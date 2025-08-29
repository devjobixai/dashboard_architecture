# CustomerController (API) Documentation

## Overview

CustomerController manages customers via REST API v1, including saving, deleting, searching, counting, and working with customer lists.

Base path: `/api/v1/customer`  
Controller type: ApiController (API Application v1)  
Namespace: `apps\\api\\versions\\v1\\controllers\\CustomerController`  
Authentication: Header `company-key` to identify the company

---

## Authentication

All endpoints require an authentication header:

| Header | Type | Required | Description |
|--------|------|----------|-------------|
| `company-key` | string | ✓ | Company API key for access |

---

## Endpoints

### 1. Save Customer

| Param | Value |
|-------|-------|
| URL | `/api/v1/customer/save` |
| Method | POST |
| Description | Save customer data with optional queue processing |

Query Parameters:
- `useQueue` (boolean, optional) — process asynchronously via queue

Input (SaveForm bodyParams):

| Field | Type | Required | Validation | Description |
|-------|------|----------|------------|-------------|
| `company_key` | string | ✓ | required, string, custom validation | Company API key |
| `customer_data` | object | ✓ | required, complex array structure | Customer data |
| `event_data` | object | ✗ | complex array structure, maxSize: 100KB | Event data (optional) |

Customer Data Structure example
```json
{
  "main": {
    "suid": "customer_123",
    "email": "customer@example.com",
    "phone": "+1234567890",
    "name": "John Doe",
    "timezone": "America/New_York"
  },
  "values": {
    "custom_field_1": "value1",
    "custom_field_2": "value2"
  }
}
```

Customer Data Fields:
- `suid` (string, required) — min:3, max:64 — unique customer ID
- `email` (string, optional) — email format, min:5, max:255
- `phone` (string, optional) — min:3, max:64
- `name` (string, optional) — min:3, max:64
- `timezone` (string, optional) — valid timezone, min:3, max:64
- `values` (object, optional) — additional fields

Event Data Structure example (optional)
```json
{
  "main": {
    "slug": "registration",
    "name": "Customer Registration"
  },
  "values": {
    "source": "website",
    "campaign": "spring_2023"
  }
}
```

Event Data Fields:
- `slug` (string, required) — min:3, max:64, unique per company
- `name` (string, optional) — min:3, max:64
- `values` (object, optional) — additional event fields

SaveForm Validation:
```php
// Required fields
[['company_key', 'customer_data'], 'required']
[['company_key'], 'string']

// Customer data validation
['customer_data', ArrayValidator::class, 'structure' => [
    'main' => [
        'suid' => [['required'], ['string', 'min' => 3, 'max' => 64]],
        'email' => [['email'], ['string', 'min' => 5, 'max' => 255]],
        'phone' => [['string', 'min' => 3, 'max' => 64]],
        'name' => [['string', 'min' => 3, 'max' => 64]],
        'timezone' => [['in', 'range' => timezone_identifiers_list()]]
    ],
    'values' => [[ArrayValidator::class]]
], 'maxSizeInBytes' => 102400] // 100KB limit

// Event data validation (optional)
['event_data', ArrayValidator::class, 'structure' => [
    'main' => [
        'slug' => [
            ['required'], ['string', 'min' => 3, 'max' => 64],
            ['unique'] // Cannot use system reserved slugs
        ],
        'name' => [['string', 'min' => 3, 'max' => 64]]
    ],
    'values' => [[ArrayValidator::class]]
], 'maxSizeInBytes' => 102400]

// Company key validation
[['company_key'], 'validateCompanyApiKey'] // Custom: check company exists
```

Save Business Logic:

1. Queue Mode (`?useQueue=true`):
   - Enqueue job `SaveCompanyCustomerJob`
   - Return queue number for tracking
   
2. Direct Mode (default):
   - Validate data and company
   - Database transaction
   - Save customer_data
   - Save event_data (if provided)
   - MongoDB integration for customer data
   - Initialize trigger nodes
   - Filter through customer lists
   - Process event nodes
   - Trigger console email command

Transaction Safety:
- Uses database transactions
- Rolls back on any error
- MongoDB cleanup on errors

MongoDB Integration:
- Automatic persistence to MongoDB
- Returns MongoDB ObjectID

Responses:

Queue Mode Response:
```json
{
  "queueNumber": 12345
}
```

Direct Mode Success:
```json
{
  "customer_data": {
    "id": 12345,
    "suid": "customer_123",
    "status": "saved"
  },
  "event_data": {
    "id": 678,
    "slug": "registration",
    "status": "created"
  }
}
```

Validation Error:
```json
{
  "company_key": ["Company not found"],
  "customer_data": {
    "main": {
      "suid": ["This field is required"]
    }
  }
}
```

---

### 2. Delete Customer

| Param | Value |
|-------|-------|
| URL | `/api/v1/customer/delete` |
| Method | POST |
| Description | Delete a customer |

Input (DeleteForm bodyParams):
- See DeleteForm for details

Responses:
- Success: `DeleteForm` object or `boolean`
- Error: `DeleteForm` object with errors

---

### 3. Add to List

| Param | Value |
|-------|-------|
| URL | `/api/v1/customer/add-to-list` |
| Method | POST |
| Description | Add a customer to a list |

Input (AddToListForm bodyParams):
- See AddToListForm for details

Responses:
- Always: `AddToListForm` object

---

### 4. Count Customers

| Param | Value |
|-------|-------|
| URL | `/api/v1/customer/count` |
| Method | POST |
| Description | Count customers by criteria |

Input (CountForm bodyParams):
- See CountForm for details

Responses:
- Success: result `array` or `CountForm` object

---

### 5. Regenerate List

| Param | Value |
|-------|-------|
| URL | `/api/v1/customer/regenerate-list` |
| Method | POST |
| Description | Regenerate a customer list |

Input (RegenerateListForm bodyParams):
- See RegenerateListForm for details

Responses:
- Success: `boolean` or `RegenerateListForm` object

---

### 6. Regenerate List Progress

| Param | Value |
|-------|-------|
| URL | `/api/v1/customer/regenerate-list-progress` |
| Method | POST |
| Description | Get regeneration progress |

Input (RegenerateListProgressForm bodyParams):
- See RegenerateListProgressForm for details

Responses:
- Success: `null`, `int` (progress) or `RegenerateListProgressForm` object

---

### 7. Search Customers

| Param | Value |
|-------|-------|
| URL | `/api/v1/customer/search` |
| Method | GET |
| Description | Search customers by criteria |

Headers:
- `company-key` (required) — company API key

Query Parameters:
- Search params via queryParams
- Processed by CustomersListForm

Responses:
- Success: `ActiveDataProvider` with results or `CustomersListForm` object

---

## Form Classes

### 1. SaveForm

File: `src/apps/api/versions/v1/forms/customers/SaveForm.php`  
Extends: `customs\\Model` (806 lines)

Public properties:
```php
public string $company_key = '';           // Company API key
public array $customer_data = [];         // Customer data
public array $event_data = [];            // Event data (optional)
```

Private properties:
```php
private bool $_isNewCustomer = true;               // Is new customer
private bool $_isNewCompanyEvent = true;           // Is new company event
private array $_customerPreviousValues = [];       // Previous values
private CompaniesAdvanced $_companyModel = null;   // Company model
private CompanyEventsAdvanced $_eventModel = null; // Event model
private CompanyCustomersAdvanced $_customerModel = null; // Customer model
private CompanyCustomerEventsAdvanced $_customerEventModel = null; // Customer event model
```

Key methods:
- `rules()` — complex validation rules with ArrayValidator
- `afterValidate()` — check existence of customer and event
- `validateCompanyApiKey()` — validate company API key
- `save()` — main save logic with transactions
- `_saveCustomerData()` — save customer data
- `_saveEventData()` — save event data
- Various getters for model access

Validation structure:
- customer_data.main.suid — required, string(3-64)
- customer_data.main.email — optional, email, string(5-255)
- customer_data.main.phone — optional, string(3-64)
- customer_data.main.name — optional, string(3-64)
- customer_data.main.timezone — optional, valid timezone
- customer_data.values — array of extra fields
- event_data.main.slug — required, string(3-64), unique
- event_data.main.name — optional, string(3-64)
- event_data.values — array of extra fields

Business Logic Features:
- Database transactions with rollback
- MongoDB integration
- Nodular component processing
- Customer list filtering
- Event node initialization
- Email processing via console commands
- Cache management
- Error logging

---

### 2. DeleteForm

File: `src/apps/api/versions/v1/forms/customers/DeleteForm.php`  
Extends: `yii\\base\\Model`

Details require further analysis for full documentation.

---

### 3. AddToListForm

File: `src/apps/api/versions/v1/forms/customers/AddToListForm.php`  
Extends: `yii\\base\\Model`

Details require further analysis for full documentation.

---

### 4. CustomersListForm (Search)

File: `src/apps/api/versions/v1/forms/customers/CustomersListForm.php`  
Extends: `yii\\base\\Model`

Details require further analysis for full documentation.

---

## Security Features

### 1. Company-based Access Control
- All operations tied to company-key
- API key validation for every request
- Cross-company data isolation

### 2. Data Validation
- Complex validation via ArrayValidator
- Size limits (100KB for customer_data and event_data)
- Email and timezone validation
- Unique constraints for event slugs

### 3. Transaction Safety
- Database transactions with automatic rollback
- Exception handling with cleanup
- MongoDB consistency checks

### 4. Input Sanitization
- Trim filters for text fields
- Array structure validation
- Type checking and constraints

---

## Business Logic

### 1. Customer Save Flow
```
1. Validate company_key
2. Validate structure of customer_data and event_data
3. Begin database transaction
4. Save customer_data to PostgreSQL
5. Save event_data to PostgreSQL (if present)
6. Save customer to MongoDB
7. Initialize trigger nodes
8. Filter via customer lists
9. Process event
10. Send email notification
11. Commit transaction
```

### 2. Queue Processing
- Asynchronous processing via Yii Queue
- Job: SaveCompanyCustomerJob
- Tracking via queue numbers

### 3. MongoDB Integration
- Automatic synchronization PostgreSQL -> MongoDB
- Object ID generation
- Error handling and cleanup

---

## Integration Points

### Dependencies
- `CompaniesAdvanced` — company management
- `CompanyCustomersAdvanced` — company customers
- `CompanyEventsAdvanced` — company events

