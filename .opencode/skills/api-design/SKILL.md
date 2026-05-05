---
name: api-design
description: REST, GraphQL, gRPC, and WebSocket API design patterns. Includes endpoint design, versioning, authentication, rate limiting, and documentation standards. Use when designing APIs, reviewing API specs, establishing API conventions, or when the user mentions REST, GraphQL, gRPC, WebSocket, endpoints, API versioning, OpenAPI, Swagger, or API documentation.
---

# API Design Skill

Design robust, scalable, and developer-friendly APIs across REST, GraphQL, gRPC, and WebSocket protocols.

## When to Use
- Designing new API endpoints or services
- Reviewing API specifications
- Establishing API conventions for a project
- Migrating between API protocols (REST → GraphQL, etc.)
- Adding authentication, rate limiting, or versioning

## REST API Design

### Endpoint Structure
```
GET    /api/v1/resources       # List resources
GET    /api/v1/resources/:id   # Get single resource
POST   /api/v1/resources       # Create resource
PUT    /api/v1/resources/:id   # Update resource (full)
PATCH  /api/v1/resources/:id   # Update resource (partial)
DELETE /api/v1/resources/:id   # Delete resource
```

### Best Practices
- Use nouns, not verbs (`/users` not `/getUsers`)
- Use proper HTTP methods (GET, POST, PUT, PATCH, DELETE)
- Return appropriate status codes (200, 201, 400, 401, 403, 404, 500)
- Use query parameters for filtering: `?status=active&sort=date`
- Version APIs early: `/api/v1/`, `/api/v2/`
- Use consistent error response format:
```json
{
  "error": {
    "code": "RESOURCE_NOT_FOUND",
    "message": "The requested resource was not found",
    "details": {...}
  }
}
```

### OpenAPI/Swagger Spec
Generate and maintain OpenAPI 3.0 spec:
```yaml
openapi: 3.0.0
info:
  title: API Name
  version: 1.0.0
paths:
  /users:
    get:
      summary: List users
      parameters:
        - name: limit
          in: query
          schema:
            type: integer
      responses:
        '200':
          description: OK
```

## GraphQL Design

### Schema Basics
```graphql
type User {
  id: ID!
  name: String!
  email: String!
  posts: [Post]
}

type Query {
  user(id: ID!): User
  users(limit: Int, offset: Int): [User]
}

type Mutation {
  createUser(input: CreateUserInput!): User!
  updateUser(id: ID!, input: UpdateUserInput!): User!
}

input CreateUserInput {
  name: String!
  email: String!
}
```

### Best Practices
- Use clear, descriptive type and field names
- Implement pagination for collections (cursor-based preferred)
- Use `@deprecated` directive for old fields
- Implement proper error handling with unions or errors in response
- Consider using DataLoader for N+1 query prevention

## gRPC Design

### Proto File Structure
```protobuf
syntax = "proto3";

package user.v1;

service UserService {
  rpc GetUser(GetUserRequest) returns (User);
  rpc ListUsers(ListUsersRequest) returns (ListUsersResponse);
  rpc CreateUser(CreateUserRequest) returns (User);
}

message GetUserRequest {
  string user_id = 1;
}

message User {
  string id = 1;
  string name = 2;
  string email = 3;
}
```

### Best Practices
- Version proto packages (`user.v1`, `user.v2`)
- Use appropriate field numbers and don't reuse them
- Design idempotent RPCs when possible
- Implement proper timeout and retry logic
- Use streaming for large data transfers

## WebSocket API Design

### Connection Management
```javascript
// Client connection
const ws = new WebSocket('wss://api.example.com/v1/ws');

// Message protocol (JSON)
{
  "type": "subscribe",
  "channel": "updates",
  "data": {...}
}
```

### Best Practices
- Use secure WebSocket (wss://) in production
- Implement heartbeat/ping-pong for connection health
- Design clear message types and protocols
- Handle reconnection with exponential backoff
- Authenticate during connection handshake

## Authentication Patterns

### API Key
```
Authorization: ApiKey your-api-key-here
```

### Bearer Token (JWT)
```
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

### OAuth 2.0
- Authorization Code (web apps)
- Client Credentials (server-to-server)
- PKCE for mobile/SPA

## Rate Limiting

### Headers
```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1640995200
```

### Strategies
- Token bucket (recommended)
- Leaky bucket
- Fixed window
- Sliding window

## Documentation Standards

- **OpenAPI/Swagger** for REST APIs
- **GraphQL Playground/Apollo Sandbox** for GraphQL
- **Proto documentation** for gRPC
- Include examples for every endpoint/operation
- Document error codes and responses
- Provide Postman/Insomnia collections

## Verification

After designing an API:
1. Validate OpenAPI spec: `swagger-cli validate spec.yaml`
2. Test GraphQL schema: `graphql-codegen` introspection
3. Ensure all endpoints have authentication specified
4. Verify rate limiting headers are documented
5. Check that error responses are consistent
