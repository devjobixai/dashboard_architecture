# CustomersController Documentation

## Overview

CustomersController manages customers in the Jobix Dashboard admin panel, including listing, creating, editing customers, and bulk upload functionality.

Base path: `/customers`  
Controller type: WebController (Admin Application)  
Namespace: `apps\\admin\\controllers\\CustomersController`

---

## Endpoints

### 1. Index — Customers List

| Param | Value |
|------|-------|
| URL | `/customers/index` |
| Method | GET |
| Description | Display all company customers with search and filters |

Input (CustomerSearchForm queryParams):
- Query parameters for searching and filtering customers
- Processed by CustomerSearchForm

Business logic:
1. Instantiate CustomerSearchForm for search
2. Load search params from the query string
3. Validate search parameters
4. Execute search and build a DataProvider

Response:
- Success: HTML view with customers list
- Data: `pageName`, `searchModel`, `dataProvider`

---

### 2. Create — Create Customer

| Param | Value |
|------|-------|
| URL | `/customers/create` |
| Methods | GET, POST, AJAX |
| Description | Create a new customer |

#### GET `/customers/create`
Purpose: Render the creation form

#### AJAX Validation
Purpose: Real-time form validation

Response Format: JSON with validation results

#### POST `/customers/create`
Purpose: Handle customer creation

Input (SaveCustomerForm bodyParams):

| Field | Type | Required | Validation | Description |
|------|------|----------|------------|-------------|
| `email` | string | ✓ | required, email, unique per company | Customer email |
| `phone` | string | ✓ | required, string, max:64 | Customer phone |
| `name` | string | ✓ | required, string, max:100 | Customer name |
| `timezone` | string | ✓ | required, valid timezone | Customer timezone |
| `values` | array | ✗ | custom field validation | Additional/custom fields |

SaveCustomerForm validation (CREATE scenario):
```php
// Required fields
[['email', 'phone', 'name', 'timezone'], 'required']

// Field-specific validation
[['phone'], 'string', 'max' => 64]
[['email'], 'email']
[['name'], 'string', 'max' => 100]
[['timezone'], 'in', 'range' => timezone_identifiers_list()]

// Custom validation
[['values'], 'validateValues'] // Custom field validation through ValueHelper
[['email'], 'unique', 'targetClass' => CompanyCustomersAdvanced::class,
 'filter' => function($query) {
     return $query->whereCompanyId($this->selectedUserCompany->id);
 }]
```

Custom Field Validation:
- Via `ValueHelper::validateValue()`
- Supports various field types (text, number, date, etc.)
- Automatic type conversion via `ValueHelper::convertValue()`
- Integrated with `FieldsAdvanced` for field definitions

Creation business logic:
1. Validate form in CREATE scenario
2. Generate customer data via `generateCustomerData()`
3. Use API SaveForm to persist
4. Handle the result (array or object with errors)
5. Generate SUID for the customer

API Integration:
- Uses API `SaveForm` for actual creation
- Sends data from `generateCustomerData()`
- Handles array/object responses

Responses:
- AJAX: JSON validation results
- Success: Redirect to `/customers/edit/{new_customer_uuid}` with success notification
- Error: HTML view with form and errors
- GET: HTML view with empty form

---

### 3. Edit — Edit Customer

| Param | Value |
|------|-------|
| URL | `/customers/edit?uuid={customer_uuid}` |
| Methods | GET, POST, AJAX |
| Description | Edit an existing customer and view their events |

URL Parameters:
- `uuid` (string) — customer UUID

#### GET `/customers/edit`
Purpose: Render the edit form

Behavior:
- Load existing customer data via `populate()`
- Load customer events via CustomerEventsSearchForm
- Render customer event history

#### AJAX Validation
Response Format: JSON with validation results

#### POST `/customers/edit`
Purpose: Handle customer update

Input (SaveCustomerForm bodyParams):

| Field | Type | Required | Validation | Description |
|------|------|----------|------------|-------------|
| `uuid` | string | ✓ | required, UUID validation, exists check | Customer UUID |
| `email` | string | ✓ | required, email, unique per company (exclude current) | Customer email |
| `phone` | string | ✓ | required, string, max:64 | Customer phone |
| `name` | string | ✓ | required, string, max:100 | Customer name |
| `timezone` | string | ✓ | required, valid timezone | Customer timezone |
| `values` | array | ✗ | custom field validation | Additional/custom fields |

SaveCustomerForm validation (UPDATE scenario):
```php
// Required fields (includes UUID for UPDATE)
[['uuid', 'email', 'phone', 'name', 'timezone'], 'required']

// UUID validation
[['uuid'], UuidHelper::class]
[['uuid'], 'exist', 'targetClass' => CompanyCustomersAdvanced::class]

// Email uniqueness (exclude current customer)
[['email'], 'unique', 'targetClass' => CompanyCustomersAdvanced::class,
 'filter' => function($query) {
     if($this->scenario == self::SCENARIO_UPDATE){
         $query->andWhere(['!=', 'uuid', $this->uuid]);
     }
     return $query->whereCompanyId($this->selectedUserCompany->id);
 }]
```

Data Population Logic:
1. Load basic customer attributes
2. Load custom field values via ValueHelper
3. Convert field types for display
4. Company-based filtering for custom fields

Customer Events Integration:
- Show customer event history
- Use CustomerEventsSearchForm
- Filter by `company_customer_id`

Responses:
- AJAX: JSON validation results
- Success: HTML view with updated form and success notification
- Error: HTML view with form and errors
- Data: `pageName`, `model`, `eventsDataProvider`

---

### 4. Upload — Bulk Upload Customers

| Param | Value |
|------|-------|
| URL | `/customers/upload` |
| Method | POST (AJAX only) |
| Description | Bulk upload customers via pasted content |

Request validation:
- AJAX request required
- POST method

Input (UploadCustomersForm bodyParams):
- `paste_content` — text content with customers data
- See UploadCustomersForm for exact structure

Business logic:
1. Validate AJAX request
2. Load UploadCustomersForm
3. Process bulk data via `process()`
4. Return processing result

Responses:

Success:
```json
{
  "success": true,
  "object": {
    "processed_count": 25,
    "created_count": 20,
    "updated_count": 5,
    "errors": []
  }
}
```

Error:
```json
{
  "success": false,
  "errors": {
    "paste_content": ["Error message"]
  }
}
```

Missing content:
```json
{
  "success": false,
  "errors": {
    "paste_content": "Paste customer content"
  }
}
```

---

## Form Classes

### 1. SaveCustomerForm

File: `src/apps/admin/forms/customers/SaveCustomerForm.php`  
Extends: `customs\\Model` (306 lines)

Scenarios:
- `SCENARIO_CREATE` — create a new customer
- `SCENARIO_UPDATE` — update an existing customer

Public properties:
```php
public mixed $uuid = null;           // Customer UUID (required for UPDATE)
public mixed $email = null;          // Customer email
public mixed $phone = null;          // Customer phone
public mixed $name = null;           // Customer name
public mixed $timezone = null;       // Timezone
public array $values = [];           // Additional/custom fields
```

Private properties:
```php
private array $_categoryFields = [];                    // Category fields
private CompanyCustomersAdvanced|null $_customer = null; // Cached customer
```

Scenario Fields:
- CREATE: `['email', 'phone', 'name', 'timezone', 'values']`
- UPDATE: `['uuid', 'email', 'phone', 'name', 'timezone', 'values']`

Attribute Labels:
```php
'suid' => 'Source unique ID'
'name' => 'Name'
'email' => 'Email'
'phone' => 'Phone'
'timezone' => 'Timezone'
```

Key methods:
- `validateValues()` — custom validation for additional fields
- `populate()` — load existing customer data
- `save()` — save via API SaveForm
- `generateCustomerData()` — build data for API
- `getSuid()` — generate source unique ID
- `getSelectedCategoryFields()` — get category fields
- `getCustomerFieldValues()` — get custom field values
- `getCustomer()` — get customer model
- `timezoneDropdownList()` — timezone list

Custom Field Validation Features:
- Field type validation via FieldsAdvanced
- ValueHelper integration for validation and conversion
- Supports multiple field types (string, number, date, etc.)
- Automatic type conversion on save

API Integration:
- Wrapper around API SaveForm
- Generates customer_data structure
- Handles different API response types
- Error handling and propagation

---

### 2. CustomerSearchForm

File: `src/apps/admin/forms/customers/CustomerSearchForm.php`  
Extends: `customs\\Model`

Details require further analysis for full documentation.

---

### 3. CustomerEventsSearchForm

File: `src/apps/admin/forms/customers/CustomerEventsSearchForm.php`  
Extends: `customs\\Model`

Details require further analysis for full documentation.

---

### 4. UploadCustomersForm

File: `src/apps/admin/forms/customers/UploadCustomersForm.php`  
Extends: `customs\\Model`

Details require further analysis for full documentation.

---

## Business Logic

### 1. Customer Creation Flow
```
1. Display creation form (GET)
2. AJAX validation (optional)
3. Form validation in CREATE scenario
4. Generate customer data structure
5. Call API SaveForm to persist
6. Handle API response (array/object)
7. Generate SUID for customer
8. Redirect to edit page
```

### 2. Customer Update Flow
```
1. Load existing customer data (GET)
2. Populate form with customer attributes
3. Load custom field values via ValueHelper
4. AJAX validation (optional)
5. Form validation in UPDATE scenario
6. Generate updated customer data
7. Call API SaveForm to update
8. Handle API response and errors
```

### 3. Custom Field Management
```
1. Load category fields for company
2. Validate field values by type
3. Convert values using ValueHelper
4. Store values with proper types
5. Display values with type conversion
```

### 4. Bulk Upload Processing
```
1. Validate AJAX request
2. Parse paste content
3. Process multiple customer records
4. Handle per-record validation errors
5. Return processing statistics
```

---

## Security Features

### 1. Company-based Access Control
- All operations are isolated by selectedUserCompany
- Email uniqueness is checked within the company
- Custom fields filtered by company relation

### 2. UUID Validation
- UuidHelper validation for customer UUID
- Existence checking in CompanyCustomersAdvanced
- Secure UUID handling

### 3. Email Uniqueness
- Per-company email uniqueness
- Scenario-aware uniqueness checks
- Exclude current customer on UPDATE

### 4. Custom Field Security
- Type validation via FieldsAdvanced
- Value validation via ValueHelper
- Field access control by company relation

### 5. Input Validation
- String length limits for all text fields
- Email format validation
- Timezone validation against PHP timezone list
- XSS protection via Yii validation

---

## Integration Points

### Dependencies
- `CompanyCustomersAdvanced` — customers model
- `FieldsAdvanced` — custom field definitions
- `ValueHelper` — validation and conversion for field values
- `UuidHelper` — UUID validation
- API `SaveForm` — persistence layer

### Related Models
- `CompanyCustomers` (basic) — base model
- `CustomerEvents` — events history

---

## Usage Examples

### Create a customer
```http
POST /customers/create
Content-Type: application/x-www-form-urlencoded

email=jane@example.com&phone=+12025550123&name=Jane Doe&timezone=UTC
```

### Bulk upload (AJAX)
```http
POST /customers/upload
X-Requested-With: XMLHttpRequest
Content-Type: application/x-www-form-urlencoded

paste_content=...
```

