## SCIM and Non-SCIM Token Handling with Rego  

This Rego policy ensures that token validation errors are handled differently for SCIM and non-SCIM endpoints. For SCIM endpoints, it returns a SCIM-compliant error response with a `401 Unauthorized` status code. For non-SCIM endpoints, it returns a generic `401 Unauthorized` response.

---

### **Rego Policy Logic**

The following Rego policy evaluates incoming requests and determines whether the request is for a SCIM endpoint or a non-SCIM endpoint. Based on the endpoint type, it constructs the appropriate error response for token validation failures.

#### `policy.rego`  

```rego  
package authz

# Default decision: Deny access with a generic 401 response
default allow = false
default response = {
    "status": 401,
    "body": {
        "message": "Unauthorized"
    }
}

# Helper function to check if the request path matches SCIM endpoints
is_scim_endpoint(path) {
    startswith(path, "/scim/v2/")
}

# Handle token validation failure for SCIM endpoints
response = scim_response {
    input.path := path
    is_scim_endpoint(path)
    not input.valid_token
    scim_response := {
        "status": 401,
        "body": {
            "schemas": ["urn:ietf:params:scim:api:messages:2.0:Error"],
            "detail": "Authentication failed.",
            "status": "401"
        }
    }
}


# Handle token validation failure for non-SCIM endpoints
response = generic_response {
    input.path := path
    not is_scim_endpoint(path)
    not input.valid_token
    generic_response := {
        "status": 401,
        "body": {
            "message": "Unauthorized"
        }
    }
}

# Allow access if the token is valid
allow = true {
    input.valid_token
}
```  
### Explanation of the Policy

1. **Default Behavior :**  
    -   By default, the policy denies access (allow = false) and returns a generic 401 Unauthorized response.
    -   This ensures that unauthorized requests are handled gracefully.  

2. **SCIM Endpoint Detection :**  
    -   The `is_scim_endpoint` function checks if the request path starts with `/scim/v2/`, identifying SCIM-specific endpoints.

3. **SCIM Error Response :**
    -   If the request is for a SCIM endpoint and the token is invalid (`not input.valid_token`), the policy constructs a SCIM-compliant error response with:  
        - `status`: `401`
        - `body`: A JSON object conforming to the SCIM error schema.  

4. **Generic Error Response :**
    -   For non-SCIM endpoints with an invalid token, the policy returns a generic `401 Unauthorized` response with a simple message.  

5. **Token Validation :**  
    - If the token is valid (`input.valid_token`), the policy allows access (`allow = true`).  

---
### Input Example  

Here is an example of the input structure expected by the Rego policy:
```json
{
    "path": "/scim/v2/Users",
    "valid_token": false
}
```

### Output Example

**SCIM Endpoint (Invalid Token)**  

**Request Path:** `/scim/v2/Users`  
**Response:**  
```json
{
    "status": 401,
    "body": {
        "schemas": ["urn:ietf:params:scim:api:messages:2.0:Error"],
        "detail": "Authentication failed.",
        "status": "401"
    }
}
```  

**Non-SCIM Endpoint (Invalid Token)**  

**Request Path:** `/api/otherApi`  
**Response:**  
```json
{
    "status": 401,
    "body": {
        "message": "Unauthorized"
    }
}
```  
---
### Key Points  

- **Endpoint Differentiation :** The policy distinguishes between SCIM and non-SCIM endpoints using the request path.
- **SCIM Compliance :** SCIM endpoints receive error responses adhering to the SCIM 2.0 specification.
- **Generic Handling :** Non-SCIM endpoints receive a standard `401 Unauthorized` response.
- **Token Validation :** The policy assumes the presence of a `valid_token` flag in the input to determine token validity.
---

### Conclusion  
This Rego policy ensures that token validation errors are handled appropriately for both SCIM and non-SCIM endpoints. It provides SCIM-compliant error responses for SCIM endpoints while maintaining simplicity for non-SCIM endpoints. This approach aligns with best practices for API security and compliance with SCIM standards.