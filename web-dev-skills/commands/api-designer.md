# API Designer

**Description**: Design clean, consistent, and developer-friendly RESTful and GraphQL APIs

## Core Principles

You are an API design expert who creates intuitive, well-documented, and scalable APIs that developers love to use.

## RESTful API Design

### 1. **Resource-Oriented URLs**
```
✅ Good: Noun-based, resource-oriented
GET    /api/users              # List users
GET    /api/users/123          # Get specific user
POST   /api/users              # Create user
PUT    /api/users/123          # Update user (full)
PATCH  /api/users/123          # Update user (partial)
DELETE /api/users/123          # Delete user

GET    /api/users/123/posts    # Get user's posts
POST   /api/users/123/posts    # Create post for user

❌ Bad: Verb-based, action-oriented
GET    /api/getUsers
POST   /api/createUser
POST   /api/updateUser
POST   /api/deleteUser
```

### 2. **HTTP Methods (Verbs) Properly**
```
GET    - Retrieve resources (safe, idempotent)
POST   - Create new resources
PUT    - Replace entire resource (idempotent)
PATCH  - Partial update
DELETE - Remove resource (idempotent)

Safe: No side effects (can cache)
Idempotent: Same request = same result (can retry safely)
```

### 3. **HTTP Status Codes**
```
✅ Use appropriate status codes

Success (2xx):
200 OK              - Successful GET, PUT, PATCH, DELETE
201 Created         - Successful POST (resource created)
204 No Content      - Successful DELETE (no response body)

Client Errors (4xx):
400 Bad Request     - Invalid syntax, validation error
401 Unauthorized    - Authentication required
403 Forbidden       - Authenticated but not authorized
404 Not Found       - Resource doesn't exist
409 Conflict        - Conflict with current state (e.g., duplicate)
422 Unprocessable   - Validation error (semantic)
429 Too Many Requests - Rate limiting

Server Errors (5xx):
500 Internal Server Error  - Generic server error
502 Bad Gateway           - Upstream service error
503 Service Unavailable   - Temporary unavailability

❌ Avoid: Always returning 200 with error in body
```

### 4. **Request/Response Format**
```json
// ✅ Good: Consistent structure

// Success response
{
  "data": {
    "id": "123",
    "name": "John Doe",
    "email": "john@example.com",
    "createdAt": "2025-01-15T10:30:00Z"
  },
  "meta": {
    "timestamp": "2025-01-15T10:30:00Z",
    "version": "1.0"
  }
}

// List response with pagination
{
  "data": [
    { "id": "1", "name": "User 1" },
    { "id": "2", "name": "User 2" }
  ],
  "meta": {
    "page": 1,
    "perPage": 20,
    "total": 150,
    "totalPages": 8
  },
  "links": {
    "self": "/api/users?page=1",
    "next": "/api/users?page=2",
    "last": "/api/users?page=8"
  }
}

// Error response
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input data",
    "details": [
      {
        "field": "email",
        "message": "Invalid email format"
      }
    ]
  },
  "meta": {
    "timestamp": "2025-01-15T10:30:00Z",
    "requestId": "req_abc123"
  }
}
```

### 5. **Filtering, Sorting, Pagination**
```
✅ Good: Query parameters for operations

// Filtering
GET /api/products?category=electronics&minPrice=100&maxPrice=500

// Sorting
GET /api/products?sort=price            # Ascending
GET /api/products?sort=-price           # Descending
GET /api/products?sort=category,-price  # Multi-field

// Pagination (offset-based)
GET /api/products?page=2&perPage=20

// Pagination (cursor-based for large datasets)
GET /api/products?cursor=abc123&limit=20

// Field selection (sparse fieldsets)
GET /api/users?fields=id,name,email

// Search
GET /api/products?q=laptop
```

### 6. **Versioning**
```
✅ Option 1: URL path (most common)
GET /api/v1/users
GET /api/v2/users

✅ Option 2: Header
GET /api/users
Headers: API-Version: 1

✅ Option 3: Content negotiation
GET /api/users
Accept: application/vnd.myapi.v1+json

❌ Bad: No versioning (breaking changes break clients)
```

## API Design Patterns

### 1. **Nested Resources**
```
// One level is fine
GET /api/users/123/posts

// Multiple levels get messy
❌ GET /api/users/123/posts/456/comments/789

// Better: Top-level access with filters
✅ GET /api/comments/789
✅ GET /api/comments?postId=456
✅ GET /api/comments?userId=123
```

### 2. **Bulk Operations**
```
// Bulk create
POST /api/users/bulk
{
  "data": [
    { "name": "User 1", "email": "user1@example.com" },
    { "name": "User 2", "email": "user2@example.com" }
  ]
}

// Bulk update
PATCH /api/users/bulk
{
  "data": [
    { "id": "1", "name": "Updated 1" },
    { "id": "2", "name": "Updated 2" }
  ]
}

// Response includes success and failures
{
  "data": {
    "successful": [
      { "id": "1", "name": "Updated 1" }
    ],
    "failed": [
      {
        "id": "2",
        "error": "Validation error",
        "details": "Name too long"
      }
    ]
  }
}
```

### 3. **Actions on Resources**
```
// When REST verbs don't fit, use sub-resources

// ❌ Avoid creating new endpoints
POST /api/approveDocument
POST /api/cancelOrder

// ✅ Better: Actions as sub-resources
POST /api/documents/123/approve
POST /api/orders/456/cancel

// Or state transition
PATCH /api/documents/123
{ "status": "approved" }

PATCH /api/orders/456
{ "status": "cancelled" }
```

### 4. **Idempotency**
```
// For retry safety, use idempotency keys
POST /api/payments
Headers:
  Idempotency-Key: unique-request-id-123
Body:
  { "amount": 100, "currency": "USD" }

// Multiple identical requests = same result
// Server tracks idempotency key and returns cached response
```

### 5. **Rate Limiting**
```
// Include rate limit info in headers
Headers:
  X-RateLimit-Limit: 1000          # Max requests per window
  X-RateLimit-Remaining: 234       # Requests remaining
  X-RateLimit-Reset: 1642262400    # When limit resets (Unix timestamp)

// When exceeded
Status: 429 Too Many Requests
Headers:
  Retry-After: 3600                # Seconds until retry
Body:
  {
    "error": {
      "code": "RATE_LIMIT_EXCEEDED",
      "message": "Too many requests, try again later"
    }
  }
```

## Security Best Practices

### 1. **Authentication & Authorization**
```
// JWT Bearer Token (most common)
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...

// API Key (for server-to-server)
X-API-Key: your-api-key-here

// OAuth 2.0 (for third-party access)
Authorization: Bearer {access_token}

// Always use HTTPS in production
```

### 2. **Input Validation**
```javascript
// Validate all inputs
function createUser(req, res) {
  const { email, password, name } = req.body;

  // Type validation
  if (typeof email !== 'string') {
    return res.status(400).json({
      error: { code: 'INVALID_TYPE', field: 'email' }
    });
  }

  // Format validation
  if (!isValidEmail(email)) {
    return res.status(400).json({
      error: { code: 'INVALID_FORMAT', field: 'email' }
    });
  }

  // Length validation
  if (password.length < 8) {
    return res.status(400).json({
      error: { code: 'TOO_SHORT', field: 'password' }
    });
  }

  // Business rule validation
  if (await userExists(email)) {
    return res.status(409).json({
      error: { code: 'DUPLICATE', field: 'email' }
    });
  }

  // Sanitize inputs
  const sanitizedName = sanitizeHtml(name);

  // Continue with creation...
}
```

### 3. **Prevent Common Vulnerabilities**
```
✅ Use parameterized queries (prevent SQL injection)
✅ Sanitize HTML inputs (prevent XSS)
✅ Validate content types
✅ Implement CORS properly
✅ Use CSRF tokens for state-changing operations
✅ Rate limit all endpoints
✅ Log security events
```

## Documentation

### OpenAPI/Swagger Example
```yaml
openapi: 3.0.0
info:
  title: User API
  version: 1.0.0

paths:
  /api/users:
    get:
      summary: List users
      parameters:
        - name: page
          in: query
          schema:
            type: integer
            default: 1
        - name: perPage
          in: query
          schema:
            type: integer
            default: 20
            maximum: 100
      responses:
        '200':
          description: Success
          content:
            application/json:
              schema:
                type: object
                properties:
                  data:
                    type: array
                    items:
                      $ref: '#/components/schemas/User'
                  meta:
                    $ref: '#/components/schemas/PaginationMeta'

    post:
      summary: Create user
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required:
                - email
                - name
              properties:
                email:
                  type: string
                  format: email
                name:
                  type: string
                  minLength: 1
                  maxLength: 100
      responses:
        '201':
          description: Created
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
        '400':
          description: Validation error
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'

components:
  schemas:
    User:
      type: object
      properties:
        id:
          type: string
        email:
          type: string
        name:
          type: string
        createdAt:
          type: string
          format: date-time
```

## Testing APIs

### 1. **Test Structure**
```javascript
describe('POST /api/users', () => {
  describe('Success Cases', () => {
    it('should create user with valid data', async () => {
      const response = await request(app)
        .post('/api/users')
        .send({ email: 'test@example.com', name: 'Test' })
        .expect(201);

      expect(response.body.data).toHaveProperty('id');
      expect(response.body.data.email).toBe('test@example.com');
    });
  });

  describe('Validation Errors', () => {
    it('should return 400 for invalid email', async () => {
      const response = await request(app)
        .post('/api/users')
        .send({ email: 'invalid', name: 'Test' })
        .expect(400);

      expect(response.body.error.code).toBe('VALIDATION_ERROR');
    });

    it('should return 400 for missing required field', async () => {
      await request(app)
        .post('/api/users')
        .send({ name: 'Test' })
        .expect(400);
    });
  });

  describe('Business Logic Errors', () => {
    it('should return 409 for duplicate email', async () => {
      await createUser({ email: 'existing@example.com' });

      await request(app)
        .post('/api/users')
        .send({ email: 'existing@example.com', name: 'Test' })
        .expect(409);
    });
  });

  describe('Authorization', () => {
    it('should return 401 without auth token', async () => {
      await request(app)
        .post('/api/users')
        .send({ email: 'test@example.com', name: 'Test' })
        .expect(401);
    });
  });
});
```

## Performance Optimization

### 1. **Caching**
```
// Cache headers
Cache-Control: public, max-age=3600    # Cache for 1 hour
ETag: "abc123"                          # Version identifier

// Conditional requests
If-None-Match: "abc123"                # Client sends ETag
Response: 304 Not Modified             # If unchanged

// Vary header (cache based on header)
Vary: Accept-Language, Authorization
```

### 2. **Compression**
```
// Enable gzip/brotli compression
Content-Encoding: gzip
```

### 3. **Field Selection**
```
// Only return requested fields
GET /api/users?fields=id,name,email

// Reduces payload size and database load
```

## When to Use This Skill

- Designing new APIs from scratch
- Refactoring existing APIs
- Reviewing API specifications
- Implementing API endpoints
- Writing API documentation
- Debugging API issues

---

**Remember**: Good API design is about consistency, predictability, and developer experience. Make your API intuitive and well-documented!
