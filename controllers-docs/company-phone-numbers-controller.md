# CompanyPhoneNumbersController Documentation

## Overview

CompanyPhoneNumbersController manages company phone numbers in the Jobix Dashboard administrative panel. This is a complex controller with external phone number provider integration, bulk operations support, fraud rating system, and advanced search capabilities.

**Base Path:** `/company-phone-numbers`  
**Controller Type:** WebController (Admin Application)  
**Namespace:** `apps\admin\controllers\CompanyPhoneNumbersController`

---

## Endpoints

### 1. Index - Phone Numbers List

| Parameter | Value |
|-----------|-------|
| **URL** | `/company-phone-numbers/index` |
| **Method** | GET |
| **Description** | Display list of all company phone numbers with search and filtering |

**Input Data (SearchCompanyPhoneNumbersForm queryParams):**

| Field | Type | Required | Validation | Description |
|-------|------|----------|------------|-------------|
| `service_provider` | string | ✗ | string | Filter by service provider |
| `phone_number` | string | ✗ | string | Search by phone number (ILIKE) |
| `title` | string | ✗ | string | Search by title (ILIKE) |
| `description` | string | ✗ | string | Search by description (ILIKE) |
| `status` | string | ✗ | string | Filter by status |
| `use_for_sms` | boolean | ✗ | string | Filter "used for SMS" |
| `use_for_call` | boolean | ✗ | string | Filter "used for calls" |
| `fraud_rating` | string | ✗ | string | Filter by fraud rating |
| `updated_at` | string | ✗ | string | Filter by update date |

**Fraud Rating System:**
- **Normal** - rating ≤ 75 or absent
- **Suspicious** - rating 75-85
- **High Risk** - rating 85-90  
- **Frequent Abusive Behavior** - rating 90-100

**Business Logic:**
1. Creating SearchCompanyPhoneNumbersForm for filtering
2. Loading search parameters from query string
3. Performing search with company-based filtering
4. JSON-based fraud rating queries through additional_info
5. Pagination (50 items per page)
6. Sorting by updated_at (DESC by default)

**Response:**
- **Success:** HTML view with phone numbers list
- **Data:** `pageName`, `searchModel`, `dataProvider`

**Available Sorting:**
- `service_provider`, `phone_number`, `title`, `description`
- `status`, `fraud_rating` (JSON field), `updated_at`

---

### 2. Create - Phone Number Creation

| Parameter | Value |
|-----------|-------|
| **URL** | `/company-phone-numbers/create` |
| **Methods** | GET, POST |
| **Description** | Create new phone number with bulk operations support |

#### GET `/company-phone-numbers/create`
**Purpose:** Display creation form

#### POST `/company-phone-numbers/create`
**Purpose:** Process phone number creation

**Input Data (SaveCompanyPhoneNumberForm bodyParams):**

| Field | Type | Required | Validation | Description |
|-------|------|----------|------------|-------------|
| `service_provider` | string | ✓ | required, string, max:20, in range | Phone service provider |
| `country_code` | string | ✓ | required, string, max:2 | Country code (ISO 2 letters) |
| `title` | string | ✓ | required, string, max:50 | Number title |
| `description` | string | ✗ | string, max:500 | Number description |
| `phone_number` | string/array | ✓ | required, custom provider validation | Phone number or array of numbers |
| `status` | boolean | ✗ | boolean, default: false | Activity status |
| `use_for_call` | boolean | ✗ | boolean, default: false | Use for calls |
| `use_for_sms` | boolean | ✗ | boolean, default: false | Use for SMS |
| `use_for_inbound_call` | boolean | ✗ | boolean, default: false | Use for inbound calls |

**Detailed Validation SaveCompanyPhoneNumberForm (CREATE scenario):**
```php
// Required fields
[['service_provider', 'country_code', 'title', 'phone_number', 'status', 'use_for_call', 'use_for_sms', 'use_for_inbound_call'], 'required']

// String validation
[['description'], 'string', 'max' => 500]
[['country_code'], 'string', 'max' => 2]
[['service_provider'], 'string', 'max' => 20]
[['title'], 'string', 'max' => 50]

// Provider validation
[['service_provider'], 'in', 'range' => CompanyPhoneNumbersAdvanced::serviceProviders('keys')]

// Custom phone validation
[['phone_number'], function($attribute) {
    if(!$this->phoneNumberProviderInstance->validate($phoneNumber)){
        $this->addError($attribute, 'Phone number is invalid!');
    }
}]

// Uniqueness validation
[['phone_number'], 'unique', 'targetClass' => CompanyPhoneNumbersAdvanced::class,
 'filter' => function($query) {
     return $query->whereCompanyId($this->selectedUserCompany->id)->active();
 }, 'on' => self::SCENARIO_CREATE]

// Boolean defaults and validation
[['status', 'use_for_call', 'use_for_sms', 'use_for_inbound_call'], 'default', 'value' => false]
[['status', 'use_for_call', 'use_for_sms', 'use_for_inbound_call'], 'boolean']
```

**Bulk Operations Support:**
1. **Multiple Phone Numbers:** If array of numbers is passed
2. **Unique Title Generation:** title + ' - ' + uniqid() for each number
3. **Iterative Creation:** Creating each number separately
4. **Error Handling:** Stop at first error
5. **Success Tracking:** Count successfully created numbers

**Provider Integration:**
1. **Phone Number Validation:** Through phoneNumberProviderInstance
2. **Phone Number Purchase:** Calling purchasePhoneNumber() API
3. **Provider Response Storage:** Saving provider response
4. **Friendly Number:** Generate user-friendly format

**Creation Business Logic:**
1. Form validation with CREATE scenario
2. Database transaction start
3. Search for existing number or create new one
4. Set country through CountriesHelper
5. Generate friendly phone format
6. Purchase number through provider API
7. Save provider response
8. Transaction commit/rollback

**Responses:**
- **Success:** Redirect to `/company-phone-numbers/update/{uuid}` of last created number
- **Bulk Success:** Notification about all created numbers + redirect
- **Validation Error:** HTML view with form and errors
- **Provider Error:** "Cannot buy this phone number. Please, contact with support!"
- **GET:** HTML view with empty form

---

### 3. Update - Phone Number Editing

| Parameter | Value |
|-----------|-------|
| **URL** | `/company-phone-numbers/update?uuid={phone_uuid}` |
| **Methods** | GET, POST, AJAX |
| **Description** | Edit existing phone number |

**URL Parameters:**
- `uuid` (string) - Phone number UUID

#### GET `/company-phone-numbers/update`
**Purpose:** Display edit form

**Behavior:**
- Automatic data loading through `populate()`
- Converting boolean fields from database values

#### AJAX Validation
**Response Format:** JSON with validation results

#### POST `/company-phone-numbers/update`
**Purpose:** Process number update

**Input Data (SaveCompanyPhoneNumberForm bodyParams):**

| Field | Type | Required | Validation | Description |
|-------|------|----------|------------|-------------|
| `uuid` | string | ✓ | required, UUID validation, exists check | Number UUID |
| `title` | string | ✓ | required, string, max:50 | Number title |
| `description` | string | ✗ | string, max:500 | Number description |
| `status` | boolean | ✗ | boolean | Activity status |
| `use_for_call` | boolean | ✗ | boolean | Use for calls |
| `use_for_sms` | boolean | ✗ | boolean | Use for SMS |
| `use_for_inbound_call` | boolean | ✗ | boolean | Use for inbound calls |

**Detailed Validation SaveCompanyPhoneNumberForm (UPDATE scenario):**
```php
// UPDATE scenario fields
[['uuid', 'title', 'description', 'status', 'use_for_call', 'use_for_sms', 'use_for_inbound_call'], 'required']

// UUID validation
[['uuid'], UuidHelper::class]

// Existence validation
[['uuid'], 'exist', 'targetClass' => CompanyPhoneNumbersAdvanced::class]

// Note: phone_number and service_provider cannot be changed in UPDATE
```

**Data Population Logic:**
```php
if ($this->uuid) {
    $this->attributes = $this->companyPhoneNumberModel->attributes;
    $this->status = $this->companyPhoneNumberModel->status == CompanyPhoneNumbersAdvanced::STATUS_ACTIVE;
    $this->use_for_call = $this->companyPhoneNumberModel->useForCall;
    $this->use_for_sms = $this->companyPhoneNumberModel->useForSMS;
    $this->use_for_inbound_call = $this->companyPhoneNumberModel->useForInboundCall;
}
```

**Responses:**
- **AJAX:** JSON validation results
- **Success:** HTML view with updated form and success notification
- **Error:** HTML view with form and errors
- **GET:** HTML view with populated form

---

### 4. Delete - Phone Number Deletion

| Parameter | Value |
|-----------|-------|
| **URL** | `/company-phone-numbers/delete?uuid={phone_uuid}` |
| **Method** | POST |
| **Description** | Delete one or multiple phone numbers |

**URL Parameters:**
- `uuid` (string, optional) - Number UUID for deletion

**POST Parameters:**
- `uuid` (array, optional) - Array of UUIDs for bulk deletion

**Input Data:**
- Single Delete: UUID through GET parameter
- Bulk Delete: Array of UUIDs through POST bodyParams

**Business Logic:**
1. **Bulk Delete Detection:** Check if UUID array is passed
2. **Iterative Processing:** Process each UUID separately
3. **DELETE Scenario:** Using SCENARIO_DELETE for each operation
4. **Error Handling:** Stop at first error in bulk operation
5. **Success Tracking:** Different messages for single/bulk operations

**Delete Operation Flow:**
```php
$model = new SaveCompanyPhoneNumberForm();
$model->scenario = SaveCompanyPhoneNumberForm::SCENARIO_DELETE;
$model->uuid = $uuid;
$isDeleted = $model->delete();
```

**Responses:**
- **Single Success:** "The Phone Number" successfully deleted
- **Bulk Success:** "All Phone Number was" successfully deleted  
- **Single Error:** Error notification with error details
- **Bulk Error:** Stop at first error with message
- **Always:** Redirect to `/company-phone-numbers/index`

---

## Form Classes

### 1. SaveCompanyPhoneNumberForm

**File:** `src/apps/admin/forms/company_phones/SaveCompanyPhoneNumberForm.php`  
**Extends:** `customs\Model` (336 lines)

**Scenarios:**
- `SCENARIO_VIEW` - view number data
- `SCENARIO_CREATE` - create new number
- `SCENARIO_UPDATE` - edit existing number  
- `SCENARIO_DELETE` - delete number
- `SCENARIO_NUMBERS_LIST` - list numbers by country

**Public Properties:**
```php
public mixed $uuid = null;                    // Number UUID (required for UPDATE/DELETE)
public mixed $service_provider = null;       // Service provider
public mixed $country_code = null;           // Country code
public mixed $title = null;                  // Number title
public mixed $description = null;            // Number description
public mixed $phone_number = null;           // Phone number
public bool $status = false;                 // Activity status
public bool $use_for_call = false;           // Use for calls
public bool $use_for_sms = false;            // Use for SMS
public bool $use_for_inbound_call = false;   // Use for inbound calls
```

**Private Properties:**
```php
private CompanyPhoneNumbersAdvanced $_companyPhoneNumberModel; // Number model
private PhoneNumberProviders $_phoneNumberProviderComponent;   // Provider component
```

**Scenario Fields:**
- **VIEW:** `['uuid']`
- **CREATE:** `['service_provider', 'country_code', 'title', 'description', 'phone_number', 'status', 'use_for_call', 'use_for_sms', 'use_for_inbound_call']`
- **UPDATE:** `['uuid', 'title', 'description', 'status', 'use_for_call', 'use_for_sms', 'use_for_inbound_call']`
- **DELETE:** `['uuid']`
- **NUMBERS_LIST:** `['country_code']`

**Key Methods:**
- `__construct()` - initialize phoneNumberProviders component
- `populate()` - load existing number data
- `create()` - create new number with provider integration
- `update()` - update existing number
- `delete()` - delete number
- `getCompanyPhoneNumberModel()` - get number model
- `getPhoneNumberProviderInstance()` - get provider instance

**Provider Integration Features:**
- Phone number validation through provider API
- Phone number purchasing through purchasePhoneNumber()
- Friendly number formatting
- Provider response storage
- Multi-provider support

**Transaction Safety:**
- Database transactions for create operations
- Rollback on provider API failures
- Error handling with user-friendly messages

---

### 2. SearchCompanyPhoneNumbersForm

**File:** `src/apps/admin/forms/company_phones/SearchCompanyPhoneNumbersForm.php`  
**Extends:** `customs\Model` (192 lines)

**Public Properties:**
```php
public mixed $service_provider = null;  // Filter by provider
public mixed $phone_number = null;      // Search by number (ILIKE)
public mixed $title = null;             // Search by title (ILIKE)
public mixed $description = null;       // Search by description (ILIKE)
public mixed $status = null;            // Filter by status
public mixed $use_for_sms = null;       // Filter "used for SMS"
public mixed $use_for_call = null;      // Filter "used for calls"
public mixed $fraud_rating = null;      // Filter by fraud rating
public mixed $updated_at = null;        // Filter by update date
```

**Search Features:**
- **ILIKE Searches:** phone_number, title, description
- **Exact Filtering:** service_provider, status
- **Boolean Filtering:** use_for_sms, use_for_call
- **JSON-based Fraud Rating:** Complex numeric range queries
- **Company Isolation:** Automatic filtering by selectedUserCompany
- **Not Deleted Default:** Automatic exclusion of deleted numbers

**Fraud Rating Queries:**
```php
// Frequent Abusive Behavior: 90-100
"CAST(additional_info->'spam_data'->>'fraud_rating' AS NUMERIC) BETWEEN 90 AND 100"

// High Risk: 85-90
"CAST(additional_info->'spam_data'->>'fraud_rating' AS NUMERIC) BETWEEN 85 AND 89"

// Suspicious: 75-85  
"CAST(additional_info->'spam_data'->>'fraud_rating' AS NUMERIC) BETWEEN 75 AND 84"

// Normal: ≤75 or NULL
"CAST(additional_info->'spam_data'->>'fraud_rating' AS NUMERIC) <= 75 OR IS NULL"
```

**Pagination:**
- Default page size: 50 items
- Custom pagination support

**Sorting Options:**
```php
'service_provider' => SORT_ASC/DESC
'phone_number' => SORT_ASC/DESC
'title' => SORT_ASC/DESC
'description' => SORT_ASC/DESC
'status' => SORT_ASC/DESC  
'fraud_rating' => SORT_ASC/DESC (JSON field)
'updated_at' => SORT_ASC/DESC (default DESC)
```

**Static Methods:**
- `statusesDropdown()` - status dropdown for UI
- `fraudRatingDropdown()` - fraud ratings dropdown for UI

---

## Business Logic

### 1. Phone Number Creation Flow
```
1. Display creation form (GET)
2. Detect single vs bulk phone number input
3. FOR EACH phone number:
   a. Generate unique title (for bulk)
   b. Validate phone number format through provider
   c. Check uniqueness within company
   d. Start database transaction
   e. Create/update phone number record
   f. Purchase phone number through provider API
   g. Store provider response
   h. Commit transaction or rollback on failure
4. Show success notification and redirect
```

### 2. Phone Number Update Flow
```
1. Load existing phone number data (GET)
2. Populate form with current values
3. Convert database boolean fields to form booleans
4. Validate updated data (UUID, title, usage flags)
5. Update record without changing core phone data
6. Show success notification
```

### 3. Phone Number Search Flow
```
1. Load search parameters from query
2. Apply company-based filtering automatically
3. Apply ILIKE searches for text fields
4. Apply boolean filters for usage flags
5. Apply complex JSON queries for fraud rating
6. Apply status filter or "not deleted" default
7. Apply sorting and pagination
8. Return ActiveDataProvider
```

### 4. Bulk Delete Flow
```
1. Detect array vs single UUID input
2. FOR EACH UUID:
   a. Create delete form with SCENARIO_DELETE
   b. Set UUID and call delete()
   c. Check success and break on first error
3. Show appropriate success message
4. Redirect to index
```

---

## Security Features

### 1. Company-based Access Control
- All operations isolated by selectedUserCompany
- Uniqueness validation within company scope
- Search results limited to user's company

### 2. Provider Integration Security
- Number validation through provider API
- Secure storage of provider responses
- Error handling for provider failures
- Transaction rollback on API errors

### 3. UUID Validation
- UuidHelper validation for all UUID operations
- Existence checking in CompanyPhoneNumbersAdvanced
- Secure UUID parameter handling

### 4. Input Validation
- String length limits for all text fields
- Boolean validation with proper defaults
- Custom phone number validation through providers
- XSS protection through Yii validation

### 5. Bulk Operations Security
- Individual validation for each element
- Transaction safety for batch operations
- Error isolation - stop on first error

---

## Integration Points

### Dependencies
- `CompanyPhoneNumbersAdvanced` - main phone numbers model
- `PhoneNumberProviders` - provider management component
- `PhoneProviderAbstract` - abstract provider class
- `CountriesHelper` - helper for country operations
- `UuidHelper` - UUID validation and generation
- `NotificationHelper` - user notifications

### Provider Integrations
- Multiple phone number provider support
- Provider-specific validation rules
- Phone number purchasing API integration
- Friendly number formatting
- Provider response storage and management

### Related Models
- `CompanyAgentPhoneNumbersAdvanced` - agent associations
- Various provider-specific models
- Company models for access control

---

## Usage Examples

### Creating single number
```http
POST /company-phone-numbers/create
Content-Type: application/x-www-form-urlencoded

SaveCompanyPhoneNumberForm[service_provider]=twilio&SaveCompanyPhoneNumberForm[country_code]=US&SaveCompanyPhoneNumberForm[title]=Main Office&SaveCompanyPhoneNumberForm[phone_number]=+1234567890&SaveCompanyPhoneNumberForm[use_for_call]=1&SaveCompanyPhoneNumberForm[use_for_sms]=1
```

### Bulk number creation
```http
POST /company-phone-numbers/create
Content-Type: application/x-www-form-urlencoded

SaveCompanyPhoneNumberForm[title]=Bulk Numbers&SaveCompanyPhoneNumberForm[service_provider]=twilio&SaveCompanyPhoneNumberForm[country_code]=US&SaveCompanyPhoneNumberForm[phone_number][]=+1234567890&SaveCompanyPhoneNumberForm[phone_number][]=+1234567891&SaveCompanyPhoneNumberForm[phone_number][]=+1234567892
```

### Search numbers with fraud filtering
```http
GET /company-phone-numbers/index?service_provider=twilio&fraud_rating=High Risk&use_for_call=1&status=1
```

### Bulk number deletion
```http
POST /company-phone-numbers/delete
Content-Type: application/x-www-form-urlencoded

uuid[]=550e8400-e29b-41d4-a716-446655440000&uuid[]=550e8400-e29b-41d4-a716-446655440001&uuid[]=550e8400-e29b-41d4-a716-446655440002
```

---

## Performance Considerations

### Provider API Integration
- Provider API calls can be slow
- Bulk operations increase execution time
- Transaction timeouts for bulk creation
- Error handling may require retries

### Database Optimization
- Company-based indexing for all queries
- JSON field queries (fraud_rating) can be slow
- Bulk operations require transaction management
- Pagination for large number lists

### Search Performance
- ILIKE searches can be resource-intensive
- JSON field sorting requires expression calculations
- Multiple filter combinations can slow down queries

---

## Error Handling

### Common Errors
- **"Phone number is invalid!"** - Provider validation failure
- **"Cannot buy this phone number. Please, contact with support!"** - Provider API failure
- **"Phone Number not found"** - Invalid UUID or access denied
- **Uniqueness violations** - Duplicate phone number in company

### Provider Integration Errors
- API connection failures
- Provider service unavailability  
- Invalid phone number format
- Purchase failures due to provider issues

### Bulk Operation Errors
- Individual validation failures
- Transaction rollback on provider errors
- Partial success scenarios

### HTTP Status Codes
- **200** - Success responses
- **500** - System errors with user-friendly messages
- **Redirect** - Success operations redirect to appropriate pages

---

*CompanyPhoneNumbersController is a critical component for managing phone numbers with advanced provider integration, bulk operations, fraud detection, and comprehensive search capabilities.*

