# CompanyUsersController Documentation

## Overview

CompanyUsersController manages company users in the Jobix Dashboard admin panel. It provides full CRUD for users with role assignment, personal settings, and migration of users between companies.

Base path: `/company-users`  
Controller type: WebController (Admin Application)  
Namespace: `apps\\admin\\controllers\\CompanyUsersController`

---

## Endpoints

### 1. Index — Company Users List

| Param | Value |
|------|-------|
| URL | `/company-users/index?company_uuid={company_uuid}` |
| Method | GET |
| Description | List all company users with search, filters, and migration tools |

URL Parameters:
- `company_uuid` (string, required) — company UUID

Input (CompanyUserSearchForm queryParams):

| Field | Type | Required | Validation | Description |
|------|------|----------|------------|-------------|
| `company_uuid` | string | ✓ | required, UUID, exists | Company UUID |
| `email` | string | ✗ | string | Filter by email (ILIKE) |
| `name` | string | ✗ | string | Filter by full name (CONCAT first_name + last_name) |
| `role` | string | ✗ | string | Filter by role |
| `status` | integer | ✗ | integer | Active/inactive |
| `language` | string | ✗ | string | Filter by language |
| `timezone` | string | ✗ | string | Filter by timezone |
| `is_admin` | boolean | ✗ | boolean | Admin filter |
| `updated_at` | array | ✗ | timestamp array | Updated date range [start, end] |

Business logic:
1. Validate company_uuid and user access
2. Create CompanyUserSearchForm and MigrateCompanyUserForm
3. Load search params from query string
4. Execute search with joins (user, role tables)
5. Pagination (50 per page)
6. Default sorting: users.created_at DESC

Response:
- Success: HTML view with users list and migration form
- Data: `pageName`, `searchModel`, `dataProvider`, `migrateModel`
- Error: NotFoundHttpException if company not found

Available sorting:
- `email`, `name` (CONCAT), `role`, `language`, `timezone`, `status`, `updated_at`

---

### 2. Create — Create User

| Param | Value |
|------|-------|
| URL | `/company-users/create?company_uuid={company_uuid}` |
| Methods | GET, POST, AJAX |
| Description | Create a new company user with role and personal data |

URL Parameters:
- `company_uuid` (string, required)

GET `/company-users/create`
- Purpose: Render creation form

AJAX Validation
- Response: JSON validation results

POST `/company-users/create`
- Purpose: Process user creation

Input (SaveCompanyUserForm bodyParams):
- Fields include email, role, name, language, timezone, status, and settings (see form for details)

Responses:
- AJAX: JSON
- Success: Redirect to index with success notification
- Error: HTML view with form and errors

---

### 3. Edit — Edit User

| Param | Value |
|------|-------|
| URL | `/company-users/edit?company_uuid={company_uuid}&user_uuid={user_uuid}` |
| Methods | GET, POST, AJAX |
| Description | Edit an existing company user |

URL Parameters:
- `company_uuid` (string, required)
- `user_uuid` (string, required)

Behavior:
- Load existing user-company relation, role, and settings
- AJAX validation supported

Responses:
- Success: HTML view with updated data and success notification
- Error: HTML view with validation errors

---

### 4. Delete — Remove User From Company

| Param | Value |
|------|-------|
| URL | `/company-users/delete?company_uuid={company_uuid}&user_uuid={user_uuid}` |
| Method | POST |
| Description | Remove a user from the company (soft or hard depending on business rules) |

Responses:
- JSON with success/error

---

### 5. Migrate — Migrate User Between Companies

| Param | Value |
|------|-------|
| URL | `/company-users/migrate` |
| Method | POST |
| Description | Migrate a user from one company to another with role mapping |

Input (MigrateCompanyUserForm):
- `source_company_uuid`, `target_company_uuid`, `user_uuid`, optional role mapping

Responses:
- JSON with migration result and messages

---

## Form Classes

### SaveCompanyUserForm
- Validates and persists user-company relation, role, personal settings
- Scenario-based validation for create/edit

### CompanyUserSearchForm
- Search filters and sorting definitions (see fields above)

### MigrateCompanyUserForm
- Validates source/target companies and user relation
- Handles migration, conflict checks, and role assignment

---

## Security Features
- Company-based access control on all actions
- UUID validation for company and user UUIDs
- Role-based permissions for managing users
- CSRF protection for POST operations

---

## Usage Examples

Create user
```http
POST /company-users/create?company_uuid=c-uuid
Content-Type: application/x-www-form-urlencoded

email=john@example.com&role=manager&language=en&timezone=UTC&status=1
```

Migrate user
```http
POST /company-users/migrate
Content-Type: application/json

{
  "source_company_uuid": "c-uuid-1",
  "target_company_uuid": "c-uuid-2",
  "user_uuid": "u-uuid"
}
```

