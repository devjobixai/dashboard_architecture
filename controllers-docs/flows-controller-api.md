# FlowsController (API) Documentation

## Overview

FlowsController manages automated workflows via REST API v1, providing endpoints for statistics, nodes listing, activating flows, and updating properties.

Base path: `/api/v1/flows`  
Controller type: ApiController (API Application v1)  
Namespace: `apps\\api\\versions\\v1\\controllers\\FlowsController`  
Authentication: Company-based access control via selectedUserCompany

---

## Endpoints

### 1. Statistic — Flow Statistics

| Param | Value |
|------|-------|
| URL | `/api/v1/flows/statistic` |
| Method | POST |
| Description | Generate detailed flow execution statistics |

Input (StatisticForm bodyParams):

Complex form with many parameters to generate flow statistics. Integrated with:
- Nodular system for node analysis
- MongoDB for statistics storage
- QueryBuilder for complex queries
- Node types: ApiRequestNode, CallNode, DelayNode, EmailNode, EventNode, FilterNode, SMSNode, SplitNode, UpdateNode

StatisticForm highlights:
- 717 lines
- Recursive statistic collection
- Generates multiple statistic table types
- Integration with MongoDB and NodularComponent
- Blocks: SingleStatBlock, TableStatBlock, TabStatBlock

Responses:
- Success: Array with detailed flow statistics
- Error: StatisticForm object with validation errors

---

### 2. Nodes — Flow Nodes List

| Param | Value |
|------|-------|
| URL | `/api/v1/flows/nodes?uuid={flow_uuid}` |
| Method | GET |
| Description | Get active nodes for a specific flow |

URL Parameters:
- `uuid` (string) — flow UUID

Validation:
- UUID format validation via UuidHelper

Business logic:
1. Read flow_uuid from GET params
2. Validate UUID format
3. Find active nodes via ListNode model
4. Filter by flow UUID

Responses:
- Success: Array of active nodes
- Invalid UUID: []
- No UUID: []

Response format
```json
[
  {
    "id": 123,
    "uuid": "node-uuid-here",
    "flow_uuid": "flow-uuid-here",
    "type": "CallNode",
    "name": "Customer Call",
    "status": "active",
    "configuration": {}
  }
]
```

---

### 3. Activate — Toggle Flow Activation

| Param | Value |
|------|-------|
| URL | `/api/v1/flows/activate?uuid={flow_uuid}` |
| Method | POST |
| Description | Toggle flow active/inactive status |

URL Parameters:
- `uuid` (string) — flow UUID

Input (ActivateForm):

| Field | Type | Required | Validation | Description |
|------|------|----------|------------|-------------|
| `flow_uuid` | string | ✓ | required, UUID validation, exists check | Flow UUID to activate |

ActivateForm validation:
```php
[['flow_uuid'], 'required']
[['flow_uuid'], UuidHelper::class]
[['flow_uuid'], 'exist', 'targetClass' => CompanyFlowsAdvanced::class,
 'filter' => function($query) {
     return $query->whereCompanyId($this->selectedUserCompany->id);
 },
 'message' => 'Company flow not found']
```

Activation business logic:
1. Validate UUID and existence within the company
2. Ensure flow is not deleted (STATUS_DELETED)
3. Toggle ACTIVE ⟷ INACTIVE
4. Save new status
5. Return boolean result

Status Logic:
- ACTIVE → INACTIVE (deactivate)
- INACTIVE → ACTIVE (activate)  
- DELETED → Error (cannot activate)
- Others → Error (cannot change)

Responses:
- Success: `true` (now active) or `false` (now inactive)
- Validation Error: ActivateForm with errors
- Deleted Flow: Error "Company flow was deleted and cannot be activated"
- Invalid Status: Error "Company flow status cannot be changed because of current status"

---

### 4. Update Name — Flow Name

| Param | Value |
|------|-------|
| URL | `/api/v1/flows/update-name?uuid={flow_uuid}` |
| Method | POST |
| Description | Update flow name |

URL Parameters:
- `uuid` (string) — flow UUID

POST Parameters:
- `name` (string) — new flow name

Input (UpdateNameForm):

| Field | Type | Required | Validation | Description |
|------|------|----------|------------|-------------|
| `uuid` | string | ✓ | UUID validation, exists check | Flow UUID |
| `name` | string | ✓ | string validation | New flow name |

Business logic:
1. Read UUID from GET params
2. Read name from POST
3. Validate and update
4. Company-based access control

Responses:
- Success: `true`
- Error: UpdateNameForm with errors

---

### 5. Update Description — Flow Description

| Param | Value |
|------|-------|
| URL | `/api/v1/flows/update-description?uuid={flow_uuid}` |
| Method | POST |
| Description | Update flow description |

URL Parameters:
- `uuid` (string) — flow UUID

POST Parameters:
- `description` (string) — new description

Input (UpdateDescriptionForm):

| Field | Type | Required | Validation | Description |
|------|------|----------|------------|-------------|
| `uuid` | string | ✓ | UUID validation, exists check | Flow UUID |
| `description` | string | ✓ | string validation | New flow description |

Business logic:
1. Read UUID from GET params
2. Read new description from POST
3. Validate and update
4. Company-based access control

Responses:
- Success: `true`
- Error: UpdateDescriptionForm with errors

---

### 6. Versions — Flow Versions

| Param | Value |
|------|-------|
| URL | `/api/v1/flows/versions?uuid={flow_uuid}` |
| Method | GET |
| Description | Get a list of flow versions |

URL Parameters:
- `uuid` (string) — flow UUID

Input (VersionsForm):

| Field | Type | Required | Validation | Description |
|------|------|----------|------------|-------------|
| `flow_uuid` | string | ✓ | UUID validation | Flow UUID |

Business logic:
1. Read UUID from GET params
2. Find versions via VersionsForm
3. Return ActiveDataProvider with versions

Responses:
- Success: ActiveDataProvider with versions
- Error: VersionsForm with errors

---

## Form Classes

### 1. StatisticForm

File: `src/apps/api/versions/v1/forms/flows/StatisticForm.php`  
Extends: `customs\\Model` (717 lines)

Key features:
- Generate complex flow statistics
- Integration with nodular system
- MongoDB support
- Recursive statistics processing
- Multiple statistic block types

Main methods:
- `generateStatistic()` — main generation method
- `_validateNodes()` — validate nodes
- `_generateOutputParams()` — generate output parameters
- `_collectChainedStatisticsRecursive()` — recursive collection
- `_buildStatisticData()` — build statistic data
- `_generateStatusesTable()` — statuses table
- `_generateGeneralTable()` — general table
- `_generateMainTable()` — main table

Supported node types:
- ApiRequestNode, CallNode, DelayNode
- EmailNode, EventNode, FilterNode
- NowNode, SMSNode, SplitNode, UpdateNode

---

### 2. ActivateForm

File: `src/apps/api/versions/v1/forms/flows/ActivateForm.php`  
Extends: `customs\\Model`

Public properties:
```php
public mixed $flow_uuid; // Flow UUID to activate
```

Validation Rules:
- Required flow_uuid
- UUID format validation
- Existence check within company
- Company-based filtering

Key methods:
- `activate()` — toggle activation
- `rules()` — validation rules

Status Management:
- Toggle ACTIVE/INACTIVE
- Protect against activating deleted flows
- Error handling for invalid statuses

---

### 3. UpdateNameForm

File: `src/apps/api/versions/v1/forms/flows/UpdateNameForm.php`  
Extends: `customs\\Model`

Details require further analysis for full documentation.

---

### 4. UpdateDescriptionForm

File: `src/apps/api/versions/v1/forms/flows/UpdateDescriptionForm.php`  
Extends: `customs\\Model`

Details require further analysis for full documentation.

---

### 5. VersionsForm

File: `src/apps/api/versions/v1/forms/flows/VersionsForm.php`  
Extends: `customs\\Model`

Details require further analysis for full documentation.

---

## Business Logic

### 1. Flow Statistics Generation
```
1. Load flow and node data
2. Validate nodes configuration
3. Collect chained statistics recursively
4. Generate different table types (status, general, main)
5. Build comprehensive statistic blocks
6. Return aggregated statistics array
```

### 2. Flow Activation Toggle
```
1. Validate flow UUID and company access
2. Check current flow status
3. Prevent activation of deleted flows
4. Toggle between ACTIVE ⟷ INACTIVE
5. Save new status to database
6. Return boolean activation result
```

### 3. Flow Nodes Retrieval
```
1. Validate UUID format
2. Query ListNode model by flow UUID
3. Filter for active nodes only
4. Return array of node objects
```

### 4. Flow Property Updates
```
1. Get UUID from URL parameter
2. Get new value from POST data
3. Validate input and company access
4. Update specific property (name/description)
5. Return success boolean
```

---

## Security Features

### 1. Company-based Access Control
- All operations isolated by selectedUserCompany
- UUID validation with company filtering
- Existence checking within the company

### 2. UUID Validation
- UuidHelper validation for all UUID params
- Format checking before database queries
- Secure UUID handling

### 3. Input Validation
- Proper form validation for all endpoints
- String validation for text fields
- Required field checking

### 4. Status Protection
- Protect against activating deleted flows
- Validate current status before changes
- Error handling for invalid statuses

---

## Integration Points

### Dependencies
- `CompanyFlowsAdvanced` — core flows model
- `ListNode` — flow node model
- `UuidHelper` — UUID validation
- `NodularComponent` — nodular system integration
- `QueryBuilder` — complex queries
- MongoDB Connection — statistics and cache

### Nodular System Integration
- Various node types (Call, Email, SMS, API, etc.)
- NodeStatBuilder for statistics
- Statistic blocks and components
- Recursive processing of node chains

### External Services
- MongoDB for statistics
- Nodular execution engine
- Flow versioning system
- Statistics caching system

---

## Usage Examples

### Get flow statistics
```http
POST /api/v1/flows/statistic
Content-Type: application/json

{
  "flow_uuid": "550e8400-e29b-41d4-a716-446655440000",
  "date_from": "2023-01-01",
  "date_to": "2023-12-31",
  "include_nodes": true
}
```

Response:
```json
{
  "total_executions": 1250,
  "success_rate": 89.6,
  "average_duration": 45.2,
  "nodes_statistics": [
    {"node_uuid": "node-uuid-1", "type": "CallNode", "executions": 1250, "success_count": 1100, "error_count": 150}
  ],
  "status_breakdown": {"completed": 1120, "failed": 130}
}
```

### Get flow nodes
```http
GET /api/v1/flows/nodes?uuid=550e8400-e29b-41d4-a716-446655440000
```

Response:
```json
[
  {
    "id": 123,
    "uuid": "node-uuid-1",
    "flow_uuid": "550e8400-e29b-41d4-a716-446655440000",
    "type": "CallNode",
    "name": "Initial Call",
    "status": "active"
  }
]
```

