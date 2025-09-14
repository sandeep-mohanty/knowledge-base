# Tutorial: Managing Azure AD B2C Objects with Microsoft Graph API

## Introduction

Azure Active Directory B2C (Azure AD B2C) is a customer identity access management (CIAM) solution that allows businesses to manage external identities. Like Azure AD, managing the lifecycle of objects such as users, groups, and applications in Azure AD B2C can be challenging when done manually. This tutorial will guide you through using Microsoft Graph API to streamline and automate the lifecycle management of Azure AD B2C objects, ensuring consistency, security, and scalability.

## Table of Contents

1. [What Is Azure AD B2C?](#what-is-azure-ad-b2c)
2. [Graph API for Azure AD B2C Object Management](#graph-api-for-azure-ad-b2c-object-management)
3. [Uses of Graph API in Azure AD B2C](#uses-of-graph-api-in-azure-ad-b2c)
4. [Lifecycle Management of Azure AD B2C Objects](#lifecycle-management-of-azure-ad-b2c-objects)
5. [Authenticating and Registering an Application with Azure AD B2C](#authenticating-and-registering-an-application-with-azure-ad-b2c)
6. [Managing Users, Groups, and Roles in Azure AD B2C](#managing-users-groups-and-roles-in-azure-ad-b2c)
7. [Other Lifecycle Management Tasks](#other-lifecycle-management-tasks)
8. [Tips for Using Graph APIs for Azure AD B2C Cloud IAM](#tips-for-using-graph-apis-for-azure-ad-b2c-cloud-iam)
9. [Conclusion](#conclusion)

## What Is Azure AD B2C?

Azure AD B2C (Business-to-Customer) is a cloud-based identity and access management service designed specifically for customer-facing applications. It allows organizations to provide secure authentication and authorization for their customers across multiple platforms, including web, mobile, and desktop apps. Azure AD B2C supports social identity providers like Google, Facebook, and Microsoft, in addition to traditional username/password authentication.

## Graph API for Azure AD B2C Object Management

From a mathematical perspective, a graph comprises nodes and their connections. In Azure AD B2C, entities like users, groups, and applications represent the nodes, while relationships like group membership or application assignments form the edges. The Graph API provides a RESTful interface to navigate and manage this content.

It simplifies CRUD (Create, Read, Update, Delete) operations across various Microsoft services—such as Teams, Office 365, OneDrive, and more—and allows synchronization between Azure AD B2C and other data stores via differential queries.

## Uses of Graph API in Azure AD B2C

Microsoft Graph API offers several uses for managing Azure AD B2C:

- **Identity and Access Management**: Automate user provisioning, role-based access control, and conditional access policies.
- **Centralized Data Access and Management**: Easily manipulate data across the Microsoft ecosystem with a unified endpoint.
- **Customer Insights**: Analyze customer activity and engagement trends, and integrate with Power BI for visualization.
- **Automating Workflow**: Streamline repetitive tasks like creating and deleting accounts, tracking task progress, processing emails, etc.

## Lifecycle Management of Azure AD B2C Objects

### What Are Azure AD B2C Objects?

An Azure AD B2C object is any entity stored and managed within your Azure AD B2C tenant. These include:
- **User Accounts**: Individual users with unique IDs and passwords.
- **Groups**: Users grouped based on roles and privileges.
- **Applications**: Software applications integrated with your Azure AD B2C tenant.

### Benefits of Lifecycle Management

- **Automated Provisioning and De-Provisioning**: Automatically create and delete user accounts, groups, and roles.
- **Password Reset**: Allow users to reset their passwords without administrative guidance.
- **Role-Based Access and Control**: Assign roles and privileges to users.
- **Conditional Access**: Allow users to access resources based on specific conditions.

## Authenticating and Registering an Application with Azure AD B2C

Accessing the Graph API requires your application to be authenticated using OAuth 2.0. Follow these steps:

1. **Register the Application**:
    - Go to Azure Portal.
    - Navigate through Azure AD B2C -> App Registrations -> New Registration.
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

## Managing Users, Groups, and Roles in Azure AD B2C

### User Management

#### Listing Users
```python
url = "https://graph.microsoft.com/v1.0/users"
response = requests.get(url, headers=headers)
print(response.json())
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

### Tips for Using Graph APIs for Azure AD B2C Cloud IAM
- Use certified app-only authentication or client secret for automation instead of interactive login.
- Only assign the necessary Graph API access permissions.
- Keep track of changes with Delta Queries (/delta).
- Handle API errors gracefully by checking response.status_code and response.json().
- Review and audit all Graph API changes by logging them.

### Conclusion
Graph API plays a crucial role in automating the lifecycle management of Azure AD B2C objects. It minimizes errors, reduces administrative oversight, enhances consistency and performance, and strengthens security. As organizations move towards hybrid and multi-cloud environments, leveraging Graph API offers scalability, flexibility, and futuristic identity governance.