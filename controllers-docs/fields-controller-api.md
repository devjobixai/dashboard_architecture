# FieldsController API Documentation

## Overview

FieldsController manages customer field definitions, values, and operations for the Jobix Dashboard API. The controller provides comprehensive functionality for field management including listing fields, managing field values, handling aggregations, and providing query building capabilities with operators and anchors support.

**Base Path:** `/api/v1/fields`  
**Controller Type:** ApiController (API Application)  
**Namespace:** `apps\api\versions\v1\controllers\FieldsController`  
**Authentication:** Requires `company-key` header

---

## Endpoints

### 1. List - Get Fields List

| Parameter | Value |
|-----------|-------|
| **URL** | `/api/v1/fields/list` |
| **Method** | GET |
| **Description** | Retrieve list of available customer fields with filtering support |

**Input Data (FieldsForm queryParams):**
- Query parameters for field filtering and pagination
- Company-based access control through authentication

**Business Logic:**
1. Parse query parameters through FieldsForm
2. Apply company-based filtering
3. Return formatted field list with metadata
4. Support pagination and sorting

**Response:**
- **Success (200):** Array of field definitions
- **Error (422):** FieldsForm object with validation errors

---

### 2. Flow Dropdown - Get Fields for Flow Dropdown

| Parameter | Value |
|-----------|-------|
| **URL** | `/api/v1/fields/flow-dropdown` |
| **Method** | GET |
| **Description** | Get field list optimized for flow builder dropdown components |

**Input Data (FlowDropdownFieldsForm queryParams):**
- Query parameters for dropdown field filtering
- Flow-specific field selection criteria

**Business Logic:**
1. Load fields suitable for flow builder dropdowns
2. Format for UI dropdown consumption
3. Apply flow-specific filtering
4. Return simplified field structure

**Response:**
- **Success (200):** Array of dropdown-optimized field data
- **Error (422):** FlowDropdownFieldsForm object with validation errors

---

### 3. Values - Get Field Values

| Parameter | Value |
|-----------|-------|
| **URL** | `/api/v1/fields/values` |
| **Method** | POST |
| **Description** | Retrieve distinct values for specified customer fields |

**Input Data (ValuesForm bodyParams):**
- Field identifiers for value retrieval
- Filtering criteria for value selection
- Company context for data isolation

**Business Logic:**
1. Validate field access permissions
2. Extract distinct field values from customer data
3. Apply filtering and pagination
4. Return formatted value list

**Response:**
- **Success (200):** Array/Object with field values
- **Error (422):** ValuesForm object with validation errors

---

### 4. Operators - Get Query Operators

| Parameter | Value |
|-----------|-------|
| **URL** | `/api/v1/fields/operators` |
| **Method** | GET |
| **Description** | Get available query operators for field filtering |

**Input Data:** None (static operator list)

**Business Logic:**
1. Access QueryBuilder component
2. Retrieve all available operators
3. Convert operators to array format
4. Return operator definitions with metadata

**Response:**
- **Success (200):** Array of operator objects with properties and methods

**Operator Structure:**
- Each operator contains metadata for query building
- Supports various comparison operations (equals, contains, greater than, etc.)
- Provides operator configuration for UI components

---

### 5. Anchors - Get Field Anchors

| Parameter | Value |
|-----------|-------|
| **URL** | `/api/v1/fields/anchors` |
| **Method** | GET |
| **Description** | Retrieve field anchor points for query building |

**Input Data (AnchorsForm queryParams):**
- Query parameters for anchor filtering
- Context-specific anchor selection

**Business Logic:**
1. Load field anchor definitions
2. Apply context-based filtering
3. Return anchor metadata for query building
4. Support hierarchical anchor structures

**Response:**
- **Success (200):** Array of anchor definitions
- **Error (422):** AnchorsForm object with validation errors

---

### 6. Customer Field Status - Change Field Status

| Parameter | Value |
|-----------|-------|
| **URL** | `/api/v1/fields/customer-field-status?uuid={field_uuid}` |
| **Method** | POST |
| **Description** | Update customer field status (active/inactive) |

**URL Parameters:**
- `uuid` (string, required) - UUID of the customer field

**Input Data (CustomerFieldStatusForm bodyParams):**
- Status change parameters
- Field configuration updates

**Business Logic:**
1. Validate field UUID and company access
2. Process status change request
3. Update field configuration
4. Return success status or validation errors

**Response:**
- **Success (200):** `true` boolean for successful status change
- **Error (422):** CustomerFieldStatusForm object with validation errors

---

### 7. Customer Field Aggregation - Get Field Aggregation Data

| Parameter | Value |
|-----------|-------|
| **URL** | `/api/v1/fields/customer-field-aggregation?uuid={field_uuid}` |
| **Method** | GET |
| **Description** | Get aggregated statistics and data for customer field |

**URL Parameters:**
- `uuid` (string, required) - UUID of the customer field

**Input Data (CustomerFieldAggregationForm queryParams):**
- Aggregation parameters and filters
- Statistical calculation options

**Business Logic:**
1. Validate field access and UUID
2. Calculate field aggregations (count, sum, average, etc.)
3. Generate statistical summaries
4. Return formatted aggregation results

**Response:**
- **Success (200):** Array with aggregation statistics
- **Error (422):** CustomerFieldAggregationForm object with validation errors

---

### 8. Delete Field - Delete Customer Field

| Parameter | Value |
|-----------|-------|
| **URL** | `/api/v1/fields/delete-field?uuid={field_uuid}` |
| **Method** | POST |
| **Description** | Delete customer field definition |

**URL Parameters:**
- `uuid` (string, required) - UUID of the customer field to delete

**Input Data:** None (UUID from URL parameter)

**Business Logic:**
1. Validate field UUID and ownership
2. Check field dependencies and usage
3. Execute safe field deletion
4. Clean up related data and references

**Response:**
- **Success (200):** Array with deletion confirmation
- **Error (422):** DeleteFieldForm object with validation errors

---

### 9. Clear Field Values - Clear Field Data

| Parameter | Value |
|-----------|-------|
| **URL** | `/api/v1/fields/clear-field-values?uuid={field_uuid}` |
| **Method** | POST |
| **Description** | Clear all values for specified customer field |

**URL Parameters:**
- `uuid` (string, required) - UUID of the customer field

**Input Data:** None (UUID from URL parameter)

**Business Logic:**
1. Validate field UUID and permissions
2. Clear all field values while preserving field definition
3. Update field statistics and metadata
4. Provide operation confirmation

**Response:**
- **Success (200):** Array with clear operation results
- **Error (422):** ClearFieldValuesForm object with validation errors

---

## Form Classes

### 1. FieldsForm

**File:** `src/apps/api/versions/v1/forms/fields/FieldsForm.php`  
**Extends:** `customs\Model`

**Purpose:** Handle field listing with filtering and pagination
**Method:** `list()` - Return formatted field list
**Features:**
- Company-based field filtering
- Pagination and sorting support
- Field metadata enrichment

---

### 2. FlowDropdownFieldsForm

**File:** `src/apps/api/versions/v1/forms/fields/FlowDropdownFieldsForm.php`  
**Extends:** `customs\Model`

**Purpose:** Provide field data optimized for flow builder dropdowns
**Method:** `list()` - Return dropdown-formatted field data
**Features:**
- Flow-specific field selection
- UI-optimized data structure
- Simplified field representation

---

### 3. ValuesForm

**File:** `src/apps/api/versions/v1/forms/fields/ValuesForm.php`  
**Extends:** `customs\Model`

**Purpose:** Extract and return distinct field values
**Method:** `list()` - Return field value collections
**Features:**
- Multi-field value extraction
- Distinct value filtering
- Pagination and search support

---

### 4. AnchorsForm

**File:** `src/apps/api/versions/v1/forms/fields/AnchorsForm.php`  
**Extends:** `customs\Model`

**Purpose:** Manage field anchor points for query building
**Method:** `list()` - Return anchor definitions
**Features:**
- Query building support
- Hierarchical anchor structures
- Context-aware anchor selection

---

### 5. CustomerFieldStatusForm

**File:** `src/apps/api/versions/v1/forms/fields/CustomerFieldStatusForm.php`  
**Extends:** `customs\Model`

**Purpose:** Handle customer field status changes
**Method:** `change()` - Process status update
**Features:**
- Field activation/deactivation
- Status validation
- Configuration updates

---

### 6. CustomerFieldAggregationForm

**File:** `src/apps/api/versions/v1/forms/fields/CustomerFieldAggregationForm.php`  
**Extends:** `customs\Model`

**Purpose:** Calculate and return field aggregation statistics
**Method:** `list()` - Return aggregation data
**Features:**
- Statistical calculations
- Data summarization
- Performance-optimized queries

---

### 7. DeleteFieldForm

**File:** `src/apps/api/versions/v1/forms/fields/DeleteFieldForm.php`  
**Extends:** `customs\Model`

**Purpose:** Handle safe field deletion operations
**Method:** `delete()` - Execute field deletion
**Features:**
- Dependency checking
- Safe deletion process
- Cleanup operations

---

### 8. ClearFieldValuesForm

**File:** `src/apps/api/versions/v1/forms/fields/ClearFieldValuesForm.php`  
**Extends:** `customs\Model`

**Purpose:** Clear field values while preserving field definition
**Method:** `clear()` - Execute value clearing
**Features:**
- Value-only clearing
- Metadata preservation
- Batch operations support

---

## Business Logic

### 1. Field Management Flow
```
1. Field Definition Management
   - Create and configure custom fields
   - Set field types and validation rules
   - Manage field visibility and permissions

2. Field Value Operations
   - Extract distinct values for filtering
   - Calculate aggregations and statistics
   - Clear field data when needed

3. Query Building Support
   - Provide operators for filtering
   - Supply anchor points for queries
   - Support dropdown field selection
```

### 2. Integration with Flow Builder
```
1. Flow Dropdown Integration
   - Optimized field list for UI components
   - Simplified field structure
   - Context-aware field filtering

2. Query Builder Integration
   - Operator definitions for filtering
   - Anchor points for query construction
   - Field value suggestions for inputs
```

### 3. Data Management
```
1. Field Lifecycle Management
   - Field creation and configuration
   - Status management (active/inactive)
   - Safe deletion with dependency checks

2. Value Management
   - Distinct value extraction
   - Statistical aggregation
   - Bulk value clearing operations
```

---

## Security Features

### 1. Company-based Access Control
- All operations filtered by company context
- Field access validation through authentication
- UUID-based field identification with ownership checks

### 2. Input Validation
- UUID validation for all field identifiers
- Parameter sanitization for query operations
- Form-based validation for all endpoints

### 3. Safe Operations
- Dependency checking before field deletion
- Validation of field modifications
- Proper error handling and reporting

---

## Integration Points

### Dependencies
- `QueryBuilder` - Query operator management
- `UuidHelper` - UUID validation and processing
- `OperatorAbstract` - Query operator definitions
- Company authentication system
- Customer field models

### Related Components
- **Flow Builder** - Field dropdown integration
- **Customer Management** - Field value storage
- **Query System** - Operator and anchor definitions
- **Statistics System** - Field aggregation calculations

### External Integrations
- Customer data storage systems
- Field definition management
- Query building interfaces
- Statistical calculation engines

---

## Usage Examples

### Get fields list
```http
GET /api/v1/fields/list?page=1&limit=50
company-key: your-company-api-key
```

### Get field values
```http
POST /api/v1/fields/values
Content-Type: application/json
company-key: your-company-api-key

{
  "field_uuid": "550e8400-e29b-41d4-a716-446655440000",
  "filters": {
    "search": "example"
  }
}
```

### Get query operators
```http
GET /api/v1/fields/operators
company-key: your-company-api-key
```

### Change field status
```http
POST /api/v1/fields/customer-field-status?uuid=550e8400-e29b-41d4-a716-446655440000
Content-Type: application/json
company-key: your-company-api-key

{
  "status": true,
  "configuration": {
    "visible": true,
    "required": false
  }
}
```

### Get field aggregation
```http
GET /api/v1/fields/customer-field-aggregation?uuid=550e8400-e29b-41d4-a716-446655440000&type=statistics
company-key: your-company-api-key
```

### Delete field
```http
POST /api/v1/fields/delete-field?uuid=550e8400-e29b-41d4-a716-446655440000
company-key: your-company-api-key
```

---

## Error Handling

### Common Errors
- **"Invalid field UUID"** - Field not found or access denied
- **"Field has dependencies"** - Cannot delete field in use
- **"Company access denied"** - Invalid company context
- **"Validation failed"** - Form validation errors

### Field Operation Errors
- Field deletion conflicts with existing data
- Status change validation failures
- Value extraction permission errors
- Aggregation calculation failures

### HTTP Status Codes
- **200 OK** - Successful operations
- **422 Unprocessable Entity** - Validation errors or business logic failures

---

*FieldsController API provides comprehensive customer field management with advanced query building capabilities, statistical operations, and flow builder integration.*