# CustomerListsController Documentation

## Overview

CustomerListsController manages customer lists within the company in the Jobix Dashboard administrative panel. The controller provides comprehensive CRUD functionality for managing customer lists with advanced filtering capabilities, MongoDB integration for customer data queries, and support for different list types and complex filter conditions.

**Base Path:** `/customer-lists`  
**Controller Type:** WebController (Admin Application)  
**Namespace:** `apps\admin\controllers\CustomerListsController`

---

## Endpoints

### 1. Index - Customer Lists Overview

| Parameter | Value |
|-----------|-------|
| **URL** | `/customer-lists/index` |
| **Method** | GET |
| **Description** | Display list of all customer lists with search and filtering capabilities |

**Input Data (CustomerListSearch queryParams):**

| Field | Type | Required | Validation | Description |
|-------|------|----------|-----------|-------------|
| `name` | string | ✗ | string | Search by list name (ILIKE) |
| `description` | string | ✗ | string | Search by list description (ILIKE) |
| `count_from` | integer | ✗ | string → int | Minimum customers count filter |
| `count_to` | integer | ✗ | string → int | Maximum customers count filter |
| `type` | integer | ✗ | string → int | Filter by list type |
| `status` | integer | ✗ | string → int | Filter by list status |
| `modify_time` | string | ✗ | string | Date range filter (format: "start - end") |

**Business Logic:**
1. Create CustomerListSearch for filtering
2. Load search parameters from query string
3. Execute search with company-based isolation
4. Apply ILIKE searches for text fields (name, description)
5. Apply numeric range filtering for customers count
6. Apply date range filtering with Carbon parsing
7. Default to "not deleted" if no status specified
8. Pagination (25 elements per page)
9. Default sorting by updated_at DESC

**Response:**
- **Success:** HTML view with customer lists
- **Data:** `pageName`, `searchModel`, `dataProvider`

**Available Sorting:**
- `name`, `description`, `customers_count`, `type`, `status`, `updated_at`

---

### 2. Create - Create New Customer List

| Parameter | Value |
|-----------|-------|
| **URL** | `/customer-lists/create` |
| **Methods** | GET, POST |
| **Description** | Create new customer list |

#### GET `/customer-lists/create`
**Purpose:** Display creation form

#### POST `/customer-lists/create`
**Purpose:** Process customer list creation

**Input Data (CreateCustomerListForm bodyParams):**

| Field | Type | Required | Validation | Description |
|-------|------|----------|-----------|-------------|
| `name` | string | ✓ | required, string, min:5, max:255, trim | List name |
| `description` | string | ✗ | string, max:1024, trim | List description |
| `type` | integer | ✗ | default: TYPE_FILTER, in types range | List type |
| `status` | boolean | ✗ | boolean, default: true | List status |

**Detailed Validation CreateCustomerListForm:**
```php
// Required fields
[['name'], 'required']

// Type validation with default
[['type'], 'default', 'value' => CompanyCustomerListsAdvanced::TYPE_FILTER]
[['type'], 'in', 'range' => CompanyCustomerListsAdvanced::types()]

// String validation with trimming
[['name', 'description'], 'filter', 'filter' => 'trim']
[['name'], 'string', 'min' => 5, 'max' => 255]
[['description'], 'string', 'max' => 1024]

// Boolean validation
[['status'], 'boolean']

// Company validation
[['name'], 'setCompany'] // Custom validator to ensure selected company exists
```

**Company Validation Logic:**
```php
public function setCompany(string $attrName): void
{
    $this->_company = CompaniesAdvanced::findByParams([
        'uuid' => \Yii::$app->user->identity->getSelectedCompanyUuid(),
    ]);
    if (is_null($this->_company)) {
        $this->addError($attrName, 'Please select company!');
    }
}
```

**Business Logic:**
1. Validate form data including company selection
2. Generate UUID for new customer list
3. Set company_id from selected company
4. Set customers_count to 0 initially
5. Convert boolean status to proper database format
6. Call CompanyCustomerListsAdvanced::registerCustomerList()

**Responses:**
- **Success:** Redirect to `/customer-lists/edit?uuid={list_uuid}`
- **Validation Error:** HTML view with form and errors
- **GET:** HTML view with empty form

---

### 3. Edit - Edit Customer List

| Parameter | Value |
|-----------|-------|
| **URL** | `/customer-lists/edit?uuid={list_uuid}` |
| **Methods** | GET, POST |
| **Description** | Edit existing customer list basic information |

**URL Parameters:**
- `uuid` (string, required) - UUID of the customer list

#### GET `/customer-lists/edit`
**Purpose:** Display edit form

**Behavior:**
- Automatic data loading in constructor
- Populate form with current list data and settings

#### POST `/customer-lists/edit`
**Purpose:** Process customer list update

**Input Data (EditCustomerListForm bodyParams):**

| Field | Type | Required | Validation | Description |
|-------|------|----------|-----------|-------------|
| `uuid` | string | ✓ | required | List UUID |
| `name` | string | ✓ | required, string, min:5, max:255, trim | List name |
| `description` | string | ✗ | string, max:1024, trim | List description |
| `type` | integer | ✗ | in types range | List type |
| `status` | boolean | ✗ | boolean | List status |

**EditCustomerListForm extends CreateCustomerListForm:**
```php
class EditCustomerListForm extends CreateCustomerListForm
{
    public string $uuid = '';
    protected object|null $_customer_list = null;
    
    public function __construct(string $uuid)
    {
        $customer_list = CompanyCustomerListsAdvanced::findByParams(['uuid' => $this->uuid]);
        if (!is_null($customer_list)) {
            $this->_customer_list = $customer_list;
            $params = ArrayHelper::toArray($this->_customer_list);
            // Merge main attributes with settings
            $this->setAttributes(ArrayHelper::merge($params, $params['settings']));
        }
    }
}
```

**Business Logic:**
1. Load existing customer list in constructor
2. Populate form with current data including settings
3. Validate updated data with parent form rules
4. Update customer list with new data and current timestamp
5. Call CompanyCustomerListsAdvanced::updateCustomerList()

**Responses:**
- **Success:** Redirect to `/customer-lists/edit?uuid={list_uuid}`
- **Validation Error:** HTML view with form and errors
- **Not Found:** Redirect to `/customer-lists/index`
- **GET:** HTML view with populated form

---

### 4. Filters - Manage List Filters

| Parameter | Value |
|-----------|-------|
| **URL** | `/customer-lists/filters?uuid={list_uuid}` |
| **Methods** | GET, POST, PJAX |
| **Description** | Configure complex JSON-based filters for customer list with MongoDB integration |

**URL Parameters:**
- `uuid` (string, required) - UUID of the customer list

#### GET `/customer-lists/filters`
**Purpose:** Display filters management interface

**Behavior:**
- Load existing customer list
- Populate form with current filters (JSON format)
- Display filter preview with customer count

#### PJAX `/customer-lists/filters`
**Purpose:** Dynamic filter preview

**Input:**
- `filters` (JSON) - Filter conditions for preview

**Behavior:**
- Validate filters using QueryBuilder
- Return updated filters view without full page reload

#### POST `/customer-lists/filters`
**Purpose:** Save filter configuration

**Input Data (FilterCustomerListForm bodyParams):**

| Field | Type | Required | Validation | Description |
|-------|------|----------|-----------|-------------|
| `uuid` | string | ✓ | required | List UUID |
| `filters` | string | ✗ | JSON format, custom validation | JSON-encoded filter conditions |

**Detailed Validation FilterCustomerListForm:**
```php
[['uuid'], 'required']
[['filters'], JsonValidator::class]
[['filters'], 'validateFilters'] // Custom validation using QueryBuilder

public function validateFilters($attribute): void
{
    $filters = json_decode($this->$attribute, true);
    
    /** @var QueryBuilder $queryBuilder */
    $queryBuilder = Yii::$app->get('queryBuilder');
    
    $errors = $queryBuilder->validateConditions($filters, $attribute);
    
    if (!empty($errors)) {
        $this->filter_errors = json_encode($errors);
        $this->addError($attribute, 'Invalid conditions data');
    }
}
```

**MongoDB Integration:**
```php
public function getCustomersPager(): array
{
    /** @var Connection $mongo */
    $mongo = Yii::$app->mongodb;
    
    /** @var QueryBuilder $queryBuilder */
    $queryBuilder = Yii::$app->get('queryBuilder');
    
    $collection = $mongo->getCollection(MongoHelper::MAIN_TABLE);
    
    $filters = json_decode($this->filters, true);
    
    $pipeline = [
        ['$match' => ['company_id' => $this->selectedUserCompany->id]]
    ];
    
    if (!empty($filters)) {
        $parseFilters = $queryBuilder->mongodbQuery($filters);
        $conditions = $mongo->queryBuilder->buildCondition($parseFilters->where);
        $pipeline[] = ['$match' => $conditions];
    }
    
    $pipeline[] = ['$sort' => ['_created_at' => -1]];
    $pipeline[] = ['$skip' => $pagination->offset];
    $pipeline[] = ['$limit' => $pagination->limit];
    
    return $collection->aggregate($pipeline);
}
```

**Business Logic:**
1. Load existing customer list and current filters
2. Validate JSON filter structure using QueryBuilder
3. Preview filtered customers using MongoDB aggregation
4. Save validated filters to customer list model
5. Handle filter errors and provide feedback

**Responses:**
- **Success:** Redirect to `/customer-lists/filters?uuid={list_uuid}`
- **PJAX:** Partial HTML with filter preview
- **Validation Error:** HTML view with errors
- **Not Found:** Redirect to `/customer-lists/index`
- **GET:** HTML view with filters management interface

---

### 5. Customers - View List Customers

| Parameter | Value |
|-----------|-------|
| **URL** | `/customer-lists/customers?uuid={list_uuid}` |
| **Methods** | GET |
| **Description** | Display customers assigned to specific customer list |

**URL Parameters:**
- `uuid` (string, required) - UUID of the customer list

**Input Data (SearchCustomersListForm):**

| Field | Type | Required | Validation | Description |
|-------|------|----------|-----------|-------------|
| `uuid` | string | ✓ | required | List UUID |

**Business Logic:**
1. Validate customer list UUID
2. Execute JOIN query with company_customer_list_data table
3. Filter customers by company and list assignment
4. Apply pagination (50 customers per page)
5. Sort by created_at DESC

**Database Query:**
```php
$jobs = CompanyCustomersAdvanced::find()
    ->whereCompanyId($this->selectedUserCompany->id)
    ->innerJoin('company_customer_list_data',
        "company_customer_list_data.customer_id = company_customers.id AND 
         company_customer_list_data.company_customer_list_id = {$this->customerList->id}")
    ->orderBy(['created_at' => SORT_DESC]);
```

**Responses:**
- **Success:** HTML view with customers list
- **Data:** `model`, `dataProvider`
- **Not Found:** NotFoundHttpException if list not found

**Available Data Fields:**
- `suid` - Source unique ID
- `name` - Customer name
- `email` - Customer email
- `phone` - Customer phone

---

### 6. Delete - Delete Customer List

| Parameter | Value |
|-----------|-------|
| **URL** | `/customer-lists/delete?uuid={list_uuid}` |
| **Method** | POST |
| **Description** | Delete customer list |

**URL Parameters:**
- `uuid` (string, required) - UUID of customer list to delete

**Business Logic:**
1. Load customer list by UUID in constructor
2. Call CompanyCustomerListsAdvanced::deleteCustomerList()
3. Return boolean success status
4. Redirect to index regardless of result

**Delete Process:**
```php
class DeleteCustomerListForm
{
    public function __construct(string $uuid)
    {
        $this->uuid = $uuid;
        $customer_list = CompanyCustomerListsAdvanced::findByParams(['uuid' => $this->uuid]);
        if (!is_null($customer_list)) {
            $this->_customer_list = $customer_list;
        }
    }
    
    public function process(): bool
    {
        $deleteCount = CompanyCustomerListsAdvanced::deleteCustomerList(['uuid' => $this->uuid]);
        return $deleteCount >= 1;
    }
}
```

**Responses:**
- **Always:** Redirect to `/customer-lists/index`
- **Success/Failure:** Handled by static method return value

---

## Form Classes

### 1. CustomerListSearch

**File:** `src/apps/admin/forms/customerlist/CustomerListSearch.php`  
**Extends:** `customs\Model` (143 lines)

**Public Properties:**
```php
public mixed $name = null;            // List name search (ILIKE)
public mixed $description = null;     // List description search (ILIKE)
public mixed $count_from = null;      // Minimum customers count
public mixed $count_to = null;        // Maximum customers count
public mixed $type = null;            // List type filter
public mixed $status = null;          // List status filter
public mixed $modify_time = null;     // Date range filter
```

**Search Features:**
- **ILIKE Searches:** name, description with case-insensitive matching
- **Range Filtering:** customers count between count_from and count_to
- **Type Filtering:** filter by list type with validation
- **Status Filtering:** filter by status or default to "not deleted"
- **Date Range Filtering:** modify_time with Carbon date parsing
- **Company Isolation:** automatic filtering by selectedUserCompany

**Pagination:**
- Default page size: 25 elements
- Custom pagination support

**Sorting Options:**
- `name`, `description`, `customers_count`, `type`, `status`, `updated_at`
- Default order: `updated_at DESC`

**Static Methods:**
- `getTypes()` - returns CompanyCustomerListsAdvanced::typesList() for dropdowns

---

### 2. CreateCustomerListForm

**File:** `src/apps/admin/forms/customerlist/CreateCustomerListForm.php`  
**Extends:** `customs\Model` (106 lines)

**Public Properties:**
```php
public mixed $name = '';              // List name (required, 5-255 chars)
public mixed $description = '';       // List description (max 1024 chars)
public mixed $type = null;            // List type (default: TYPE_FILTER)
public bool $status = true;           // List status (default: true)
```

**Protected Properties:**
```php
protected mixed $_company = null;     // Selected company model
```

**Key Features:**
- **Company Validation:** Custom setCompany() validator ensures selected company exists
- **Type Defaulting:** Automatic TYPE_FILTER default with validation against types list
- **String Trimming:** Automatic trimming for name and description fields
- **UUID Generation:** Automatic UUID generation in process() method
- **Status Conversion:** Boolean to database format conversion

**Process Method:**
```php
public function process(): array
{
    return CompanyCustomerListsAdvanced::registerCustomerList(
        ArrayHelper::merge($this->attributes, [
            'uuid' => UuidHelper::generate(),
            'company_id' => $this->_company->id,
            'type' => $this->type,
            'customers_count' => 0,
            'status' => ($this->status ? ActiveRecordPostgres::STATUSES['active'] : 
                                       ActiveRecordPostgres::STATUSES['inactive'])
        ])
    );
}
```

---

### 3. EditCustomerListForm

**File:** `src/apps/admin/forms/customerlist/EditCustomerListForm.php`  
**Extends:** `CreateCustomerListForm` (84 lines)

**Additional Properties:**
```php
public string $uuid = '';                      // List UUID (required)
protected object|null $_customer_list = null; // Loaded customer list model
```

**Constructor Logic:**
```php
public function __construct(string $uuid)
{
    parent::__construct();
    $this->uuid = $uuid;
    
    $customer_list = CompanyCustomerListsAdvanced::findByParams(['uuid' => $this->uuid]);
    
    if (!is_null($customer_list)) {
        $this->_customer_list = $customer_list;
        $params = ArrayHelper::toArray($this->_customer_list);
        // Merge main attributes with settings for complex data loading
        $this->setAttributes(ArrayHelper::merge($params, $params['settings']));
    }
}
```

**Key Features:**
- **Inheritance:** Extends CreateCustomerListForm for shared validation
- **Auto-population:** Constructor automatically loads and populates data
- **Settings Integration:** Merges main attributes with settings field
- **Update Process:** Uses CompanyCustomerListsAdvanced::updateCustomerList()
- **Timestamp Management:** Automatic updated_at with Carbon

---

### 4. FilterCustomerListForm

**File:** `src/apps/admin/forms/customerlist/FilterCustomerListForm.php`  
**Extends:** `customs\Model` (139 lines)

**Public Properties:**
```php
public mixed $uuid = null;               // List UUID (required)
public mixed $filters = '[]';            // JSON-encoded filter conditions
public mixed $filter_errors = '[]';      // JSON-encoded validation errors
```

**Protected Properties:**
```php
protected CompanyCustomerListsAdvanced|null $_customerList = null; // List model
```

**Advanced Features:**
- **JSON Validation:** Custom JsonValidator for filters structure
- **QueryBuilder Integration:** Uses QueryBuilder component for filter validation
- **MongoDB Integration:** Advanced MongoDB aggregation for filtered results
- **Error Tracking:** Separate filter_errors field for detailed error reporting
- **PJAX Support:** Supports partial page updates for filter preview

**MongoDB Query Pipeline:**
```php
$pipeline = [
    ['$match' => ['company_id' => $this->selectedUserCompany->id]]
];

if (!empty($filters)) {
    $parseFilters = $queryBuilder->mongodbQuery($filters);
    $conditions = $mongo->queryBuilder->buildCondition($parseFilters->where);
    $pipeline[] = ['$match' => $conditions];
}

$pipeline[] = ['$sort' => ['_created_at' => -1]];
$pipeline[] = ['$skip' => $pagination->offset];
$pipeline[] = ['$limit' => $pagination->limit];

$customers = $collection->aggregate($pipeline);
```

**Key Methods:**
- `populate()` - loads existing filters from customer list
- `save()` - validates and saves filters with error handling
- `getCustomersPager()` - returns MongoDB-based filtered results with pagination

---

### 5. SearchCustomersListForm

**File:** `src/apps/admin/forms/customerlist/SearchCustomersListForm.php`  
**Extends:** `customs\Model` (69 lines)

**Public Properties:**
```php
public mixed $uuid = null;  // List UUID (required)
```

**Protected Properties:**
```php
protected CompanyCustomerListsAdvanced|null $_customerList = null; // List model
```

**Key Features:**
- **JOIN Query:** INNER JOIN with company_customer_list_data table
- **Company Isolation:** Filters by selectedUserCompany
- **Pagination:** 50 customers per page
- **Sorting:** Default created_at DESC ordering
- **Attribute Labels:** Localized labels for UI display

**Database Query:**
```php
$jobs = CompanyCustomersAdvanced::find()
    ->whereCompanyId($this->selectedUserCompany->id)
    ->innerJoin('company_customer_list_data',
        "company_customer_list_data.customer_id = company_customers.id AND 
         company_customer_list_data.company_customer_list_id = {$this->customerList->id}")
    ->orderBy(['created_at' => SORT_DESC]);
```

---

### 6. DeleteCustomerListForm

**File:** `src/apps/admin/forms/customerlist/DeleteCustomerListForm.php`  
**Simple Class** (52 lines)

**Public Properties:**
```php
public string $uuid = '';  // List UUID
```

**Protected Properties:**
```php
protected object|null $_customer_list = null; // Loaded list model
```

**Key Features:**
- **Simple Design:** No validation, direct deletion process
- **Constructor Loading:** Automatic model loading by UUID
- **Static Method Call:** Uses CompanyCustomerListsAdvanced::deleteCustomerList()
- **Boolean Return:** Simple success/failure indication

---

## Business Logic

### 1. Customer List Creation Flow
```
1. Display creation form (GET)
2. Validate list data (name length, type, company selection)
3. Generate UUID for new list
4. Set company_id from selected company
5. Initialize customers_count to 0
6. Convert boolean status to database format
7. Call registerCustomerList() static method
8. Redirect to edit page for further configuration
```

### 2. Customer List Editing Flow
```
1. Load existing list data in constructor (GET)
2. Populate form with current data and settings
3. Display form with current values
4. Validate updated data using parent form rules
5. Update list with new data and current timestamp
6. Call updateCustomerList() static method
7. Redirect to same edit page with success message
```

### 3. Advanced Filtering Flow
```
1. Load existing list and current filters (GET)
2. Display filters management interface
3. Validate JSON filter structure using QueryBuilder
4. Preview filtered results using MongoDB aggregation pipeline
5. Save validated filters to list model (POST)
6. Handle PJAX requests for dynamic filter preview
7. Provide detailed error feedback for invalid filters
```

### 4. Customer Viewing Flow
```
1. Validate list UUID and load list model
2. Execute JOIN query with company_customer_list_data
3. Filter customers by company and list assignment
4. Apply pagination (50 customers per page)
5. Sort by creation date descending
6. Display customers with localized attribute labels
```

### 5. Customer List Search Flow
```
1. Load search parameters from query
2. Apply company-based filtering automatically
3. Apply ILIKE searches for text fields (name, description)
4. Apply numeric range filtering for customers count
5. Apply type and status filtering
6. Apply date range filtering with Carbon parsing
7. Default to "not deleted" if no status specified
8. Apply sorting and pagination (25 elements)
9. Return ActiveDataProvider with results
```

---

## Security Features

### 1. Company-based Access Control
- All operations isolated by selectedUserCompany
- List existence validation within company
- Customer results limited to company data
- MongoDB queries filtered by company_id

### 2. Input Validation
- String length limits for all text fields
- Name trimming to prevent whitespace issues
- UUID validation for all UUID operations
- JSON structure validation for filters
- Type validation against allowed types list

### 3. MongoDB Security
- Company-based filtering in all MongoDB queries
- QueryBuilder validation for filter conditions
- Proper pipeline construction with company isolation
- Error handling for invalid filter structures

### 4. Advanced Filter Validation
- JsonValidator for filter structure
- QueryBuilder integration for condition validation
- Error tracking with detailed feedback
- PJAX security for partial updates

---

## Integration Points

### Dependencies
- `CompanyCustomerListsAdvanced` - main customer list model
- `CompanyCustomersAdvanced` - customer model for list assignments
- `CompaniesAdvanced` - company model for access control
- `UuidHelper` - UUID validation and generation
- `Carbon` - date parsing and manipulation
- `QueryBuilder` - advanced filter validation and MongoDB query building
- `MongoHelper` - MongoDB utilities and table constants
- MongoDB connection - customer data storage and querying

### MongoDB Integration
- Customer data stored in MongoDB collections
- Advanced aggregation pipelines for filtering
- Company-based data isolation
- Real-time filter preview with PJAX

### Related Models
- `company_customer_list_data` - junction table for list-customer assignments
- Customer models for data relationships
- Company models for access control

---

## Usage Examples

### Create customer list
```http
POST /customer-lists/create
Content-Type: application/x-www-form-urlencoded

CreateCustomerListForm[name]=VIP Customers&CreateCustomerListForm[description]=High value customers&CreateCustomerListForm[type]=1&CreateCustomerListForm[status]=1
```

### Search lists with filters
```http
GET /customer-lists/index?name=VIP&count_from=100&count_to=1000&type=1&status=1&modify_time=2024-01-01 - 2024-12-31
```

### Configure list filters
```http
POST /customer-lists/filters?uuid=550e8400-e29b-41d4-a716-446655440000
Content-Type: application/x-www-form-urlencoded

FilterCustomerListForm[filters]={"conditions":[{"field":"email","operator":"contains","value":"@gmail.com"}],"logic":"and"}
```

### Edit customer list
```http
POST /customer-lists/edit?uuid=550e8400-e29b-41d4-a716-446655440000
Content-Type: application/x-www-form-urlencoded

EditCustomerListForm[name]=Premium Customers&EditCustomerListForm[description]=Updated description&EditCustomerListForm[type]=2
```

### View list customers
```http
GET /customer-lists/customers?uuid=550e8400-e29b-41d4-a716-446655440000
```

### Delete customer list
```http
POST /customer-lists/delete?uuid=550e8400-e29b-41d4-a716-446655440000
```

---

## Performance Considerations

### Database Optimization
- Company-based indexing for all queries
- ILIKE searches may be resource-intensive
- JOIN operations with customer list data table
- Date range queries with proper indexing

### MongoDB Performance
- Aggregation pipelines with proper indexing
- Company-based filtering at database level
- Pagination for large result sets
- Complex filter validation overhead

### Advanced Filtering
- QueryBuilder validation processing overhead
- MongoDB aggregation pipeline complexity
- Real-time filter preview performance
- PJAX update efficiency

---

## Error Handling

### Common Errors
- **"Company Customers List not found!"** - Invalid UUID or no access
- **"Please select company!"** - No company selected in session
- **"Invalid conditions data"** - Filter validation failure
- **Name validation errors** - Length or trimming issues

### MongoDB Integration Errors
- Connection failures to MongoDB
- Invalid aggregation pipeline construction
- Query timeout for complex filters
- Data type mismatches in filter conditions

### Filter Validation Errors
- JSON structure validation failures
- QueryBuilder condition validation errors
- Unsupported filter operators
- Invalid field references

### HTTP Status Codes
- **200** - Success responses
- **404** - NotFoundHttpException for missing lists
- **500** - System errors with user-friendly messages
- **Redirect** - Success operations redirect to appropriate pages

---

*CustomerListsController is a sophisticated component for managing customer lists with advanced MongoDB-based filtering capabilities, comprehensive CRUD operations, and complex customer data management with real-time filter preview and validation.*