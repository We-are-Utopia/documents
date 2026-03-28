# API Contracts

| Field         | Value                                |
|---------------|--------------------------------------|
| **Version**   | 1.0.0                                |
| **Status**    | Draft                                |
| **Author**    | Vox                                  |
| **Reviewer**  | Vox                                  |
| **Created**   | 2026-03-27                           |
| **Updated**   | 2026-03-27                           |
| **Standard**  | OpenAPI 3.1; RFC 7807; REST          |

---

## 1. Purpose

This document specifies the detailed API endpoint contracts per module for the Utopia platform. For overarching API design principles, response envelope, and event schemas, see [INTEGRATION-ARCHITECTURE.md](../02-architecture/INTEGRATION-ARCHITECTURE.md).

## 2. Base URL & Versioning

| Environment | Base URL |
|-------------|----------|
| **dev** | `https://api.utopia.local/api/v1` |
| **staging** | `https://api-staging.utopia.local/api/v1` |

URL pattern: `/api/v{version}/{module}/{resource}`

## 3. Authentication & Authorization

All endpoints (except health checks) require a valid JWT Bearer token from Keycloak. See [ACCESS-CONTROL-POLICY.md](../04-security/ACCESS-CONTROL-POLICY.md).

```
Authorization: Bearer <access_token>
```

### 3.1. RBAC Roles

| Role | Description |
|------|-------------|
| `admin` | Full system access |
| `manager` | Manage catalog, view users |
| `user` | View catalog, manage own profile |
| `viewer` | Read-only access |

---

## 4. Identity Module Endpoints

### 4.1. User Profile

#### `GET /api/v1/identity/users/me`

Get the current authenticated user's profile.

| Property | Value |
|----------|-------|
| **Auth** | Any authenticated user |
| **Rate Limit** | 60 req/min |
| **Cache** | `private, max-age=60` |

**Response 200:**

```json
{
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "email": "vox@utopia.dev",
    "firstName": "Vox",
    "lastName": "Nguyen",
    "avatarUrl": "https://api.utopia.local/avatars/vox.jpg",
    "roles": ["admin"],
    "createdAt": "2026-03-27T10:00:00Z",
    "updatedAt": "2026-03-27T10:00:00Z"
  }
}
```

---

#### `PUT /api/v1/identity/users/me`

Update the current user's profile.

| Property | Value |
|----------|-------|
| **Auth** | Any authenticated user |
| **Rate Limit** | 10 req/min |

**Request Body:**

```json
{
  "firstName": "Vox",
  "lastName": "Nguyen",
  "avatarUrl": "https://api.utopia.local/avatars/vox-new.jpg"
}
```

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| `firstName` | `string` | Yes | 1–100 characters |
| `lastName` | `string` | Yes | 1–100 characters |
| `avatarUrl` | `string?` | No | Valid URL, max 2048 characters |

**Response 200:** Updated user profile (same shape as GET).

**Response 422:**

```json
{
  "type": "https://utopia.dev/errors/validation",
  "title": "Validation Failed",
  "status": 422,
  "detail": "One or more validation errors occurred.",
  "errors": {
    "firstName": ["First name is required."]
  }
}
```

**Domain Event Published:** `UserProfileUpdated`

---

#### `PUT /api/v1/identity/users/me/password`

Change password (delegates to Keycloak).

| Property | Value |
|----------|-------|
| **Auth** | Any authenticated user |
| **Rate Limit** | 5 req/min |

**Request Body:**

```json
{
  "currentPassword": "...",
  "newPassword": "...",
  "confirmPassword": "..."
}
```

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| `currentPassword` | `string` | Yes | Non-empty |
| `newPassword` | `string` | Yes | Min 12 chars, complexity rules per [SECURITY-STANDARD.md](../00-standards/SECURITY-STANDARD.md) |
| `confirmPassword` | `string` | Yes | Must match `newPassword` |

**Response 204:** No content (success).

**Response 400:** Current password incorrect.

---

### 4.2. User Management (Admin)

#### `GET /api/v1/identity/users`

List all users with pagination.

| Property | Value |
|----------|-------|
| **Auth** | `admin` role |
| **Rate Limit** | 30 req/min |
| **Cache** | `private, max-age=30` |

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `page` | `int` | 1 | Page number |
| `pageSize` | `int` | 20 | Items per page (max 100) |
| `search` | `string?` | — | Search by name or email |
| `role` | `string?` | — | Filter by role |
| `sort` | `string` | `createdAt:desc` | Sort field and direction |

**Response 200:**

```json
{
  "data": [
    {
      "id": "...",
      "email": "vox@utopia.dev",
      "firstName": "Vox",
      "lastName": "Nguyen",
      "roles": ["admin"],
      "isActive": true,
      "createdAt": "2026-03-27T10:00:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "pageSize": 20,
    "totalCount": 1,
    "totalPages": 1,
    "hasNextPage": false,
    "hasPreviousPage": false
  }
}
```

---

#### `GET /api/v1/identity/users/{id}`

Get user by ID.

| Property | Value |
|----------|-------|
| **Auth** | `admin` role |

**Response 200:** Single user object.

**Response 404:** User not found.

---

#### `PUT /api/v1/identity/users/{id}/roles`

Assign roles to a user.

| Property | Value |
|----------|-------|
| **Auth** | `admin` role |
| **Rate Limit** | 10 req/min |

**Request Body:**

```json
{
  "roles": ["manager", "user"]
}
```

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| `roles` | `string[]` | Yes | 1–4 valid role names |

**Response 200:** Updated user with new roles.

**Response 422:** Invalid role name.

**Domain Event Published:** `RoleAssigned`

---

#### `POST /api/v1/identity/users/{id}/deactivate`

Deactivate a user account.

| Property | Value |
|----------|-------|
| **Auth** | `admin` role |

**Request Body:**

```json
{
  "reason": "Account closure requested by user"
}
```

**Response 204:** No content (success).

**Domain Event Published:** `UserDeactivated`

---

## 5. Catalog Module Endpoints

### 5.1. Products

#### `GET /api/v1/catalog/products`

List products with filtering, searching, and pagination.

| Property | Value |
|----------|-------|
| **Auth** | `user`, `manager`, `admin` |
| **Rate Limit** | 60 req/min |
| **Cache** | `public, max-age=30` (published only) |

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `page` | `int` | 1 | Page number |
| `pageSize` | `int` | 20 | Items per page (max 100) |
| `categoryId` | `guid?` | — | Filter by category |
| `isPublished` | `bool?` | — | Filter by publish status |
| `search` | `string?` | — | Search by name, SKU, description |
| `minPrice` | `decimal?` | — | Minimum price filter |
| `maxPrice` | `decimal?` | — | Maximum price filter |
| `sort` | `string` | `createdAt:desc` | Sort: `name`, `price`, `createdAt` |

**Response 200:**

```json
{
  "data": [
    {
      "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
      "name": "Wireless Keyboard",
      "slug": "wireless-keyboard",
      "sku": "KB-WL-001",
      "description": "Ergonomic wireless keyboard with backlight",
      "price": {
        "amount": 49.99,
        "currency": "USD"
      },
      "categoryId": "cat-abc-123",
      "categoryName": "Peripherals",
      "isPublished": true,
      "imageUrl": "https://api.utopia.local/images/kb-wl-001.jpg",
      "createdAt": "2026-03-27T10:30:00Z",
      "updatedAt": "2026-03-27T10:30:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "pageSize": 20,
    "totalCount": 142,
    "totalPages": 8,
    "hasNextPage": true,
    "hasPreviousPage": false
  }
}
```

---

#### `GET /api/v1/catalog/products/{id}`

Get product by ID.

| Property | Value |
|----------|-------|
| **Auth** | `user`, `manager`, `admin` |
| **Cache** | `public, max-age=60` (if published) |

**Response 200:**

```json
{
  "data": {
    "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "name": "Wireless Keyboard",
    "slug": "wireless-keyboard",
    "sku": "KB-WL-001",
    "description": "Ergonomic wireless keyboard with backlight",
    "longDescription": "Full product description with features...",
    "price": {
      "amount": 49.99,
      "currency": "USD"
    },
    "categoryId": "cat-abc-123",
    "categoryName": "Peripherals",
    "isPublished": true,
    "images": [
      {
        "id": "img-001",
        "url": "https://api.utopia.local/images/kb-wl-001.jpg",
        "altText": "Wireless Keyboard front view",
        "sortOrder": 1
      }
    ],
    "attributes": {
      "color": "Black",
      "connectivity": "Bluetooth 5.2",
      "batteryLife": "6 months"
    },
    "createdBy": {
      "id": "usr-abc",
      "name": "Vox Nguyen"
    },
    "createdAt": "2026-03-27T10:30:00Z",
    "updatedAt": "2026-03-27T10:30:00Z"
  }
}
```

**Response 404:** Product not found.

---

#### `POST /api/v1/catalog/products`

Create a new product.

| Property | Value |
|----------|-------|
| **Auth** | `manager`, `admin` |
| **Rate Limit** | 30 req/min |

**Request Body:**

```json
{
  "name": "Wireless Keyboard",
  "sku": "KB-WL-001",
  "description": "Ergonomic wireless keyboard with backlight",
  "longDescription": "Full product description...",
  "price": {
    "amount": 49.99,
    "currency": "USD"
  },
  "categoryId": "cat-abc-123",
  "isPublished": false,
  "attributes": {
    "color": "Black",
    "connectivity": "Bluetooth 5.2"
  }
}
```

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| `name` | `string` | Yes | 1–200 characters |
| `sku` | `string` | Yes | 1–50 characters, unique, alphanumeric + dash |
| `description` | `string` | Yes | 1–500 characters |
| `longDescription` | `string?` | No | Max 5000 characters |
| `price.amount` | `decimal` | Yes | > 0, max 2 decimal places |
| `price.currency` | `string` | Yes | ISO 4217 (3 chars), allowed: `USD` |
| `categoryId` | `guid` | Yes | Must reference existing category |
| `isPublished` | `bool` | No | Default `false` |
| `attributes` | `object?` | No | Key-value pairs, max 20 keys |

**Response 201:** Created product (full object). `Location: /api/v1/catalog/products/{id}`

**Response 409:** SKU already exists.

**Response 422:** Validation errors.

**Domain Event Published:** `ProductCreated`

---

#### `PUT /api/v1/catalog/products/{id}`

Update an existing product.

| Property | Value |
|----------|-------|
| **Auth** | `manager`, `admin` |
| **Rate Limit** | 30 req/min |

**Request Body:** Same as POST (all fields optional except `name`).

**Response 200:** Updated product.

**Response 404:** Product not found.

**Response 422:** Validation errors.

**Domain Events Published:** `ProductUpdated`, `PriceChanged` (if price changed)

---

#### `DELETE /api/v1/catalog/products/{id}`

Soft-delete a product (sets `isDeleted = true`, unpublishes).

| Property | Value |
|----------|-------|
| **Auth** | `admin` |
| **Rate Limit** | 10 req/min |

**Response 204:** No content (success).

**Response 404:** Product not found.

---

#### `POST /api/v1/catalog/products/{id}/publish`

Publish a product (make it visible to users).

| Property | Value |
|----------|-------|
| **Auth** | `manager`, `admin` |

**Response 200:** Updated product with `isPublished: true`.

**Response 422:** Product fails publish validation (missing required fields).

---

#### `POST /api/v1/catalog/products/{id}/unpublish`

Unpublish a product.

| Property | Value |
|----------|-------|
| **Auth** | `manager`, `admin` |

**Response 200:** Updated product with `isPublished: false`.

---

### 5.2. Product Images

#### `GET /api/v1/catalog/products/{id}/images`

List images for a product.

| Property | Value |
|----------|-------|
| **Auth** | `user`, `manager`, `admin` |
| **Cache** | `public, max-age=300` |

**Response 200:**

```json
{
  "data": [
    {
      "id": "img-001",
      "url": "https://api.utopia.local/images/kb-wl-001.jpg",
      "altText": "Wireless Keyboard front view",
      "sortOrder": 1,
      "createdAt": "2026-03-27T10:30:00Z"
    }
  ]
}
```

---

#### `POST /api/v1/catalog/products/{id}/images`

Upload an image for a product.

| Property | Value |
|----------|-------|
| **Auth** | `manager`, `admin` |
| **Rate Limit** | 10 req/min |
| **Content-Type** | `multipart/form-data` |

**Request:**

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| `file` | `IFormFile` | Yes | JPEG, PNG, WebP; max 5MB |
| `altText` | `string` | Yes | 1–200 characters |
| `sortOrder` | `int` | No | Default: next in sequence |

**Response 201:** Created image object. Max 10 images per product.

**Response 422:** Invalid file type or size.

---

#### `DELETE /api/v1/catalog/products/{id}/images/{imageId}`

Delete a product image.

| Property | Value |
|----------|-------|
| **Auth** | `manager`, `admin` |

**Response 204:** No content.

---

### 5.3. Categories

#### `GET /api/v1/catalog/categories`

List all categories (flat list, supports tree via `parentId`).

| Property | Value |
|----------|-------|
| **Auth** | `user`, `manager`, `admin` |
| **Cache** | `public, max-age=300` |

**Response 200:**

```json
{
  "data": [
    {
      "id": "cat-abc-123",
      "name": "Electronics",
      "slug": "electronics",
      "parentId": null,
      "productCount": 42,
      "sortOrder": 1
    },
    {
      "id": "cat-def-456",
      "name": "Peripherals",
      "slug": "peripherals",
      "parentId": "cat-abc-123",
      "productCount": 18,
      "sortOrder": 1
    }
  ]
}
```

---

#### `POST /api/v1/catalog/categories`

Create a new category.

| Property | Value |
|----------|-------|
| **Auth** | `admin` |
| **Rate Limit** | 10 req/min |

**Request Body:**

```json
{
  "name": "Peripherals",
  "parentId": "cat-abc-123",
  "sortOrder": 1
}
```

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| `name` | `string` | Yes | 1–100 characters, unique within parent |
| `parentId` | `guid?` | No | Must reference existing category, max 3 levels deep |
| `sortOrder` | `int` | No | Default 0 |

**Response 201:** Created category.

**Response 409:** Category name already exists.

---

#### `PUT /api/v1/catalog/categories/{id}`

Update a category.

| Property | Value |
|----------|-------|
| **Auth** | `admin` |

**Response 200:** Updated category.

---

#### `DELETE /api/v1/catalog/categories/{id}`

Delete a category (only if no products assigned).

| Property | Value |
|----------|-------|
| **Auth** | `admin` |

**Response 204:** No content.

**Response 409:** Category has assigned products.

---

## 6. Health Check Endpoints

No authentication required.

#### `GET /health/live`

Liveness probe — application is running.

**Response 200:**

```json
{
  "status": "Healthy"
}
```

#### `GET /health/ready`

Readiness probe — application and all dependencies are ready.

**Response 200:**

```json
{
  "status": "Healthy",
  "checks": {
    "postgresql": "Healthy",
    "redis": "Healthy",
    "rabbitmq": "Healthy",
    "keycloak": "Healthy"
  }
}
```

**Response 503:** One or more dependencies unhealthy.

---

## 7. Common Error Responses

All errors follow [RFC 7807](https://www.rfc-editor.org/rfc/rfc7807) format.

| Status | Type URI | When |
|--------|----------|------|
| 400 | `https://utopia.dev/errors/bad-request` | Malformed JSON, missing content-type |
| 401 | `https://utopia.dev/errors/unauthorized` | Missing or expired JWT |
| 403 | `https://utopia.dev/errors/forbidden` | Insufficient role/permissions |
| 404 | `https://utopia.dev/errors/not-found` | Resource does not exist |
| 409 | `https://utopia.dev/errors/conflict` | Duplicate SKU, slug, or name |
| 422 | `https://utopia.dev/errors/validation` | Business validation failure |
| 429 | `https://utopia.dev/errors/rate-limited` | Rate limit exceeded |
| 500 | `https://utopia.dev/errors/internal` | Unhandled exception (no details exposed) |

## 8. Rate Limiting Summary

| Endpoint Group | Limit | Window | Scope |
|----------------|-------|--------|-------|
| GET (read) | 60 req | 1 minute | Per user |
| POST/PUT (write) | 30 req | 1 minute | Per user |
| DELETE | 10 req | 1 minute | Per user |
| Password change | 5 req | 1 minute | Per user |
| Image upload | 10 req | 1 minute | Per user |
| Global | 1000 req | 1 minute | All users |

Rate limit headers returned:

```
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 42
X-RateLimit-Reset: 1711540800
Retry-After: 30
```

## 9. OpenAPI Specification

The OpenAPI 3.1 spec is auto-generated from code during CI and published to [contracts/](./contracts/). See [CI-PIPELINE.md](../06-devops/CI-PIPELINE.md) for the generation step.

- Swagger UI is available in `dev` at `/swagger`
- Swagger UI is disabled in `staging`

## 10. References

- [INTEGRATION-ARCHITECTURE.md](../02-architecture/INTEGRATION-ARCHITECTURE.md) — API design principles, events
- [ACCESS-CONTROL-POLICY.md](../04-security/ACCESS-CONTROL-POLICY.md) — RBAC roles and permissions
- [DATA-ARCHITECTURE.md](../02-architecture/DATA-ARCHITECTURE.md) — Database schemas
- [SECURITY-STANDARD.md](../00-standards/SECURITY-STANDARD.md) — Password and input validation rules
- [CI-PIPELINE.md](../06-devops/CI-PIPELINE.md) — OpenAPI spec generation

## Changelog

| Version | Date       | Author | Description          |
|---------|------------|--------|----------------------|
| 1.0.0   | 2026-03-27 | Vox    | Initial draft        |
