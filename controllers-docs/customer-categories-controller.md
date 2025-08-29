# CustomerCategoriesController Documentation

## Overview

CustomerCategoriesController manages customer categories within the company in the Jobix Dashboard administrative panel. The controller provides full CRUD functionality for managing customer categories with support for custom fields management, search and filtering capabilities, and integration with copy/delete widgets.

**Base Path:** `/customer-categories`  
**Controller Type:** WebController (Admin Application)  
**Namespace:** `apps\admin\controllers\CustomerCategoriesController`

---

## Endpoints

### 1. Index - Customer Categories List

| Parameter | Value |
|-----------|-------|
| **URL** | `/customer-categories/index` |
| **Method** | GET |
| **Description** | Display list of all customer categories with search and filtering |

**Input Data (SearchCustomerCategoryForm queryParams):**

| Field | Type | Required | Validation | Description |
|-------|------|----------|-----------|-------------|
| `name` | string | ✗ | string | Search by category name (ILIKE) |
| `description` | string | ✗ | string | Search by category description (ILIKE) |
| `status` | string | ✗ | string | Filter by status |
| `updated_at` | string | ✗ | string | Filter by update date (date range) |

**Business Logic:**
1. Create SearchCustomerCategoryForm for filtering
2. Load search parameters from query string
3. Execute search with company-based isolation
4. Apply ILIKE searches for text fields
5. Apply date range filtering with Carbon parsing
6. Pagination (25 elements per page)
7. Default sorting by updated_at DESC

**Response:**
- **Success:** HTML view with categories list
- **Data:** `pageName`, `searchModel`, `dataProvider`

**Available Sorting:**
- `uuid`, `name`, `description`, `updated_at`

---

### 2. Fields - Manage Category Fields

| Parameter | Value |
|-----------|-------|
| **URL** | `/customer-categories/fields?uuid={category_uuid}` |
| **Methods** | GET, POST |
| **Description** | Manage additional fields assigned to customer category |

**URL Parameters:**
- `uuid` (string, required) - UUID of the category

#### GET `/customer-categories/fields`
**Purpose:** Display fields management form

**Behavior:**
- Automatic data loading through `populate()`
- Load current assigned fields

#### POST `/customer-categories/fields`
**Purpose:** Save category fields configuration

**Input Data (SaveCustomerCategoryForm bodyParams):**

| Field | Type | Required | Validation | Description |
|-------|------|----------|-----------|-------------|
| `uuid` | string | ✓ | required, UUID, exists | Category UUID |
| `fields` | array | ✗ | each: integer | Array of field IDs |

**Detailed Validation SaveCustomerCategoryForm (FIELDS scenario):**
```php
// FIELDS scenario fields
[['uuid', 'fields'], 'required']

// UUID validation
[['uuid'], UuidHelper::class]

// Fields array validation
[['fields'], 'each', 'rule' => ['integer']]

// Category existence validation
[['uuid'], 'exist', 'targetClass' => CompanyCustomerCategoriesAdvanced::class,
 'filter' => function($query) {
     return $query->whereCompanyId($this->selectedUserCompany->id);
 }]
```

**Business Logic:**
1. Validate form with FIELDS scenario
2. Call `saveFields()` method to update category fields
3. Handle field assignments and removals
4. Update category model with new fields configuration

**Responses:**
- **Success:** Redirect to `/customer-categories/fields?uuid={category_uuid}` with success notification
- **Error:** HTML view with form and validation errors
- **GET:** HTML view with populated form
- **Not Found:** NotFoundHttpException if category not found

---

### 3. Create - Create New Category

| Parameter | Value |
|-----------|-------|
| **URL** | `/customer-categories/create` |
| **Methods** | GET, POST, AJAX |
| **Description** | Create new customer category |

#### GET `/customer-categories/create`
**Purpose:** Display creation form

#### AJAX Validation
**Response Format:** JSON with validation results

#### POST `/customer-categories/create`
**Purpose:** Process category creation

**Input Data (SaveCustomerCategoryForm bodyParams):**

| Field | Type | Required | Validation | Description |
|-------|------|----------|-----------|-------------|
| `name` | string | ✓ | required, string, min:3, max:255, trim | Category name |
| `description` | string | ✗ | string, max:1024, trim | Category description |
| `status` | boolean | ✗ | boolean | Category status |

**Detailed Validation SaveCustomerCategoryForm (CREATE scenario):**
```php
// Required fields
[['uuid', 'name'], 'required'] // uuid auto-generated

// String validation with trimming
[['name', 'description'], 'filter', 'filter' => 'trim']
[['name'], 'string', 'min' => 3, 'max' => 255]
[['description'], 'string', 'max' => 1024]

// Boolean validation
[['status'], 'boolean']

// UUID validation (auto-generated)
[['uuid'], UuidHelper::class]
```

**Business Logic:**
1. Validate form with CREATE scenario
2. Generate UUID for new category
3. Set company_id from selectedUserCompany
4. Create CompanyCustomerCategoriesAdvanced record
5. Set default status if not provided
6. Save category to database

**Responses:**
- **AJAX:** JSON validation results with clean output buffer
- **Success:** Redirect to `/customer-categories/update?uuid={category_uuid}` with success notification
- **Validation Error:** HTML view with form and errors
- **GET:** HTML view with empty form

---

### 4. Update - Edit Category

| Parameter | Value |
|-----------|-------|
| **URL** | `/customer-categories/update?uuid={category_uuid}` |
| **Methods** | GET, POST, AJAX |
| **Description** | Edit existing customer category |

**URL Parameters:**
- `uuid` (string, required) - UUID of the category

#### GET `/customer-categories/update`
**Purpose:** Display edit form

**Behavior:**
- Automatic data loading through `populate()`
- Load category attributes and current fields

#### AJAX Validation
**Response Format:** JSON with validation results

#### POST `/customer-categories/update`
**Purpose:** Process category update

**Input Data (SaveCustomerCategoryForm bodyParams):**

| Field | Type | Required | Validation | Description |
|-------|------|----------|-----------|-------------|
| `uuid` | string | ✓ | required, UUID, exists | Category UUID |
| `name` | string | ✓ | required, string, min:3, max:255, trim | Category name |
| `description` | string | ✗ | string, max:1024, trim | Category description |
| `status` | boolean | ✗ | boolean | Category status |

**Detailed Validation SaveCustomerCategoryForm (UPDATE scenario):**
```php
// UPDATE scenario includes all fields from CREATE plus uuid
// Same validation rules as CREATE scenario
// Category existence validation within company
[['uuid'], 'exist', 'targetClass' => CompanyCustomerCategoriesAdvanced::class,
 'filter' => function($query) {
     return $query->whereCompanyId($this->selectedUserCompany->id);
 }]
```

**Data Population Logic:**
```php
public function populate(): bool
{
    if ($this->uuid && $this->categoryModel) {
        $this->attributes = $this->categoryModel->attributes;
        $this->fields = $this->categoryModel->fields;
        return true;
    }
    return false;
}
```

**Responses:**
- **AJAX:** JSON validation results with clean output buffer
- **Success:** HTML view with updated form and success notification
- **Error:** HTML view with form and validation errors
- **GET:** HTML view with populated form
- **Not Found:** NotFoundHttpException if category not found

---

### 5. Delete - Delete Category

| Parameter | Value |
|-----------|-------|
| **URL** | `/customer-categories/delete?uuid={category_uuid}` |
| **Method** | POST |
| **Description** | Delete customer category using widget |

**URL Parameters:**
- `uuid` (string, required) - UUID of category to delete

**Business Logic:**
1. Use DeleteForm widget for standardized deletion
2. Configure entity name as 'customer category'
3. Set target class as CompanyCustomerCategoriesAdvanced
4. Use soft delete with status update
5. Update status to STATUS_DELETED

**Delete Widget Configuration:**
```php
$form = new DeleteForm([
    'entityName' => 'customer category',
    'targetClass' => CompanyCustomerCategoriesAdvanced::class,
    'condition' => ['uuid' => $this->request->get('uuid')],
    'softDelete' => true,
    'updateFields' => [
        'status' => CompanyCustomerCategoriesAdvanced::STATUS_DELETED
    ]
]);
```

**Responses:**
- **Success:** JSON response with success message
- **Error:** JSON response with error details
- **Not Found:** JSON response if category not found

---

### 6. Copy - Copy Category

| Parameter | Value |
|-----------|-------|
| **URL** | `/customer-categories/copy?uuid={category_uuid}` |
| **Method** | POST |
| **Description** | Copy customer category using widget |

**URL Parameters:**
- `uuid` (string, required) - UUID of category to copy

**Business Logic:**
1. Use CopyForm widget for standardized copying
2. Configure entity name as 'customer category'
3. Set target class as CompanyCustomerCategoriesAdvanced
4. Generate new UUID for copy
5. Append '_copy' to name for uniqueness

**Copy Widget Configuration:**
```php
$form = new CopyForm([
    'entityName' => 'customer category',
    'targetClass' => CompanyCustomerCategoriesAdvanced::class,
    'condition' => ['uuid' => $this->request->get('uuid')],
    'updateFields' => [
        'uuid' => UuidHelper::generate(),
        'name' => fn($value) => $value . '_copy'
    ]
]);
```

**Responses:**
- **Success:** JSON response with success message and new category UUID
- **Error:** JSON response with error details
- **Not Found:** JSON response if source category not found

---

## Form Classes

### 1. SaveCustomerCategoryForm

**File:** `src/apps/admin/forms/customer_categories/SaveCustomerCategoryForm.php`  
**Extends:** `customs\Model` (259 lines)

**Scenarios:**
- `SCENARIO_CREATE` - creating new category
- `SCENARIO_UPDATE` - editing existing category
- `SCENARIO_FIELDS` - managing category fields

**Public Properties:**
```php
public mixed $uuid = null;                // Category UUID (required for UPDATE/FIELDS)
public mixed $name = null;                // Category name (required)
public mixed $description = null;         // Category description
public bool $status = true;               // Category status (default: true)
public mixed $fields = [];                // Array of field IDs
```

**Private Properties:**
```php
private CompanyCustomerCategoriesAdvanced $_categoryModel;  // Category model
private array $_fieldsModels = [];                         // Fields models cache
```

**Scenario Fields:**
- **CREATE:** `['name', 'description', 'status']`
- **UPDATE:** `['uuid', 'name', 'description', 'status']`
- **FIELDS:** `['uuid', 'fields']`

**Key Methods:**
- `populate()` - load existing category data and assigned fields
- `create()` - create new category with company association
- `update()` - update existing category data
- `saveFields()` - update category fields assignments
- `getCategoryModel()` - get category model with lazy loading
- `getFieldsModels()` - get available fields for assignment
- `getFieldsCheckboxData()` - get fields data for checkbox UI

**Validation Features:**
- Name trimming and length validation (3-255 characters)
- Description length validation (max 1024 characters)
- UUID validation with company filtering
- Fields array validation with integer field IDs
- Category existence validation within company

**Company Isolation:**
- All operations filtered by selectedUserCompany
- Category existence validation within company
- Fields assignment respects company boundaries

---

### 2. SearchCustomerCategoryForm

**File:** `src/apps/admin/forms/customer_categories/SearchCustomerCategoryForm.php`  
**Extends:** `customs\Model` (115 lines)

**Public Properties:**
```php
public mixed $name = null;            // Name search filter
public mixed $description = null;     // Description search filter
public mixed $status = null;          // Status filter
public mixed $updated_at = null;      // Date range filter
```

**Search Features:**
- **ILIKE Searches:** name, description fields with case-insensitive matching
- **Status Filtering:** filter by category status or default to "not deleted"
- **Date Range Filtering:** updated_at with Carbon date parsing
- **Company Isolation:** automatic filtering by selectedUserCompany
- **Not Deleted Default:** automatic exclusion of deleted categories

**Validation Rules:**
```php
[['name', 'description', 'status', 'updated_at'], 'string']
```

**Search Logic:**
```php
if ($this->needToSearch('name')) {
    $query->ilikeName($this->name);
}

if ($this->needToSearch('description')) {
    $query->ilikeDescription($this->description);
}

if ($this->needToSearch('status')) {
    $query->whereStatus($this->status);
} else {
    $query->notDeleted();
}

// Date range filtering with Carbon
if (!$this->hasErrors('updated_at') && is_string($this->updated_at)) {
    $times = explode(' - ', $this->updated_at);
    $query->andWhere(['>=', 'company_customer_categories.updated_at', 
                     Carbon::parse($times[0])->startOfDay()->toDateTimeString()]);
    $query->andWhere(['<=', 'company_customer_categories.updated_at', 
                     Carbon::parse($times[1])->endOfDay()->toDateTimeString()]);
}
```

**Pagination:**
- Default page size: 25 elements
- Custom pagination support

**Sorting Options:**
```php
'uuid' => SORT_ASC/DESC
'name' => SORT_ASC/DESC
'description' => SORT_ASC/DESC
'updated_at' => SORT_ASC/DESC (default DESC)
```

**Static Methods:**
- `statusesDropdown()` - status options for UI dropdown

---

## Business Logic

### 1. Category Creation Flow
```
1. Display creation form (GET)
2. Validate category data (name, description)
3. Generate UUID for new category
4. Set company_id from selectedUserCompany
5. Create CompanyCustomerCategoriesAdvanced record
6. Set default status and timestamps
7. Save category to database
8. Redirect to update page for fields configuration
```

### 2. Category Update Flow
```
1. Load existing category data and populate form (GET)
2. Load current assigned fields
3. Display form with current values
4. Validate updated data with company filtering
5. Update CompanyCustomerCategoriesAdvanced record
6. Update timestamps
7. Show success notification
```

### 3. Fields Management Flow
```
1. Load existing category and current fields (GET)
2. Display fields selection interface
3. Validate field assignments (integer field IDs)
4. Update category fields relationship
5. Save fields configuration
6. Redirect with success message
```

### 4. Category Search Flow
```
1. Load search parameters from query
2. Apply company-based filtering automatically
3. Apply ILIKE searches for text fields
4. Apply status filtering or "not deleted" default
5. Apply date range filtering with Carbon parsing
6. Apply sorting and pagination (25 elements)
7. Return ActiveDataProvider
```

### 5. Copy/Delete Operations Flow
```
1. Use standardized widget components
2. Validate entity existence and permissions
3. Execute operation with proper configuration
4. Return JSON response with operation result
```

---

## Security Features

### 1. Company-based Access Control
- All operations isolated by selectedUserCompany
- Category existence validation within company
- Search results limited to company categories
- Fields assignment respects company boundaries

### 2. Input Validation
- String length limits for all text fields
- Name trimming to prevent whitespace issues
- UUID validation for all UUID operations
- Fields array validation with type checking
- XSS protection through Yii validation

### 3. Widget Integration Security
- DeleteForm widget with soft delete configuration
- CopyForm widget with proper field updates
- Entity name and target class validation
- Condition-based operation execution

### 4. Data Population Security
- Safe attribute assignment in populate()
- Model existence validation before operations
- Company filtering in all database queries
- Proper error handling for missing entities

---

## Integration Points

### Dependencies
- `CompanyCustomerCategoriesAdvanced` - main category model
- `FieldsAdvanced` - model for custom fields
- `UuidHelper` - UUID validation and generation
- `NotificationHelper` - user notifications
- `Carbon` - date parsing and manipulation
- `DeleteForm` widget - standardized deletion
- `CopyForm` widget - standardized copying

### Widget Integrations
- DeleteForm for soft delete operations
- CopyForm for category duplication
- Standard entity manipulation patterns
- JSON response formatting

### Related Models
- Customer models for category assignment
- Field models for custom field management
- Company models for access control

---

## Usage Examples

### Create category
```http
POST /customer-categories/create
Content-Type: application/x-www-form-urlencoded

SaveCustomerCategoryForm[name]=VIP Customers&SaveCustomerCategoryForm[description]=High priority customers&SaveCustomerCategoryForm[status]=1
```

### Search categories with filtering
```http
GET /customer-categories/index?name=VIP&status=1&updated_at=2024-01-01 - 2024-12-31
```

### Manage category fields
```http
POST /customer-categories/fields?uuid=550e8400-e29b-41d4-a716-446655440000
Content-Type: application/x-www-form-urlencoded

SaveCustomerCategoryForm[fields][]=1&SaveCustomerCategoryForm[fields][]=2&SaveCustomerCategoryForm[fields][]=3
```

### Update category
```http
POST /customer-categories/update?uuid=550e8400-e29b-41d4-a716-446655440000
Content-Type: application/x-www-form-urlencoded

SaveCustomerCategoryForm[name]=Premium Customers&SaveCustomerCategoryForm[description]=Updated description&SaveCustomerCategoryForm[status]=1
```

### Copy category
```http
POST /customer-categories/copy?uuid=550e8400-e29b-41d4-a716-446655440000
```

### Delete category
```http
POST /customer-categories/delete?uuid=550e8400-e29b-41d4-a716-446655440000
```

---

## Performance Considerations

### Database Optimization
- Company-based indexing for all queries
- ILIKE searches may be resource-intensive
- Date range queries with proper indexing
- Soft delete queries with status filtering

### Search Performance
- Text searches with ILIKE operations
- Date range parsing with Carbon overhead
- Pagination for large category lists
- Proper query optimization for joins

### Widget Operations
- JSON response overhead for copy/delete
- Widget validation and processing costs
- Database transaction management

---

## Error Handling

### Common Errors
- **"Customer Category not found"** - Invalid UUID or no access
- **Name validation errors** - Length or trimming issues
- **Description validation errors** - Length limit exceeded
- **Fields validation errors** - Invalid field IDs

### Widget Operation Errors
- Delete operation failures
- Copy operation failures
- Permission denied errors
- Entity not found errors

### HTTP Status Codes
- **200** - Success responses
- **404** - NotFoundHttpException for missing categories
- **500** - System errors with user-friendly messages
- **JSON** - Widget operations return JSON responses

---

*CustomerCategoriesController is a comprehensive component for managing customer categories with advanced field assignment capabilities, integrated search functionality, and standardized copy/delete operations through widget components.*