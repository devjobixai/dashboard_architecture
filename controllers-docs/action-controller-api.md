# ActionController API Documentation

## Overview

ActionController provides API endpoints for executing company actions through external integrations in the Jobix Dashboard API. This controller handles action execution requests with support for HTTP client integration, customer data processing, event data management, anchor replacement, and activity history tracking.

**Base Path:** `/api/v1/action`  
**Controller Type:** ApiController (API Application v1)  
**Namespace:** `apps\api\versions\v1\controllers\ActionController`  
**Authentication:** Header `company-key` for company identification

---

## Endpoints

### 1. Run - Execute Action

| Parameter | Value |
|-----------|-------|
| **URL** | `/api/v1/action/run` |
| **Method** | POST |
| **Description** | Execute a company action with customer and event data processing |

**Authentication:**
- **Header:** `company-key` (required) - Company API key for authentication

**Input Data (ActionRunForm bodyParams):**

| Field | Type | Required | Validation | Description |
|-------|------|----------|-----------|-------------|
| `company_key` | string | ✓ | required, string, exists in companies | Company API key for authentication |
| `action` | mixed | ✓ | required, exists as ID or UUID | Action ID or UUID to execute |
| `customer_data` | array | ✗ | custom validation, safe | Customer data for action context |
| `event_data` | array | ✗ | EventDataForm validation | Event data for action processing |

**Detailed Validation ActionRunForm:**
```php
[
    ['company_key', 'action'], 'required'
]
[
    ['company_key'], 'string'
]
[
    ['company_key'], 'validateCompanyApiKey'  // Custom company validation
]
[
    ['action'], 'validateAction'              // Custom action validation
]
[
    ['customer_data'], 'setCustomer'          // Customer data processing
]
[
    ['event_data'], 'validateData'            // Event data validation
]
```

**Custom Validation Methods:**

#### Company API Key Validation
```php
public function validateCompanyApiKey(string $attrName): void
{
    $this->_companyModel = CompaniesAdvanced::findByApiKey($this->$attrName);
    if (is_null($this->_companyModel)) {
        $this->addError($attrName, 'Company not found');
    }
}
```

#### Action Validation
```php
public function validateAction(string $attrName): void
{
    // Try to find by ID first
    $this->_actionModel = CompanyActionsAdvanced::findById(intval($this->$attrName));
    if (empty($this->_actionModel)) {
        // Fall back to UUID search
        $this->_actionModel = CompanyActionsAdvanced::findByUUid(strval($this->$attrName));
        if (empty($this->_actionModel)) {
            $this->addError($attrName, 'Action not found');
        }
    }
}
```

#### Event Data Validation
```php
public function validateData(string $attrName): void
{
    $model = new EventDataForm();
    $model->load(array_merge($this->$attrName, ['event_data' => $this->$attrName]), '');
    $model->process();
    
    if ($model->hasErrors()) {
        $this->addErrors($model->getErrors());
        return;
    }
    
    $this->event_data = $model->event_data;
}
```

**Business Logic:**
1. Clear output buffer for clean JSON response
2. Load request body parameters without wrapper
3. Validate company API key and find company model
4. Validate action existence (by ID or UUID)
5. Process customer data and set customer model
6. Validate and process event data through EventDataForm
7. Execute action with HTTP client integration
8. Store action execution history
9. Return structured response array

**Action Execution Process:**
```php
public function save(): array
{
    $this->_start = microtime(true);
    
    if (!$this->validate()) {
        return [
            'success' => false,
            'errors' => $this->getErrors(),
            'execution_time' => microtime(true) - $this->_start
        ];
    }
    
    return $this->runAction();
}
```

**HTTP Client Integration:**
- Uses GuzzleHttp\Client for external API calls
- Supports custom headers with anchor replacement
- URL parameter processing and anchor substitution
- Request/response tracking for history
- Promise-based asynchronous requests
- Error handling for request exceptions

**Response Format:**

**Success Response:**
```json
{
    "success": true,
    "data": {
        "action_id": "action_uuid_or_id",
        "response_data": "external_api_response",
        "execution_time": 0.123
    },
    "execution_time": 0.123
}
```

**Error Response:**
```json
{
    "success": false,
    "errors": {
        "company_key": ["Company not found"],
        "action": ["Action not found"],
        "event_data": ["Invalid event data structure"]
    },
    "execution_time": 0.045
}
```

**History Tracking:**
- Automatic history storage on successful execution
- Request/response logging through CompanyActionActivitiesAdvanced
- Execution time tracking
- Customer and event data association
- Action performance metrics

---

## Form Classes

### 1. ActionRunForm

**File:** `src/apps/api/versions/v1/forms/actions/ActionRunForm.php`  
**Extends:** `customs\Model` (474 lines)

**Public Properties:**
```php
public mixed $company_key = null;         // Company API key (required)
public mixed $action = null;              // Action ID or UUID (required)
public mixed $customer_data = [];         // Customer context data
public mixed $event_data = [];            // Event processing data
```

**Private Properties:**
```php
private null|CompaniesAdvanced $_companyModel = null;           // Company model
private null|CompanyCustomersAdvanced $_customerModel = null;   // Customer model
private null|CompanyActionsAdvanced $_actionModel = null;       // Action model
private array|null $_request = null;                            // HTTP request data
private array|null $_response = null;                           // HTTP response data
private int|float $_start = 0;                                  // Execution start time
```

**Key Methods:**
- `save()` - main execution method with validation and timing
- `runAction()` - execute action logic with HTTP integration
- `runCustomAction()` - custom action execution with anchor replacement
- `validateCompanyApiKey()` - validate company through API key
- `validateAction()` - validate action existence by ID or UUID
- `validateData()` - validate event data through EventDataForm
- `setCustomer()` - process and set customer data
- `storeHistory()` - store execution history and metrics

**Advanced Features:**

#### Anchor Replacement System
```php
public function replaceAnchors(array $data, array $anchorsValues): array
{
    // Replace placeholder anchors with actual values
    // Supports nested array processing
    // Used for dynamic URL, header, and content generation
}

public function setUrlParams(array $anchors): string
{
    // Process URL parameters with anchor replacement
    // Handles GET parameters and path variables
}

public function setHeaders(array $headers): array
{
    // Process HTTP headers with anchor replacement
    // Supports dynamic header generation
}
```

#### Customer Data Management
```php
public function getCustomerData(): array
{
    // Extract customer data with field validation
    // Supports custom field processing
    // Company-based customer isolation
}

public function getCustomerFields(): array
{
    // Get customer custom fields configuration
    // Field type and validation support
}
```

#### HTTP Client Integration
```php
public function runCustomAction(): array
{
    // GuzzleHttp client integration
    // Support for GET, POST, PUT, DELETE methods
    // Custom headers and content processing
    // Anchor replacement in URLs and data
    // Promise-based request handling
    // Exception handling for API failures
}
```

**Event Data Processing:**
- Integration with EventDataForm for validation
- Nested event data structure support
- Event type validation and processing
- Context data extraction and management

---

### 2. EventDataForm

**File:** `src/apps/api/versions/v1/forms/actions/event/EventDataForm.php`  
**Purpose:** Validate and process event data for action execution

**Key Features:**
- Event data structure validation
- Event type processing
- Context data management
- Integration with ActionRunForm

---

## Business Logic

### 1. Action Execution Flow
```
1. Receive POST request with company_key and action data
2. Clear output buffer for clean JSON response
3. Load request body parameters without wrapper
4. Start execution timer for performance tracking
5. Validate company API key and load company model
6. Validate action existence (try ID first, then UUID)
7. Process customer data and set customer context
8. Validate event data through EventDataForm
9. Execute action with HTTP client integration:
   a. Set up anchor values from customer and event data
   b. Replace anchors in URL, headers, and content
   c. Make HTTP request to external API
   d. Process response and handle errors
10. Store execution history with metrics
11. Return structured JSON response with results
```

### 2. Company Authentication Flow
```
1. Extract company_key from request body
2. Look up company by API key in CompaniesAdvanced
3. Validate company existence and status
4. Set company context for subsequent operations
5. Use company model for data isolation
```

### 3. Action Resolution Flow
```
1. Receive action parameter (ID or UUID)
2. First attempt: Find action by integer ID
3. If not found: Find action by UUID string
4. Validate action belongs to authenticated company
5. Load action configuration (URL, headers, method, etc.)
6. Prepare action for execution
```

### 4. Customer Data Processing Flow
```
1. Load customer_data array from request
2. Extract customer identifier or data
3. Find or create customer record in company context
4. Load customer custom fields configuration
5. Validate customer data against field definitions
6. Set customer model for anchor replacement
```

### 5. HTTP Request Processing Flow
```
1. Load action configuration (method, URL, headers, content)
2. Set up anchor values from customer and event data
3. Replace anchors in URL path and parameters
4. Process and replace anchors in headers
5. Replace anchors in request content/body
6. Create GuzzleHttp request with processed data
7. Execute request with error handling
8. Process response and extract relevant data
9. Track request/response for history
```

---

## Security Features

### 1. API Key Authentication
- Company-based authentication through `company-key` header
- API key validation against CompaniesAdvanced records
- Company context isolation for all operations
- Secure company identification and authorization

### 2. Action Access Control
- Action existence validation within company context
- Support for both ID and UUID-based action identification
- Company-based action filtering and access control
- Prevention of cross-company action execution

### 3. Input Validation
- Comprehensive validation for all input parameters
- Custom validators for company, action, and event data
- EventDataForm integration for structured event validation
- Safe handling of customer and event data arrays

### 4. HTTP Security
- HTTPS enforcement for external API calls
- Secure header processing with anchor replacement
- Request/response sanitization and validation
- Error handling to prevent information leakage

---

## Integration Points

### Dependencies
- `CompaniesAdvanced` - company model and API key validation
- `CompanyActionsAdvanced` - action configuration and execution
- `CompanyCustomersAdvanced` - customer data management
- `CompanyActionActivitiesAdvanced` - execution history tracking
- `FieldsAdvanced` - customer field definitions
- `EventDataForm` - event data validation and processing
- `GuzzleHttp\Client` - HTTP client for external API calls
- `JsonValidator` - JSON structure validation
- `UuidHelper` - UUID validation and generation

### HTTP Client Integration
- GuzzleHttp for external API communication
- Promise-based asynchronous request handling
- Custom header and content processing
- Multiple HTTP method support (GET, POST, PUT, DELETE)
- Request exception handling and error reporting

### Anchor System Integration
- `AnchorsHelper` for anchor processing
- `NodeHelper` for nodular system integration
- Dynamic content replacement based on customer and event data
- URL, header, and content anchor substitution
- Context-aware data injection

### Related Components
- **Admin ActionsController** - action configuration management
- **Customer Management** - customer data integration
- **Event Processing** - event data validation and handling
- **History Tracking** - execution monitoring and metrics
- **External APIs** - integration with third-party services

---

## Usage Examples

### Execute action with customer data
```http
POST /api/v1/action/run
Content-Type: application/json
company-key: your-company-api-key

{
    "company_key": "your-company-api-key",
    "action": "550e8400-e29b-41d4-a716-446655440000",
    "customer_data": {
        "email": "customer@example.com",
        "name": "John Doe",
        "phone": "+1234567890"
    },
    "event_data": {
        "event_type": "order_completed",
        "order_id": "12345",
        "amount": 99.99
    }
}
```

### Execute action by ID with minimal data
```http
POST /api/v1/action/run
Content-Type: application/json
company-key: your-company-api-key

{
    "company_key": "your-company-api-key",
    "action": 123
}
```

### Execute action with complex event data
```http
POST /api/v1/action/run
Content-Type: application/json
company-key: your-company-api-key

{
    "company_key": "your-company-api-key",
    "action": "action-uuid",
    "customer_data": {
        "customer_id": "cust_123",
        "custom_fields": {
            "priority": "high",
            "segment": "premium"
        }
    },
    "event_data": {
        "event_type": "support_ticket_created",
        "ticket_data": {
            "subject": "Billing Issue",
            "priority": "urgent",
            "category": "billing"
        }
    }
}
```

**Success Response Example:**
```json
{
    "success": true,
    "data": {
        "action_id": "550e8400-e29b-41d4-a716-446655440000",
        "external_response": {
            "status": "sent",
            "message_id": "msg_123456",
            "timestamp": "2024-01-01T12:00:00Z"
        },
        "customer_processed": true,
        "event_processed": true
    },
    "execution_time": 0.234
}
```

**Error Response Example:**
```json
{
    "success": false,
    "errors": {
        "company_key": ["Company not found"],
        "action": ["Action not found"],
        "event_data": ["Invalid event data structure"]
    },
    "execution_time": 0.045
}
```

---

## Performance Considerations

### HTTP Client Optimization
- GuzzleHttp client with connection pooling
- Promise-based asynchronous requests where possible
- Request timeout management
- Error handling to prevent hanging requests

### Data Processing Performance
- Efficient anchor replacement with minimal string operations
- Customer data caching within request context
- Field definition caching for custom field processing
- Event data validation optimization

### History Tracking Efficiency
- Asynchronous history storage when possible
- Batch history operations for high-volume scenarios
- Execution time tracking with microsecond precision
- Selective history data storage

### Memory Management
- Request/response data cleanup after processing
- Model instance reuse within request context
- Efficient array processing for large datasets
- Output buffer management for clean responses

---

## Error Handling

### Common Errors
- **"Company not found"** - Invalid company API key
- **"Action not found"** - Invalid action ID or UUID
- **EventDataForm validation errors** - Invalid event data structure
- **Customer data processing errors** - Invalid customer information

### HTTP Client Errors
- **RequestException** - External API communication failures
- **Timeout errors** - Request timeout exceeded
- **Connection errors** - Network connectivity issues
- **Response parsing errors** - Invalid response format

### Validation Errors
- Company API key format validation
- Action ID/UUID format validation
- Customer data structure validation
- Event data type and content validation

### System Errors
- Database connection failures during validation
- Model loading errors
- History storage failures
- Memory or processing limits

### Error Response Format
```json
{
    "success": false,
    "errors": {
        "field_name": ["Error message 1", "Error message 2"],
        "general": ["System error message"]
    },
    "execution_time": 0.123,
    "debug_info": {
        "request_id": "req_123456",
        "timestamp": "2024-01-01T12:00:00Z"
    }
}
```

---

## HTTP Status Codes
- **200 OK** - Successful action execution
- **400 Bad Request** - Validation failures or malformed requests
- **401 Unauthorized** - Invalid or missing company API key
- **404 Not Found** - Action or company not found
- **500 Internal Server Error** - System errors or external API failures

---

*ActionController API provides a robust and secure platform for executing company actions with comprehensive validation, HTTP client integration, and detailed activity tracking for enterprise automation workflows.*