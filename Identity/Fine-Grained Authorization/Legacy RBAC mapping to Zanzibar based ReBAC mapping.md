# RBAC to Zanzibar/OpenFGA/SpiceDB Migration Reference

## Existing Resource → Permission → Role Mapping

| **Resource** | **Permission** | **Roles with Access** |
|---------------|----------------|------------------------|
| Users | UserRead | SA, OA, UA, HELPDESK |
| Users | UserReadExpanded | SA, OA, UA, HELPDESK, USER (only self) |
| Users | UserWrite | SA, OA, UA |
| Users | UserCreate | SA, OA, UA |
| Users | UserDelete | SA, OA, UA |
| Users | UserUpdate | SA, OA, UA |
| Users | UserBulkImport | SA, OA, UA |
| Organizations | OrgRead | SA, OA, UA, HELPDESK, PA, USER |
| Organizations | OrgReadAndSelf | SA, OA, UA, HELPDESK, PA, USER |
| Organizations | OrgWrite | SA, OA |
| Organizations | OrgCreate | SA, OA |
| Organizations | OrgDelete | SA, OA |
| Organizations | OrgReadForUserSearch | SA, OA |
| OrganizationIdpConfigurations | IdpConfigRead | SA, OA, HELPDESK |
| OrganizationIdpConfigurations | IdpConfigWrite | SA, OA |
| OrganizationChildren | OrgReadChildren | SA, PA, HELPDESK |
| OrganizationChildren | OrgWriteChildren | SA, PA |
| OrganizationChildren | OrgDeleteChildren | OA |
| OrganizationDescendants | OrgReadDescendants | SA, PA |
| OrganizationRoleGrants | RoleGrantsOrgRead | SA, OA, UA, HELPDESK |
| OrganizationRoleGrants | RoleGrantsOrgWrite | SA, OA |
| FederationProfiles | FederationProfilesRead | SA, OA, HELPDESK |
| FederationProfiles | FederationProfilesWrite | SA, OA |
| B2C Policies | PolicyDeploymentsRead | SA |
| B2C Policies | PolicyDeploymentsWrite | SA |
| Roles | RolesRead | SA, OA, UA, PA, HELPDESK, USER |
| Roles | RolesWrite | SA |
| Groups | GroupsRead | SA, OA, UA, HELPDESK, USER |
| Groups | GroupsWrite | SA, OA, UA |
| Groups | GroupsReadById | SA, OA, UA, HELPDESK, USER |
| Groups | GroupsSearch | SA, OA, UA, HELPDESK, USER |
| Groups | GroupsDelete | SA, OA, UA |
| GroupMembers | GroupMembersRead | SA, OA, UA, HELPDESK |
| GroupMembers | GroupMembersWrite | SA, OA, UA |
| GroupRoleGrants | GroupRoleGrantsRead | SA, OA, UA, HELPDESK |
| GroupRoleGrants | GroupRoleGrantsWrite | SA, OA, UA |
| Domains | DomainsRead | SA, OA, HELPDESK |
| Domains | DomainsWrite | SA, OA |
| Offerings | OfferingsRead | SA, OA, UA, PA, HELPDESK, USER |
| Offerings | OfferingsWrite | SA |
| RoleGrants | RoleGrantsRead | SA, OA, UA, HELPDESK |
| RoleGrants | RoleGrantsWrite | SA, OA, UA |
| UserRoleGrants | UserRoleGrantsRead | SA, OA, UA, HELPDESK |
| UserRoleGrants | RoleGrantsWrite | SA, OA, UA |
| RoleManagementGroups | RoleManagementGroupsRead | SA |
| RoleManagementGroups | RoleManagementGroupsWrite | SA, OA |
| SupportedEntitlements | SupportedEntitlementsRead | SA, PA |
| Entitlements | EntitlementsRead | SA, PA, USER |
| Entitlements | EntitlementsWrite | SA, OA |
| HomeRealm | HomeRealmDiscovery | SA |
| FederatedUsers | FederatedUsersRead | SA |
| FederatedUsers | FederatedUsersWrite | SA |
| InternalMpsOrganizations | InternalMpsReadAccess | SA |

---

## Proposed Zanzibar/OpenFGA/SpiceDB Mapping

| **Resource** | **Proposed Relation & Permission Definition** |
|---------------|----------------------------------------------|
| organization_children |     definition organization_children {<br>  relation system_admin: user<br>  relation partner_admin: user<br>  relation helpdesk: user<br>  relation org_admin: user<br><br>  permission OrgReadChildren = system_admin + partner_admin + helpdesk<br>  permission OrgWriteChildren = system_admin + partner_admin<br>  permission OrgDeleteChildren = org_admin<br>     } |
| organization_descendants |     definition organization_descendants {<br>  relation system_admin: user<br>  relation partner_admin: user<br><br>  permission OrgReadDescendants = system_admin + partner_admin<br>     } |
| organizations |     definition organizations {<br>  relation system_admin: user<br>  relation org_admin: user<br>  relation user_admin: user<br>  relation partner_admin: user<br>  relation helpdesk: user<br>  relation user: user<br>  relation self: user<br><br>  permission OrgRead = system_admin + org_admin + user_admin + partner_admin + helpdesk + user<br>  permission OrgReadAndSelf = system_admin + org_admin + user_admin + partner_admin + helpdesk + user + self<br>  permission OrgWrite = system_admin + org_admin<br>  permission OrgCreate = system_admin + org_admin<br>  permission OrgDelete = system_admin + org_admin<br>  permission OrgReadForUserSearch = system_admin + org_admin<br>     } |
| organization_idp_configurations |     definition organization_idp_configurations {<br>  relation system_admin: user<br>  relation org_admin: user<br>  relation helpdesk: user<br><br>  permission IdpConfigRead = system_admin + org_admin + helpdesk<br>  permission IdpConfigWrite = system_admin + org_admin<br>     } |
| organization_role_grants |     definition organization_role_grants {<br>  relation system_admin: user<br>  relation org_admin: user<br>  relation user_admin: user<br>  relation helpdesk: user<br><br>  permission RoleGrantsOrgRead = system_admin + org_admin + user_admin + helpdesk<br>  permission RoleGrantsOrgWrite = system_admin + org_admin<br>     } |
| federation_profiles |     definition federation_profiles {<br>  relation system_admin: user<br>  relation org_admin: user<br>  relation helpdesk: user<br><br>  permission FederationProfilesRead = system_admin + org_admin + helpdesk<br>  permission FederationProfilesWrite = system_admin + org_admin<br>     } |
| b2c_policies |     definition b2c_policies {<br>  relation system_admin: user<br><br>  permission PolicyDeploymentsRead = system_admin<br>  permission PolicyDeploymentsWrite = system_admin<br>     } |
| roles |     definition roles {<br>  relation system_admin: user<br>  relation org_admin: user<br>  relation user_admin: user<br>  relation partner_admin: user<br>  relation helpdesk: user<br>  relation user: user<br><br>  permission RolesRead = system_admin + org_admin + user_admin + partner_admin + helpdesk + user<br>  permission RolesWrite = system_admin<br>     } |
| groups |     definition groups {<br>  relation system_admin: user<br>  relation org_admin: user<br>  relation user_admin: user<br>  relation helpdesk: user<br>  relation user: user<br><br>  permission GroupsRead = system_admin + org_admin + user_admin + helpdesk + user<br>  permission GroupsWrite = system_admin + org_admin + user_admin<br>  permission GroupsReadById = system_admin + org_admin + user_admin + helpdesk + user<br>  permission GroupsSearch = system_admin + org_admin + user_admin + helpdesk + user<br>  permission GroupsDelete = system_admin + org_admin + user_admin<br>     } |
| group_members |     definition group_members {<br>  relation system_admin: user<br>  relation org_admin: user<br>  relation user_admin: user<br>  relation helpdesk: user<br><br>  permission GroupMembersRead = system_admin + org_admin + user_admin + helpdesk<br>  permission GroupMembersWrite = system_admin + org_admin + user_admin<br>     } |
| group_role_grants |     definition group_role_grants {<br>  relation system_admin: user<br>  relation org_admin: user<br>  relation user_admin: user<br>  relation helpdesk: user<br><br>  permission GroupRoleGrantsRead = system_admin + org_admin + user_admin + helpdesk<br>  permission GroupRoleGrantsWrite = system_admin + org_admin + user_admin<br>     } |
| domains |     definition domains {<br>  relation system_admin: user<br>  relation org_admin: user<br>  relation helpdesk: user<br><br>  permission DomainsRead = system_admin + org_admin + helpdesk<br>  permission DomainsWrite = system_admin + org_admin<br>     } |
| offerings |     definition offerings {<br>  relation system_admin: user<br>  relation org_admin: user<br>  relation user_admin: user<br>  relation partner_admin: user<br>  relation helpdesk: user<br>  relation user: user<br><br>  permission OfferingsRead = system_admin + org_admin + user_admin + partner_admin + helpdesk + user<br>  permission OfferingsWrite = system_admin<br>     } |
| role_grants |     definition role_grants {<br>  relation system_admin: user<br>  relation org_admin: user<br>  relation user_admin: user<br>  relation helpdesk: user<br><br>  permission RoleGrantsRead = system_admin + org_admin + user_admin + helpdesk<br>  permission RoleGrantsWrite = system_admin + org_admin + user_admin<br>     } |
| user_role_grants |     definition user_role_grants {<br>  relation system_admin: user<br>  relation org_admin: user<br>  relation user_admin: user<br>  relation helpdesk: user<br><br>  permission UserRoleGrantsRead = system_admin + org_admin + user_admin + helpdesk<br>  permission RoleGrantsWrite = system_admin + org_admin + user_admin<br>     } |
| role_management_groups |     definition role_management_groups {<br>  relation system_admin: user<br>  relation org_admin: user<br><br>  permission RoleManagementGroupsRead = system_admin<br>  permission RoleManagementGroupsWrite = system_admin + org_admin<br>     } |
| supported_entitlements |     definition supported_entitlements {<br>  relation system_admin: user<br>  relation partner_admin: user<br><br>  permission SupportedEntitlementsRead = system_admin + partner_admin<br>     } |
| entitlements |     definition entitlements {<br>  relation system_admin: user<br>  relation org_admin: user<br>  relation partner_admin: user<br>  relation user: user<br><br>  permission EntitlementsRead = system_admin + partner_admin + user<br>  permission EntitlementsWrite = system_admin + org_admin<br>     } |
| home_realm |     definition home_realm {<br>  relation system_admin: user<br><br>  permission HomeRealmDiscovery = system_admin<br>     } |
| federated_users |     definition federated_users {<br>  relation system_admin: user<br><br>  permission FederatedUsersRead = system_admin<br>  permission FederatedUsersWrite = system_admin<br>     } |
| internal_mps_organizations |     definition internal_mps_organizations {<br>  relation system_admin: user<br><br>  permission InternalMpsReadAccess = system_admin<br>     } |

---

## Explanation and Rationale

### **1. Users Resource**

The **Users** resource governs identity and account lifecycle.  
- **System Admin (SA)**, **Org Admin (OA)**, and **User Admin (UA)** retain all creation, modification, deletion, and import powers.  
- **Helpdesk** can view (`UserRead`) user data for support purposes but cannot modify it.  
- Regular **User** accounts gain visibility only into their own user record via `UserReadExpanded` through the `self` relation.  

This separation preserves administrative control while introducing relational expressiveness for future expansions such as delegated or group-based user management.

---

### **2. Organizations Resource**

The **Organizations** resource handles higher-level organizational entities and structures.  
- **SA** and **OA** continue to own full mutation privileges — they can create, edit, or delete organizations.  
- **Partner Admin (PA)**, **Helpdesk**, and **User** roles retain various read capabilities as before.  
- `OrgReadAndSelf` uses the `self` relation to model scoped or personal organization-level access.

A new relation `OrgReadForUserSearch` accommodates a specific admin/query scenario, providing exact parity with the legacy permission intent.

---

### **3. Organization Children and Descendants**

Introduced to model hierarchical data relationships between parent and child organizations.  
- **SA**, **PA**, and **HELPDESK** can read organizational children for administrative or support insight.  
- Only **SA** and **PA** can perform modifications (`OrgWriteChildren`), maintaining oversight coherence between platform and partner levels.  
- **OA** holds the restricted deletion authority (`OrgDeleteChildren`), matching real-world ownership control within their org scope.  
- For broader hierarchy introspection, **SA** and **PA** can also read descendant organizations via `OrgReadDescendants`.

---

### **4. Organization Role Grants, IDP Configurations, and Federation Profiles**

These resources collectively manage **authorization policies** and **federated identity integrations**.  
- **IDP Configurations**: Read permission (`IdpConfigRead`) allows **Helpdesk** participation, while writes are limited to **SA** and **OA** due to sensitive nature.  
- **Role Grants**, **Organization Role Grants**, and **User Role Grants**: Allow unified management by **SA**, **OA**, and **UA** (write), with **Helpdesk** retaining readonly diagnostic visibility.
- **Federation Profiles**: Maintain connection metadata — readable by **Helpdesk** and writable only by administrative roles.

These mappings enforce clear separation between **observation** (for troubleshooting) and **modification** (for governance).

---

### **5. Groups, Group Members, and Group Role Grants**

Group-based membership and permission management follow tiered visibility control.  
- **Read**, **ReadById**, and **Search** permissions include **user-level** access, ensuring self-service and member visibility.  
- Write operations are reserved for administrative roles (SA, OA, UA).  
- Group member modifications mimic this pattern: **Helpdesk** maintains read access, while only admins can modify membership.

These granular relation permissions model complex collaboration controls with minimal duplication.

---

### **6. Domains, Offerings, and Partner Entities**

- **Domains**: Configuration and ownership reserved for **SA** and **OA**, with **Helpdesk** granted observational access.  
- **Offerings**: Broader read access extends to users, partners, and helpdesk for catalog visibility; only **SA** can modify offerings to protect master data consistency.  
- **Partner Admin (PA)** relations mirror organizational partnership control, preserving multi-tenant alignment.

---

### **7. Roles and Role Management Groups**

- **RolesRead**: Provided universally to SA, OA, UA, PA, HELPDESK, and USER for visibility across granted capabilities.  
- **RolesWrite**: Exclusively controlled by **System Admin**, guaranteeing central integrity of role definitions.  
- **Role Management Groups**: Introduces complementary schema for managing grouped roles. Read is SA-only; write includes SA and OA for collaborative role delegation.

These distinctions ensure stable, traceable permission lifecycle handling.

---

### **8. Entitlements and Supported Entitlements**

These represent internal and external service/feature allocations:  
- **SupportedEntitlementsRead** allows visibility to SA and PA only.  
- **EntitlementsRead** expands to **User** roles where appropriate (e.g., checking their entitlements).  
- **EntitlementsWrite** remains restricted to **SA** and **OA** for authorized system actions.

This pattern encapsulates business logic integrity: users can *view* entitlements, administrators can *edit* them.

---

### **9. B2C Policies, Federated Users, and Home Realm**

These components underpin **external identity federation and consumer policy control**.  
- **B2C Policies**: Deployment permissions locked to **SA** only, reflecting production release sensitivity.  
- **Federated Users**: Read/write both restricted to **SA**, ensuring federations stay authoritative and immune to drift.  
- **HomeRealmDiscovery**: Similarly, limited to **SA**, as it affects global authentication routing.

Such strict isolation enforces regulated control over top-tier authentication mechanisms.

---
