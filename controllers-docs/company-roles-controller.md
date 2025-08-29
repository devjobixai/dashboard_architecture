# CompanyRolesController Documentation

## Overview

CompanyRolesController manages user roles within a company in the Jobix Dashboard administrative panel. The controller provides full CRUD functionality for managing role hierarchy with support for permissions, parent-child relationships, and integration with the RBAC authorization system.

**Base Path:** `/company-roles`  
**Controller Type:** WebController (Admin Application)  
**Namespace:** `apps\admin\controllers\CompanyRolesController`

---

## Endpoints

### 1. Index - Company Roles List

| Parameter | Value |
|----------|----------|
| **URL** | `/company-roles/index?company_uuid={company_uuid}` |
| **Method** | GET |
| **Description** | Display list of all company roles with search and filtering |

**URL Parameters:**
- `company_uuid` (string, required) - Company UUID

**Input Data (CompanyRoleSearchForm queryParams):**

| Field | Type | Required | Validation | Description |
|-------|------|----------|------------|-------------|
| `company_uuid` | string | ✓ | required, UUID, exists | Company UUID |
| `parent` | string | ✗ | string | Filter by parent role |
| `name` | string | ✗ | string | Search by role name |
| `slug` | string | ✗ | string | Search by role slug |
| `status` | integer | ✗ | integer | Filter by status (active/inactive) |
| `description` | string | ✗ | string | Search by role description |
| `updated_at` | array | ✗ | timestamp array validation | Filter by update date [start, end] |

**Business Logic:**
1. Validate company_uuid and check user access to company
2. Create CompanyRoleSearchForm for filtering
3. Load search parameters from query string
4. Execute search with company-based isolation
5. Pagination (50 elements per page)
6. Default sorting by created_at DESC

**Response:**
- **Success:** HTML view with roles list
- **Data:** `pageName`, `searchModel`, `dataProvider`
- **Error:** NotFoundHttpException if company not found

**Available Sorting:**
- `parent`, `name`, `slug`, `status`, `updated_at`

---

### 2. Create - Create New Role

| Parameter | Value |
|----------|----------|
| **URL** | `/company-roles/create?company_uuid={company_uuid}` |
| **Methods** | GET, POST, AJAX |
| **Description** | Create new company role with permissions configuration |

**URL Parameters:**
- `company_uuid` (string, required) - Company UUID

#### GET `/company-roles/create`
**Purpose:** Display creation form

#### AJAX Validation
**Response Format:** JSON with validation results

#### POST `/company-roles/create`
**Purpose:** Process role creation

**Input Data (SaveCompanyRolesForm bodyParams):**

| Field | Type | Required | Validation | Description |
|-------|------|----------|------------|-------------|
| `company_uuid` | string | ✓ | required, UUID, exists | Company UUID |
| `parent_uuid` | string | ✗ | UUID, exists within company | Parent role UUID |
| `name` | string | ✓ | required, string, max:64, unique within company | Role name |
| `slug` | string | ✓ | required, string, max:64, unique within company | Role slug |
| `description` | string | ✗ | string, max:255 | Role description |
| `permissions` | array | ✗ | safe | Array of permissions for the role |

**Detailed Validation SaveCompanyRolesForm (CREATE scenario):**
```php
// Required fields
[['uuid', 'company_uuid', 'name', 'slug', 'status'], 'required']

// String validation
[['name', 'slug'], 'string', 'max' => 64]
[['description'], 'string', 'max' => 255]

// UUID validation
[['uuid', 'company_uuid', 'parent_uuid'], UuidHelper::class]

// Uniqueness validation within company
[['name'], 'unique', 'targetClass' => CompanyRolesAdvanced::class,
 'filter' => function($query) {
     return $query->whereCompanyId($this->company->id);
 }]

[['slug'], 'unique', 'targetClass' => CompanyRolesAdvanced::class,
 'filter' => function($query) {
     return $query->whereCompanyId($this->company->id);
 }]

// Parent role validation
[['parent_uuid'], 'exist', 'targetClass' => CompanyRolesAdvanced::class,
 'filter' => function($query) {
     return $query->whereCompanyId($this->company->id);
 }]

// Company validation
[['company_uuid'], 'exist', 'targetClass' => CompaniesAdvanced::class,
 'filter' => function($query) {
     return $query->whereUserId($this->selectedUserCompany->user_id);
 }]
```

**Creation Business Logic:**
1. Validate form with CREATE scenario
2. Create new CompanyRolesAdvanced model
3. Set basic attributes (name, slug, description, parent_id)
4. Generate UUID for new role
5. Save role to database
6. Process permissions through _processPermissions()
7. Integration with AuthManager for RBAC

**Responses:**
- **AJAX:** JSON validation results
- **Success:** Redirect to `/company-roles/update?uuid={role_uuid}&company_uuid={company_uuid}`
- **Validation Error:** HTML view with form and errors
- **GET:** HTML view with empty form

---

### 3. Update - Edit Role

| Parameter | Value |
|----------|----------|
| **URL** | `/company-roles/update?uuid={role_uuid}&company_uuid={company_uuid}` |
| **Methods** | GET, POST, AJAX |
| **Description** | Edit existing company role |

**URL Parameters:**
- `uuid` (string, required) - Role UUID
- `company_uuid` (string, required) - Company UUID

#### GET `/company-roles/update`
**Purpose:** Display edit form

**Behavior:**
- Automatic role data loading through `populate()`
- Load current role permissions

#### AJAX Validation
**Response Format:** JSON with validation results

#### POST `/company-roles/update`
**Purpose:** Process role update

**Input Data (SaveCompanyRolesForm bodyParams):**

| Field | Type | Required | Validation | Description |
|-------|------|----------|------------|-------------|
| `uuid` | string | ✓ | required, UUID, exists | Role UUID |
| `company_uuid` | string | ✓ | required, UUID, exists | Company UUID |
| `parent_uuid` | string | ✗ | UUID, exists within company | Parent role UUID |
| `name` | string | ✓ | required, string, max:64, unique within company | Role name |
| `slug` | string | ✓ | required, string, max:64, unique within company | Role slug |
| `description` | string | ✗ | string, max:255 | Role description |
| `permissions` | array | ✗ | safe | Array of permissions for the role |

**Detailed Validation SaveCompanyRolesForm (UPDATE scenario):**
```php
// UPDATE scenario includes all fields from CREATE plus uuid
// Uniqueness validation excludes current record
[['name'], 'unique', 'targetClass' => CompanyRolesAdvanced::class,
 'filter' => function($query) {
     return $query->whereCompanyId($this->company->id)
                  ->andWhere(["!=", "uuid", $this->uuid]);
 }]

[['slug'], 'unique', 'targetClass' => CompanyRolesAdvanced::class,
 'filter' => function($query) {
     return $query->whereCompanyId($this->company->id)
                  ->andWhere(["!=", "uuid", $this->uuid]);
 }]

// Role existence validation
[['uuid'], 'exist', 'targetClass' => CompanyRolesAdvanced::class]
```

**Data Population Logic:**
```php
public function populate(): static
{
    if ($this->uuid && $this->company_uuid) {
        $role = $this->getRole(); // CompanyRolesAdvanced model
        if ($role) {
            $this->attributes = $role->attributes;
            $this->parent_uuid = $role->parent?->uuid;
            // Load current permissions from AuthManager
        }
    }
    return $this;
}
```

**Responses:**
- **AJAX:** JSON validation results
- **Success:** HTML view with updated form and success notification
- **Error:** HTML view with form and errors
- **GET:** HTML view with populated form

---

### 4. Delete - Delete Role

| Parameter | Value |
|----------|----------|
| **URL** | `/company-roles/delete?uuid={role_uuid}&company_uuid={company_uuid}` |
| **Method** | POST |
| **Description** | Delete company role |

**URL Parameters:**
- `uuid` (string, required) - UUID of role to delete
- `company_uuid` (string, required) - Company UUID

**Business Logic:**
1. Create SaveCompanyRolesForm with SCENARIO_DELETE
2. Set uuid and company_uuid
3. Call form's delete() method
4. Soft delete role (set deleted_at)
5. Remove related permissions from AuthManager
6. Handle child roles (if any)

**Delete Operation Flow:**
```php
$model = new SaveCompanyRolesForm();
$model->scenario = SaveCompanyRolesForm::SCENARIO_DELETE;
$model->uuid = $this->request->get('uuid');
$model->company_uuid = $this->request->get('company_uuid');
$model->delete();
```

**Responses:**
- **Success:** Redirect to `/company-roles/index?company_uuid={company_uuid}`
- **Error:** Redirect with error notification
- **Not Found:** Error if role doesn't exist or no access

---

### 5. Permissions - Get Role Permissions

| Parameter | Value |
|----------|----------|
| **URL** | `/company-roles/permissions?role_name={role_name}` |
| **Method** | GET |
| **Description** | API endpoint to get list of permissions for specific role |

**URL Parameters:**
- `role_name` (string, required) - Role name

**Business Logic:**
1. Get AuthManager instance
2. Call getPermissionsByRole() to get permissions
3. Filter only permission items (TYPE_PERMISSION)
4. Return array of permission names

**Response Format:**
```json
[
    "permission_name_1",
    "permission_name_2",
    "permission_name_3"
]
```

**Responses:**
- **Success:** JSON array with permission names
- **Empty role_name:** Empty array `[]`
- **Invalid role:** Empty array `[]`

---

## Form Classes

### 1. SaveCompanyRolesForm

**File:** `src/apps/admin/forms/company_roles/SaveCompanyRolesForm.php`  
**Extends:** `customs\Model` (544 lines)

**Scenarios:**
- `SCENARIO_POPULATE` - load existing role data
- `SCENARIO_CREATE` - create new role
- `SCENARIO_UPDATE` - edit existing role
- `SCENARIO_DELETE` - delete role

**Public Properties:**
```php
public mixed $uuid = null;                // Role UUID (required for UPDATE/DELETE)
public mixed $company_uuid = null;        // Company UUID (required)
public mixed $parent_uuid = null;         // Parent role UUID
public mixed $name = null;                // Role name (required)
public mixed $slug = null;                // Role slug (required)
public mixed $description = null;         // Role description
public mixed $status = true;              // Activity status (default: true)
public mixed $permissions = [];           // Array of role permissions
```

**Private Properties:**
```php
private CompaniesAdvanced|null $_company = null;           // Company model
private CompanyRolesAdvanced|null $_parentRole = null;     // Parent role
private CompanyRolesAdvanced|null $_role = null;           // Current role
```

**Scenario Fields:**
- **POPULATE:** `['uuid', 'company_uuid']`
- **CREATE:** `['company_uuid', 'parent_uuid', 'name', 'slug', 'description', 'permissions']`
- **UPDATE:** `['uuid', 'company_uuid', 'parent_uuid', 'name', 'slug', 'description', 'permissions']`
- **DELETE:** `['uuid', 'company_uuid']`

**Key Methods:**
- `populate()` - load existing role data
- `create()` - create new role with permission processing
- `update()` - update existing role with permission processing
- `delete()` - delete role and related permissions
- `getPermissionsData()` - get permissions structure for UI
- `_processPermissions()` - process permissions through AuthManager
- `getRolesDropdownItems()` - dropdown list of roles for UI

**RBAC Integration Features:**
- Integration with Yii2 AuthManager for permission management
- Automatic permission synchronization when creating/updating roles
- Support for role hierarchy with permission inheritance
- Permission parsing with lock mechanism for protected permissions

**Transaction Safety:**
- Database transactions for create/update operations
- Rollback on AuthManager integration failures
- Error handling with detailed messages

---

### 2. CompanyRoleSearchForm

**File:** `src/apps/admin/forms/company_roles/CompanyRoleSearchForm.php`  
**Extends:** `customs\Model` (159 lines)

**Public Properties:**
```php
public mixed $company_uuid = null;    // Company UUID (required)
public mixed $parent = null;          // Filter by parent role
public mixed $name = null;            // Search by role name
public mixed $slug = null;            // Search by role slug
public mixed $status = null;          // Filter by status
public mixed $description = null;     // Search by role description
public mixed $updated_at = null;      // Filter by update date (array)
```

**Search Features:**
- **Company Isolation:** Automatic filtering by selectedUserCompany
- **Hierarchical Search:** Search by parent roles
- **Text Search:** name, slug, description fields
- **Status Filtering:** active/inactive roles
- **Date Range Filtering:** updated_at with timestamp validation
- **Not Deleted Default:** Automatic exclusion of deleted roles

**Validation Rules:**
```php
[['company_uuid'], 'required']
[['parent', 'name', 'slug', 'language', 'timezone'], 'string']
[['status'], 'integer']
[['updated_at'], 'validateUpdatedAt'] // Custom timestamp array validation
[['company_uuid'], 'exist', 'targetClass' => CompaniesAdvanced::class,
 'filter' => function($query) {
     return $query->whereUserId($this->selectedUserCompany->user_id);
 }]
```

**Pagination:**
- Default page size: 50 elements
- Custom pagination support

**Sorting Options:**
```php
'parent' => SORT_ASC/DESC (parent_id field)
'name' => SORT_ASC/DESC
'slug' => SORT_ASC/DESC  
'status' => SORT_ASC/DESC
'updated_at' => SORT_ASC/DESC (default DESC)
```

**Static Methods:**
- `attributeLabels()` - localized field names

---

## Business Logic

### 1. Role Creation Flow
```
1. Display creation form with company context (GET)
2. Load available parent roles for dropdown
3. Validate role data (name, slug uniqueness within company)
4. Create CompanyRolesAdvanced record with generated UUID
5. Process permissions through AuthManager integration
6. Set up parent-child relationships if parent_uuid provided
7. Commit transaction or rollback on failure
8. Redirect to update page for further configuration
```

### 2. Role Update Flow
```
1. Load existing role data and populate form (GET)
2. Load current role permissions from AuthManager
3. Display form with current values and permissions
4. Validate updated data (excluding current record from uniqueness)
5. Update CompanyRolesAdvanced record
6. Process permission changes through AuthManager
7. Handle parent role changes and inheritance
8. Show success notification
```

### 3. Role Search Flow
```
1. Validate company_uuid and user access
2. Load search parameters from query
3. Apply company-based filtering automatically
4. Apply text searches for name, slug, description
5. Apply parent role filtering
6. Apply status filtering or "not deleted" default
7. Apply date range filtering with timestamp validation
8. Apply sorting and pagination (50 elements)
9. Return ActiveDataProvider
```

### 4. Role Delete Flow
```
1. Validate role existence and company access
2. Check for dependent child roles
3. Remove role permissions from AuthManager
4. Perform soft delete (set deleted_at timestamp)
5. Handle child role reassignment if needed
6. Redirect to index with success message
```

### 5. Permissions Management Flow
```
1. Parse available permissions from AuthManager rules
2. Identify locked/protected permissions
3. Load current role permissions
4. Handle parent role permission inheritance
5. Process permission changes through AuthManager
6. Maintain permission hierarchy consistency
```

---

## Security Features

### 1. Company-based Access Control
- All operations isolated by selectedUserCompany
- Uniqueness validation within company scope
- Search results limited to user's company
- Parent role validation only within the same company

### 2. RBAC Integration Security
- Secure integration with Yii2 AuthManager
- Permission inheritance through parent roles
- Protected permissions with lock mechanism
- Consistent authorization rule application

### 3. UUID Validation
- UuidHelper validation for all UUID operations
- Existence checking in CompanyRolesAdvanced
- Secure UUID parameter handling
- Cross-reference validation between role and company

### 4. Input Validation
- String length limits for all text fields
- Uniqueness validation within company scope
- Parent role circular reference prevention
- XSS protection through Yii validation

### 5. Hierarchical Security
- Parent-child relationship validation
- Circular reference detection
- Permission inheritance consistency
- Access control through company boundaries

---

## Integration Points

### Dependencies
- `CompanyRolesAdvanced` - main company roles model
- `CompaniesAdvanced` - companies model for access control
- `AuthManager` - Yii2 RBAC component for permission management
- `UuidHelper` - UUID validation and generation
- `TimeHelper` - timestamp validation for search

### RBAC Integration
- Yii2 AuthManager for permission management
- Role-based access control with inheritance
- Custom AuthRuleAbstract integration
- Permission parsing and validation

### Related Models
- `CompanyUsersAdvanced` - relationship with users
- Parent-child role relationships
- Company models for access control

---

## Usage Examples

### Create role with permissions
```http
POST /company-roles/create?company_uuid=550e8400-e29b-41d4-a716-446655440000
Content-Type: application/x-www-form-urlencoded

SaveCompanyRolesForm[name]=Manager&SaveCompanyRolesForm[slug]=manager&SaveCompanyRolesForm[description]=Team manager role&SaveCompanyRolesForm[parent_uuid]=550e8400-e29b-41d4-a716-446655440001&SaveCompanyRolesForm[permissions][]=user.create&SaveCompanyRolesForm[permissions][]=user.update
```

### Search roles with filtering
```http
GET /company-roles/index?company_uuid=550e8400-e29b-41d4-a716-446655440000&name=manager&status=1&parent=admin
```

### Get role permissions
```http
GET /company-roles/permissions?role_name=manager
```

### Update role
```http
POST /company-roles/update?uuid=550e8400-e29b-41d4-a716-446655440002&company_uuid=550e8400-e29b-41d4-a716-446655440000
Content-Type: application/x-www-form-urlencoded

SaveCompanyRolesForm[name]=Senior Manager&SaveCompanyRolesForm[description]=Updated description&SaveCompanyRolesForm[permissions][]=user.create&SaveCompanyRolesForm[permissions][]=user.update&SaveCompanyRolesForm[permissions][]=user.delete
```

---

## Performance Considerations

### RBAC Integration
- AuthManager operations can be resource-intensive
- Permission inheritance calculations for complex hierarchies
- Caching considerations for frequently accessed permissions

### Database Optimization
- Company-based indexing for all queries
- Parent-child relationship queries optimization
- Uniqueness validation performance within company scope
- Soft delete queries with proper indexing

### Search Performance
- Text searches can be resource-intensive
- Parent role filtering requires JOIN operations
- Date range queries with timestamp validation

---

## Error Handling

### Common Errors
- **"User not found"** - Invalid role UUID or access denied
- **"Parent role not found"** - Invalid parent_uuid or cross-company reference
- **"Company not found"** - Invalid company_uuid or access denied
- **Uniqueness violations** - Duplicate name or slug within company

### RBAC Integration Errors
- AuthManager connection failures
- Permission assignment errors
- Circular role dependency detection
- Permission inheritance conflicts

### Hierarchy Errors
- Parent role circular reference attempts
- Cross-company parent role assignments
- Permission inheritance conflicts

### HTTP Status Codes
- **200** - Success responses
- **404** - NotFoundHttpException for invalid company_uuid
- **500** - System errors with user-friendly messages
- **Redirect** - Success operations redirect to appropriate pages

---

*CompanyRolesController is a key component of the authorization system with full RBAC support, hierarchical roles, company isolation, and comprehensive permission management capabilities.*