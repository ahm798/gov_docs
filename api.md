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

**Response:** `200 OK`

```json
{
  "id": "123e4567-e89b-12d3-a456-426614174000",
  "slug": "ministry-of-health",
  "name": {
    "en": "Ministry of Health",
    "ar": "وزارة الصحة"
  },
  "active": true,
  "currency": "OMR",
  "countryCode": "OM",
  "availableLanguages": ["en", "ar"],
  "logoUrl": "https://example.com/logos/ministry-health.png",
  "assignedModules": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "name": {
        "en": "Strategic Management",
        "ar": "الإدارة الاستراتيجية"
      },
      "iconUrl": "https://example.com/icons/strategic.svg",
      "modules": [
        {
          "id": "6ba7b810-9dad-11d1-80b4-00c04fd430c8",
          "slug": "maps-360",
          "name": {
            "en": "360 Maps",
            "ar": "خرائط 360"
          },
          "description": {
            "en": "Interactive strategic maps",
            "ar": "خرائط استراتيجية تفاعلية"
          },
          "iconUrl": "https://example.com/icons/maps.svg"
        }
      ]
    }
  ],
  "localized": {
    "name": "Ministry of Health"
  },
  "_links": {
    "self": {
      "href": "/organizations/123e4567-e89b-12d3-a456-426614174000"
    }
  }
}
```

#### Get Organization by ID

```http
GET /organizations/{id}
```

**Response:** Same as Get My Organization

#### Create Organization

```http
POST /organizations
Content-Type: application/json

{
  "slug": "new-ministry",
  "name": {
    "en": "New Ministry",
    "ar": "وزارة جديدة"
  },
  "currency": "OMR",
  "countryCode": "OM",
  "availableLanguages": ["en", "ar"],
  "logoUrl": "https://example.com/logos/new-ministry.png",
  "organizationUnitRootId": "550e8400-e29b-41d4-a716-446655440000",
  "moduleIds": ["6ba7b810-9dad-11d1-80b4-00c04fd430c8"]
}
```

**Required Fields:**
- `slug`: Kebab-case string (lowercase letters, digits, hyphens)
- `name`: LocalizedValue object with language keys

**Optional Fields:**
- `currency`: ISO 4217 code (default: "OMR")
- `countryCode`: ISO 3166-1 alpha-2 code (default: "OM")
- `availableLanguages`: Array of language codes (default: ["en", "ar"])
- `logoUrl`: URL to organization logo
- `organizationUnitRootId`: UUID of root organization unit
- `moduleIds`: Set of module UUIDs to assign
- `organizationGroup`: Organization group object

**Response:** `201 Created` with same structure as GET response

#### Update Organization (PATCH)

```http
PATCH /organizations/{id}?operation={operation}
Content-Type: application/json
```

**Operations:**

1. **update-metadata** (requires body)
```json
{
  "name": {
    "en": "Updated Ministry",
    "ar": "وزارة محدثة"
  },
  "currency": "SAR",
  "countryCode": "SA",
  "availableLanguages": ["en", "ar", "fr"],
  "logoUrl": "https://example.com/logos/updated.png"
}
```

2. **move-to-group** (requires body)
```json
{
  "groupId": "550e8400-e29b-41d4-a716-446655440000"
}
```

3. **set-root-unit** (requires body)
```json
{
  "unitId": "6ba7b810-9dad-11d1-80b4-00c04fd430c8"
}
```

4. **assign-modules** (requires body)
```json
{
  "moduleIds": ["uuid1", "uuid2", "uuid3"]
}
```

5. **activate** (no body required)
```http
PATCH /organizations/{id}?operation=activate
```

6. **deactivate** (no body required)
```http
PATCH /organizations/{id}?operation=deactivate
```

**Response:** `200 OK` with updated organization object

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

System modules grouped by functionality. Modules are read-only (managed by admins).

#### List Module Groups (with modules)

```http
GET /modules/groups
```

**Response:** `200 OK`

```json
{
  "_embedded": {
    "moduleGroupResponseList": [
      {
        "id": "550e8400-e29b-41d4-a716-446655440000",
        "name": {
          "en": "Strategic Management",
          "ar": "الإدارة الاستراتيجية"
        },
        "iconUrl": "https://example.com/icons/strategic.svg",
        "modules": [
          {
            "id": "6ba7b810-9dad-11d1-80b4-00c04fd430c8",
            "slug": "maps-360",
            "name": {
              "en": "360 Maps",
              "ar": "خرائط 360"
            },
            "description": {
              "en": "Interactive strategic maps and dashboards",
              "ar": "خرائط ولوحات استراتيجية تفاعلية"
            },
            "iconUrl": "https://example.com/icons/maps.svg"
          },
          {
            "id": "7c9e6679-7425-40de-944b-e07fc1f90ae7",
            "slug": "kpi-management",
            "name": {
              "en": "KPI Management",
              "ar": "إدارة مؤشرات الأداء"
            },
            "description": {
              "en": "Key Performance Indicators tracking",
              "ar": "تتبع مؤشرات الأداء الرئيسية"
            },
            "iconUrl": "https://example.com/icons/kpi.svg"
          }
        ]
      }
    ]
  },
  "_links": {
    "self": {
      "href": "/modules/groups"
    }
  }
}
```

#### Assign Modules to Organization

```http
PUT /modules/organizations/{organizationId}/assignments
Content-Type: application/json

{
  "moduleIds": [
    "6ba7b810-9dad-11d1-80b4-00c04fd430c8",
    "7c9e6679-7425-40de-944b-e07fc1f90ae7"
  ]
}
```

**Response:** `204 No Content`

#### Assign Modules to Membership

```http
PUT /modules/memberships/{membershipId}/assignments
Content-Type: application/json

{
  "moduleIds": [
    "6ba7b810-9dad-11d1-80b4-00c04fd430c8"
  ]
}
```

**Response:** `204 No Content`

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
  "slug": "string (unique, kebab-case)",
  "name": "LocalizedValue<String>",
  "active": "boolean",
  "currency": "string (ISO 4217, 3 chars)",
  "countryCode": "string (ISO 3166-1 alpha-2, 2 chars)",
  "availableLanguages": ["string"],
  "logoUrl": "string (nullable)",
  "assignedModules": "AssignedModuleGroupResponse[]",
  "localized": {
    "name": "string (based on Accept-Language header)"
  }
}
```

### Module

```json
{
  "id": "uuid",
  "slug": "string (unique, kebab-case)",
  "name": "LocalizedValue<String>",
  "description": "LocalizedValue<String>",
  "iconUrl": "string (nullable)"
}
```

### ModuleGroup

```json
{
  "id": "uuid",
  "name": "LocalizedValue<String>",
  "iconUrl": "string (nullable)",
  "modules": "Module[]"
}
```

### AssignedModuleGroupResponse

```json
{
  "id": "uuid",
  "name": "LocalizedValue<String>",
  "iconUrl": "string (nullable)",
  "modules": "Module[]"
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
