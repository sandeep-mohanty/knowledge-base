# Mock API Tutorial: Complete Guide with Examples

## Table of Contents

1. [Introduction](#introduction)
2. [Installation](#installation)
3. [Getting Started](#getting-started)
4. [Route Parameters](#route-parameters)
5. [Response Handlers](#response-handlers)
6. [Environment Variables](#environment-variables)
7. [Proxy Mode - Extending Existing APIs](#proxy-mode)
8. [Stateful APIs](#stateful-apis)
9. [Multi-Language Response Handlers](#multi-language-response-handlers)
10. [Advanced Features](#advanced-features)
11. [Real-World Use Cases](#real-world-use-cases)

***

## Introduction

**Mock** is a language-agnostic API mocking and testing utility that provides a command-line-first approach to creating mock APIs. Unlike traditional mocking frameworks that require extensive code, Mock allows you to define API routes through command-line parameters or configuration files, making it incredibly fast to prototype and test.[1][2]

### Key Features

- **Command-line based**: No need to write code to create simple mocks[2]
- **Flexible response handlers**: Use shell scripts, Python, Node.js, or any executable program[1]
- **API proxy mode**: Extend existing APIs with new routes[2]
- **Testing assertions**: Verify endpoint requests during testing[1]
- **Stateful mocking**: Create APIs that maintain state across requests[3]

### When to Use Mock

Mock excels in scenarios where you need to:

- Test frontend applications before backend APIs are ready[2]
- Simulate slow or failing endpoints for resilience testing[3]
- Add experimental routes to existing APIs without modifying them[2]
- Create disposable test environments quickly[1]
- Mock third-party APIs for integration testing[2]

***

## Installation

Mock is distributed as a single-file executable, making installation straightforward.[1]

### Linux Installation

```bash
# Download the latest release for Linux
wget https://github.com/dhuan/mock/releases/latest/download/mock-linux-amd64.tar.gz

# Extract the tarball
tar -xzf mock-linux-amd64.tar.gz

# Move to a directory in your PATH
sudo mv mock /usr/local/bin/

# Verify installation
mock --version
```

### macOS Installation

```bash
# Download the latest release for macOS
wget https://github.com/dhuan/mock/releases/latest/download/mock-darwin-amd64.tar.gz

# Extract and install
tar -xzf mock-darwin-amd64.tar.gz
sudo mv mock /usr/local/bin/

# Verify installation
mock --version
```

### Docker Installation (Optional)

If you prefer containerized environments:

```bash
# Create a Dockerfile
cat > Dockerfile <<EOF
FROM alpine:latest
RUN apk add --no-cache wget tar
RUN wget https://github.com/dhuan/mock/releases/latest/download/mock-linux-amd64.tar.gz && \
    tar -xzf mock-linux-amd64.tar.gz && \
    mv mock /usr/local/bin/ && \
    rm mock-linux-amd64.tar.gz
ENTRYPOINT ["mock"]
EOF

# Build the image
docker build -t mock-api .

# Run mock in a container
docker run -p 3000:3000 mock-api serve --port 3000 --route hello --response "Hello, World!"
```

***

## Getting Started

Let's start with the simplest possible example and build from there.

### Your First Mock API

Create a basic API with a single endpoint that returns a static message:

```bash
# Start a mock server on port 3000 with one route
mock serve --port 3000 \
  --route "hello" \
  --method GET \
  --response "Hello, World!"
```

**Explanation:**
- `mock serve`: Starts the mock server[2]
- `--port 3000`: Listens on port 3000[2]
- `--route "hello"`: Defines the endpoint path[2]
- `--method GET`: Specifies HTTP method (GET is default)[2]
- `--response`: Sets the response body[2]

**Test it:**

```bash
# Using curl
curl http://localhost:3000/hello

# Expected output:
# Hello, World!

# Using wget
wget -qO- http://localhost:3000/hello

# Or open in your browser
# http://localhost:3000/hello
```

### Multiple Routes Example

Create an API with multiple endpoints:

```bash
mock serve --port 3000 \
  --route "users" \
  --method GET \
  --response '{"users": ["Alice", "Bob", "Charlie"]}' \
  --route "status" \
  --method GET \
  --response '{"status": "OK", "timestamp": "2025-11-02T22:14:00Z"}' \
  --route "health" \
  --method GET \
  --response "healthy"
```

**Test the routes:**

```bash
# Get users
curl http://localhost:3000/users
# Output: {"users": ["Alice", "Bob", "Charlie"]}

# Check status
curl http://localhost:3000/status
# Output: {"status": "OK", "timestamp": "2025-11-02T22:14:00Z"}

# Health check
curl http://localhost:3000/health
# Output: healthy
```

***

## Route Parameters

Route parameters allow you to create dynamic endpoints that respond differently based on URL segments.

### Basic Route Parameters

Use curly braces `{}` to define parameters in your routes:[2]

```bash
mock serve --port 3000 \
  --route "greet/{name}" \
  --method GET \
  --response "Hello, ${name}! Welcome to our API."
```

**Explanation:**
- `{name}`: Defines a route parameter called "name"[2]
- `${name}`: References the parameter value in the response[2]

**Test it:**

```bash
curl http://localhost:3000/greet/Alice
# Output: Hello, Alice! Welcome to our API.

curl http://localhost:3000/greet/Bob
# Output: Hello, Bob! Welcome to our API.

curl http://localhost:3000/greet/john_doe
# Output: Hello, john_doe! Welcome to our API.
```

### Multiple Route Parameters

Create routes with multiple parameters:

```bash
mock serve --port 3000 \
  --route "users/{userId}/posts/{postId}" \
  --method GET \
  --response '{"userId": "${userId}", "postId": "${postId}", "title": "Post ${postId} by User ${userId}"}'
```

**Test it:**

```bash
curl http://localhost:3000/users/123/posts/456
# Output: {"userId": "123", "postId": "456", "title": "Post 456 by User 123"}

curl http://localhost:3000/users/alice/posts/hello-world
# Output: {"userId": "alice", "postId": "hello-world", "title": "Post hello-world by User alice"}
```

### RESTful API Example

Create a complete RESTful user API structure:

```bash
mock serve --port 3000 \
  --route "api/v1/users" \
  --method GET \
  --response '{"users": [{"id": 1, "name": "Alice"}, {"id": 2, "name": "Bob"}]}' \
  --route "api/v1/users/{id}" \
  --method GET \
  --response '{"id": "${id}", "name": "User ${id}", "email": "user${id}@example.com"}' \
  --route "api/v1/users" \
  --method POST \
  --response '{"message": "User created", "status": "success"}' \
  --route "api/v1/users/{id}" \
  --method DELETE \
  --response '{"message": "User ${id} deleted", "status": "success"}'
```

**Test the RESTful API:**

```bash
# List all users
curl http://localhost:3000/api/v1/users

# Get specific user
curl http://localhost:3000/api/v1/users/42

# Create user (POST)
curl -X POST http://localhost:3000/api/v1/users

# Delete user
curl -X DELETE http://localhost:3000/api/v1/users/42
```

***

## Response Handlers

Response handlers are where Mock really shines. Instead of static responses, you can use executable programs to generate dynamic responses.[1][2]

### Using Shell Scripts with --exec

The `--exec` flag allows you to run shell commands to generate responses:[2]

```bash
mock serve --port 3000 \
  --route "time" \
  --method GET \
  --exec 'printf "Current time: %s" $(date +"%H:%M:%S") | mock write'
```

**Explanation:**
- `--exec`: Executes a shell command[2]
- `mock write`: Pipes the output as the HTTP response[2]

**Test it:**

```bash
curl http://localhost:3000/time
# Output: Current time: 22:14:35 (actual current time)

# Wait a few seconds and try again
curl http://localhost:3000/time
# Output: Current time: 22:14:42 (updated time)
```

### Dynamic JSON Responses

Generate JSON with shell scripts:

```bash
mock serve --port 3000 \
  --route "server-info" \
  --method GET \
  --exec 'printf "{\"hostname\": \"%s\", \"uptime\": \"%s\", \"load\": \"%s\"}" \
    "$(hostname)" \
    "$(uptime -p)" \
    "$(uptime | awk -F"load average:" "{print \$2}")" | mock write'
```

**Test it:**

```bash
curl http://localhost:3000/server-info
# Output: {"hostname": "my-machine", "uptime": "up 2 days, 5 hours", "load": " 0.52, 0.48, 0.51"}
```

### Python Response Handler

Use Python for complex logic:

```bash
mock serve --port 3000 \
  --route "calculate/{operation}/{a}/{b}" \
  --method GET \
  --exec '
python3 <<EOF | mock write
import sys
import json

# Access route parameters via environment variables
operation = "${operation}"
a = float("${a}")
b = float("${b}")

result = 0
if operation == "add":
    result = a + b
elif operation == "subtract":
    result = a - b
elif operation == "multiply":
    result = a * b
elif operation == "divide":
    result = a / b if b != 0 else "Error: Division by zero"

response = {
    "operation": operation,
    "operands": [a, b],
    "result": result
}

print(json.dumps(response))
EOF
'
```

**Test it:**

```bash
curl http://localhost:3000/calculate/add/10/5
# Output: {"operation": "add", "operands": [10.0, 5.0], "result": 15.0}

curl http://localhost:3000/calculate/multiply/7/6
# Output: {"operation": "multiply", "operands": [7.0, 6.0], "result": 42.0}
```

### Node.js Response Handler

Use Node.js for JavaScript-based responses:

```bash
mock serve --port 3000 \
  --route "uuid" \
  --method GET \
  --exec '
node <<EOF | mock write
const crypto = require("crypto");
const uuid = crypto.randomUUID();
console.log(JSON.stringify({
    uuid: uuid,
    timestamp: new Date().toISOString()
}));
EOF
'
```

**Test it:**

```bash
curl http://localhost:3000/uuid
# Output: {"uuid": "a1b2c3d4-e5f6-7890-abcd-ef1234567890", "timestamp": "2025-11-02T22:14:35.123Z"}
```

### Reading External Files

Create responses from files:

```bash
# First, create a sample data file
cat > users.json <<EOF
{
  "users": [
    {"id": 1, "name": "Alice Johnson", "role": "admin"},
    {"id": 2, "name": "Bob Smith", "role": "user"},
    {"id": 3, "name": "Charlie Brown", "role": "user"}
  ]
}
EOF

# Now serve it
mock serve --port 3000 \
  --route "users" \
  --method GET \
  --exec 'cat users.json | mock write'
```

**Test it:**

```bash
curl http://localhost:3000/users
# Output: Full contents of users.json
```

***

## Environment Variables

Mock provides access to request information through environment variables.[3]

### Available Environment Variables

Mock sets several environment variables that your response handlers can access:

- `MOCK_REQUEST_ENDPOINT`: The requested endpoint path[3]
- `MOCK_REQUEST_METHOD`: HTTP method (GET, POST, etc.)
- Route parameters (e.g., `{userId}` becomes available as `${userId}`)

### Using Environment Variables in Responses

```bash
mock serve --port 3000 \
  --route "debug" \
  --method GET \
  --exec '
printf "{\n  \"endpoint\": \"%s\",\n  \"method\": \"%s\",\n  \"timestamp\": \"%s\"\n}" \
  "${MOCK_REQUEST_ENDPOINT}" \
  "${MOCK_REQUEST_METHOD}" \
  "$(date -Iseconds)" | mock write
'
```

**Test it:**

```bash
curl http://localhost:3000/debug
# Output:
# {
#   "endpoint": "debug",
#   "method": "GET",
#   "timestamp": "2025-11-02T22:14:35+05:30"
# }
```

### Conditional Responses Based on Environment

```bash
mock serve --port 3000 \
  --route "conditional/{type}" \
  --method GET \
  --exec '
if [ "${type}" = "json" ]; then
  printf "{\"format\": \"json\", \"message\": \"JSON response\"}" | mock write
elif [ "${type}" = "xml" ]; then
  printf "<?xml version=\"1.0\"?><response><format>xml</format><message>XML response</message></response>" | mock write
else
  printf "Unknown format: ${type}" | mock write
fi
'
```

**Test it:**

```bash
curl http://localhost:3000/conditional/json
# Output: {"format": "json", "message": "JSON response"}

curl http://localhost:3000/conditional/xml
# Output: <?xml version="1.0"?><response><format>xml</format>...

curl http://localhost:3000/conditional/text
# Output: Unknown format: text
```

***

## Proxy Mode - Extending Existing APIs

One of Mock's most powerful features is its ability to act as a proxy to existing APIs while adding new routes.[1][2]

### Basic Proxy Setup

Extend an existing API by proxying all requests except your custom routes:

```bash
# Proxy to example.com and add a new route
mock serve --port 3000 \
  --base example.com \
  --route "custom/endpoint" \
  --method GET \
  --response '{"message": "This is a custom endpoint added to example.com"}'
```

**How it works:**
1. Requests to `/custom/endpoint` are handled by Mock[2]
2. All other requests are proxied to `example.com`[2]
3. The client sees a unified API[2]

**Test it:**

```bash
# Your custom route
curl http://localhost:3000/custom/endpoint
# Output: {"message": "This is a custom endpoint added to example.com"}

# Proxied to example.com
curl http://localhost:3000
# Output: (HTML from example.com)
```

### Practical Example: Adding Analytics to Existing API

```bash
# Extend your production API with a mock analytics endpoint
mock serve --port 8080 \
  --base api.yourapp.com \
  --route "analytics/events" \
  --method POST \
  --exec '
python3 <<EOF | mock write
import json
import time

event_data = {
    "event_recorded": True,
    "timestamp": time.time(),
    "message": "Analytics event received (mocked)"
}

print(json.dumps(event_data))
EOF
'
```

### Delaying Specific Endpoints

Use middleware to add delays to specific routes:[3]

```bash
mock serve --port 8000 \
  --base example.com \
  --middleware '
if [ "${MOCK_REQUEST_ENDPOINT}" = "slow/endpoint" ]; then
  sleep 3  # Wait 3 seconds for this specific endpoint
fi
'
```

**Explanation:**
- `--middleware`: Runs before each request[3]
- Check `MOCK_REQUEST_ENDPOINT` to identify the route[3]
- Use `sleep` to introduce delay[3]

**Use case:** Test how your application handles slow API responses without affecting the actual backend.

***

## Stateful APIs

Create APIs that maintain state across requests using temporary files.[3]

### Request Counter Example

```bash
# Create a temporary file to store state
export TMP=$(mktemp)
printf "0" > "${TMP}"

# Start server with stateful endpoint
mock serve --port 3000 \
  --route "counter" \
  --method GET \
  --exec '
# Read current count, increment, and save
current=$(cat '"${TMP}"')
next=$((current + 1))
echo "${next}" > '"${TMP}"'

printf "Request #%d received" "${next}" | mock write
'
```

**Test it:**

```bash
curl http://localhost:3000/counter
# Output: Request #1 received

curl http://localhost:3000/counter
# Output: Request #2 received

curl http://localhost:3000/counter
# Output: Request #3 received
```

### Session Management Example

Create a simple session-based API:

```bash
# Create session storage directory
SESSION_DIR=$(mktemp -d)

mock serve --port 3000 \
  --route "session/{userId}" \
  --method POST \
  --exec '
SESSION_FILE="'"${SESSION_DIR}"'/session_${userId}.json"

# Create or update session
printf "{\"userId\": \"${userId}\", \"lastAccess\": \"$(date -Iseconds)\", \"visits\": 1}" > "${SESSION_FILE}"

cat "${SESSION_FILE}" | mock write
' \
  --route "session/{userId}" \
  --method GET \
  --exec '
SESSION_FILE="'"${SESSION_DIR}"'/session_${userId}.json"

if [ -f "${SESSION_FILE}" ]; then
  # Update visit count
  visits=$(jq -r ".visits" "${SESSION_FILE}")
  new_visits=$((visits + 1))
  jq ".visits = ${new_visits} | .lastAccess = \"$(date -Iseconds)\"" "${SESSION_FILE}" > "${SESSION_FILE}.tmp"
  mv "${SESSION_FILE}.tmp" "${SESSION_FILE}"
  
  cat "${SESSION_FILE}" | mock write
else
  printf "{\"error\": \"Session not found for user ${userId}\"}" | mock write
fi
'
```

**Test it:**

```bash
# Create session for user alice
curl -X POST http://localhost:3000/session/alice
# Output: {"userId": "alice", "lastAccess": "2025-11-02T22:14:35+05:30", "visits": 1}

# Get session (increments visit count)
curl http://localhost:3000/session/alice
# Output: {"userId": "alice", "lastAccess": "2025-11-02T22:14:40+05:30", "visits": 2}

# Get again
curl http://localhost:3000/session/alice
# Output: {"userId": "alice", "lastAccess": "2025-11-02T22:14:45+05:30", "visits": 3}
```

***

## Multi-Language Response Handlers

Mock's language-agnostic design allows you to use different programming languages for different endpoints.[3]

### Complete Multi-Language API

```bash
mock serve --port 3000 \
  --route "js" \
  --method GET \
  --exec '
node <<EOF | mock write
console.log("Hello from Node.js! Server time: " + new Date().toISOString());
EOF
' \
  --route "python" \
  --method GET \
  --exec '
python3 <<EOF | mock write
import sys
import json
from datetime import datetime

response = {
    "language": "Python",
    "version": f"{sys.version_info.major}.{sys.version_info.minor}",
    "timestamp": datetime.now().isoformat()
}
print(json.dumps(response))
EOF
' \
  --route "php" \
  --method GET \
  --exec '
php <<EOF | mock write
<?php
echo json_encode([
    "language" => "PHP",
    "version" => phpversion(),
    "timestamp" => date("c")
]);
?>
EOF
' \
  --route "ruby" \
  --method GET \
  --exec '
ruby <<EOF | mock write
require "json"
puts JSON.generate({
  language: "Ruby",
  version: RUBY_VERSION,
  timestamp: Time.now.iso8601
})
EOF
'
```

**Test all endpoints:**

```bash
# Node.js endpoint
curl http://localhost:3000/js
# Output: Hello from Node.js! Server time: 2025-11-02T22:14:35.123Z

# Python endpoint
curl http://localhost:3000/python
# Output: {"language": "Python", "version": "3.11", "timestamp": "2025-11-02T22:14:35.123456"}

# PHP endpoint
curl http://localhost:3000/php
# Output: {"language":"PHP","version":"8.2.0","timestamp":"2025-11-02T22:14:35+05:30"}

# Ruby endpoint
curl http://localhost:3000/ruby
# Output: {"language":"Ruby","version":"3.2.0","timestamp":"2025-11-02T22:14:35+05:30"}
```

***

## Advanced Features

### HTTP Status Codes and Headers

Control response status codes and headers:

```bash
mock serve --port 3000 \
  --route "created" \
  --method POST \
  --exec '
printf "HTTP/1.1 201 Created\r\n"
printf "Content-Type: application/json\r\n"
printf "Location: /users/123\r\n"
printf "\r\n"
printf "{\"id\": 123, \"message\": \"Resource created\"}"
' | mock write
```

### Simulating API Failures

Create endpoints that simulate various failure scenarios:

```bash
mock serve --port 3000 \
  --route "unreliable" \
  --method GET \
  --exec '
# Randomly fail 30% of the time
if [ $((RANDOM % 10)) -lt 3 ]; then
  printf "HTTP/1.1 500 Internal Server Error\r\n\r\n"
  printf "{\"error\": \"Service temporarily unavailable\"}" | mock write
else
  printf "{\"status\": \"success\", \"data\": \"Request processed\"}" | mock write
fi
'
```

### Rate Limiting Simulation

```bash
# Track request counts in a file
RATE_LIMIT_DIR=$(mktemp -d)

mock serve --port 3000 \
  --route "limited/{apiKey}" \
  --method GET \
  --exec '
LIMIT_FILE="'"${RATE_LIMIT_DIR}"'/limit_${apiKey}"

# Initialize or read counter
if [ -f "${LIMIT_FILE}" ]; then
  count=$(cat "${LIMIT_FILE}")
else
  count=0
fi

count=$((count + 1))
echo "${count}" > "${LIMIT_FILE}"

# Allow 10 requests per "session"
if [ ${count} -gt 10 ]; then
  printf "{\"error\": \"Rate limit exceeded\", \"limit\": 10, \"count\": ${count}}" | mock write
else
  printf "{\"success\": true, \"requests_remaining\": $((10 - count))}" | mock write
fi
'
```

### Complex JSON Responses

Generate complex nested JSON structures:

```bash
mock serve --port 3000 \
  --route "api/products/{category}" \
  --method GET \
  --exec '
python3 <<EOF | mock write
import json
import random

category = "${category}"

products = [
    {
        "id": i,
        "name": f"{category.title()} Product {i}",
        "category": category,
        "price": round(random.uniform(10.0, 100.0), 2),
        "inStock": random.choice([True, False]),
        "tags": random.sample(["popular", "new", "sale", "featured"], k=2)
    }
    for i in range(1, 6)
]

response = {
    "category": category,
    "total": len(products),
    "products": products
}

print(json.dumps(response, indent=2))
EOF
'
```

**Test it:**

```bash
curl http://localhost:3000/api/products/electronics
# Output: Complex JSON with 5 randomly generated products
```

***

## Real-World Use Cases

### Use Case 1: Frontend Development Before Backend is Ready

**Scenario:** Your frontend team needs to start work, but the backend API isn't complete yet.

```bash
mock serve --port 3000 \
  --route "api/auth/login" \
  --method POST \
  --response '{"token": "mock-jwt-token-12345", "expiresIn": 3600, "userId": 1}' \
  --route "api/users/me" \
  --method GET \
  --response '{"id": 1, "username": "testuser", "email": "test@example.com", "role": "admin"}' \
  --route "api/dashboard/stats" \
  --method GET \
  --exec '
python3 <<EOF | mock write
import json
import random

stats = {
    "totalUsers": random.randint(1000, 5000),
    "activeUsers": random.randint(100, 500),
    "revenue": round(random.uniform(10000, 50000), 2),
    "growth": round(random.uniform(-5, 15), 1)
}

print(json.dumps(stats))
EOF
'
```

### Use Case 2: Integration Testing with Third-Party APIs

**Scenario:** You need to test integration with a third-party payment API without making real transactions.

```bash
mock serve --port 3000 \
  --route "api/payments/charge" \
  --method POST \
  --exec '
python3 <<EOF | mock write
import json
import random
import uuid

# Simulate payment processing
success = random.random() > 0.1  # 90% success rate

response = {
    "transactionId": str(uuid.uuid4()),
    "status": "success" if success else "failed",
    "amount": 99.99,
    "currency": "USD",
    "timestamp": "2025-11-02T22:14:35Z"
}

if not success:
    response["errorCode"] = "INSUFFICIENT_FUNDS"
    response["errorMessage"] = "Payment declined"

print(json.dumps(response))
EOF
'
```

### Use Case 3: Performance Testing with Delays

**Scenario:** Test how your application handles slow API responses.

```bash
mock serve --port 3000 \
  --base yourapi.com \
  --middleware '
# Add delays to specific endpoints
case "${MOCK_REQUEST_ENDPOINT}" in
  "slow/endpoint")
    sleep 5
    ;;
  "api/reports/generate")
    sleep 10
    ;;
  *)
    # No delay for other endpoints
    ;;
esac
'
```

### Use Case 4: Testing Error Handling

**Scenario:** Ensure your application properly handles various HTTP error codes.

```bash
mock serve --port 3000 \
  --route "api/test/400" \
  --method GET \
  --response '{"error": "Bad Request", "code": 400}' \
  --route "api/test/401" \
  --method GET \
  --response '{"error": "Unauthorized", "code": 401}' \
  --route "api/test/403" \
  --method GET \
  --response '{"error": "Forbidden", "code": 403}' \
  --route "api/test/404" \
  --method GET \
  --response '{"error": "Not Found", "code": 404}' \
  --route "api/test/500" \
  --method GET \
  --response '{"error": "Internal Server Error", "code": 500}' \
  --route "api/test/503" \
  --method GET \
  --response '{"error": "Service Unavailable", "code": 503}'
```

### Use Case 5: Mock Microservices Architecture

**Scenario:** Test your application with multiple mock microservices running on different ports.

```bash
# User Service (Port 3001)
mock serve --port 3001 \
  --route "users/{id}" \
  --method GET \
  --response '{"id": "${id}", "name": "User ${id}", "service": "user-service"}' &

# Order Service (Port 3002)
mock serve --port 3002 \
  --route "orders/{id}" \
  --method GET \
  --response '{"id": "${id}", "userId": "123", "status": "shipped", "service": "order-service"}' &

# Inventory Service (Port 3003)
mock serve --port 3003 \
  --route "inventory/{productId}" \
  --method GET \
  --exec '
printf "{\"productId\": \"${productId}\", \"quantity\": %d, \"service\": \"inventory-service\"}" $((RANDOM % 100))  | mock write
' &

# API Gateway (Port 3000) - Proxies to all services
echo "Mock microservices architecture running:"
echo "  - User Service: http://localhost:3001"
echo "  - Order Service: http://localhost:3002"
echo "  - Inventory Service: http://localhost:3003"
```

**Test the microservices:**

```bash
# Test user service
curl http://localhost:3001/users/42

# Test order service
curl http://localhost:3002/orders/789

# Test inventory service
curl http://localhost:3003/inventory/prod-123
```

***

## Best Practices

### 1. Keep Response Handlers Simple

For simple responses, use `--response`. Reserve `--exec` for dynamic content:

```bash
# Good for static responses
mock serve --port 3000 \
  --route "status" \
  --response '{"status": "ok"}'

# Good for dynamic responses
mock serve --port 3000 \
  --route "timestamp" \
  --exec 'date -Iseconds | mock write'
```

### 2. Use Configuration Files for Complex APIs

Instead of long command lines, create a script or configuration wrapper:

```bash
#!/bin/bash
# mock-api.sh

mock serve --port 3000 \
  --route "users" \
  --response '{"users": []}' \
  --route "users/{id}" \
  --response '{"id": "${id}"}' \
  --route "posts" \
  --response '{"posts": []}' \
  --route "posts/{id}" \
  --response '{"id": "${id}"}'
```

Make it executable:

```bash
chmod +x mock-api.sh
./mock-api.sh
```

### 3. Version Your Mock APIs

Include version information in your responses:

```bash
mock serve --port 3000 \
  --route "api/version" \
  --response '{"version": "1.0.0", "environment": "mock"}'
```

### 4. Use Meaningful Status Codes

Even in mock APIs, proper HTTP status codes improve testing:

```bash
mock serve --port 3000 \
  --route "success" \
  --response '{"status": "ok"}' \  # 200 OK (default)
  --route "created" \
  --exec 'printf "HTTP/1.1 201 Created\r\n\r\n{\"id\": 1}" | mock write'
```

### 5. Document Your Mock Endpoints

Create a README or endpoint listing:

```bash
mock serve --port 3000 \
  --route "" \
  --response '{"endpoints": ["/users", "/posts", "/comments"]}' \
  --route "users" \
  --response '[{"id": 1}, {"id": 2}]'
```

### 6. Clean Up Stateful Data

If using temporary files for state, clean them up on exit:

```bash
#!/bin/bash

TMP_DIR=$(mktemp -d)
trap "rm -rf ${TMP_DIR}" EXIT

mock serve --port 3000 \
  --route "data" \
  --exec 'echo "data" > '"${TMP_DIR}"'/state.txt; cat '"${TMP_DIR}"'/state.txt | mock write'
```

***

## Troubleshooting

### Port Already in Use

```bash
# Error: address already in use
# Solution: Use a different port or kill the process using the port

# Find process using port 3000
lsof -i :3000

# Kill the process
kill -9 <PID>

# Or use a different port
mock serve --port 3001 ...
```

### Shell Script Escaping Issues

When using complex shell scripts in `--exec`, watch out for quote escaping:

```bash
# Problem: Quotes interfere with command parsing
# Solution: Use single quotes for the outer string, double quotes inside

mock serve --port 3000 \
  --route "test" \
  --exec '
printf "{\"key\": \"value\"}" | mock write
'
```

### Mock Write Not Found

If you get "mock write: command not found":

```bash
# Ensure mock is in your PATH
which mock

# If not, add it:
export PATH=$PATH:/path/to/mock

# Or use absolute path
/usr/local/bin/mock serve --port 3000 \
  --route "test" \
  --exec 'printf "test" | /usr/local/bin/mock write'
```

***

## Summary

Mock is a powerful, flexible tool for creating API mocks quickly without writing extensive code. Key takeaways:[1][2]

1. **Quick Setup**: Create mock APIs with simple command-line parameters[2]
2. **Language Agnostic**: Use any programming language for response handlers[3]
3. **Proxy Mode**: Extend existing APIs without modification[2]
4. **Stateful Mocking**: Maintain state across requests using file systems[3]
5. **Testing Integration**: Perfect for frontend development, integration testing, and CI/CD pipelines[1]

The combination of simplicity and power makes Mock ideal for developers who need fast, reliable API mocking during development and testing phases.[1][2]

***

## Additional Resources

- **GitHub Repository**: <https://github.com/dhuan/mock>[1]
- **Official Documentation**: <https://dhuan.github.io/mock/latest/>[3]
- **Download Latest Release**: <https://github.com/dhuan/mock/releases/latest>[1]
- **License**: MIT[1]

***
