# ManagementController Documentation

## Overview

ManagementController handles management operations for customer field definitions in the Jobix Dashboard administrative panel. The controller provides functionality for managing custom fields that can be assigned to customers, with support for searching, creating, and updating field definitions through both HTML and AJAX interfaces.

**Base Path:** `/management`  
**Controller Type:** WebController (Admin Application)  
**Namespace:** `apps\admin\controllers\ManagementController`

---

## Endpoints

### 1. Customer Fields - Manage Customer Field Definitions

| Parameter | Value |
|-----------|-------|
| **URL** | `/management/customer-fields` |
| **Methods** | GET, POST (AJAX) |
| **Description** | Manage customer field definitions with search and batch operations |

#### GET `/management/customer-fields`
**Purpose:** Display customer fields management interface

**Input Data (CustomerFieldSearchForm queryParams):**

| Field | Type | Required | Validation | Description |
|-------|------|----------|-----------|-------------|
| `query` | string | ✗ | string | Search query for field label and slug (ILIKE) |

**Business Logic:**
1. Create CustomerFieldSearchForm for field searching
2. Load search parameters from query string
3. Create UpdateOrCreateCustomerFieldForm for field operations
4. Execute search with company-based isolation
5. Apply ILIKE searches on label and slug fields
6. Return HTML view with fields list and creation form

**Response:**
- **Success:** HTML view with customer fields management interface
- **Data:** `fields` (search results array), `formModel` (creation form)

#### POST `/management/customer-fields` (AJAX)
**Purpose:** Create or update customer field definitions

**Input Data (UpdateOrCreateCustomerFieldForm bodyParams):**

| Field | Type | Required | Validation | Description |
|-------|------|----------|-----------|-------------|
| `fields` | array | ✓ | FieldsValidator, company validation | Array of field definitions |

**Field Array Structure:**
Each element in the `fields` array should contain:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `uuid` | string | ✗ | Field UUID (for updates), empty for new fields |
| `name` | string | ✓ | Field label/display name |
| `slug` | string | ✓ | Field slug/identifier |
| `type` | string | ✓ | Field type (text, number, select, etc.) |
| `description` | string | ✗ | Field description |

**Detailed Validation:**
```php
[
    'fields', 
    FieldsValidator::class, 
    'selectedCompanyId' => $this->selectedUserCompany->id
]
```

**Business Logic:**
1. Validate AJAX request and form data loading
2. Set response format to JSON
3. Call form create() method for batch processing
4. Handle both create (new UUID) and update (existing UUID) operations
5. Use database transactions for data consistency
6. Return JSON response with operation results

**Create/Update Process:**
```php
foreach ($this->fields as $fieldData) {
    $field = null;
    
    if (!empty($fieldData['uuid'])) {
        // Update existing field
        $field = FieldsAdvanced::findOne(['uuid' => $fieldData['uuid']]);
    } else {
        // Create new field
        $field = new FieldsAdvanced();
        $field->uuid = UuidHelper::generate();
        $field->relation_id = $this->selectedUserCompany->id;
        $field->relation_name = CompaniesAdvanced::tableName();
        $field->status = FieldsAdvanced::STATUS_ACTIVE;
    }
    
    // Set field attributes
    $field->label = $fieldData['name'] ?? null;
    $field->type = $fieldData['type'];
    $field->slug = $fieldData['slug'];
    $field->description = $fieldData['description'] ?? null;
    
    $field->save();
}
```

**JSON Responses:**

**Success Response:**
```json
{
    "success": true,
    "message": "Fields were saved successfully!"
}
```

**Error Response:**
```json
{
    "success": false,
    "errors": {
        "field_name": ["Error message"],
        "toast_error": ["You have some errors, check the form."]
    }
}
```

---

## Form Classes

### 1. CustomerFieldSearchForm

**File:** `src/apps/admin/forms/management/CustomerFieldSearchForm.php`  
**Extends:** `customs\Model` (47 lines)

**Public Properties:**
```php
public ?string $query = null;  // Search query for field label and slug
```

**Search Features:**
- **ILIKE Search:** Searches both field label and slug with case-insensitive matching
- **Company Isolation:** Automatic filtering by selectedUserCompany
- **Field Selection:** Returns specific field attributes (uuid, label, slug, type, description, status)
- **Ordering:** Results ordered by field ID ascending
- **Array Return:** Returns raw array data instead of ActiveDataProvider

**Search Implementation:**
```php
public function search(): array
{
    $fieldsTableName = FieldsAdvanced::tableName();
    
    $query = FieldsAdvanced::find()
        ->select([
            "$fieldsTableName.uuid",
            "$fieldsTableName.label",
            "$fieldsTableName.slug",
            "$fieldsTableName.type",
            "$fieldsTableName.description",
            "$fieldsTableName.status",
        ])
        ->whereCompanyId($this->selectedUserCompany->id);
    
    if (!empty($this->query)) {
        $query->andWhere([
            'or',
            ['ilike', "$fieldsTableName.label", $this->query],
            ['ilike', "$fieldsTableName.slug", $this->query],
        ]);
    }
    
    return $query
        ->orderBy(["$fieldsTableName.id" => SORT_ASC])
        ->asArray()
        ->all();
}
```

---

### 2. UpdateOrCreateCustomerFieldForm

**File:** `src/apps/admin/forms/management/UpdateOrCreateCustomerFieldForm.php`  
**Extends:** `customs\Model` (82 lines)

**Public Properties:**
```php
public array $fields = [];  // Array of field definitions for batch processing
```

**Key Features:**
- **Batch Processing:** Handles multiple field operations in single request
- **Create/Update Logic:** Automatically determines create vs update based on UUID presence
- **Transaction Safety:** Uses database transactions for data consistency
- **Custom Validation:** Uses FieldsValidator with company context
- **Error Handling:** Comprehensive error handling with user-friendly messages

**Validation:**
- `FieldsValidator` - Custom validator for field definitions
- Company isolation through `selectedCompanyId` parameter
- Array structure validation for field data

**Create Method Features:**
```php
public function create(): bool|static
{
    // 1. Form validation
    if (!$this->validate()) {
        return $this;
    }
    
    // 2. Database transaction
    $transaction = Yii::$app->db->beginTransaction();
    
    try {
        // 3. Process each field
        foreach ($this->fields as $fieldData) {
            // 4. Determine create vs update
            if (!empty($fieldData['uuid'])) {
                // Update existing field
                $field = FieldsAdvanced::findOne(['uuid' => $fieldData['uuid']]);
            } else {
                // Create new field
                $field = new FieldsAdvanced();
                // Set company relation and generate UUID
            }
            
            // 5. Save field with validation
            if (!$field->save()) {
                $transaction->rollBack();
                return $this;
            }
        }
        
        // 6. Commit transaction
        $transaction->commit();
        return true;
        
    } catch (Exception $e) {
        // 7. Handle exceptions
        $transaction->rollBack();
        return $this;
    }
}
```

**Error Handling:**
- **Validation Errors:** Form validation errors with field-specific messages
- **Field Not Found:** UUID validation for update operations
- **Save Failures:** Database save operation error handling
- **System Errors:** Exception handling with transaction rollback
- **Toast Messages:** User-friendly error messages for UI display

---

## Business Logic

### 1. Customer Fields Display Flow
```
1. Receive GET request to /management/customer-fields
2. Create CustomerFieldSearchForm for searching
3. Load search parameters from query string
4. Create UpdateOrCreateCustomerFieldForm for operations
5. Execute field search with company isolation
6. Apply ILIKE searches on label and slug if query provided
7. Render HTML view with fields list and operation form
8. Display search results and creation interface
```

### 2. Field Creation/Update Flow (AJAX)
```
1. Receive AJAX POST request with field data
2. Set response format to JSON
3. Load form data into UpdateOrCreateCustomerFieldForm
4. Validate field array structure and content
5. Start database transaction for consistency
6. FOR EACH field in array:
   a. Check UUID presence (update vs create)
   b. Load existing field or create new instance
   c. Set field attributes from request data
   d. Save field with validation
   e. Rollback transaction on any failure
7. Commit transaction if all successful
8. Return JSON success/error response
```

### 3. Field Search Flow
```
1. Load search query from request parameters
2. Build database query with company filtering
3. Apply ILIKE searches on label and slug fields
4. Select specific field attributes
5. Order results by field ID ascending
6. Return raw array data for display
7. No pagination (returns all matching fields)
```

---

## Security Features

### 1. Company-based Access Control
- All field operations isolated by selectedUserCompany
- Field searches limited to company fields only
- Create operations automatically set company relation
- Update operations validate field ownership

### 2. Transaction Safety
- Database transactions ensure data consistency
- Rollback on any validation or save failure
- Batch operations maintain atomicity
- Exception handling prevents partial updates

### 3. Input Validation
- FieldsValidator ensures field definition validity
- UUID validation for update operations
- Field existence validation for updates
- Company context validation in custom validator

### 4. AJAX Security
- JSON response format prevents HTML injection
- Proper error message handling
- Controlled data exposure in error responses
- Request method validation (POST for operations)

---

## Integration Points

### Dependencies
- `FieldsAdvanced` - main customer field model
- `CompaniesAdvanced` - company model for relations
- `FieldsValidator` - custom field validation logic
- `UuidHelper` - UUID generation for new fields
- Yii Framework transaction handling

### Custom Validator
- `FieldsValidator` - validates field definitions with company context
- Handles field type validation
- Validates slug uniqueness within company
- Ensures proper field structure

### Related Components
- **Customer Categories** - uses custom fields for categorization
- **Customer Management** - displays and uses custom fields
- **Field Assignment** - assigns fields to customer categories
- **Data Validation** - validates customer data against field definitions

---

## Usage Examples

### Display customer fields management
```http
GET /management/customer-fields
```

### Search customer fields
```http
GET /management/customer-fields?query=email
```

### Create new customer fields (AJAX)
```http
POST /management/customer-fields
Content-Type: application/x-www-form-urlencoded
X-Requested-With: XMLHttpRequest

UpdateOrCreateCustomerFieldForm[fields][0][name]=Customer Email&UpdateOrCreateCustomerFieldForm[fields][0][slug]=customer_email&UpdateOrCreateCustomerFieldForm[fields][0][type]=email&UpdateOrCreateCustomerFieldForm[fields][0][description]=Customer primary email address
```

### Update existing customer field (AJAX)
```http
POST /management/customer-fields
Content-Type: application/x-www-form-urlencoded
X-Requested-With: XMLHttpRequest

UpdateOrCreateCustomerFieldForm[fields][0][uuid]=550e8400-e29b-41d4-a716-446655440000&UpdateOrCreateCustomerFieldForm[fields][0][name]=Updated Field Name&UpdateOrCreateCustomerFieldForm[fields][0][slug]=updated_field&UpdateOrCreateCustomerFieldForm[fields][0][type]=text&UpdateOrCreateCustomerFieldForm[fields][0][description]=Updated description
```

### Batch field operations (AJAX)
```http
POST /management/customer-fields
Content-Type: application/x-www-form-urlencoded
X-Requested-With: XMLHttpRequest

UpdateOrCreateCustomerFieldForm[fields][0][name]=New Field&UpdateOrCreateCustomerFieldForm[fields][0][slug]=new_field&UpdateOrCreateCustomerFieldForm[fields][0][type]=text&UpdateOrCreateCustomerFieldForm[fields][1][uuid]=550e8400-e29b-41d4-a716-446655440001&UpdateOrCreateCustomerFieldForm[fields][1][name]=Updated Field&UpdateOrCreateCustomerFieldForm[fields][1][slug]=updated_field&UpdateOrCreateCustomerFieldForm[fields][1][type]=number
```

---

## Performance Considerations

### Database Optimization
- Company-based indexing for field queries
- ILIKE searches may be resource-intensive
- Transaction overhead for batch operations
- Field count should be reasonable for UI performance

### Search Performance
- Simple query structure with company filtering
- ILIKE operations on label and slug fields
- No pagination may impact performance with many fields
- Consider limiting results for large field sets

### AJAX Operations
- Batch processing reduces request overhead
- Transaction safety ensures consistency
- JSON response format minimizes data transfer
- Form validation overhead for complex field arrays

---

## Error Handling

### Common Errors
- **"You have some errors, check the form."** - Form validation failure
- **"Field id not found."** - Invalid UUID for update operation
- **"Failed to save field."** - Database save operation failure
- **"System error."** - Exception during transaction processing

### Validation Errors
- Field structure validation through FieldsValidator
- UUID format validation for update operations
- Field existence validation for updates
- Company context validation errors

### Transaction Errors
- Database constraint violations
- Connection failures during transaction
- Partial update prevention through rollback
- Exception logging for debugging

### HTTP Status Codes
- **200 OK** - Successful operations (both HTML and AJAX)
- **400 Bad Request** - Validation failures (handled in JSON response)
- **500 Internal Server Error** - System errors with proper handling

---

*ManagementController is a specialized component for managing customer field definitions with batch processing capabilities, transaction safety, and comprehensive validation through custom field validators.*