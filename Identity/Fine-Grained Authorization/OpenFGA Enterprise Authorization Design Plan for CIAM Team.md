# OpenFGA Enterprise Authorization Design Plan for CIAM Team

## Executive Summary

This comprehensive design plan outlines a modular, scalable approach to implementing OpenFGA for your enterprise-wide authorization system, specifically tailored to your existing permission structure and role-based access control (RBAC) model. The design leverages OpenFGA's modular models feature to enable independent development by CIAM and domain teams while maintaining a cohesive authorization framework.[1][2]

***

## Architecture Overview

### Design Principles

1. **CIAM Core Resources**: CIAM team manages Organization, Users, Groups, and Applications as foundational resources[3][4]
2. **Domain Team Extensions**: Domain teams define their own resources (invoices, contracts, etc.) that relate to core CIAM resources[2]
3. **Organization-Context Authorization**: All permissions operate within organization scope[5][3]
4. **Permission Granularity**: Fine-grained permissions (Read, Write, Create, Delete, Update) mapped to system roles[6]
5. **Self-Service Capabilities**: Users can access their own data (e.g., UserReadExpanded for self)[7][5]
6. **Hierarchical Organizations**: Support for organization children and descendants[8][9]

### System Roles Mapping

| Role Code | Role Name | Scope | Description |
|-----------|-----------|-------|-------------|
| **SA** | System Admin | Global | Super admin with full access across all organizations |
| **OA** | Organization Admin | Organization | Full admin rights within a specific organization |
| **UA** | Users Admin | Organization | Manages users and user-related operations within organization |
| **PA** | Partner Admin | Organization | Manages partner relationships and organization hierarchies |
| **HELPDESK** | Helpdesk | Organization | Read-only support access to most resources |
| **USER** | Regular User | Organization | Standard member with limited self-service capabilities |

### High-Level Structure

```
enterprise-authz/
├── fga.mod                          # Manifest file
├── core/
│   ├── identity.fga                 # Core CIAM identity resources
│   ├── system-roles.fga             # System role definitions and permissions
│   └── tests/
│       ├── identity.yaml            # Core identity tests
│       └── permissions.yaml         # Permission mapping tests
├── modules/
│   ├── finance/
│   │   ├── finance.fga             # Finance domain resources (invoices, etc.)
│   │   └── tests/
│   │       └── finance.yaml
│   ├── hr/
│   │   ├── hr.fga                  # HR domain resources (employees, etc.)
│   │   └── tests/
│   │       └── hr.yaml
│   └── procurement/
│       ├── procurement.fga         # Procurement domain resources
│       └── tests/
│           └── procurement.yaml
└── .github/
    ├── workflows/
    │   └── openfga-tests.yml
    └── CODEOWNERS                   # Team ownership definitions
```

***

## Step 1: Core CIAM Model Design

### 1.1 Create the fga.mod Manifest

```yaml
# fga.mod
schema: '1.2'
contents:
  - core/identity.fga
  - core/system-roles.fga
  - modules/finance/finance.fga
  - modules/hr/hr.fga
  - modules/procurement/procurement.fga
```

### 1.2 Define Core Identity Resources

Create `core/identity.fga` with CIAM-managed resources:[9][3]

```openfga
# core/identity.fga
module identity

type user

# Organization with hierarchical support
type organization
  relations
    # Direct membership and role assignments
    define member: [user]
    define system_admin: [user]
    define org_admin: [user]
    define users_admin: [user]
    define partner_admin: [user]
    define helpdesk: [user]
    
    # Organization hierarchy
    define parent: [organization]
    define child: [organization]
    
    # Computed admin relation
    define admin: system_admin or org_admin
    
    # Organization Read permissions
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

# User resource (managed by CIAM)
type ciam_user
  relations
    define organization: [organization]
    define is_self: [user]
    
    # User Read permissions
    define user_read: system_admin from organization or org_admin from organization or users_admin from organization or helpdesk from organization
    define user_read_expanded: user_read or is_self
    
    # User Write permissions
    define user_write: system_admin from organization or org_admin from organization or users_admin from organization
    define user_create: system_admin from organization or org_admin from organization or users_admin from organization
    define user_delete: system_admin from organization or org_admin from organization or users_admin from organization
    define user_update: system_admin from organization or org_admin from organization or users_admin from organization
    define user_bulk_import: system_admin from organization or org_admin from organization or users_admin from organization

# Group resource
type group
  relations
    define organization: [organization]
    define owner: [user]
    define member: [user, group#member]
    
    # Groups Read permissions
    define groups_read: system_admin from organization or org_admin from organization or users_admin from organization or helpdesk from organization or member from organization
    define groups_write: system_admin from organization or org_admin from organization or users_admin from organization
    define groups_read_by_id: groups_read
    define groups_search: groups_read
    define groups_delete: system_admin from organization or org_admin from organization or users_admin from organization
    
    # Group Members permissions
    define group_members_read: system_admin from organization or org_admin from organization or users_admin from organization or helpdesk from organization
    define group_members_write: system_admin from organization or org_admin from organization or users_admin from organization
    
    # Group Role Grants permissions
    define group_role_grants_read: system_admin from organization or org_admin from organization or users_admin from organization or helpdesk from organization
    define group_role_grants_write: system_admin from organization or org_admin from organization or users_admin from organization

# Role resource
type role
  relations
    define organization: [organization]
    define assignee: [user, group#member]
    
    # Roles Read permissions (everyone in org can read)
    define roles_read: system_admin from organization or org_admin from organization or users_admin from organization or partner_admin from organization or helpdesk from organization or member from organization
    
    # Roles Write permissions (only SA)
    define roles_write: system_admin from organization
    
    # Role Grants permissions
    define role_grants_read: system_admin from organization or org_admin from organization or users_admin from organization or helpdesk from organization
    define role_grants_write: system_admin from organization or org_admin from organization or users_admin from organization
    
    # User Role Grants permissions
    define user_role_grants_read: role_grants_read
    define user_role_grants_write: role_grants_write

# Application/Offering resource
type application
  relations
    define organization: [organization]
    define owner: [user]
    
    # Offerings Read permissions
    define offerings_read: system_admin from organization or org_admin from organization or users_admin from organization or partner_admin from organization or helpdesk from organization or member from organization
    
    # Offerings Write permissions (only SA)
    define offerings_write: system_admin from organization

# Organization IDP Configuration
type organization_idp_configuration
  relations
    define organization: [organization]
    
    # IDP Config permissions
    define idp_config_read: system_admin from organization or org_admin from organization or helpdesk from organization
    define idp_config_write: system_admin from organization or org_admin from organization

# Federation Profile
type federation_profile
  relations
    define organization: [organization]
    
    # Federation Profiles permissions
    define federation_profiles_read: system_admin from organization or org_admin from organization or helpdesk from organization
    define federation_profiles_write: system_admin from organization or org_admin from organization

# Domain
type domain
  relations
    define organization: [organization]
    
    # Domains permissions
    define domains_read: system_admin from organization or org_admin from organization or helpdesk from organization
    define domains_write: system_admin from organization or org_admin from organization

# Role Management Group
type role_management_group
  relations
    define organization: [organization]
    
    # Role Management Groups permissions
    define role_management_groups_read: system_admin from organization
    define role_management_groups_write: system_admin from organization or org_admin from organization

# Entitlements
type entitlement
  relations
    define organization: [organization]
    
    # Supported Entitlements permissions
    define supported_entitlements_read: system_admin from organization or partner_admin from organization
    
    # Entitlements permissions
    define entitlements_read: system_admin from organization or partner_admin from organization or member from organization
    define entitlements_write: system_admin from organization or org_admin from organization

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

# Self-service condition for user reading own data
condition is_same_user(user_id: string, resource_user_id: string) {
  user_id == resource_user_id
}
```

***

## Step 2: Domain Module Extension Pattern

### 2.1 Template for Domain Teams

Provide this template to domain teams extending core CIAM resources:[1][2]

```openfga
# modules/[domain]/[domain].fga
module [domain_name]

# Import core types (implicit with modular models)

# Extend organization to add domain-specific permissions
extend type organization
  relations
    # Define domain-specific admin or manager roles
    define [domain]_admin: [role#assignee]
    define [domain]_manager: [role#assignee]
    define [domain]_viewer: [role#assignee]

# Define domain-specific resource types
type [domain]_resource
  relations
    # Link to core organization
    define organization: [organization]
    
    # Link to users
    define owner: [user]
    define editor: [user, role#assignee, group#member]
    define viewer: [user, role#assignee, group#member]
    
    # Resource-level permissions
    define [resource]_read: viewer or editor or owner or 
                            (system_admin from organization) or 
                            (org_admin from organization) or
                            ([domain]_admin from organization)
    
    define [resource]_write: editor or owner or 
                             (system_admin from organization) or 
                             (org_admin from organization) or
                             ([domain]_admin from organization)
    
    define [resource]_create: owner or 
                              (system_admin from organization) or 
                              (org_admin from organization) or
                              ([domain]_manager from organization)
    
    define [resource]_delete: (system_admin from organization) or 
                              (org_admin from organization) or
                              ([domain]_admin from organization)
    
    define [resource]_update: [resource]_write

# Define domain-specific roles (if needed beyond core)
type [domain]_role
  relations
    define organization: [organization]
    define assignee: [user, group#member]
    
    # Role management (only OA and SA can manage)
    define can_assign: (system_admin from organization) or (org_admin from organization)
    define can_revoke: (system_admin from organization) or (org_admin from organization)
```

### 2.2 Example: Finance Module Implementation

```openfga
# modules/finance/finance.fga
module finance

# Extend organization for finance-specific roles
extend type organization
  relations
    define finance_admin: [role#assignee]
    define finance_manager: [role#assignee]
    define finance_approver: [role#assignee]
    define finance_viewer: [role#assignee]

# Invoice resource
type invoice
  relations
    define organization: [organization]
    define owner: [user]
    define approver: [user, role#assignee]
    define viewer: [user, role#assignee, group#member]
    
    # Invoice Read permission
    define invoice_read: viewer or approver or owner or 
                         (system_admin from organization) or 
                         (org_admin from organization) or
                         (finance_admin from organization) or
                         (finance_viewer from organization) or
                         (helpdesk from organization)
    
    # Invoice Write permission
    define invoice_write: owner or 
                          (system_admin from organization) or 
                          (org_admin from organization) or
                          (finance_admin from organization) or
                          (finance_manager from organization)
    
    # Invoice Create permission
    define invoice_create: (system_admin from organization) or 
                           (org_admin from organization) or
                           (finance_admin from organization) or
                           (finance_manager from organization)
    
    # Invoice Delete permission
    define invoice_delete: (system_admin from organization) or 
                           (org_admin from organization) or
                           (finance_admin from organization)
    
    # Invoice Update permission
    define invoice_update: invoice_write
    
    # Invoice Approve permission (specific to finance domain)
    define invoice_approve: approver or 
                            (system_admin from organization) or 
                            (org_admin from organization) or
                            (finance_approver from organization)

# Financial Report resource
type financial_report
  relations
    define organization: [organization]
    define creator: [user]
    define allowed_viewers: [user, role#assignee, group#member]
    
    # Financial Report Read permission
    define financial_report_read: allowed_viewers or creator or
                                  (system_admin from organization) or 
                                  (org_admin from organization) or
                                  (finance_admin from organization) or
                                  (finance_viewer from organization)
    
    # Financial Report Write permission
    define financial_report_write: creator or
                                   (system_admin from organization) or 
                                   (org_admin from organization) or
                                   (finance_admin from organization)
    
    # Financial Report Generate permission
    define financial_report_generate: (system_admin from organization) or 
                                      (org_admin from organization) or
                                      (finance_manager from organization)
    
    # Financial Report Delete permission
    define financial_report_delete: (system_admin from organization) or 
                                    (org_admin from organization) or
                                    (finance_admin from organization)

# Expense resource
type expense
  relations
    define organization: [organization]
    define submitter: [user]
    define approver: [user, role#assignee]
    
    # Expense Read permission
    define expense_read: submitter or approver or
                         (system_admin from organization) or 
                         (org_admin from organization) or
                         (finance_admin from organization) or
                         (helpdesk from organization)
    
    # Expense Write permission (submitter can update before approval)
    define expense_write: submitter or
                          (system_admin from organization) or 
                          (org_admin from organization) or
                          (finance_admin from organization)
    
    # Expense Create permission
    define expense_create: (member from organization)
    
    # Expense Delete permission
    define expense_delete: (system_admin from organization) or 
                           (org_admin from organization) or
                           (finance_admin from organization)
    
    # Expense Approve permission
    define expense_approve: approver or
                            (system_admin from organization) or 
                            (org_admin from organization) or
                            (finance_approver from organization)
    
    # Expense Reject permission
    define expense_reject: approver or
                           (system_admin from organization) or 
                           (org_admin from organization) or
                           (finance_approver from organization)

# Finance-specific role type
type finance_role
  relations
    define organization: [organization]
    define assignee: [user, group#member]
    define role_type: [user]  # finance_admin, finance_manager, etc.
    
    # Only OA and SA can manage finance roles
    define can_assign: (system_admin from organization) or (org_admin from organization)
    define can_revoke: (system_admin from organization) or (org_admin from organization)
```

### 2.3 Example: HR Module Implementation

```openfga
# modules/hr/hr.fga
module hr

# Extend organization for HR-specific roles
extend type organization
  relations
    define hr_admin: [role#assignee]
    define hr_manager: [role#assignee]
    define hr_viewer: [role#assignee]

# Employee resource
type employee
  relations
    define organization: [organization]
    define manager: [user]
    define hr_partner: [user, role#assignee]
    define self: [user]
    
    # Employee Read permission
    define employee_read: self or manager or hr_partner or
                          (system_admin from organization) or 
                          (org_admin from organization) or
                          (users_admin from organization) or
                          (hr_admin from organization) or
                          (hr_viewer from organization) or
                          (helpdesk from organization)
    
    # Employee Write permission
    define employee_write: (system_admin from organization) or 
                           (org_admin from organization) or
                           (users_admin from organization) or
                           (hr_admin from organization) or
                           (hr_manager from organization)
    
    # Employee Create permission
    define employee_create: (system_admin from organization) or 
                            (org_admin from organization) or
                            (hr_admin from organization) or
                            (hr_manager from organization)
    
    # Employee Delete permission
    define employee_delete: (system_admin from organization) or 
                            (org_admin from organization)
    
    # Employee Update permission
    define employee_update: employee_write

# Employee Record resource (sensitive data)
type employee_record
  relations
    define organization: [organization]
    define employee: [employee]
    define record_category: [user]  # personal, compensation, performance
    
    # Employee Record Read permission (more restrictive)
    define employee_record_read: (self from employee) or 
                                 (manager from employee) or
                                 (system_admin from organization) or 
                                 (org_admin from organization) or
                                 (hr_admin from organization) or
                                 (hr_manager from organization)
    
    # Employee Record Write permission
    define employee_record_write: (system_admin from organization) or 
                                  (org_admin from organization) or
                                  (hr_admin from organization) or
                                  (hr_manager from organization)
    
    # Employee Record Delete permission
    define employee_record_delete: (system_admin from organization) or 
                                   (org_admin from organization)

# Performance Review resource
type performance_review
  relations
    define organization: [organization]
    define employee: [employee]
    define reviewer: [user]
    
    # Performance Review Read permission
    define performance_review_read: (self from employee) or reviewer or
                                    (manager from employee) or
                                    (system_admin from organization) or 
                                    (org_admin from organization) or
                                    (hr_admin from organization)
    
    # Performance Review Write permission
    define performance_review_write: reviewer or
                                     (system_admin from organization) or 
                                     (org_admin from organization) or
                                     (hr_admin from organization)
    
    # Performance Review Submit permission
    define performance_review_submit: reviewer
    
    # Performance Review Approve permission
    define performance_review_approve: (manager from employee) or
                                       (system_admin from organization) or 
                                       (org_admin from organization)
```

***

## Step 3: Tuple Structure and Role Assignment

### 3.1 System Role Assignment Tuples

Example tuples for assigning system roles:[4][6]

```javascript
// System Admin assignment (global or per org)
{
  "user": "user:alice@company.com",
  "relation": "system_admin",
  "object": "organization:root"
}

// Organization Admin assignment
{
  "user": "user:bob@company.com",
  "relation": "org_admin",
  "object": "organization:acme-corp"
}

// Users Admin assignment
{
  "user": "user:charlie@company.com",
  "relation": "users_admin",
  "object": "organization:acme-corp"
}

// Partner Admin assignment
{
  "user": "user:dave@company.com",
  "relation": "partner_admin",
  "object": "organization:acme-corp"
}

// Helpdesk assignment
{
  "user": "user:eve@company.com",
  "relation": "helpdesk",
  "object": "organization:acme-corp"
}

// Regular User (member) assignment
{
  "user": "user:frank@company.com",
  "relation": "member",
  "object": "organization:acme-corp"
}
```

### 3.2 Organization Hierarchy Tuples

Example tuples for organization parent-child relationships:[8][9]

```javascript
// Define parent-child relationship
{
  "user": "organization:acme-corp",
  "relation": "parent",
  "object": "organization:acme-emea"
}

{
  "user": "organization:acme-emea",
  "relation": "child",
  "object": "organization:acme-corp"
}

// Partner Admin in parent org can manage children
// This is inherited through the model definition
```

### 3.3 Self-Service User Access Tuples

Example tuples for users accessing their own data:[7][5]

```javascript
// Link user to their CIAM user resource
{
  "user": "user:frank@company.com",
  "relation": "is_self",
  "object": "ciam_user:frank@company.com"
}

// Link CIAM user to organization
{
  "user": "organization:acme-corp",
  "relation": "organization",
  "object": "ciam_user:frank@company.com"
}

// Now user:frank can perform user_read_expanded on their own ciam_user
// because: user_read_expanded = user_read or is_self
```

### 3.4 Domain Role Assignment Tuples

Example tuples for assigning domain-specific roles:

```javascript
// Create finance role
{
  "user": "organization:acme-corp",
  "relation": "organization",
  "object": "role:finance-manager-acme"
}

// Assign user to finance role
{
  "user": "user:sarah@company.com",
  "relation": "assignee",
  "object": "role:finance-manager-acme"
}

// Link role to organization for finance_manager relation
{
  "user": "role:finance-manager-acme",
  "relation": "finance_manager",
  "object": "organization:acme-corp"
}

// Link invoice to organization
{
  "user": "organization:acme-corp",
  "relation": "organization",
  "object": "invoice:inv-2025-001"
}

// Now user:sarah can invoice_write because:
// - sarah is assignee of role:finance-manager-acme
// - role:finance-manager-acme is finance_manager of organization:acme-corp
// - invoice:inv-2025-001 belongs to organization:acme-corp
// - invoice_write includes finance_manager from organization
```

***

## Step 4: Comprehensive Testing Strategy

### 4.1 Core Permission Tests

Create `core/tests/permissions.yaml`:[10][11]

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
      - user: user:regular-user
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
      - user: user:org-admin
        relation: org_read
        object: organization:acme
        expected: true
      - user: user:users-admin
        relation: org_read
        object: organization:acme
        expected: true
      - user: user:partner-admin
        relation: org_read
        object: organization:acme
        expected: true
      - user: user:helpdesk
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
      - user: user:partner-admin
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
      - user: user:org-admin
        relation: groups_read
        object: group:engineering
        expected: true
      - user: user:users-admin
        relation: groups_read
        object: group:engineering
        expected: true
      - user: user:helpdesk
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
      - user: user:org-admin
        relation: groups_write
        object: group:engineering
        expected: true
      - user: user:users-admin
        relation: groups_write
        object: group:engineering
        expected: true
      - user: user:helpdesk
        relation: groups_write
        object: group:engineering
        expected: false
      - user: user:regular-user
        relation: groups_write
        object: group:engineering
        expected: false
  
  # Role permissions tests
  - name: Roles Read - Everyone in org can read roles
    check:
      - user: user:system-admin
        relation: roles_read
        object: role:custom-role-1
        expected: true
      - user: user:org-admin
        relation: roles_read
        object: role:custom-role-1
        expected: true
      - user: user:regular-user
        relation: roles_read
        object: role:custom-role-1
        expected: true
  
  - name: Roles Write - Only SA can write roles
    tuples:
      - user: organization:acme
        relation: organization
        object: role:custom-role-1
    check:
      - user: user:system-admin
        relation: roles_write
        object: role:custom-role-1
        expected: true
      - user: user:org-admin
        relation: roles_write
        object: role:custom-role-1
        expected: false
```

### 4.2 Organization Hierarchy Tests

Create `core/tests/org-hierarchy.yaml`:[9][8]

```yaml
# core/tests/org-hierarchy.yaml
name: Organization Hierarchy Tests
model_file: ../../fga.mod

tuples:
  # Parent organization
  - user: user:system-admin
    relation: system_admin
    object: organization:acme-global
  - user: user:partner-admin
    relation: partner_admin
    object: organization:acme-global
  
  # Child organization
  - user: user:org-admin-emea
    relation: org_admin
    object: organization:acme-emea
  
  # Hierarchy relationship
  - user: organization:acme-global
    relation: parent
    object: organization:acme-emea
  - user: organization:acme-emea
    relation: child
    object: organization:acme-global

tests:
  - name: Org Read Children - SA and PA can read children
    check:
      - user: user:system-admin
        relation: org_read_children
        object: organization:acme-global
        expected: true
      - user: user:partner-admin
        relation: org_read_children
        object: organization:acme-global
        expected: true
      - user: user:org-admin-emea
        relation: org_read_children
        object: organization:acme-global
        expected: false
  
  - name: Org Write Children - SA and PA can write children
    check:
      - user: user:system-admin
        relation: org_write_children
        object: organization:acme-emea
        expected: true
      - user: user:partner-admin
        relation: org_write_children
        object: organization:acme-emea
        expected: true
  
  - name: Org Write Children - Inherited from parent
    check:
      # Partner admin in parent org can write to child org
      - user: user:partner-admin
        relation: org_write_children
        object: organization:acme-emea
        expected: true
  
  - name: Org Delete Children - Only OA can delete
    check:
      - user: user:org-admin-emea
        relation: org_delete_children
        object: organization:acme-emea
        expected: true
      - user: user:partner-admin
        relation: org_delete_children
        object: organization:acme-emea
        expected: false
  
  - name: Org Read Descendants - SA and PA with inheritance
    check:
      - user: user:system-admin
        relation: org_read_descendants
        object: organization:acme-global
        expected: true
      - user: user:partner-admin
        relation: org_read_descendants
        object: organization:acme-global
        expected: true
      # PA in parent can read descendants
      - user: user:partner-admin
        relation: org_read_descendants
        object: organization:acme-emea
        expected: true
```

### 4.3 Domain Module Tests - Finance Example

Create `modules/finance/tests/finance.yaml`:

```yaml
# modules/finance/tests/finance.yaml
name: Finance Module Permission Tests
model_file: ../../../fga.mod

tuples:
  # Organization and users
  - user: user:system-admin
    relation: system_admin
    object: organization:acme
  - user: user:org-admin
    relation: org_admin
    object: organization:acme
  - user: user:helpdesk
    relation: helpdesk
    object: organization:acme
  - user: user:regular-user
    relation: member
    object: organization:acme
  
  # Finance role setup
  - user: organization:acme
    relation: organization
    object: role:finance-manager
  - user: user:finance-manager
    relation: assignee
    object: role:finance-manager
  - user: role:finance-manager
    relation: finance_manager
    object: organization:acme
  
  # Finance approver role
  - user: organization:acme
    relation: organization
    object: role:finance-approver
  - user: user:finance-approver
    relation: assignee
    object: role:finance-approver
  - user: role:finance-approver
    relation: finance_approver
    object: organization:acme
  
  # Invoice setup
  - user: organization:acme
    relation: organization
    object: invoice:inv-2025-001
  - user: user:invoice-owner
    relation: owner
    object: invoice:inv-2025-001
  - user: user:finance-approver
    relation: approver
    object: invoice:inv-2025-001

tests:
  - name: Invoice Read - Owner, approver, admins, helpdesk can read
    check:
      - user: user:system-admin
        relation: invoice_read
        object: invoice:inv-2025-001
        expected: true
      - user: user:org-admin
        relation: invoice_read
        object: invoice:inv-2025-001
        expected: true
      - user: user:helpdesk
        relation: invoice_read
        object: invoice:inv-2025-001
        expected: true
      - user: user:invoice-owner
        relation: invoice_read
        object: invoice:inv-2025-001
        expected: true
      - user: user:finance-approver
        relation: invoice_read
        object: invoice:inv-2025-001
        expected: true
      - user: user:regular-user
        relation: invoice_read
        object: invoice:inv-2025-001
        expected: false
  
  - name: Invoice Write - Owner, SA, OA, finance manager can write
    check:
      - user: user:system-admin
        relation: invoice_write
        object: invoice:inv-2025-001
        expected: true
      - user: user:org-admin
        relation: invoice_write
        object: invoice:inv-2025-001
        expected: true
      - user: user:finance-manager
        relation: invoice_write
        object: invoice:inv-2025-001
        expected: true
      - user: user:invoice-owner
        relation: invoice_write
        object: invoice:inv-2025-001
        expected: true
      - user: user:helpdesk
        relation: invoice_write
        object: invoice:inv-2025-001
        expected: false
  
  - name: Invoice Create - SA, OA, finance manager can create
    check:
      - user: user:system-admin
        relation: invoice_create
        object: invoice:inv-2025-001
        expected: true
      - user: user:org-admin
        relation: invoice_create
        object: invoice:inv-2025-001
        expected: true
      - user: user:finance-manager
        relation: invoice_create
        object: invoice:inv-2025-001
        expected: true
      - user: user:regular-user
        relation: invoice_create
        object: invoice:inv-2025-001
        expected: false
  
  - name: Invoice Delete - Only SA and OA can delete
    check:
      - user: user:system-admin
        relation: invoice_delete
        object: invoice:inv-2025-001
        expected: true
      - user: user:org-admin
        relation: invoice_delete
        object: invoice:inv-2025-001
        expected: true
      - user: user:finance-manager
        relation: invoice_delete
        object: invoice:inv-2025-001
        expected: false
      - user: user:invoice-owner
        relation: invoice_delete
        object: invoice:inv-2025-001
        expected: false
  
  - name: Invoice Approve - Approver and finance approver role can approve
    check:
      - user: user:system-admin
        relation: invoice_approve
        object: invoice:inv-2025-001
        expected: true
      - user: user:org-admin
        relation: invoice_approve
        object: invoice:inv-2025-001
        expected: true
      - user: user:finance-approver
        relation: invoice_approve
        object: invoice:inv-2025-001
        expected: true
      - user: user:invoice-owner
        relation: invoice_approve
        object: invoice:inv-2025-001
        expected: false
      - user: user:finance-manager
        relation: invoice_approve
        object: invoice:inv-2025-001
        expected: false
```

### 4.4 Self-Service Access Tests

Create `core/tests/self-service.yaml`:[5][7]

```yaml
# core/tests/self-service.yaml
name: Self-Service Permission Tests
model_file: ../../fga.mod

tuples:
  # Organization and users
  - user: user:system-admin
    relation: system_admin
    object: organization:acme
  - user: user:alice
    relation: member
    object: organization:acme
  - user: user:bob
    relation: member
    object: organization:acme
  
  # Alice's user resource with self-link
  - user: organization:acme
    relation: organization
    object: ciam_user:alice
  - user: user:alice
    relation: is_self
    object: ciam_user:alice
  
  # Bob's user resource with self-link
  - user: organization:acme
    relation: organization
    object: ciam_user:bob
  - user: user:bob
    relation: is_self
    object: ciam_user:bob

tests:
  - name: User can read own expanded profile
    check:
      - user: user:alice
        relation: user_read_expanded
        object: ciam_user:alice
        expected: true
      - user: user:bob
        relation: user_read_expanded
        object: ciam_user:bob
        expected: true
  
  - name: User cannot read other user's expanded profile
    check:
      - user: user:alice
        relation: user_read_expanded
        object: ciam_user:bob
        expected: false
      - user: user:bob
        relation: user_read_expanded
        object: ciam_user:alice
        expected: false
  
  - name: User cannot read other user's basic profile without admin role
    check:
      - user: user:alice
        relation: user_read
        object: ciam_user:bob
        expected: false
  
  - name: System admin can read all user profiles
    check:
      - user: user:system-admin
        relation: user_read
        object: ciam_user:alice
        expected: true
      - user: user:system-admin
        relation: user_read
        object: ciam_user:bob
        expected: true
      - user: user:system-admin
        relation: user_read_expanded
        object: ciam_user:alice
        expected: true
```

### 4.5 CI/CD Pipeline Configuration

Create `.github/workflows/openfga-tests.yml`:[12]

```yaml
name: OpenFGA Authorization Tests

on:
  pull_request:
    paths:
      - 'core/**'
      - 'modules/**'
      - 'fga.mod'
      - '.github/workflows/openfga-tests.yml'
  push:
    branches:
      - main
      - develop

jobs:
  validate-and-test:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Install FGA CLI
        run: |
          brew install fga
      
      - name: Validate model syntax
        run: |
          echo "Validating OpenFGA model syntax..."
          fga model validate --file fga.mod
      
      - name: Run core identity tests
        run: |
          echo "Running core identity tests..."
          fga model test --tests core/tests/identity.yaml
      
      - name: Run core permission tests
        run: |
          echo "Running core permission mapping tests..."
          fga model test --tests core/tests/permissions.yaml
      
      - name: Run organization hierarchy tests
        run: |
          echo "Running organization hierarchy tests..."
          fga model test --tests core/tests/org-hierarchy.yaml
      
      - name: Run self-service tests
        run: |
          echo "Running self-service access tests..."
          fga model test --tests core/tests/self-service.yaml
      
      - name: Run finance module tests
        if: hashFiles('modules/finance/tests/*.yaml') != ''
        run: |
          echo "Running finance module tests..."
          for test_file in modules/finance/tests/*.yaml; do
            fga model test --tests "$test_file"
          done
      
      - name: Run HR module tests
        if: hashFiles('modules/hr/tests/*.yaml') != ''
        run: |
          echo "Running HR module tests..."
          for test_file in modules/hr/tests/*.yaml; do
            fga model test --tests "$test_file"
          done
      
      - name: Run all domain module tests
        run: |
          echo "Running all domain module tests..."
          for test_file in modules/*/tests/*.yaml; do
            echo "Testing: $test_file"
            fga model test --tests "$test_file"
          done
      
      - name: Test model compilation
        run: |
          echo "Compiling model..."
          fga model transform --file fga.mod
      
      - name: Generate test coverage report
        run: |
          echo "Generating test coverage report..."
          echo "Total test files:" $(find . -name "*.yaml" -path "*/tests/*" | wc -l)
          echo "Core tests:" $(find core/tests -name "*.yaml" | wc -l)
          echo "Module tests:" $(find modules/*/tests -name "*.yaml" | wc -l)
      
      - name: Deploy to development environment
        if: github.ref == 'refs/heads/main' && github.event_name == 'push'
        env:
          FGA_STORE_ID: ${{ secrets.FGA_DEV_STORE_ID }}
          FGA_API_URL: ${{ secrets.FGA_API_URL }}
          FGA_API_TOKEN: ${{ secrets.FGA_API_TOKEN }}
        run: |
          echo "Deploying to development environment..."
          fga model write --store-id=$FGA_STORE_ID --file fga.mod
          echo "Deployment successful!"
      
      - name: Notify on failure
        if: failure()
        run: |
          echo "Tests failed! Please review the errors above."
```

***

## Step 5: Application Integration Patterns

### 5.1 Node.js/TypeScript Integration

```typescript
// authorization-service.ts
import { OpenFgaClient } from '@openfga/sdk';

interface CheckPermissionParams {
  userId: string;
  permission: string;
  resourceType: string;
  resourceId: string;
  context?: Record<string, any>;
}

interface AssignSystemRoleParams {
  userId: string;
  role: 'system_admin' | 'org_admin' | 'users_admin' | 'partner_admin' | 'helpdesk' | 'member';
  organizationId: string;
}

class AuthorizationService {
  private fgaClient: OpenFgaClient;

  constructor() {
    this.fgaClient = new OpenFgaClient({
      apiUrl: process.env.FGA_API_URL!,
      storeId: process.env.FGA_STORE_ID!,
      authorizationModelId: process.env.FGA_MODEL_ID,
    });
  }

  /**
   * Check if user has specific permission on a resource
   */
  async checkPermission(params: CheckPermissionParams): Promise<boolean> {
    const { userId, permission, resourceType, resourceId, context } = params;
    
    try {
      const { allowed } = await this.fgaClient.check({
        user: `user:${userId}`,
        relation: permission,
        object: `${resourceType}:${resourceId}`,
        context: context,
      });
      
      return allowed;
    } catch (error) {
      console.error('Authorization check failed:', error);
      throw new Error('Authorization check failed');
    }
  }

  /**
   * Assign system role to user in organization
   */
  async assignSystemRole(params: AssignSystemRoleParams): Promise<void> {
    const { userId, role, organizationId } = params;
    
    await this.fgaClient.write({
      writes: [{
        user: `user:${userId}`,
        relation: role,
        object: `organization:${organizationId}`
      }]
    });
  }

  /**
   * Assign domain-specific role to user
   */
  async assignDomainRole(userId: string, roleId: string): Promise<void> {
    await this.fgaClient.write({
      writes: [{
        user: `user:${userId}`,
        relation: 'assignee',
        object: `role:${roleId}`
      }]
    });
  }

  /**
   * Add user to organization as member
   */
  async addUserToOrganization(userId: string, organizationId: string): Promise<void> {
    await this.fgaClient.write({
      writes: [
        // Add as member
        {
          user: `user:${userId}`,
          relation: 'member',
          object: `organization:${organizationId}`
        },
        // Create CIAM user resource
        {
          user: `organization:${organizationId}`,
          relation: 'organization',
          object: `ciam_user:${userId}`
        },
        // Link user to their CIAM resource for self-service
        {
          user: `user:${userId}`,
          relation: 'is_self',
          object: `ciam_user:${userId}`
        }
      ]
    });
  }

  /**
   * Create organization hierarchy (parent-child)
   */
  async createOrganizationHierarchy(parentOrgId: string, childOrgId: string): Promise<void> {
    await this.fgaClient.write({
      writes: [
        {
          user: `organization:${parentOrgId}`,
          relation: 'parent',
          object: `organization:${childOrgId}`
        },
        {
          user: `organization:${childOrgId}`,
          relation: 'child',
          object: `organization:${parentOrgId}`
        }
      ]
    });
  }

  /**
   * Check CIAM-specific permissions
   */
  async checkUserPermissions(currentUserId: string, targetUserId: string, organizationId: string) {
    return {
      canRead: await this.checkPermission({
        userId: currentUserId,
        permission: 'user_read',
        resourceType: 'ciam_user',
        resourceId: targetUserId
      }),
      canReadExpanded: await this.checkPermission({
        userId: currentUserId,
        permission: 'user_read_expanded',
        resourceType: 'ciam_user',
        resourceId: targetUserId
      }),
      canWrite: await this.checkPermission({
        userId: currentUserId,
        permission: 'user_write',
        resourceType: 'ciam_user',
        resourceId: targetUserId
      }),
      canDelete: await this.checkPermission({
        userId: currentUserId,
        permission: 'user_delete',
        resourceType: 'ciam_user',
        resourceId: targetUserId
      })
    };
  }

  /**
   * List all resources of a type that user can access with specific permission
   */
  async listAccessibleResources(userId: string, resourceType: string, permission: string): Promise<string[]> {
    const { objects } = await this.fgaClient.listObjects({
      user: `user:${userId}`,
      relation: permission,
      type: resourceType
    });
    
    return objects;
  }

  /**
   * Batch permission check for multiple resources
   */
  async batchCheckPermissions(
    userId: string,
    checks: Array<{ permission: string; resourceType: string; resourceId: string }>
  ): Promise<Record<string, boolean>> {
    const results: Record<string, boolean> = {};
    
    await Promise.all(
      checks.map(async (check) => {
        const key = `${check.resourceType}:${check.resourceId}:${check.permission}`;
        results[key] = await this.checkPermission({
          userId,
          permission: check.permission,
          resourceType: check.resourceType,
          resourceId: check.resourceId
        });
      })
    );
    
    return results;
  }
}

export default AuthorizationService;

// Usage examples
const authService = new AuthorizationService();

// Check if user can read another user's profile
const canRead = await authService.checkPermission({
  userId: 'alice@acme.com',
  permission: 'user_read',
  resourceType: 'ciam_user',
  resourceId: 'bob@acme.com'
});

// Check if user can read their own expanded profile (self-service)
const canReadOwn = await authService.checkPermission({
  userId: 'alice@acme.com',
  permission: 'user_read_expanded',
  resourceType: 'ciam_user',
  resourceId: 'alice@acme.com'
});

// Assign organization admin role
await authService.assignSystemRole({
  userId: 'bob@acme.com',
  role: 'org_admin',
  organizationId: 'acme-corp'
});

// Check invoice permissions (domain resource)
const canApproveInvoice = await authService.checkPermission({
  userId: 'finance-manager@acme.com',
  permission: 'invoice_approve',
  resourceType: 'invoice',
  resourceId: 'inv-2025-001'
});

// List all invoices user can read
const accessibleInvoices = await authService.listAccessibleResources(
  'finance-viewer@acme.com',
  'invoice',
  'invoice_read'
);
```

### 5.2 REST API Middleware Integration

```typescript
// express-middleware.ts
import { Request, Response, NextFunction } from 'express';
import AuthorizationService from './authorization-service';

const authService = new AuthorizationService();

/**
 * Generic permission check middleware factory
 */
export function requirePermission(
  permission: string,
  resourceTypeExtractor: (req: Request) => string,
  resourceIdExtractor: (req: Request) => string
) {
  return async (req: Request, res: Response, next: NextFunction) => {
    try {
      const userId = req.user?.id; // Assuming authentication middleware sets req.user
      
      if (!userId) {
        return res.status(401).json({ error: 'Unauthorized' });
      }
      
      const resourceType = resourceTypeExtractor(req);
      const resourceId = resourceIdExtractor(req);
      
      const allowed = await authService.checkPermission({
        userId,
        permission,
        resourceType,
        resourceId
      });
      
      if (!allowed) {
        return res.status(403).json({ 
          error: 'Forbidden',
          message: `User does not have ${permission} permission on ${resourceType}:${resourceId}`
        });
      }
      
      next();
    } catch (error) {
      console.error('Authorization middleware error:', error);
      res.status(500).json({ error: 'Internal server error' });
    }
  };
}

/**
 * CIAM-specific middleware examples
 */

// Check if user can read users
export const requireUserRead = requirePermission(
  'user_read',
  () => 'ciam_user',
  (req) => req.params.userId
);

// Check if user can write users
export const requireUserWrite = requirePermission(
  'user_write',
  () => 'ciam_user',
  (req) => req.params.userId
);

// Check if user can manage groups
export const requireGroupsWrite = requirePermission(
  'groups_write',
  () => 'group',
  (req) => req.params.groupId
);

// Check if user can read organization
export const requireOrgRead = requirePermission(
  'org_read',
  () => 'organization',
  (req) => req.params.orgId
);

/**
 * Domain-specific middleware examples
 */

// Check if user can read invoice (finance domain)
export const requireInvoiceRead = requirePermission(
  'invoice_read',
  () => 'invoice',
  (req) => req.params.invoiceId
);

// Check if user can approve invoice (finance domain)
export const requireInvoiceApprove = requirePermission(
  'invoice_approve',
  () => 'invoice',
  (req) => req.params.invoiceId
);

// Usage in Express routes:
/*
import express from 'express';
import { 
  requireUserRead, 
  requireUserWrite,
  requireInvoiceRead,
  requireInvoiceApprove 
} from './express-middleware';

const app = express();

// CIAM routes
app.get('/api/users/:userId', requireUserRead, async (req, res) => {
  // User has user_read permission, fetch and return user data
  const user = await getUserById(req.params.userId);
  res.json(user);
});

app.put('/api/users/:userId', requireUserWrite, async (req, res) => {
  // User has user_write permission, update user
  await updateUser(req.params.userId, req.body);
  res.json({ success: true });
});

// Finance domain routes
app.get('/api/invoices/:invoiceId', requireInvoiceRead, async (req, res) => {
  // User has invoice_read permission
  const invoice = await getInvoice(req.params.invoiceId);
  res.json(invoice);
});

app.post('/api/invoices/:invoiceId/approve', requireInvoiceApprove, async (req, res) => {
  // User has invoice_approve permission
  await approveInvoice(req.params.invoiceId);
  res.json({ success: true });
});
*/
```

***

## Step 6: Governance and Team Collaboration

### 6.1 CODEOWNERS Configuration

Create `.github/CODEOWNERS`:

```
# OpenFGA Authorization Model Code Owners

# Global owners - CIAM team has oversight on all changes
* @ciam-team

# Core model - CIAM team has exclusive ownership
fga.mod @ciam-team
core/ @ciam-team

# Domain modules - require both CIAM and domain team approval
# CIAM ensures integration integrity, domain teams own business logic

modules/finance/ @ciam-team @finance-team
modules/hr/ @ciam-team @hr-team
modules/procurement/ @ciam-team @procurement-team
modules/sales/ @ciam-team @sales-team
modules/marketing/ @ciam-team @marketing-team
modules/customer-support/ @ciam-team @support-team

# Test files must be reviewed by both teams
modules/finance/tests/ @ciam-team @finance-team
modules/hr/tests/ @ciam-team @hr-team

# CI/CD workflows - CIAM team only
.github/workflows/ @ciam-team
```

### 6.2 Module Development Guidelines

Create `CONTRIBUTING.md`:

```markdown
# OpenFGA Authorization Model Contribution Guidelines

## Overview
This repository contains the enterprise-wide authorization model managed by the CIAM team and extended by domain teams.

## CIAM Team Responsibilities

### Core Resources Managed by CIAM
- **Organization**: Including hierarchy (parent/child/descendants)
- **Users** (ciam_user type)
- **Groups**
- **Roles** (system roles and role infrastructure)
- **Applications/Offerings**
- **Domains**
- **OrganizationIdpConfigurations**
- **FederationProfiles**
- **B2CPolicies**
- **Entitlements**
- **HomeRealm**
- **FederatedUsers**
- **InternalMpsOrganizations**

### System Roles Managed by CIAM
- **SA** (System Admin)
- **OA** (Organization Admin)
- **UA** (Users Admin)
- **PA** (Partner Admin)
- **HELPDESK** (Helpdesk)
- **USER** (Regular User/Member)

## Domain Team Responsibilities

### What Domain Teams Define
1. **Domain-Specific Resources**: Invoices, Contracts, Tickets, Campaigns, etc.
2. **Domain-Specific Roles**: Finance Manager, HR Partner, Sales Rep, etc.
3. **Domain-Specific Permissions**: invoice_approve, contract_sign, etc.

### What Domain Teams Must Follow
1. **All resources MUST link to organization**
2. **All permissions MUST respect system role hierarchy**
3. **All modules MUST extend, not modify, core types**
4. **All modules MUST include comprehensive tests**

## Module Structure Requirements

### 1. File Organization
```
modules/[domain]/
├── [domain].fga          # Domain model
└── tests/
    └── [domain].yaml     # Domain tests
```

### 2. Required Module Pattern
```
# modules/[domain]/[domain].fga
module [domain_name]

# Extend organization for domain roles
extend type organization
  relations
    define [domain]_admin: [role#assignee]
    define [domain]_manager: [role#assignee]

# Define domain resources
type [domain]_resource
  relations
    define organization: [organization]  # REQUIRED
    
    # Permission pattern following CIAM standards
    define [resource]_read: [viewers] or (system_admin from organization) or (org_admin from organization)
    define [resource]_write: [editors] or (system_admin from organization) or (org_admin from organization)
    define [resource]_create: (system_admin from organization) or (org_admin from organization)
    define [resource]_delete: (system_admin from organization) or (org_admin from organization)
```

### 3. Permission Naming Convention
Follow existing CIAM pattern:
- **Read permissions**: `[resource]_read`, `[resource]_read_expanded`
- **Write permissions**: `[resource]_write`, `[resource]_create`, `[resource]_update`, `[resource]_delete`
- **Special permissions**: `[resource]_[action]` (e.g., `invoice_approve`, `contract_sign`)

### 4. System Role Integration
Always include system roles in permission definitions:
```
define [resource]_read: [specific_viewers] or 
                        (system_admin from organization) or 
                        (org_admin from organization) or
                        (helpdesk from organization)  # if appropriate
```

### 5. Testing Requirements
- **Minimum 80% coverage** of permission scenarios
- Test system role access (SA, OA, UA, PA, HELPDESK, USER)
- Test domain role access
- Test cross-organization isolation
- Test edge cases and negative scenarios

## Development Workflow

### For CIAM Team
1. Modify core model in `core/` directory
2. Update tests in `core/tests/`
3. Run all tests: `./scripts/test-all.sh`
4. Create PR with "CIAM:" prefix
5. Require 2 CIAM team approvals
6. Merge after CI passes

### For Domain Teams
1. Create feature branch from `main`
2. Develop module in `modules/[domain]/`
3. Write comprehensive tests
4. Run local tests: `fga model test --tests modules/[domain]/tests/[domain].yaml`
5. Run full suite: `./scripts/test-all.sh`
6. Create PR with "[Domain]:" prefix (e.g., "Finance: Add invoice approval workflow")
7. Require 1 CIAM team approval + 1 domain team approval
8. Address feedback
9. Merge after CI passes

## PR Template

```
## Description
[Describe the changes]

## Type of Change
- [ ] CIAM Core Model Update
- [ ] New Domain Module
- [ ] Domain Module Update
- [ ] Bug Fix
- [ ] Documentation

## Domain Team (if applicable)
- [ ] Finance
- [ ] HR
- [ ] Procurement
- [ ] Sales
- [ ] Marketing
- [ ] Other: _____

## Checklist
- [ ] Tests added/updated
- [ ] All tests pass locally
- [ ] Documentation updated
- [ ] No breaking changes to existing permissions
- [ ] System roles (SA, OA, etc.) properly integrated
- [ ] Organization context maintained

## Test Coverage
- Core permission tests: [X/Y passed]
- Domain permission tests: [X/Y passed]
- Integration tests: [X/Y passed]

## Impact Analysis
[Describe impact on existing modules and permissions]
```

## Testing Locally

```
# Install FGA CLI
brew install fga

# Validate syntax
fga model validate --file fga.mod

# Test specific module
fga model test --tests modules/finance/tests/finance.yaml

# Test all modules
./scripts/test-all.sh

# Transform and inspect compiled model
fga model transform --file fga.mod
```

## Common Patterns

### Pattern 1: Self-Service Access
```
type my_resource
  relations
    define owner: [user]
    define is_owner: owner
    
    define [resource]_read: is_owner or (admin from organization)
```

### Pattern 2: Approval Workflow
```
type my_resource
  relations
    define submitter: [user]
    define approver: [user, role#assignee]
    
    define [resource]_submit: (member from organization)
    define [resource]_approve: approver or (org_admin from organization)
```

### Pattern 3: Hierarchical Access
```
type parent_resource
  relations
    define viewer: [user, group#member]

type child_resource
  relations
    define parent: [parent_resource]
    
    define [resource]_read: [user] or (viewer from parent)
```

## Support and Questions
- **Slack**: #ciam-authorization
- **Email**: ciam-team@company.com
- **Office Hours**: Tuesdays 2-3 PM, Thursdays 10-11 AM
- **Documentation**: https://wiki.company.com/openfga
```

### 6.3 Testing Script

Create `scripts/test-all.sh`:

```bash
#!/bin/bash
# scripts/test-all.sh - Comprehensive test runner

set -e

# Colors for output
GREEN='\033[0;32m'
RED='\033[0;31m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m' # No Color

# Counters
TOTAL_TESTS=0
PASSED_TESTS=0
FAILED_TESTS=0
FAILED_FILES=()

echo -e "${BLUE}==================================================${NC}"
echo -e "${BLUE}OpenFGA Enterprise Authorization Test Suite${NC}"
echo -e "${BLUE}==================================================${NC}"
echo ""

# Function to run test file
run_test() {
    local test_file=$1
    local test_name=$(basename $test_file .yaml)
    local test_dir=$(dirname $test_file)
    
    echo -n "Testing $test_dir/$test_name... "
    TOTAL_TESTS=$((TOTAL_TESTS + 1))
    
    if fga model test --tests "$test_file" > /tmp/fga_test_output.log 2>&1; then
        echo -e "${GREEN}✓ PASSED${NC}"
        PASSED_TESTS=$((PASSED_TESTS + 1))
    else
        echo -e "${RED}✗ FAILED${NC}"
        FAILED_TESTS=$((FAILED_TESTS + 1))
        FAILED_FILES+=("$test_file")
        echo -e "${RED}Error output:${NC}"
        cat /tmp/fga_test_output.log
        echo ""
    fi
}

# Validate model syntax first
echo -e "${YELLOW}Step 1: Validating model syntax...${NC}"
if fga model validate --file fga.mod; then
    echo -e "${GREEN}✓ Model syntax valid${NC}"
    echo ""
else
    echo -e "${RED}✗ Model syntax invalid. Fix errors before running tests.${NC}"
    exit 1
fi

# Run core tests
echo -e "${YELLOW}Step 2: Running Core CIAM Tests${NC}"
echo "----------------------------------------"
for test_file in core/tests/*.yaml; do
    if [ -f "$test_file" ]; then
        run_test "$test_file"
    fi
done
echo ""

# Run domain module tests
echo -e "${YELLOW}Step 3: Running Domain Module Tests${NC}"
echo "----------------------------------------"
for test_file in modules/*/tests/*.yaml; do
    if [ -f "$test_file" ]; then
        run_test "$test_file"
    fi
done
echo ""

# Summary
echo -e "${BLUE}==================================================${NC}"
echo -e "${BLUE}Test Summary${NC}"
echo -e "${BLUE}==================================================${NC}"
echo "Total tests: $TOTAL_TESTS"
echo -e "Passed: ${GREEN}$PASSED_TESTS${NC}"
echo -e "Failed: ${RED}$FAILED_TESTS${NC}"

if [ $FAILED_TESTS -gt 0 ]; then
    echo ""
    echo -e "${RED}Failed test files:${NC}"
    for failed_file in "${FAILED_FILES[@]}"; do
        echo -e "  ${RED}✗${NC} $failed_file"
    done
    echo ""
    echo -e "${RED}Some tests failed. Please fix the issues above.${NC}"
    exit 1
else
    echo ""
    echo -e "${GREEN}All tests passed! ✓${NC}"
    echo ""
    
    # Optional: Generate model transform
    echo -e "${YELLOW}Generating compiled model...${NC}"
    fga model transform --file fga.mod > compiled-model.json
    echo -e "${GREEN}Compiled model saved to compiled-model.json${NC}"
    
    exit 0
fi
```

Make the script executable:
```bash
chmod +x scripts/test-all.sh
```

***

## Step 7: Implementation Roadmap

### Phase 1: Foundation (Weeks 1-4)

**Week 1-2: Core CIAM Model Development**
- [ ] Set up repository structure
- [ ] Create `fga.mod` manifest
- [ ] Develop `core/identity.fga` with all CIAM resources
- [ ] Map all existing permissions to OpenFGA relations
- [ ] Develop `core/system-roles.fga`
- [ ] Write comprehensive core tests matching existing permission table

**Week 3-4: Testing Infrastructure and Documentation**
- [ ] Create `core/tests/permissions.yaml` covering all permission mappings
- [ ] Create `core/tests/org-hierarchy.yaml` for organization hierarchies
- [ ] Create `core/tests/self-service.yaml` for USER self-access scenarios
- [ ] Set up CI/CD pipeline
- [ ] Create module development guidelines
- [ ] Set up CODEOWNERS
- [ ] Create comprehensive documentation
- [ ] Conduct CIAM team training

### Phase 2: Pilot Domain Module (Weeks 5-8)

**Week 5-6: Finance Module (Pilot)**
- [ ] Select Finance team as pilot
- [ ] Conduct joint workshop with Finance team
- [ ] Develop `modules/finance/finance.fga`
- [ ] Map Finance resources (invoices, expenses, financial reports)
- [ ] Define finance-specific roles (finance_admin, finance_manager, etc.)
- [ ] Write comprehensive finance module tests
- [ ] Deploy to development environment

**Week 7-8: Refinement Based on Pilot**
- [ ] Gather feedback from Finance team
- [ ] Identify pain points and areas for improvement
- [ ] Refine module development template
- [ ] Update documentation with learnings
- [ ] Conduct performance testing
- [ ] Security review of pilot implementation
- [ ] Document best practices

### Phase 3: Expansion to Additional Domains (Weeks 9-16)

**Week 9-10: HR Module**
- [ ] Onboard HR team
- [ ] Develop HR module (employees, records, reviews)
- [ ] Implement self-service access for employees
- [ ] Write HR module tests
- [ ] Integration testing with core

**Week 11-12: Procurement Module**
- [ ] Onboard Procurement team
- [ ] Develop Procurement module
- [ ] Implement approval workflows
- [ ] Write Procurement module tests

**Week 13-14: Additional Domains**
- [ ] Onboard Sales team
- [ ] Onboard Marketing team
- [ ] Onboard Customer Support team
- [ ] Develop respective modules
- [ ] Integration testing across modules

**Week 15-16: Production Preparation**
- [ ] Comprehensive integration testing
- [ ] Load and performance testing
- [ ] Security audit
- [ ] Create runbooks and operational procedures
- [ ] User acceptance testing
- [ ] Production deployment plan

### Phase 4: Production Rollout (Weeks 17-20)

**Week 17-18: Staging Deployment**
- [ ] Deploy to staging environment
- [ ] Migrate existing authorization data
- [ ] Parallel run with existing system
- [ ] Validate permission parity
- [ ] Performance benchmarking

**Week 19-20: Production Deployment**
- [ ] Phased production rollout (one domain at a time)
- [ ] Monitor authorization check latency
- [ ] Monitor error rates
- [ ] Gradual traffic migration
- [ ] Rollback plan ready
- [ ] Full cutover

### Phase 5: Optimization and Maturity (Ongoing)

- [ ] Regular model reviews (quarterly)
- [ ] Performance optimization based on metrics
- [ ] Additional domain team onboarding
- [ ] Advanced features implementation (conditions, contextual access)
- [ ] Training programs for new team members
- [ ] Documentation updates
- [ ] Continuous improvement based on feedback

***

## Step 8: Monitoring and Observability

### 8.1 Key Metrics Dashboard

```yaml
# monitoring/metrics-dashboard.yaml
dashboard: OpenFGA Authorization Metrics

panels:
  - title: Authorization Check Performance
    metrics:
      - name: authorization_check_latency_p50
        query: histogram_quantile(0.50, rate(fga_check_duration_seconds_bucket[5m]))
        threshold_warning: 50ms
        threshold_critical: 100ms
      
      - name: authorization_check_latency_p95
        query: histogram_quantile(0.95, rate(fga_check_duration_seconds_bucket[5m]))
        threshold_warning: 100ms
        threshold_critical: 200ms
      
      - name: authorization_check_latency_p99
        query: histogram_quantile(0.99, rate(fga_check_duration_seconds_bucket[5m]))
        threshold_warning: 200ms
        threshold_critical: 500ms
  
  - title: Authorization Check Volume
    metrics:
      - name: authorization_checks_total
        query: sum(rate(fga_check_total[5m])) by (relation, object_type)
        
      - name: authorization_checks_by_role
        query: sum(rate(fga_check_total[5m])) by (user_role)
  
  - title: Authorization Denials
    metrics:
      - name: authorization_denied_rate
        query: sum(rate(fga_check_total{allowed="false"}[5m]))
        threshold_warning: 100/s
        threshold_critical: 500/s
      
      - name: authorization_denied_by_permission
        query: sum(rate(fga_check_total{allowed="false"}[5m])) by (relation, object_type)
  
  - title: System Role Distribution
    metrics:
      - name: system_admin_count
        query: count(fga_tuple{relation="system_admin"})
      
      - name: org_admin_count
        query: count(fga_tuple{relation="org_admin"})
      
      - name: users_admin_count
        query: count(fga_tuple{relation="users_admin"})
      
      - name: partner_admin_count
        query: count(fga_tuple{relation="partner_admin"})
  
  - title: Model Updates
    metrics:
      - name: model_updates_total
        query: sum(rate(fga_model_write_total[1h])) by (environment)
      
      - name: model_validation_failures
        query: sum(rate(fga_model_validation_failure_total[1h]))
  
  - title: Domain Module Usage
    metrics:
      - name: checks_by_domain
        query: sum(rate(fga_check_total[5m])) by (module)

alerts:
  - name: HighAuthorizationLatency
    condition: authorization_check_latency_p99 > 500ms
    severity: critical
    description: "P99 authorization check latency exceeds 500ms"
    action: "Investigate OpenFGA performance, check database latency"
  
  - name: HighDenialRate
    condition: authorization_denied_rate > 500/s
    severity: warning
    description: "High rate of authorization denials"
    action: "Check for potential security issues or permission misconfigurations"
  
  - name: ModelValidationFailures
    condition: model_validation_failures > 0
    severity: critical
    description: "Model validation failures detected"
    action: "Review failed model updates, check CI/CD pipeline"
```

### 8.2 Audit Logging Implementation

```typescript
// audit-logger.ts
import { OpenFgaClient } from '@openfga/sdk';

interface AuditEvent {
  event_type: 'authorization_check' | 'role_assignment' | 'role_removal' | 'permission_grant' | 'permission_revoke';
  timestamp: string;
  user_id: string;
  resource_type?: string;
  resource_id?: string;
  permission?: string;
  result?: boolean;
  target_user?: string;
  role?: string;
  organization_id?: string;
  ip_address?: string;
  user_agent?: string;
  duration_ms?: number;
  error?: string;
}

class AuditLogger {
  private logDestination: string;

  constructor(logDestination: string) {
    this.logDestination = logDestination;
  }

  /**
   * Log authorization check
   */
  async logAuthorizationCheck(event: {
    userId: string;
    permission: string;
    resourceType: string;
    resourceId: string;
    result: boolean;
    durationMs: number;
    ipAddress?: string;
    userAgent?: string;
  }): Promise<void> {
    const auditEvent: AuditEvent = {
      event_type: 'authorization_check',
      timestamp: new Date().toISOString(),
      user_id: event.userId,
      resource_type: event.resourceType,
      resource_id: event.resourceId,
      permission: event.permission,
      result: event.result,
      duration_ms: event.durationMs,
      ip_address: event.ipAddress,
      user_agent: event.userAgent
    };

    await this.writeAuditLog(auditEvent);
  }

  /**
   * Log role assignment
   */
  async logRoleAssignment(event: {
    adminUserId: string;
    targetUserId: string;
    role: string;
    organizationId: string;
    ipAddress?: string;
  }): Promise<void> {
    const auditEvent: AuditEvent = {
      event_type: 'role_assignment',
      timestamp: new Date().toISOString(),
      user_id: event.adminUserId,
      target_user: event.targetUserId,
      role: event.role,
      organization_id: event.organizationId,
      ip_address: event.ipAddress
    };

    await this.writeAuditLog(auditEvent);
  }

  /**
   * Log role removal
   */
  async logRoleRemoval(event: {
    adminUserId: string;
    targetUserId: string;
    role: string;
    organizationId: string;
    ipAddress?: string;
  }): Promise<void> {
    const auditEvent: AuditEvent = {
      event_type: 'role_removal',
      timestamp: new Date().toISOString(),
      user_id: event.adminUserId,
      target_user: event.targetUserId,
      role: event.role,
      organization_id: event.organizationId,
      ip_address: event.ipAddress
    };

    await this.writeAuditLog(auditEvent);
  }

  /**
   * Write audit log to destination (CloudWatch, Splunk, etc.)
   */
  private async writeAuditLog(event: AuditEvent): Promise<void> {
    // Implementation depends on your logging infrastructure
    // Examples: CloudWatch Logs, Splunk, ELK Stack, Datadog
    
    console.log('AUDIT:', JSON.stringify(event));
    
    // Example: Send to CloudWatch
    // await cloudWatchClient.putLogEvents({...});
    
    // Example: Send to Splunk HEC
    // await splunkClient.send(event);
    
    // Example: Send to Datadog
    // await datadogClient.log(event);
  }
}

// Wrapper service with audit logging
class AuditedAuthorizationService extends AuthorizationService {
  private auditLogger: AuditLogger;

  constructor(auditLogger: AuditLogger) {
    super();
    this.auditLogger = auditLogger;
  }

  async checkPermission(params: CheckPermissionParams, context?: { ipAddress?: string; userAgent?: string }): Promise<boolean> {
    const startTime = Date.now();
    const result = await super.checkPermission(params);
    const duration = Date.now() - startTime;

    // Log the authorization check
    await this.auditLogger.logAuthorizationCheck({
      userId: params.userId,
      permission: params.permission,
      resourceType: params.resourceType,
      resourceId: params.resourceId,
      result,
      durationMs: duration,
      ipAddress: context?.ipAddress,
      userAgent: context?.userAgent
    });

    return result;
  }

  async assignSystemRole(params: AssignSystemRoleParams, context?: { adminUserId: string; ipAddress?: string }): Promise<void> {
    await super.assignSystemRole(params);

    // Log the role assignment
    if (context?.adminUserId) {
      await this.auditLogger.logRoleAssignment({
        adminUserId: context.adminUserId,
        targetUserId: params.userId,
        role: params.role,
        organizationId: params.organizationId,
        ipAddress: context.ipAddress
      });
    }
  }
}

export { AuditLogger, AuditedAuthorizationService };
```

***

## Step 9: Security Considerations

### 9.1 Security Best Practices

```markdown
# Security Guidelines for OpenFGA Authorization

## 1. Principle of Least Privilege
- Assign minimum necessary permissions
- Regular permission audits (quarterly)
- Review and revoke unused permissions
- Time-bound access for contractors/temporary users

## 2. System Role Protection
- **System Admin (SA)**: 
  - Manually provisioned only
  - Require MFA
  - Limit to 2-5 users
  - Log all SA actions
  - Annual recertification

- **Organization Admin (OA)**:
  - Require approval workflow
  - MFA required
  - Maximum 10 per organization
  - Quarterly access review

- **Users Admin (UA)**:
  - Scoped to specific organizations
  - Cannot escalate to OA or SA
  - Audit user creation/deletion

## 3. Organization Isolation
- All resources MUST link to organization
- Cross-organization access requires explicit tuples
- Implement organization context in all checks
- Validate organization membership before permission checks

## 4. Self-Service Security
- Users can only access their own ciam_user resource
- Implement `is_self` relation validation
- Log all self-service access
- Rate limit self-service operations

## 5. Domain Module Security
- Domain teams cannot modify core CIAM types
- System roles always have override access
- Validate organization context in domain permissions
- Security review required for new modules

## 6. Tuple Management Security
- Validate tuple writes before committing
- Implement approval workflows for sensitive role assignments
- Audit trail for all tuple modifications
- Implement tuple TTL for temporary access

## 7. API Security
- Rate limiting on authorization checks
- Authentication required before authorization
- Validate user identity before checks
- Implement request signing for sensitive operations

## 8. Monitoring and Alerting
- Alert on unusual authorization patterns
- Monitor for privilege escalation attempts
- Track authorization denial spikes
- Alert on SA/OA role assignments
```

### 9.2 Security Testing

Create `core/tests/security.yaml`:

```yaml
# core/tests/security.yaml
name: Security Tests
model_file: ../../fga.mod

tests:
  - name: Cross-organization isolation
    tuples:
      - user: user:alice
        relation: org_admin
        object: organization:org-a
      - user: user:bob
        relation: member
        object: organization:org-b
      - user: organization:org-a
        relation: organization
        object: ciam_user:john-a
      - user: organization:org-b
        relation: organization
        object: ciam_user:john-b
    check:
      # Alice from org-a cannot access users in org-b
      - user: user:alice
        relation: user_read
        object: ciam_user:john-b
        expected: false
      # Bob from org-b cannot access users in org-a
      - user: user:bob
        relation: user_read
        object: ciam_user:john-a
        expected: false
  
  - name: Privilege escalation prevention
    tuples:
      - user: user:regular-user
        relation: member
        object: organization:acme
      - user: user:users-admin
        relation: users_admin
        object: organization:acme
    check:
      # Regular user cannot become admin
      - user: user:regular-user
        relation: org_admin
        object: organization:acme
        expected: false
      - user: user:regular-user
        relation: system_admin
        object: organization:acme
        expected: false
      # Users admin cannot escalate to org admin
      - user: user:users-admin
        relation: org_admin
        object: organization:acme
        expected: false
  
  - name: Self-service isolation
    tuples:
      - user: user:alice
        relation: member
        object: organization:acme
      - user: user:bob
        relation: member
        object: organization:acme
      - user: organization:acme
        relation: organization
        object: ciam_user:alice
      - user: user:alice
        relation: is_self
        object: ciam_user:alice
      - user: organization:acme
        relation: organization
        object: ciam_user:bob
      - user: user:bob
        relation: is_self
        object: ciam_user:bob
    check:
      # Alice can read own expanded profile
      - user: user:alice
        relation: user_read_expanded
        object: ciam_user:alice
        expected: true
      # Alice cannot read Bob's profile
      - user: user:alice
        relation: user_read_expanded
        object: ciam_user:bob
        expected: false
      # Bob can read own expanded profile
      - user: user:bob
        relation: user_read_expanded
        object: ciam_user:bob
        expected: true
      # Bob cannot read Alice's profile
      - user: user:bob
        relation: user_read_expanded
        object: ciam_user:alice
        expected: false
  
  - name: System Admin override
    tuples:
      - user: user:system-admin
        relation: system_admin
        object: organization:acme
      - user: organization:acme
        relation: organization
        object: invoice:inv-001
    check:
      # SA can perform all operations
      - user: user:system-admin
        relation: invoice_read
        object: invoice:inv-001
        expected: true
      - user: user:system-admin
        relation: invoice_write
        object: invoice:inv-001
        expected: true
      - user: user:system-admin
        relation: invoice_delete
        object: invoice:inv-001
        expected: true
  
  - name: Helpdesk read-only enforcement
    tuples:
      - user: user:helpdesk
        relation: helpdesk
        object: organization:acme
      - user: organization:acme
        relation: organization
        object: ciam_user:john
      - user: organization:acme
        relation: organization
        object: group:engineering
    check:
      # Helpdesk can read
      - user: user:helpdesk
        relation: user_read
        object: ciam_user:john
        expected: true
      - user: user:helpdesk
        relation: groups_read
        object: group:engineering
        expected: true
      # Helpdesk cannot write
      - user: user:helpdesk
        relation: user_write
        object: ciam_user:john
        expected: false
      - user: user:helpdesk
        relation: groups_write
        object: group:engineering
        expected: false
      - user: user:helpdesk
        relation: user_delete
        object: ciam_user:john
        expected: false
```

***

## Conclusion

This comprehensive design plan provides a production-ready implementation of OpenFGA tailored to your existing permission structure and organizational needs. The design ensures:[13][2][1]

1. **Accurate Permission Mapping**: All existing permissions (UserRead, UserWrite, OrgRead, etc.) are mapped to OpenFGA relations[6]
2. **System Role Integration**: SA, OA, UA, PA, HELPDESK, and USER roles properly implemented with correct access levels[14][4]
3. **Organization Hierarchies**: Support for parent-child and descendant relationships[8][9]
4. **Self-Service Capabilities**: Users can access their own data (UserReadExpanded)[7][5]
5. **Modular Architecture**: CIAM core + domain extensions for independent team management[2][1]
6. **Comprehensive Testing**: Full test coverage for all permission scenarios[11][10]
7. **Production Ready**: Complete with CI/CD, monitoring, audit logging, and security controls[12]

### Next Steps

1. **Week 1**: Review this plan with CIAM team and key stakeholders
2. **Week 2**: Set up repository an
