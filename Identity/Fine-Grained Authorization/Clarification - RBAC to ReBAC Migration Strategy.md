# OpenFGA Enterprise Authorization Design Plan for CIAM Team

## Executive Summary

This comprehensive design plan outlines a modular, scalable approach to implementing OpenFGA for your enterprise-wide authorization system, built on **ReBAC (Relationship-Based Access Control)** principles from day one. The design maintains clear separation between CIAM team responsibilities (Organizations, Users, Groups, Applications) and domain team responsibilities (Printers, IoT Devices, and other domain-specific resources).[1][11][12]

**Critical Architecture Principle**: System roles (SA, OA, UA, PA, HELPDESK, USER) manage **ONLY** CIAM resources. Domain resources like printers and IoT devices are managed independently by domain teams using their own domain-specific roles, all within organization context.[13][14]

***

## Architecture Overview

### Design Principles

1. **CIAM Responsibility Boundary**: CIAM team manages Organizations, Users, Groups, and Applications only
2. **Domain Team Independence**: Domain teams manage their resources (printers, IoT devices, etc.) with their own roles
3. **Organization as Context**: All resources belong to organizations, but organization membership ≠ automatic access
4. **ReBAC Foundation**: Permissions derived from relationships, enabling progressive adoption of advanced patterns[11][12]
5. **Clear Separation**: System admins cannot directly access domain team APIs or resources[13]

### CIAM vs Domain Boundaries

| Responsibility | Managed By | Authorization Model |
|----------------|------------|---------------------|
| **Organizations** | CIAM team | System roles (SA, OA, UA, PA, HELPDESK, USER) |
| **Users (Identity)** | CIAM team | System roles |
| **Groups** | CIAM team | System roles |
| **Applications (OAuth)** | CIAM team | System roles |
| **Printers** | Device Management team | Domain roles (printer_admin, printer_technician, etc.) |
| **IoT Devices** | IoT team | Domain roles (iot_admin, iot_technician, etc.) |
| **Invoices** | Finance team | Domain roles (finance_admin, finance_approver, etc.) |

### High-Level Structure

```
enterprise-authz/
├── fga.mod                          # Manifest file
├── core/
│   ├── identity.fga                 # CIAM core resources
│   ├── system-roles.fga             # System role definitions
│   └── tests/
│       ├── identity.yaml
│       ├── permissions.yaml
│       └── ciam-boundary.yaml       # Test CIAM/domain separation
├── modules/
│   ├── device-management/
│   │   ├── device-management.fga    # Printers, IoT devices
│   │   └── tests/
│   │       └── device-access.yaml
│   ├── finance/
│   │   ├── finance.fga
│   │   └── tests/
│   │       └── finance.yaml
│   └── hr/
│       ├── hr.fga
│       └── tests/
│           └── hr.yaml
└── .github/
    └── CODEOWNERS
```

***

## Step 1: Core CIAM Model Design (CIAM Team Ownership)

### 1.1 Create the fga.mod Manifest

```yaml
# fga.mod
schema: '1.2'
contents:
  - core/identity.fga
  - core/system-roles.fga
  - modules/device-management/device-management.fga
  - modules/finance/finance.fga
  - modules/hr/hr.fga
```

### 1.2 Define Core CIAM Resources

Create `core/identity.fga` with CIAM-managed resources only:

```openfga
# core/identity.fga
module identity

type user

# Organization - Provides CONTEXT only, not domain resource access
type organization
  relations
    # CIAM system roles - manage ONLY CIAM resources
    define member: [user]
    define system_admin: [user]
    define org_admin: [user]
    define users_admin: [user]
    define partner_admin: [user]
    define helpdesk: [user]
    
    # Organization hierarchy
    define parent: [organization]
    define child: [organization]
    
    # Computed admin relation for CIAM resources
    define ciam_admin: system_admin or org_admin
    
    # Organization Read permissions (CIAM context only)
    define org_read: system_admin or org_admin or users_admin or helpdesk or partner_admin or member
    define org_read_and_self: org_read
    define org_write: system_admin or org_admin
    define org_create: system_admin or org_admin
    define org_delete: system_admin or org_admin
    define org_read_for_user_search: system_admin or org_admin
    
    # Organization Children permissions
    define org_read_children: system_admin or partner_admin or helpdesk
    define org_write_children: system_admin or partner_admin or (org_write_children from parent)
    define org_delete_children: org_admin
    
    # Organization Descendants permissions
    define org_read_descendants: system_admin or partner_admin or (org_read_descendants from parent)
    
    # Role Grants at organization level
    define role_grants_org_read: system_admin or org_admin or users_admin or helpdesk
    define role_grants_org_write: system_admin or org_admin

# CIAM User resource - Managed by CIAM team
type ciam_user
  relations
    define organization: [organization]
    define is_self: [user]
    
    # User Read permissions
    define user_read: system_admin from organization or 
                      org_admin from organization or 
                      users_admin from organization or 
                      helpdesk from organization
    
    define user_read_expanded: user_read or is_self
    
    # User Write permissions
    define user_write: system_admin from organization or 
                       org_admin from organization or 
                       users_admin from organization
    define user_create: user_write
    define user_delete: user_write
    define user_update: user_write
    define user_bulk_import: user_write

# Group - Managed by CIAM team
type group
  relations
    define organization: [organization]
    define parent_group: [group]
    define member: [user, group#member]
    define owner: [user]
    
    # Groups Read permissions
    define groups_read: system_admin from organization or 
                        org_admin from organization or 
                        users_admin from organization or 
                        helpdesk from organization or 
                        member from organization
    
    define groups_write: system_admin from organization or 
                         org_admin from organization or 
                         users_admin from organization
    
    define groups_read_by_id: groups_read
    define groups_search: groups_read
    define groups_delete: groups_write
    
    # Group Members permissions
    define group_members_read: system_admin from organization or 
                               org_admin from organization or 
                               users_admin from organization or 
                               helpdesk from organization
    
    define group_members_write: system_admin from organization or 
                                org_admin from organization or 
                                users_admin from organization
    
    # Group Role Grants permissions
    define group_role_grants_read: system_admin from organization or 
                                   org_admin from organization or 
                                   users_admin from organization or 
                                   helpdesk from organization
    
    define group_role_grants_write: system_admin from organization or 
                                    org_admin from organization or 
                                    users_admin from organization

# Role infrastructure - Managed by CIAM team
type role
  relations
    define organization: [organization]
    define assignee: [user, group#member]
    
    # Roles Read permissions (everyone in org can read)
    define roles_read: system_admin from organization or 
                       org_admin from organization or 
                       users_admin from organization or 
                       partner_admin from organization or 
                       helpdesk from organization or 
                       member from organization
    
    # Roles Write permissions (only SA)
    define roles_write: system_admin from organization
    
    # Role Grants permissions
    define role_grants_read: system_admin from organization or 
                             org_admin from organization or 
                             users_admin from organization or 
                             helpdesk from organization
    
    define role_grants_write: system_admin from organization or 
                              org_admin from organization or 
                              users_admin from organization
    
    # User Role Grants permissions
    define user_role_grants_read: role_grants_read
    define user_role_grants_write: role_grants_write

# Application (OAuth app) - Managed by CIAM team
type application
  relations
    define organization: [organization]
    define owner: [user]
    
    # Offerings Read permissions
    define offerings_read: system_admin from organization or 
                          org_admin from organization or 
                          users_admin from organization or 
                          partner_admin from organization or 
                          helpdesk from organization or 
                          member from organization
    
    # Offerings Write permissions (only SA)
    define offerings_write: system_admin from organization

# Organization IDP Configuration
type organization_idp_configuration
  relations
    define organization: [organization]
    
    # IDP Config permissions
    define idp_config_read: system_admin from organization or 
                           org_admin from organization or 
                           helpdesk from organization
    define idp_config_write: system_admin from organization or 
                            org_admin from organization

# Federation Profile
type federation_profile
  relations
    define organization: [organization]
    
    # Federation Profiles permissions
    define federation_profiles_read: system_admin from organization or 
                                    org_admin from organization or 
                                    helpdesk from organization
    define federation_profiles_write: system_admin from organization or 
                                     org_admin from organization

# Domain
type domain
  relations
    define organization: [organization]
    
    # Domains permissions
    define domains_read: system_admin from organization or 
                        org_admin from organization or 
                        helpdesk from organization
    define domains_write: system_admin from organization or 
                         org_admin from organization

# Role Management Group
type role_management_group
  relations
    define organization: [organization]
    
    # Role Management Groups permissions
    define role_management_groups_read: system_admin from organization
    define role_management_groups_write: system_admin from organization or 
                                        org_admin from organization

# Entitlements
type entitlement
  relations
    define organization: [organization]
    
    # Supported Entitlements permissions
    define supported_entitlements_read: system_admin from organization or 
                                       partner_admin from organization
    
    # Entitlements permissions
    define entitlements_read: system_admin from organization or 
                             partner_admin from organization or 
                             member from organization
    define entitlements_write: system_admin from organization or 
                              org_admin from organization

# B2C Policies
type b2c_policy
  relations
    define organization: [organization]
    
    # Policy Deployments permissions
    define policy_deployments_read: system_admin from organization
    define policy_deployments_write: system_admin from organization

# Home Realm
type home_realm
  relations
    define organization: [organization]
    
    # Home Realm Discovery permissions
    define home_realm_discovery: system_admin from organization

# Federated Users
type federated_user
  relations
    define organization: [organization]
    
    # Federated Users permissions
    define federated_users_read: system_admin from organization
    define federated_users_write: system_admin from organization

# Internal MPS Organizations
type internal_mps_organization
  relations
    define organization: [organization]
    
    # Internal MPS Read Access permissions
    define internal_mps_read_access: system_admin from organization
```

***

## Step 2: Domain Module Pattern (Independent from CIAM System Roles)

### 2.1 Template for Domain Teams

**CRITICAL**: Domain resources are NOT accessible by CIAM system roles

```openfga
# modules/[domain]/[domain].fga
module [domain_name]

# Domain-specific role type
type [domain]_role
  relations
    define organization: [organization]  # Context only
    define assignee: [user, group#member]
    define role_name: [user]
    
    # Only domain admins can manage domain roles
    # NOT CIAM system admins
    define can_assign: [user with domain_admin_privilege]
    define can_revoke: [user with domain_admin_privilege]

# Domain resource - COMPLETELY independent of CIAM system roles
type [domain]_resource
  relations
    # Organization for context (not automatic access)
    define organization: [organization]
    
    # Association with CIAM entities (reference, not access)
    define assigned_user: [user]
    define assigned_application: [application]
    
    # Domain-specific roles
    define [domain]_admin: [user, group#member, [domain]_role#assignee]
    define [domain]_manager: [user, group#member, [domain]_role#assignee]
    define [domain]_viewer: [user, group#member, [domain]_role#assignee]
    
    # Permissions based ONLY on domain roles
    # NO system_admin, NO org_admin, NO CIAM roles
    define [resource]_read: [domain]_viewer or [domain]_manager or [domain]_admin
    define [resource]_write: [domain]_manager or [domain]_admin
    define [resource]_create: [domain]_admin
    define [resource]_delete: [domain]_admin

condition domain_admin_privilege(has_domain_admin: bool) {
  has_domain_admin == true
}
```

### 2.2 Example: Device Management Module

```openfga
# modules/device-management/device-management.fga
module device_management

# Location hierarchy for device context
type location
  relations
    define organization: [organization]
    define parent_location: [location]
    
    # Device management roles (NOT CIAM system roles)
    define facility_manager: [user, group#member]
    define location_admin: [user, group#member]
    
    # ReBAC: Inherited device management
    define can_manage_devices_at_location: facility_manager or 
                                           location_admin or
                                           (can_manage_devices_at_location from parent_location)

# Device Fleet hierarchy
type device_fleet
  relations
    define organization: [organization]
    define parent_fleet: [device_fleet]
    
    # Fleet management roles (domain-specific)
    define fleet_admin: [user, group#member, device_role#assignee]
    define fleet_manager: [user, group#member, device_role#assignee]
    define fleet_viewer: [user, group#member, device_role#assignee]
    
    # Fleet permissions - NO CIAM system roles
    define fleet_read: fleet_viewer or fleet_manager or fleet_admin
    define fleet_manage: fleet_manager or fleet_admin or (fleet_manage from parent_fleet)

# Device Role (domain-specific role type)
type device_role
  relations
    define organization: [organization]
    define assignee: [user, group#member]
    define role_name: [user]
    
    # Only device admins can manage device roles
    define can_assign: [user with device_admin_privilege]
    define can_revoke: [user with device_admin_privilege]

# Printer - Managed ONLY by device management team roles
type printer
  relations
    # Context (not access)
    define organization: [organization]
    define location: [location]
    define fleet: [device_fleet]
    
    # Association with CIAM entities (reference only)
    define assigned_user: [user]
    define monitoring_application: [application]
    
    # Device-specific roles
    define printer_admin: [user, group#member, device_role#assignee]
    define printer_technician: [user, group#member, device_role#assignee]
    define printer_user: [user, group#member]
    
    # ReBAC: Sharing relationships
    define shared_with_user: [user]
    define shared_with_group: [group#member]
    define shared_with_department: [group#member]
    
    # Printer permissions - NO CIAM SYSTEM ROLES
    define printer_read: printer_user or 
                         printer_technician or 
                         printer_admin or
                         shared_with_user or
                         shared_with_group or
                         shared_with_department or
                         (fleet_viewer from fleet) or
                         assigned_user
    
    define printer_use: printer_read
    
    define printer_configure: printer_technician or 
                              printer_admin or
                              (fleet_manager from fleet) or
                              (can_manage_devices_at_location from location)
    
    define printer_manage: printer_admin or
                           (fleet_admin from fleet) or
                           (can_manage_devices_at_location from location)
    
    define printer_delete: printer_admin or (fleet_admin from fleet)

# IoT Device - Similar pattern
type iot_device
  relations
    # Context
    define organization: [organization]
    define location: [location]
    define fleet: [device_fleet]
    
    # Association with CIAM entities
    define assigned_user: [user]
    define monitoring_application: [application]
    define control_application: [application]
    
    # Device-specific roles
    define iot_admin: [user, group#member, device_role#assignee]
    define iot_technician: [user, group#member, device_role#assignee]
    define iot_operator: [user, group#member, device_role#assignee]
    define iot_viewer: [user, group#member]
    
    # Device type grouping (ReBAC)
    define device_type_group: [device_type_group]
    
    # IoT permissions - NO CIAM SYSTEM ROLES
    define device_read: iot_viewer or 
                        iot_operator or 
                        iot_technician or 
                        iot_admin or
                        assigned_user or
                        (can_monitor_devices from device_type_group) or
                        (fleet_viewer from fleet)
    
    define device_read_telemetry: device_read
    define device_read_alerts: device_read
    
    define device_configure: iot_technician or 
                             iot_admin or
                             (fleet_manager from fleet) or
                             (can_manage_devices_at_location from location)
    
    define device_control: iot_operator or device_configure
    
    define device_manage: iot_admin or (fleet_admin from fleet)
    
    define device_delete: iot_admin

# Device Type Group for cross-functional monitoring
type device_type_group
  relations
    define organization: [organization]
    define monitoring_team: [group]
    
    # Monitoring permissions
    define can_monitor_devices: [user] or (member from monitoring_team)

condition device_admin_privilege(has_device_admin: bool) {
  has_device_admin == true
}
```

***

## Step 3: Comprehensive Testing Strategy

### 3.1 CIAM Boundary Tests

Create `core/tests/ciam-boundary.yaml`:

```yaml
# core/tests/ciam-boundary.yaml
name: CIAM Boundary Tests - System Roles Do Not Access Domain Resources
model_file: ../../fga.mod

tuples:
  # CIAM setup
  - user: user:system-admin
    relation: system_admin
    object: organization:acme
  - user: user:org-admin
    relation: org_admin
    object: organization:acme
  
  # CIAM user
  - user: organization:acme
    relation: organization
    object: ciam_user:john@acme.com
  
  # Domain resource (printer)
  - user: organization:acme
    relation: organization
    object: printer:test-printer
  
  # Domain admin
  - user: user:device-admin
    relation: printer_admin
    object: printer:test-printer

tests:
  - name: System Admin CAN manage CIAM users
    check:
      - user: user:system-admin
        relation: user_read
        object: ciam_user:john@acme.com
        expected: true
      - user: user:system-admin
        relation: user_write
        object: ciam_user:john@acme.com
        expected: true
  
  - name: System Admin CANNOT access domain resources (printers)
    check:
      - user: user:system-admin
        relation: printer_read
        object: printer:test-printer
        expected: false
      - user: user:system-admin
        relation: printer_manage
        object: printer:test-printer
        expected: false
  
  - name: Org Admin CAN manage CIAM resources in their org
    check:
      - user: user:org-admin
        relation: user_read
        object: ciam_user:john@acme.com
        expected: true
      - user: user:org-admin
        relation: org_write
        object: organization:acme
        expected: true
  
  - name: Org Admin CANNOT access domain resources (printers)
    check:
      - user: user:org-admin
        relation: printer_read
        object: printer:test-printer
        expected: false
  
  - name: Domain Admin CAN access domain resources
    check:
      - user: user:device-admin
        relation: printer_read
        object: printer:test-printer
        expected: true
      - user: user:device-admin
        relation: printer_manage
        object: printer:test-printer
        expected: true
  
  - name: Domain Admin CANNOT access CIAM admin functions
    check:
      - user: user:device-admin
        relation: user_write
        object: ciam_user:john@acme.com
        expected: false
      - user: user:device-admin
        relation: org_write
        object: organization:acme
        expected: false
```

### 3.2 Core CIAM Permission Tests

Create `core/tests/permissions.yaml`:

```yaml
# core/tests/permissions.yaml
name: Core CIAM Permission Tests
model_file: ../../fga.mod

tuples:
  # Organization setup
  - user: user:system-admin
    relation: system_admin
    object: organization:acme
  - user: user:org-admin
    relation: org_admin
    object: organization:acme
  - user: user:users-admin
    relation: users_admin
    object: organization:acme
  - user: user:partner-admin
    relation: partner_admin
    object: organization:acme
  - user: user:helpdesk
    relation: helpdesk
    object: organization:acme
  - user: user:regular-user
    relation: member
    object: organization:acme
  
  # User resource setup
  - user: organization:acme
    relation: organization
    object: ciam_user:john@acme.com
  - user: user:john@acme.com
    relation: is_self
    object: ciam_user:john@acme.com
  
  # Group setup
  - user: organization:acme
    relation: organization
    object: group:engineering
  - user: user:regular-user
    relation: member
    object: group:engineering

tests:
  # User permissions tests
  - name: User Read - SA, OA, UA, HELPDESK can read users
    check:
      - user: user:system-admin
        relation: user_read
        object: ciam_user:john@acme.com
        expected: true
      - user: user:org-admin
        relation: user_read
        object: ciam_user:john@acme.com
        expected: true
      - user: user:users-admin
        relation: user_read
        object: ciam_user:john@acme.com
        expected: true
      - user: user:helpdesk
        relation: user_read
        object: ciam_user:john@acme.com
        expected: true
      - user: user:regular-user
        relation: user_read
        object: ciam_user:john@acme.com
        expected: false
  
  - name: User Read Expanded - USER can read own data
    check:
      - user: user:john@acme.com
        relation: user_read_expanded
        object: ciam_user:john@acme.com
        expected: true
      - user: user:regular-user
        relation: user_read_expanded
        object: ciam_user:john@acme.com
        expected: false
  
  - name: User Write - SA, OA, UA can write users
    check:
      - user: user:system-admin
        relation: user_write
        object: ciam_user:john@acme.com
        expected: true
      - user: user:org-admin
        relation: user_write
        object: ciam_user:john@acme.com
        expected: true
      - user: user:users-admin
        relation: user_write
        object: ciam_user:john@acme.com
        expected: true
      - user: user:helpdesk
        relation: user_write
        object: ciam_user:john@acme.com
        expected: false
  
  # Organization permissions tests
  - name: Org Read - Everyone in org can read
    check:
      - user: user:system-admin
        relation: org_read
        object: organization:acme
        expected: true
      - user: user:regular-user
        relation: org_read
        object: organization:acme
        expected: true
  
  - name: Org Write - Only SA and OA can write
    check:
      - user: user:system-admin
        relation: org_write
        object: organization:acme
        expected: true
      - user: user:org-admin
        relation: org_write
        object: organization:acme
        expected: true
      - user: user:users-admin
        relation: org_write
        object: organization:acme
        expected: false
  
  # Group permissions tests
  - name: Groups Read - SA, OA, UA, HELPDESK, USER can read
    check:
      - user: user:system-admin
        relation: groups_read
        object: group:engineering
        expected: true
      - user: user:regular-user
        relation: groups_read
        object: group:engineering
        expected: true
  
  - name: Groups Write - SA, OA, UA can write
    check:
      - user: user:system-admin
        relation: groups_write
        object: group:engineering
        expected: true
      - user: user:helpdesk
        relation: groups_write
        object: group:engineering
        expected: false
```

### 3.3 Device Management Tests

Create `modules/device-management/tests/device-access.yaml`:

```yaml
# modules/device-management/tests/device-access.yaml
name: Device Management Access Tests
model_file: ../../../fga.mod

tuples:
  # Organization
  - user: user:system-admin
    relation: system_admin
    object: organization:acme
  
  # Device role
  - user: organization:acme
    relation: organization
    object: device_role:printer-tech
  - user: user:john@acme.com
    relation: assignee
    object: device_role:printer-tech
  
  # Printer
  - user: organization:acme
    relation: organization
    object: printer:office-p01
  - user: device_role:printer-tech#assignee
    relation: printer_technician
    object: printer:office-p01
  
  # Group sharing
  - user: organization:acme
    relation: organization
    object: group:engineering
  - user: user:alice@acme.com
    relation: member
    object: group:engineering
  - user: group:engineering#member
    relation: shared_with_group
    object: printer:office-p01

tests:
  - name: User with device role can access printer
    check:
      - user: user:john@acme.com
        relation: printer_read
        object: printer:office-p01
        expected: true
      - user: user:john@acme.com
        relation: printer_configure
        object: printer:office-p01
        expected: true
  
  - name: System Admin CANNOT access printer
    check:
      - user: user:system-admin
        relation: printer_read
        object: printer:office-p01
        expected: false
  
  - name: User in group can access shared printer
    check:
      - user: user:alice@acme.com
        relation: printer_use
        object: printer:office-p01
        expected: true
```

***

## Step 4: Application Integration Patterns

### 4.1 CIAM API Service

```typescript
// ciam-service.ts
class CiamAuthorizationService {
  async getCiamUser(adminUserId: string, targetUserId: string) {
    const allowed = await authService.checkPermission({
      userId: adminUserId,
      permission: 'user_read',
      resourceType: 'ciam_user',
      resourceId: targetUserId
    });

    if (!allowed) throw new Error('Forbidden: No CIAM user_read permission');
    return fetchUserFromIdentityServer(targetUserId);
  }

  async createCiamUser(adminUserId: string, userData: any) {
    const allowed = await authService.checkPermission({
      userId: adminUserId,
      permission: 'user_write',
      resourceType: 'ciam_user',
      resourceId: userData.organizationId
    });

    if (!allowed) throw new Error('Forbidden: No CIAM user_write permission');
    return createUserInIdentityServer(userData);
  }
}
```

### 4.2 Device Management API Service

```typescript
// device-service.ts
class DeviceAuthorizationService {
  async getPrinter(userId: string, printerId: string) {
    // Check DEVICE permission (not CIAM permission)
    const allowed = await authService.checkPermission({
      userId,
      permission: 'printer_read',
      resourceType: 'printer',
      resourceId: printerId
    });

    if (!allowed) {
      throw new Error('Forbidden: No printer_read permission. User must have device role.');
    }

    return fetchPrinterFromDatabase(printerId);
  }

  async assignDeviceRole(
    deviceAdminUserId: string,
    targetUserId: string,
    deviceRoleId: string,
    organizationId: string
  ) {
    // Only device admins can do this, not CIAM system admins
    const isDeviceAdmin = await authService.checkPermission({
      userId: deviceAdminUserId,
      permission: 'can_assign',
      resourceType: 'device_role',
      resourceId: deviceRoleId,
      context: { has_device_admin: true }
    });

    if (!isDeviceAdmin) {
      throw new Error('Forbidden: Only device admins can assign device roles');
    }

    // Assign role...
  }
}
```

***

## Step 5: Implementation Roadmap

### Phase 1: Foundation (Weeks 1-4)

**Week 1-2: Core CIAM Model**
- [ ] Set up repository structure
- [ ] Create `fga.mod` manifest
- [ ] Develop `core/identity.fga` with CIAM resources only
- [ ] Map all CIAM permissions to OpenFGA relations
- [ ] Write core CIAM tests
- [ ] Write CIAM boundary tests

**Week 3-4: Testing Infrastructure**
- [ ] Set up CI/CD pipeline
- [ ] Create module development guidelines
- [ ] Document CIAM/domain separation
- [ ] Set up CODEOWNERS
- [ ] Conduct CIAM team training

### Phase 2: Pilot Domain Module (Weeks 5-8)

**Week 5-6: Device Management Module (Pilot)**
- [ ] Collaborate with Device Management team
- [ ] Develop device-management.fga
- [ ] Define device-specific roles
- [ ] Implement printer and IoT device types
- [ ] Write device management tests
- [ ] Test boundary enforcement

**Week 7-8: Refinement**
- [ ] Gather feedback
- [ ] Refine module template
- [ ] Update documentation
- [ ] Performance testing
- [ ] Security review

### Phase 3-5: Continue with original roadmap...

***

## Conclusion

This design provides a **production-ready OpenFGA authorization system** with:

1. ✅ **Clear CIAM Boundary**: System roles manage only Organizations, Users, Groups, Applications
2. ✅ **Domain Independence**: Each team manages their resources (printers, IoT devices, etc.) independently
3. ✅ **ReBAC Foundation**: Built on relationships from day one, enabling advanced patterns
4. ✅ **Organization Context**: All resources scoped to organizations without automatic access
5. ✅ **Scalable Architecture**: Modular design supports unlimited domain teams
6. ✅ **Security by Design**: No cross-boundary access, clear separation of concerns

The architecture ensures CIAM and domain teams can work independently while maintaining cohesive, secure authorization across your enterprise printer and IoT device fleet management system.

