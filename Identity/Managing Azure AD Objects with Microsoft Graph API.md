# Tutorial: Managing Azure AD Objects with Microsoft Graph API

## Introduction

Azure Active Directory (Azure AD) is essential for identity and access management. However, manually handling the lifecycles of these objects can be challenging. This tutorial will guide you through using Microsoft Graph API to streamline and automate lifecycle management of Azure AD objects, ensuring consistency, security, and scalability.

## Table of Contents

1. [What Is Graph API?](#what-is-graph-api)
2. [Uses of Graph API](#uses-of-graph-api)
3. [Lifecycle Management of Azure AD Objects](#lifecycle-management-of-azure-ad-objects)
4. [Authenticating and Registering an Application with Azure AD](#authenticating-and-registering-an-application-with-azure-ad)
5. [Managing Service Principals, Groups, and Roles](#managing-service-principals-groups-and-roles)
6. [Other Lifecycle Management Tasks](#other-lifecycle-management-tasks)
7. [Tips for Using Graph APIs for Azure AD Cloud IAM](#tips-for-using-graph-apis-for-azure-ad-cloud-iam)
8. [Conclusion](#conclusion)

## What Is Graph API?

From a mathematical perspective, a graph consists of nodes and their connections. Azure Active Directory has a graph-like model structure where entities (groups, users, applications) represent nodes, and relationships (application assignments, group membership) represent edges. The Graph API provides a RESTful interface to navigate and manage this content.

## Uses of Graph API

Microsoft Graph API simplifies developer interaction with Microsoft’s resources and services. Its uses include:

- **Identity and Access Management**: Automate user provisioning, role-based access control, and conditional access policies.
- **Centralized Data Access and Management**: Easily access and manipulate data across the Microsoft ecosystem.
- **Handling Event Logistics**: Administer events like sending meeting invites or scheduling meetings using Outlook Calendar.
- **Managing User Insights**: Analyze user activity and engagement trends or integrate with Power BI for visualization.
- **Automating Workflow**: Streamline repetitive tasks such as creating and deleting accounts, tracking task progress, and processing emails.

## Lifecycle Management of Azure AD Objects

### What Are Azure AD Objects?

Azure AD objects are any entities stored and managed in an active directory tenant. These include:
- **User Accounts**: Individual users with unique IDs and passwords.
- **Group Accounts**: Users grouped based on roles and privileges.
- **Devices**: Computers, phones, or any device connected to your Azure AD.
- **Applications**: Software applications integrated with your Azure AD.

### Benefits of Lifecycle Management

- **Automated Provisioning and De-Provisioning**: Automatically create and delete user accounts, groups, and roles.
- **Password Reset**: Allow users to reset their passwords without administrative guidance.
- **Role-Based Access and Control**: Assign roles and privileges to users.
- **Conditional Access**: Allow users to access certain resources based on specific conditions.

## Authenticating and Registering an Application with Azure AD

Accessing the Graph API requires your application to be authenticated using OAuth 2.0. Follow these steps:

1. **Register the Application**:
    - Go to Azure Portal.
    - Navigate through Azure Active Directory -> App Registrations -> New Registration.
    - Enter the name of your application and select a supported account type.
    - Copy the Application ID and Directory ID.
    - Create and copy the new client secret under Certificates and Secrets.

2. **Assign API Permissions**:
    - Go to API Permissions.
    - Select Add a Permission -> Microsoft Graph.
    - Select the necessary permissions (e.g., `User.ReadWrite.All`) and grant admin consent.

3. **Authenticate Application With OAuth 2.0**:
    - Authenticate your application with OAuth 2.0 by requesting the OAuth 2.0 token endpoint.

4. **Generate the Graph API Access Token**:
    ```python
    import requests

    tenant_id = "your-tenant-id"
    client_id = "your-client-id"
    client_secret = "your-client-secret"
    scope = "https://graph.microsoft.com/.default"

    url = f"https://login.microsoftonline.com/{tenant_id}/oauth2/v2.0/token"

    payload = {
        "client_id": client_id,
        "scope": scope,
        "client_secret": client_secret,
        "grant_type": "client_credentials"
    }

    response = requests.post(url, data=payload)

    if response.status_code == 200:
        access_token = response.json().get("access_token")
        print("Access Token:", access_token)
    else:
        print("Failed to get access token:", response.json())
    ```

## Managing Service Principals, Groups, and Roles

### Service Principals Management

#### Listing Service Principals
```python
url = "https://graph.microsoft.com/v1.0/servicePrincipals"
response = requests.get(url, headers=headers)
print(response.json())
```
#### Creating a Service Principal
```python
import requests

access_token = "[your_access_token]"

url = "https://graph.microsoft.com/v1.0/servicePrincipals"

payload = {
    "displayName": "My Service Principal",
    "appId": "<application-id-of-the-app>"
}

headers = {
    "Authorization": f"Bearer {access_token}",
    "Content-Type": "application/json"
}

response = requests.post(url, json=payload, headers=headers)

if response.status_code == 201:
    print("Service Principal Created:", response.json())
else:
    print("Error:", response.json())
```

### Groups Management

#### Creating a Group
```python
url = "https://graph.microsoft.com/v1.0/groups"

payload = {
    "displayName": "Developers Team",
    "mailEnabled": False,
    "mailNickname": "devteam",
    "securityEnabled": True
}

response = requests.post(url, json=payload, headers=headers)

if response.status_code == 201:
    print("Group Created:", response.json())
else:
    print("Error:", response.json())
```
#### Adding Members to the Group
```python
group_id = "[group-id]"
user_id = "[user-id]"

url = f"https://graph.microsoft.com/v1.0/groups/{group_id}/members/$ref"

payload = {
    "@odata.id": f"https://graph.microsoft.com/v1.0/users/{user_id}"
}

response = requests.post(url, json=payload, headers=headers)

if response.status_code == 204:
    print("Member Added to Group")
else:
    print("Error:", response.json())
```
#### Deleting a Group
```python
group_id = "[group-id]"
url = f"https://graph.microsoft.com/v1.0/groups/{group_id}"

response = requests.delete(url, headers=headers)

if response.status_code == 204:
    print("Group Deleted")
else:
    print("Error:", response.json())
```
#### Removing a Member
```python
group_id = "[group-id]"
member_id = "[user-or-service-principal-id]"

url = f"https://graph.microsoft.com/v1.0/groups/{group_id}/members/{member_id}/$ref"

response = requests.delete(url, headers=headers)

if response.status_code == 204:
    print("Member Removed")
else:
    print("Error:", response.json())
```
### Roles Management

#### Assigning Roles
```python
role_id = "[role-id]"
principal_id = "[service-principal-id]"

url = "https://graph.microsoft.com/v1.0/directoryRoleAssignments"

payload = {
    "principalId": principal_id,
    "roleDefinitionId": f"{role_id}",
    "directoryScopeId": "/"
}

response = requests.post(url, json=payload, headers=headers)

if response.status_code == 201:
    print("Role Assigned:", response.json())
else:
    print("Error:", response.json())
```
#### Listing Roles
```python
url = "https://graph.microsoft.com/v1.0/directoryRoles"
response = requests.get(url, headers=headers)

if response.status_code == 200:
    print("Roles:", response.json())
else:
    print("Error:", response.json())
```

### Other Lifecycle Management Tasks

- Provisioning New Users : Onboard new users on Azure AD.
    - Endpoint: `POST https://graph.microsoft.com/v1.0/users`
- Updating User Information : Change user roles, departments, or other information.
    - Endpoint: `PATCH https://graph.microsoft.com/v1.0/users/{id}`
- Deactivating or Soft Deleting a User : Temporarily delete or deactivate user accounts.
    - Endpoint: `PATCH https://graph.microsoft.com/v1.0/users/{id}`
- Permanently Deleting a User : Hard delete the users.
    - Endpoint: `DELETE https://graph.microsoft.com/v1.0/directory/deletedItems/{id}`

### Tips for Using Graph APIs for Azure AD Cloud IAM
- Use certified app-only authentication or client secret for automation instead of interactive login.
- Only assign the necessary Graph API access permissions.
- Keep track of changes with Delta Queries (/delta).
- Handle API errors gracefully by checking response.status_code and response.json().
- Review and audit all Graph API changes by logging them.

### Conclusion
Graph API plays a crucial role in automating the lifecycle management of Azure AD objects. It minimizes errors, reduces administrative oversight, enhances consistency and performance, and strengthens security. As organizations move towards hybrid and multi-cloud environments, leveraging Graph API offers scalability, flexibility, and futuristic identity governance.
