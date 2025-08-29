# FlowsController (Admin) Documentation

## Overview

FlowsController manages automated workflows (flows) in the Jobix Dashboard admin panel. It is the key controller for creating, editing, and managing a company's business processes.

Base path: `/flows`  
Controller type: WebController (Admin Application)  
Namespace: `apps\\admin\\controllers\\FlowsController`

---

### 4. Builder — Visual Flow Builder

| Param | Value |
|------|-------|
| URL | `/flows/builder?uuid={flow_uuid}` |
| Method | GET |
| Description | Visual interface to build workflows |

URL Parameters:
- `uuid` (string) — flow UUID

Business logic:
1. Use SaveFlowForm with UPDATE scenario
2. Load existing flow data
3. Render a specialized builder view

Features:
- Specialized UI for drag-and-drop flow building
- Visual representation of business logic
- Integration with the nodular system

Responses:
- Success: HTML view with the flow builder interface
- Not Found: 404 NotFoundHttpException

---

### 5. Delete — Soft Delete a Flow

| Param | Value |
|------|-------|
| URL | `/flows/delete?uuid={flow_uuid}` |
| Method | POST |
| Description | Soft delete a workflow |

URL Parameters:
- `uuid` (string) — flow UUID to delete

Input (DeleteForm):
- UUID via GET param

Business logic:
- Uses a generic DeleteForm widget
- Soft delete by updating status
- Sets STATUS_DELETED

DeleteForm Configuration:
```php
new DeleteForm([
    'entityName' => 'flow',
    'targetClass' => CompanyFlowsAdvanced::class,
    'condition' => ['uuid' => $uuid],
    'softDelete' => true,
    'updateFields' => [
        'status' => CompanyPhoneNumbersAdvanced::STATUS_DELETED
    ]
])
```

Response:
- Format: JSON operation result
- Success: `{\"success\": true, \"message\": \"Flow deleted successfully\"}`
- Error: `{\"success\": false, \"message\": \"Error message\"}`

---

### 6. Copy — Duplicate a Flow

| Param | Value |
|------|-------|
| URL | `/flows/copy?uuid={flow_uuid}` |
| Method | POST |
| Description | Create a copy of an existing workflow |

URL Parameters:
- `uuid` (string) — flow UUID to copy

Business logic:
- Uses a generic CopyForm widget
- Generate a new UUID
- Append "_copy" to the name
- Create with INACTIVE status

CopyForm Configuration:
```php
new CopyForm([
    'entityName' => 'Flow',
    'targetClass' => CompanyFlowsAdvanced::class,
    'condition' => ['uuid' => $uuid],
    'updateFields' => [
        'uuid' => UuidHelper::generate(),
        'name' => fn($value) => $value . '_copy',
        'status' => CompanyFlowsAdvanced::STATUS_INACTIVE
    ]
])
```

Response:
- Format: JSON copy result
- Success: `{\"success\": true, \"new_uuid\": \"generated-uuid\", \"message\": \"Flow copied\"}`
- Error: `{\"success\": false, \"message\": \"Error message\"}`

---

## Form Classes

### 1. SaveFlowForm

File: `src/apps/admin/forms/flows/SaveFlowForm.php`  
**Extends:** `customs\\Model`

**Scenarios:**
- `SCENARIO_CREATE` — create a new flow
- `SCENARIO_UPDATE` — update an existing flow

Public properties:
```php
public mixed $uuid = null;           // Flow UUID (required for UPDATE)
public mixed $name = null;           // Flow name
public mixed $description = null;    // Flow description
public bool $status = true;          // Active status
```

Private properties:
```php
private ?CompanyFlowsAdvanced $_flowModel = null; // Cached flow model
```

Scenario Fields:
- CREATE: `['name', 'description', 'status']`
- UPDATE: `['uuid', 'name', 'description', 'status']`

Attribute Labels:
```php
'name' => 'Name'
'status' => 'Status' 
'description' => 'Description'
```

Key methods:
- `populate()` — load existing data
- `create()` — create a new flow
- `update()` — update an existing flow
- `getFlowModel()` — get flow model

Transaction Safety:
- Database transactions for create/update operations
- Automatic rollback on errors
- Error handling with user-friendly messages

---

### 2. SearchFlowForm

File: `src/apps/admin/forms/flows/SearchFlowForm.php`  
**Extends:** `customs\\Model`

Public properties:
```php
public mixed $name = null;           // Search by name
public mixed $description = null;    // Search by description  
public mixed $status = null;         // Status filter
public mixed $updated_at = null;     // Update date range
```

Private properties:
```php
protected null|int $_companyId = null; // Company ID for filtering
```

Search Features:
- Name Search: ILIKE search by name
- Description Search: ILIKE search by description
- Status Filter: Exact match or "not deleted" by default
- Date Range: Carbon-based parsing with start/end of day

Pagination:
- Default page size: 25 items
- Supports custom page sizes

Sorting Options:
```php
'uuid' => SORT_ASC/DESC
'name' => SORT_ASC/DESC  
'description' => SORT_ASC/DESC
'updated_at' => SORT_ASC/DESC (default DESC)
```

Static methods:
- `statusesDropdown()` — UI dropdown with labels

Company Filtering:
- Automatic filter by selectedUserCompany
- Security via company-based access control

---

## Business Logic

### 1. Flow Creation Flow
```
1. Display creation form (GET)
2. AJAX validation (optional)  
3. Form validation in CREATE scenario
4. Database transaction start
5. Generate UUID and set company_id
6. Create with INACTIVE status
7. Transaction commit/rollback
8. Redirect to edit page
```

### 2. Flow Update Flow
```
1. Load existing flow data (GET)
2. Validate UUID and company access
3. AJAX validation (optional)
4. Form validation in UPDATE scenario  
5. Database transaction start
6. Update name and description
7. Transaction commit/rollback
8. Success notification
```

### 3. Search and Filter Flow
```
1. Load search parameters from query
2. Apply ILIKE searches for text fields
3. Apply status filter or "not deleted" default
4. Parse date range with Carbon
5. Apply sorting and pagination
6. Return ActiveDataProvider
```

### 4. Copy Flow
```
1. Find source flow by UUID
2. Generate new UUID for copy
3. Append "_copy" to name
4. Set INACTIVE status
5. Create new record
6. Return success with new UUID
```

---

## Security Features

### 1. Company-based Access Control
- All operations are isolated by company_id
- UUID validation with company filtering
- selectedUserCompany integration

### 2. UUID Validation
- UuidHelper validation for format
- Existence checking within the company
- Secure UUID generation for new records

### 3. Input Validation
- String length limits
- Trim filters for text fields
- Boolean validation for status
- XSS protection via Yii validation

### 4. Transaction Safety
- Database transactions for critical operations
- Automatic rollback on errors
- Error logging for system errors

---

## Integration Points

### Dependencies
- `CompanyFlowsAdvanced` — flows core model
- `UuidHelper` — UUID generation and validation
- `NotificationHelper` — user notifications
- Generic `DeleteForm`/`CopyForm` widgets
- `ActiveForm` — AJAX validation

### Related Models
- `CompanyFlowsQuery` — query builder with custom scopes
- `CompanyFlows` (basic) — base model

### External Integrations
- Carbon — date/time parsing
- Nodular system — for flow execution
- Visual flow builder UI

---

## Usage Examples

### Create a flow
```http
POST /flows/create
Content-Type: application/x-www-form-urlencoded

name=Customer Registration Flow&description=Automated customer onboarding process&status=1
```

### Search flows
```http
GET /flows/index?name=registration&status=active&updated_at=2023-01-01 - 2023-12-31
```

### AJAX validation
```http
POST /flows/create
X-Requested-With: XMLHttpRequest
Content-Type: application/x-www-form-urlencoded

name=Test&description=Test flow

Response:
{
    "name": [],
    "description": []
}
```

### Copy a flow
```http
POST /flows/copy?uuid=550e8400-e29b-41d4-a716-446655440000

Response:
{
    "success": true,
    "new_uuid": "550e8400-e29b-41d4-a716-446655440001", 
    "message": "Flow copied successfully"
}
```

### Delete a flow
```http
POST /flows/delete?uuid=550e8400-e29b-41d4-a716-446655440000

Response:
{
    "success": true,
    "message": "Flow deleted successfully"
}
```

---

## Performance Considerations

### Database Optimization
- ILIKE searches can be slow on large datasets
- Indexes on company_id, name, status
- Pagination for large lists

### UI Performance  
- AJAX validation for responsive UX
- Lazy loading for the builder interface
- Efficient DataProvider usage

---

## Error Handling

### Common Errors
- "Flow not found" — Invalid UUID or no access
- "System error, please contact with Administrator!" — Database or system errors
- Validation errors — Form validation failures

### HTTP Status Codes
- 200 — Success responses
- 404 — Flow not found
- 500 — System errors (JSON response for AJAX)

---

FlowsController is the central component for managing business processes with a visual builder and advanced search capabilities.

## Endpoints

### 1. Index — Flows List

| Param | Value |
|------|-------|
| URL | `/flows/index` |
| Method | GET |
| Description | Display all company flows with search and filtering |

Input (SearchFlowForm queryParams):

| Field | Type | Required | Validation | Description |
|------|------|----------|------------|-------------|
| `name` | string | ✗ | string | Search by name (ILIKE) |
| `description` | string | ✗ | string | Search by description (ILIKE) |
| `status` | string | ✗ | string | Filter by status |
| `updated_at` | string | ✗ | string (date range) | Update date range (format: "YYYY-MM-DD - YYYY-MM-DD") |

Statuses for filtering:
- `active` — Active flows
- `inactive` — Inactive flows  
- `deleted` — Deleted flows

Business logic:
1. Instantiate SaveFlowForm for quick creation
2. Instantiate SearchFlowForm for filtering
3. Load search params from query string
4. Execute search with pagination (25 per page)
5. Sort by updated_at (DESC by default)

Response:
- Success: HTML view with list of flows
- Data: `pageName`, `searchModel`, `saveModel`, `dataProvider`

Available sorting:
- `uuid` — by UUID
- `name` — by name
- `description` — by description
- `updated_at` — by update date (default)

---

### 2. Create — Create a Flow

| Param | Value |
|------|-------|
| URL | `/flows/create` |
| Methods | GET, POST, AJAX |
| Description | Create a new workflow |

#### GET `/flows/create`
Purpose: Render the creation form

#### AJAX Validation
Purpose: Real-time form validation

Response Format: JSON with validation results

#### POST `/flows/create`
Purpose: Handle flow creation

Input (SaveFlowForm bodyParams):

| Field | Type | Required | Validation | Description |
|------|------|----------|------------|-------------|
| `name` | string | ✓ | required, string, min:3, max:255, trim | Flow name |
| `description` | string | ✗ | string, max:1024 | Flow description |
| `status` | boolean | ✗ | boolean | Active status (default: true) |

SaveFlowForm validation (CREATE scenario):
```php
// Required fields
[['name'], 'required']

// String validation
[['name', 'description'], 'filter', 'filter' => 'trim', 'skipOnEmpty' => true]
[['name'], 'string', 'min' => 3, 'max' => 255]
[['description'], 'string', 'max' => 1024]

// Boolean validation
[['status'], 'boolean']
```

Creation business logic:
1. Validate form in CREATE scenario
2. Database transaction
3. Generate UUID for the new flow
4. Set current company's `company_id`
5. Create with STATUS_INACTIVE (requires activation)
6. Save in CompanyFlowsAdvanced

Auto-set fields:
- `uuid` — generated UUID
- `company_id` — current company ID
- `status` — CompanyFlowsAdvanced::STATUS_INACTIVE

Responses:
- AJAX: JSON validation results
- Success: Redirect to `/flows/update/{new_flow_uuid}` with success notification
- Error: HTML view with form and errors
- GET: HTML view with empty form

---

### 3. Update — Edit a Flow

| Param | Value |
|------|-------|
| URL | `/flows/update?uuid={flow_uuid}` |
| Methods | GET, POST, AJAX |
| Description | Edit an existing workflow |

URL Parameters:
- `uuid` (string) — flow UUID

#### GET `/flows/update`
Purpose: Render the edit form

Behavior:
- If flow not found — NotFoundHttpException
- Otherwise — load flow data into the form

#### AJAX Validation
Response Format: JSON with validation results

#### POST `/flows/update`
Purpose: Handle flow update

Input (SaveFlowForm bodyParams):

| Field | Type | Required | Validation | Description |
|------|------|----------|------------|-------------|
| `uuid` | string | ✓ | required, UUID validation, exists check | Flow UUID |
| `name` | string | ✓ | required, string, min:3, max:255, trim | Flow name |
| `description` | string | ✗ | string, max:1024 | Flow description |
| `status` | boolean | ✗ | boolean | Active status |

SaveFlowForm validation (UPDATE scenario):
```php
// Required fields
[['uuid', 'name'], 'required']

// UUID validation
[['uuid'], UuidHelper::class]

// Existence check
[['uuid'], 'exist', 'targetClass' => CompanyFlowsAdvanced::class, 
 'filter' => function($query) {
     return $query->whereCompanyId($this->selectedUserCompany->id);
 }]

// String validation
[['name', 'description'], 'filter', 'filter' => 'trim', 'skipOnEmpty' => true]
[['name'], 'string', 'min' => 3, 'max' => 255]
[['description'], 'string', 'max' => 1024]

// Boolean validation
[['status'], 'boolean']
```

Update business logic:
1. Validate form in UPDATE scenario
2. Verify flow exists within the current company
3. Database transaction
4. Update name and description
5. Save changes in CompanyFlowsAdvanced

Security Features:
- UUID validation via UuidHelper
- Company-based access control
- Existence validation

Responses:
- AJAX: JSON validation results
- Success: HTML view with updated form and success notification
- Not Found: 404 NotFoundHttpException
- Error: HTML view with form and errors

---
