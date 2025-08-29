# NodesController API Documentation

## Overview

NodesController manages flow builder nodes for the Jobix Dashboard API. The controller provides comprehensive CRUD functionality for workflow nodes including creation, updating, evaluation, position management, execution, and filtering operations. It serves as the core API for flow builder visual node management and workflow automation.

**Base Path:** `/api/v1/nodes`  
**Controller Type:** ApiController (API Application)  
**Namespace:** `apps\api\versions\v1\controllers\NodesController`  
**Authentication:** Requires `company-key` header

---

## Endpoints

### 1. Evaluate - Evaluate Node Logic

| Parameter | Value |
|-----------|-------|
| **URL** | `/api/v1/nodes/evaluate` |
| **Method** | POST |
| **Description** | Evaluate node logic and conditions for workflow processing |

**Input Data (NodeForm bodyParams):**
- Node configuration and evaluation parameters
- Workflow context data for processing
- Conditional logic parameters

**Business Logic:**
1. Parse node configuration from request body
2. Initialize node evaluation context
3. Execute node logic evaluation
4. Process conditional statements and rules
5. Return evaluation results or validation errors

**Response:**
- **Success (200):** Array with evaluation results
- **Error (422):** NodeForm object with validation errors

---

### 2. Save - Save Multiple Nodes

| Parameter | Value |
|-----------|-------|
| **URL** | `/api/v1/nodes/save` |
| **Method** | POST |
| **Description** | Save multiple nodes in batch operation for workflow updates |

**Input Data (NodeForm bodyParams):**
- Array of node configurations
- Batch operation parameters
- Workflow context information

**Business Logic:**
1. Validate batch node data structure
2. Process multiple node configurations
3. Execute batch save operations with transaction safety
4. Update node relationships and dependencies
5. Return batch operation results

**Response:**
- **Success (200):** Array with batch save results
- **Error (422):** NodeForm object with validation errors

---

### 3. Create - Create New Node

| Parameter | Value |
|-----------|-------|
| **URL** | `/api/v1/nodes/create` |
| **Method** | POST |
| **Description** | Create new workflow node with configuration |

**Input Data (NodeForm bodyParams):**
- Node type and configuration parameters
- Position and visual settings
- Workflow integration settings

**Business Logic:**
1. Validate new node configuration
2. Initialize node with default settings
3. Create node record with proper relationships
4. Set up node positioning and visual properties
5. Return created node data or validation errors

**Response:**
- **Success (200):** ViewNode object with created node data
- **Error (422):** NodeForm object with validation errors
- **Database Error:** NodeForm with exception details

---

### 4. Update - Update Existing Node

| Parameter | Value |
|-----------|-------|
| **URL** | `/api/v1/nodes/update?uuid={node_uuid}` |
| **Method** | POST |
| **Description** | Update existing workflow node configuration |

**URL Parameters:**
- `uuid` (string, required) - UUID of the node to update

**Input Data (NodeForm bodyParams):**
- Updated node configuration parameters
- Modified settings and properties
- Relationship updates

**Business Logic:**
1. Validate node UUID and ownership
2. Load existing node configuration
3. Apply configuration updates
4. Validate updated node settings
5. Save changes and update relationships

**Response:**
- **Success (200):** ViewNode object with updated node data
- **Error (422):** NodeForm object with validation errors
- **Database Error:** NodeForm with exception details

---

### 5. View - Get Node Details

| Parameter | Value |
|-----------|-------|
| **URL** | `/api/v1/nodes/view?uuid={node_uuid}` |
| **Method** | GET |
| **Description** | Retrieve detailed information about specific workflow node |

**URL Parameters:**
- `uuid` (string, required) - UUID of the node to view

**Input Data:** None (UUID from URL parameter)

**Business Logic:**
1. Validate node UUID and access permissions
2. Load node configuration and metadata
3. Include relationships and dependencies
4. Return formatted node data

**Response:**
- **Success (200):** ViewNode object with complete node data
- **Error (422):** NodeForm object with validation errors
- **Not Found:** `null` if node doesn't exist

---

### 6. Delete - Delete Node

| Parameter | Value |
|-----------|-------|
| **URL** | `/api/v1/nodes/delete?uuid={node_uuid}` |
| **Method** | POST |
| **Description** | Delete workflow node with dependency handling |

**URL Parameters:**
- `uuid` (string, required) - UUID of the node to delete

**Input Data:** None (UUID from URL parameter)

**Business Logic:**
1. Validate node UUID and ownership
2. Check node dependencies and relationships
3. Execute safe node deletion
4. Clean up related data and connections
5. Update workflow structure

**Response:**
- **Success (200):** NodeForm with deletion confirmation
- **Error (422):** NodeForm object with validation or dependency errors
- **Database Error:** NodeForm with exception details

---

### 7. Update Position - Update Node Position

| Parameter | Value |
|-----------|-------|
| **URL** | `/api/v1/nodes/update-position?uuid={node_uuid}` |
| **Method** | POST |
| **Description** | Update visual position of node in flow builder |

**URL Parameters:**
- `uuid` (string, required) - UUID of the node

**Input Data (POST bodyParams):**
- `position` - New position coordinates and layout data

**Business Logic:**
1. Validate node UUID and access
2. Process new position data
3. Update node visual positioning
4. Maintain flow builder layout consistency
5. Return updated node information

**Response:**
- **Success (200):** ViewNode object with updated position
- **Error (422):** NodeForm object with validation errors
- **Database Error:** NodeForm with exception details

---

### 8. Run - Execute Node for Client

| Parameter | Value |
|-----------|-------|
| **URL** | `/api/v1/nodes/run?uuid={node_uuid}` |
| **Method** | POST |
| **Description** | Execute node logic for specific client workflow |

**URL Parameters:**
- `uuid` (string, required) - UUID of the node to execute

**Input Data (RunNodesForClientForm bodyParams):**
- Client context and execution parameters
- Workflow state and variables
- Execution configuration settings

**Business Logic:**
1. Validate node execution permissions
2. Load client context and workflow state
3. Execute node logic with client data
4. Process node outputs and state changes
5. Return execution results

**Response:**
- **Success (200):** Array/Object with execution results
- **Error (422):** RunNodesForClientForm object with validation errors

---

### 9. Filter Count - Count Filtered Nodes

| Parameter | Value |
|-----------|-------|
| **URL** | `/api/v1/nodes/filter-count` |
| **Method** | POST |
| **Description** | Count nodes matching specified filter criteria |

**Input Data (FilterCountForm bodyParams):**
- `filters` - Filter criteria for node counting

**Business Logic:**
1. Parse filter criteria from request
2. Apply filters to node dataset
3. Calculate count of matching nodes
4. Return count result or validation errors

**Response:**
- **Success (200):** Integer count of matching nodes
- **Error (422):** FilterCountForm object with validation errors

---

## Form Classes

### 1. NodeForm

**File:** `src/apps/api/versions/v1/forms/node/NodeForm.php`  
**Extends:** `customs\Model`

**Purpose:** Primary form class for all node operations
**Key Methods:**
- `evaluate()` - Process node logic evaluation
- `multipleSave()` - Handle batch node save operations
- `create()` - Create new workflow node
- `update()` - Update existing node configuration
- `view()` - Retrieve node details
- `delete()` - Delete node with cleanup
- `updatePosition()` - Update node visual position

**Features:**
- Comprehensive node validation
- Batch operations support
- Position and layout management
- Relationship handling
- Error management with exceptions

---

### 2. RunNodesForClientForm

**File:** `src/apps/api/versions/v1/forms/node/RunNodesForClientForm.php`  
**Extends:** `customs\Model`

**Public Properties:**
```php
public mixed $node_uuid = null;        // UUID of node to execute
// Additional execution context properties
```

**Purpose:** Handle client-specific node execution
**Key Methods:**
- `run()` - Execute node for client with context

**Features:**
- Client context integration
- Workflow state management
- Node execution logic
- Result processing and formatting

---

### 3. FilterCountForm

**File:** `src/apps/api/versions/v1/forms/node/FilterCountForm.php`  
**Extends:** `customs\Model`

**Public Properties:**
```php
public mixed $filters = null;          // Filter criteria array
```

**Purpose:** Handle node filtering and counting operations
**Key Methods:**
- `count()` - Count nodes matching filters

**Features:**
- Dynamic filter processing
- Query optimization
- Count aggregation
- Performance-optimized queries

---

## Model Classes

### 1. ViewNode

**File:** `src/apps/api/versions/v1/models/node/ViewNode.php`

**Purpose:** Data model for node representation and serialization
**Features:**
- Node data formatting
- Relationship management
- View-optimized data structure
- API response formatting

---

## Business Logic

### 1. Node Management Flow
```
1. Node Creation and Configuration
   - Initialize node with type and settings
   - Set up visual positioning
   - Establish workflow relationships
   - Configure node-specific parameters

2. Node Updates and Modifications
   - Validate existing node access
   - Apply configuration changes
   - Update relationships and dependencies
   - Maintain workflow integrity

3. Node Deletion and Cleanup
   - Check dependencies before deletion
   - Remove node connections
   - Clean up related data
   - Update workflow structure
```

### 2. Node Execution Flow
```
1. Evaluation Process
   - Parse node configuration
   - Load execution context
   - Process conditional logic
   - Return evaluation results

2. Client Execution
   - Validate client permissions
   - Load workflow state
   - Execute node logic
   - Process outputs and state changes

3. Batch Operations
   - Process multiple nodes
   - Maintain transaction safety
   - Handle batch validation
   - Return aggregate results
```

### 3. Visual Management Flow
```
1. Position Management
   - Update node coordinates
   - Maintain layout consistency
   - Handle visual relationships
   - Optimize flow builder display

2. Filter and Search
   - Apply dynamic filters
   - Count matching nodes
   - Optimize query performance
   - Return aggregated results
```

---

## Security Features

### 1. Company-based Access Control
- All node operations filtered by company context
- Node ownership validation through authentication
- UUID-based node identification with access checks

### 2. Input Validation
- UUID validation for all node identifiers
- Configuration parameter sanitization
- Form-based validation for all endpoints
- Exception handling for database operations

### 3. Safe Operations
- Dependency checking before node deletion
- Transaction safety for batch operations
- Validation of node modifications
- Proper error handling and reporting

---

## Integration Points

### Dependencies
- `ViewNode` - Node data model and representation
- `UuidHelper` - UUID validation and processing
- Company authentication system
- Workflow management components
- Database transaction handling

### Related Components
- **Flow Builder** - Visual node management interface
- **Workflow Engine** - Node execution and processing
- **Customer Management** - Client context integration
- **Statistics System** - Node performance tracking

### External Integrations
- Flow builder visual interface
- Workflow execution engine
- Node configuration storage
- Client data processing systems

---

## Usage Examples

### Create new node
```http
POST /api/v1/nodes/create
Content-Type: application/json
company-key: your-company-api-key

{
  "type": "condition",
  "name": "Customer Status Check",
  "configuration": {
    "condition": "customer.status == 'active'",
    "true_path": "send_email",
    "false_path": "end_flow"
  },
  "position": {
    "x": 100,
    "y": 200
  }
}
```

### Update node
```http
POST /api/v1/nodes/update?uuid=550e8400-e29b-41d4-a716-446655440000
Content-Type: application/json
company-key: your-company-api-key

{
  "name": "Updated Customer Check",
  "configuration": {
    "condition": "customer.status == 'premium'",
    "timeout": 300
  }
}
```

### View node details
```http
GET /api/v1/nodes/view?uuid=550e8400-e29b-41d4-a716-446655440000
company-key: your-company-api-key
```

### Execute node
```http
POST /api/v1/nodes/run?uuid=550e8400-e29b-41d4-a716-446655440000
Content-Type: application/json
company-key: your-company-api-key

{
  "client_uuid": "customer-uuid-here",
  "context": {
    "customer_data": {...},
    "workflow_state": {...}
  }
}
```

### Update node position
```http
POST /api/v1/nodes/update-position?uuid=550e8400-e29b-41d4-a716-446655440000
Content-Type: application/x-www-form-urlencoded
company-key: your-company-api-key

position[x]=150&position[y]=250
```

### Count filtered nodes
```http
POST /api/v1/nodes/filter-count
Content-Type: application/json
company-key: your-company-api-key

{
  "filters": {
    "type": "condition",
    "status": "active",
    "created_after": "2024-01-01"
  }
}
```

---

## Error Handling

### Common Errors
- **"Invalid node UUID"** - Node not found or access denied
- **"Node has dependencies"** - Cannot delete node in use
- **"Configuration validation failed"** - Invalid node configuration
- **"Position update failed"** - Invalid position data

### Node Operation Errors
- Node creation conflicts with existing data
- Update validation failures
- Execution permission errors
- Batch operation partial failures

### Database Exceptions
- Transaction rollback on batch failures
- Connection errors during operations
- Constraint violations on relationships
- Data integrity validation failures

### HTTP Status Codes
- **200 OK** - Successful operations
- **422 Unprocessable Entity** - Validation errors or business logic failures

---

*NodesController API provides comprehensive workflow node management with advanced execution capabilities, visual positioning, and batch operations for enterprise flow builder functionality.*