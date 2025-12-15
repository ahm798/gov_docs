# GOV360 Organization Service API Documentation

## Overview

The Organization Service provides APIs for managing organizations, organizational units, memberships, and module assignments in the GOV360 platform.

**Version:** 0.2.1  
**Base URL (Production):** `https://v2.gov360.tadafur.com/api/organization`  
**Base URL (Development):** `http://localhost/api/organization`

## Authentication

All API endpoints require OAuth2 authentication via Keycloak.

### Obtaining an Access Token

**Endpoint:** `POST /realms/gov360/protocol/openid-connect/token`  
**Auth Server:** `https://v2.auth.gov360.tadafur.com` (production) or `http://localhost:8080` (dev)

#### Password Grant (User Login)

```bash
curl -X POST "https://v2.auth.gov360.tadafur.com/realms/gov360/protocol/openid-connect/token" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "client_id={{client_id}}" \
  -d "grant_type=password" \
  -d "username=YOUR_USERNAME" \
  -d "password=YOUR_PASSWORD"
```

#### Response

```json
{
  "access_token": "...",
  "expires_in": 300,
  "refresh_expires_in": 1800,
  "refresh_token": "...",
  "token_type": "Bearer"
}
```

### Using the Access Token

Include the access token in the `Authorization` header:

```bash
Authorization: Bearer ...
```

## Core Resources

### Organizations

Organizations are the top-level entities in the system.

#### Get My Organization

```http
GET /organizations/my-organization
```

#### Get Organization by ID

```http
GET /organizations/{id}
```

#### Create Organization

```http
POST /organizations
Content-Type: application/json

{
  "slug": "new-org",
  "name": {
    "en": "New Organization",
    "ar": "منظمة جديدة"
  }
}
```

#### Update Organization (PATCH)

```http
PATCH /organizations/{id}?operation=update-name
Content-Type: application/json

{
  "name": {
    "en": "Updated Organization",
    "ar": "منظمة محدثة"
  }
}
```

Operation options: `update-name`, `update-slug`, `activate`, `deactivate`

---

### Memberships

User memberships in organizations.

#### Get My Membership

```http
GET /organization-memberships/my-membership
```

#### Get Membership by ID

```http
GET /organization-memberships/{id}
```

#### List Memberships in My Organization

```http
GET /organizations/my-organization/memberships
```

#### Create Membership

```http
POST /organization-memberships
Content-Type: application/json

{
  "userId": "user-123",
  "organizationId": "...",
  "roleType": "ADMIN"
}
```

#### Update Membership (PATCH)

```http
PATCH /organization-memberships/{id}?operation=update-role
Content-Type: application/json

{
  "roleType": "MEMBER"
}
```

Operation options: `update-role`, `activate`, `deactivate`

#### Delete Membership

```http
DELETE /organization-memberships/{id}
```

---

### Modules & Module Groups

System modules grouped by functionality. Modules are read-only (no CRUD).

#### List Module Groups (with modules)

```http
GET /modules/groups
```

#### Assign Modules to Organization

```http
PUT /modules/organizations/{organizationId}/assignments
Content-Type: application/json

{
  "moduleIds": [1, 2, 3, 4]
}
```

#### Assign Modules to Membership

```http
PUT /modules/memberships/{membershipId}/assignments
Content-Type: application/json

{
  "moduleIds": [1, 2, 5]
}
```

---

### Organization Units

Hierarchical units within an organization (stored in Neo4j graph database).

#### Get Unit Tree by Unit ID

```http
GET /organization-units/{unitId}/tree?depth=10
```

#### Get Unit Tree by Organization ID

```http
GET /organizations/{organizationId}/organization-unit/tree?depth=10
```

---

### Users

#### Create User

```http
POST /users
Content-Type: application/json

{
  "username": "newuser",
  "email": "user@example.com",
  "firstName": "John",
  "lastName": "Doe",
  "password": "SecurePass123!"
}
```

#### Disable User

```http
DELETE /users/{sub}
```

---

### Permissions

#### Grant Organization Permission

```http
POST /permissions/organizations/{organizationId}/writers
Content-Type: application/json

{
  "sub": "user-uuid"
}
```

#### Revoke Organization Permission

```http
DELETE /permissions/organizations/{organizationId}/writers
Content-Type: application/json

{
  "sub": "user-uuid"
}
```

---

### Health & Monitoring

#### Health Check

```http
GET /actuator/health
```

**Response:**

```json
{
  "status": "UP",
  "components": {
    "db": { "status": "UP" },
    "neo4j": { "status": "UP" }
  }
}
```

## Error Responses

### Standard Error Format

```json
{
  "timestamp": "...",
  "status": 400,
  "error": "Bad Request",
  "message": "Invalid organization slug",
  "path": "/organizations"
}
```

### Common Status Codes

- `200 OK` - Successful request
- `201 Created` - Resource created successfully
- `204 No Content` - Successful deletion
- `400 Bad Request` - Invalid input
- `401 Unauthorized` - Missing or invalid token
- `403 Forbidden` - Insufficient permissions
- `404 Not Found` - Resource not found
- `500 Internal Server Error` - Server error

## Data Models

### LocalizedValue

Multilingual support for text fields:

```json
{
  "en": "English text",
  "ar": "النص العربي"
}
```

### Organization

```json
{
  "id": "uuid",
  "slug": "string (unique)",
  "name": "LocalizedValue<String>",
  "active": "boolean",
  "assignedModules": "AssignedModuleGroupResponse[]"
}
```

### Module

```json
{
  "id": "integer",
  "name": "LocalizedValue<String>",
  "description": "LocalizedValue<String>",
  "moduleGroup": "ModuleGroupResponse"
}
```

### ModuleGroup

```json
{
  "id": "integer",
  "name": "LocalizedValue<String>",
  "description": "LocalizedValue<String>"
}
```

## Authorization

The service uses OpenFGA for fine-grained authorization:

- Organization-level permissions (view, edit, delete)
- Module assignments (which modules an organization can access)
- Membership-level permissions (user roles within organizations)

Authorization checks are performed automatically based on JWT claims and OpenFGA tuples.

## Rate Limiting

Currently no rate limiting is enforced. This may be added in future versions.

## Versioning

API version is included in the base path: `/api/organization`

Future versions may use: `/api/v2/organization`

## Support

For API support and questions:
- **Email:** support@tadafur.com
- **Documentation:** https://docs.gov360.tadafur.com
