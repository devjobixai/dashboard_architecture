# QueuesController Documentation

## Overview

QueuesController manages queue operations for call center functionality in the Jobix Dashboard administrative panel. The controller provides comprehensive queue management including creation, editing, monitoring, customer management, real-time statistics, file downloads, and inbound call handling with extensive search and filtering capabilities.

**Base Path:** `/queues`  
**Controller Type:** WebController (Admin Application)  
**Namespace:** `apps\admin\controllers\QueuesController`

---

## Endpoints

### Queue Management

### 1. Index - Queue List

| Parameter | Value |
|-----------|-------|
| **URL** | `/queues/index` |
| **Method** | GET |
| **Description** | Display list of all queues with search and filtering |

**Input Data (QueuesSearch queryParams):**
- Search parameters loaded from query string
- Company-based filtering applied automatically

**Business Logic:**
1. Create QueuesSearch form for filtering
2. Load search parameters from query string
3. Execute search with company isolation
4. Return HTML view with queue list and search interface

**Response:**
- **Success:** HTML view with queues list
- **Data:** `dataProvider`, `searchModel`

---

### 2. Action - Queue Status Control (AJAX)

| Parameter | Value |
|-----------|-------|
| **URL** | `/queues/action` |
| **Method** | POST (AJAX only) |
| **Description** | Handle queue status changes and actions |

**Input Data (ActionQueueForm bodyParams):**
- Form data loaded without wrapper
- Queue action parameters

**Business Logic:**
1. Validate AJAX request (throws 404 if not AJAX)
2. Clean output buffer
3. Set JSON response format
4. Create ActionQueueForm and load data
5. Process queue action and return results

**Response:**
- **Success:** JSON array with action results
- **Error:** NotFoundHttpException for non-AJAX requests
- **Format:** JSON response format

---

### 3. Create - Create New Queue

| Parameter | Value |
|-----------|-------|
| **URL** | `/queues/create` |
| **Methods** | GET, POST |
| **Description** | Create new queue |

**Input Data (CreateQueueForm bodyParams):**
- Queue creation parameters
- Company context automatic

#### GET `/queues/create`
**Response:** HTML view with empty creation form

#### POST `/queues/create`
**Business Logic:**
1. Load and validate CreateQueueForm
2. Process queue creation
3. Redirect to edit page on success

**Response:**
- **Success:** Redirect to `/queues/edit?uuid={queue_uuid}`
- **Validation Error:** HTML view with form and errors

---

### 4. Edit - Edit Queue

| Parameter | Value |
|-----------|-------|
| **URL** | `/queues/edit?uuid={queue_uuid}` |
| **Methods** | GET, POST |
| **Description** | Edit existing queue |

**URL Parameters:**
- `uuid` (string, required) - Queue UUID

**Input Data (EditQueueForm bodyParams):**
- Queue update parameters

#### GET `/queues/edit`
**Business Logic:**
1. Create EditQueueForm with UUID
2. Check queue existence
3. Auto-populate form with current data

#### POST `/queues/edit`
**Business Logic:**
1. Load and validate form data
2. Process queue update
3. Redirect to same edit page on success

**Response:**
- **Success:** Redirect to `/queues/edit?uuid={queue_uuid}`
- **Not Found:** Redirect to `/queues/index`
- **Validation Error:** HTML view with form and errors

---

### 5. Delete - Delete Queue

| Parameter | Value |
|-----------|-------|
| **URL** | `/queues/delete?uuid={queue_uuid}` |
| **Method** | POST |
| **Description** | Delete queue |

**URL Parameters:**
- `uuid` (string, required) - Queue UUID

**Business Logic:**
1. Create DeleteQueueForm with UUID
2. Process queue deletion
3. Always redirect to index

**Response:**
- **Always:** Redirect to `/queues/index`

---

### Queue Monitoring

### 6. Calls - Queue Calls List

| Parameter | Value |
|-----------|-------|
| **URL** | `/queues/calls?uuid={queue_uuid}` |
| **Method** | GET |
| **Description** | Display calls associated with specific queue |

**URL Parameters:**
- `uuid` (string, required) - Queue UUID

**Input Data (QueueCallSearch queryParams):**
- Search parameters with queue UUID context

**Business Logic:**
1. Validate company selection (redirect if not set)
2. Create QueueCallSearch with queue UUID
3. Load search parameters and validate
4. Execute call search for queue

**Response:**
- **Success:** HTML view with queue calls
- **Data:** `dataProvider`, `searchModel`
- **No Company:** Redirect to `/companies/index`

---

### 7. Customers - Queue Customers List

| Parameter | Value |
|-----------|-------|
| **URL** | `/queues/customers?uuid={queue_uuid}` |
| **Method** | GET |
| **Description** | Display customers associated with specific queue |

**URL Parameters:**
- `uuid` (string, required) - Queue UUID

**Input Data (QueueCustomersSearch queryParams):**
- Search parameters with queue UUID context

**Business Logic:**
1. Validate company selection (redirect if not set)
2. Create QueueCustomersSearch with queue UUID
3. Load search parameters and validate
4. Execute customer search for queue

**Response:**
- **Success:** HTML view with queue customers
- **Data:** `dataProvider`, `searchModel`
- **No Company:** Redirect to `/companies/index`

---

### 8. Real Time - Queue Real-time Monitoring

| Parameter | Value |
|-----------|-------|
| **URL** | `/queues/real-time?uuid={queue_uuid}` |
| **Method** | GET |
| **Description** | Display real-time queue statistics and monitoring |

**URL Parameters:**
- `uuid` (string, required) - Queue UUID

**Business Logic:**
1. Validate company selection (redirect if not set)
2. Find queue by UUID using CompanyQueuesAdvanced
3. Render real-time monitoring interface

**Response:**
- **Success:** HTML view with real-time monitoring
- **Data:** `uuid`, `queue` model
- **No Company:** Redirect to `/companies/index`

---

### 9. Customers Download - Export Queue Customers

| Parameter | Value |
|-----------|-------|
| **URL** | `/queues/customers-download?uuid={queue_uuid}&type={export_type}` |
| **Method** | GET |
| **Description** | Download/export queue customers data |

**URL Parameters:**
- `uuid` (string, required) - Queue UUID
- `type` (string, required) - Export type/format

**Input Data (QueueCustomersDownloadForm):**
- UUID and export type parameters

**Business Logic:**
1. Validate company selection (redirect if not set)
2. Create QueueCustomersDownloadForm
3. Load UUID and type parameters
4. Process export generation
5. Send file response or redirect on failure

**Response:**
- **Success:** File download with appropriate headers
- **Headers:** `mimeType`, `Content-Disposition: attachment`
- **Failure:** Redirect to `/queues/customers?uuid={queue_uuid}`
- **No Company:** Redirect to `/companies/index`

---

### Inbound Call Management

### 10. Inbound Call - Inbound Calls List

| Parameter | Value |
|-----------|-------|
| **URL** | `/queues/inbound-call` |
| **Method** | GET |
| **Description** | Display list of inbound calls with search |

**Input Data (InboundCallSearch queryParams):**
- Search parameters from query string

**Business Logic:**
1. Create InboundCallSearch form
2. Load search parameters from query
3. Execute inbound call search

**Response:**
- **Success:** HTML view with inbound calls list
- **Data:** `dataProvider`, `searchModel`

---

### 11. Inbound Call Create - Create Inbound Call

| Parameter | Value |
|-----------|-------|
| **URL** | `/queues/inbound-call-create` |
| **Methods** | GET, POST |
| **Description** | Create new inbound call configuration |

**Input Data (CreateInboundCallForm bodyParams):**
- Inbound call creation parameters

#### GET `/queues/inbound-call-create`
**Response:** HTML view with creation form

#### POST `/queues/inbound-call-create`
**Business Logic:**
1. Load CreateInboundCallForm
2. Process inbound call creation
3. Set success flash message
4. Redirect to edit page on success

**Response:**
- **Success:** Redirect to `/queues/inbound-call-edit?uuid={call_uuid}` with flash message
- **Validation Error:** HTML view with form and errors

---

### 12. Inbound Call Edit - Edit Inbound Call

| Parameter | Value |
|-----------|-------|
| **URL** | `/queues/inbound-call-edit?uuid={call_uuid}` |
| **Methods** | GET, POST |
| **Description** | Edit existing inbound call configuration |

**URL Parameters:**
- `uuid` (string, required) - Inbound call UUID

**Input Data (EditInboundCallForm bodyParams):**
- Inbound call update parameters

#### GET `/queues/inbound-call-edit`
**Business Logic:**
1. Create EditInboundCallForm with UUID
2. Auto-populate with current data

#### POST `/queues/inbound-call-edit`
**Business Logic:**
1. Load form data
2. Process inbound call update
3. Set success flash message
4. Redirect to same edit page

**Response:**
- **Success:** Redirect to `/queues/inbound-call-edit?uuid={call_uuid}` with flash message
- **Validation Error:** HTML view with form and errors

---

### 13. Inbound Call Delete - Delete Inbound Call

| Parameter | Value |
|-----------|-------|
| **URL** | `/queues/inbound-call-delete?uuid={call_uuid}` |
| **Method** | POST |
| **Description** | Delete inbound call configuration |

**URL Parameters:**
- `uuid` (string, required) - Inbound call UUID

**Business Logic:**
1. Create DeleteInboundCallForm with UUID
2. Process deletion
3. Set appropriate flash message based on result
4. Redirect to inbound call list

**Response:**
- **Success:** Redirect to `/queues/inbound-call` with success flash
- **Failure:** Redirect to `/queues/inbound-call` with error flash

---

### 14. Inbound Call Statistics - Inbound Call Statistics

| Parameter | Value |
|-----------|-------|
| **URL** | `/queues/inbound-call-statistics?uuid={call_uuid}` |
| **Method** | GET |
| **Description** | Display statistics for specific inbound call |

**URL Parameters:**
- `uuid` (string, required) - Inbound call UUID

**Input Data (InboundCallStatisticsSearch queryParams):**
- Statistics search parameters with call UUID context

**Business Logic:**
1. Create InboundCallStatisticsSearch with UUID
2. Load search parameters
3. Execute statistics search

**Response:**
- **Success:** HTML view with inbound call statistics
- **Data:** `dataProvider`, `searchModel`

---

## Form Classes Overview

### Queue Management Forms
1. **QueuesSearch** - Search and filter queues
2. **ActionQueueForm** - Handle queue status actions (AJAX)
3. **CreateQueueForm** - Create new queues
4. **EditQueueForm** - Edit existing queues
5. **DeleteQueueForm** - Delete queues

### Queue Monitoring Forms
6. **QueueCallSearch** - Search calls within specific queue
7. **QueueCustomersSearch** - Search customers within specific queue
8. **QueueCustomersDownloadForm** - Export queue customer data

### Inbound Call Management Forms
9. **InboundCallSearch** - Search and filter inbound calls
10. **CreateInboundCallForm** - Create inbound call configurations
11. **EditInboundCallForm** - Edit inbound call configurations
12. **DeleteInboundCallForm** - Delete inbound call configurations
13. **InboundCallStatisticsSearch** - Search inbound call statistics

---

## Business Logic

### 1. Queue Management Flow
```
1. List queues with search/filtering (index)
2. Create new queue with form validation (create)
3. Edit queue with data population (edit)
4. Control queue status via AJAX (action)
5. Delete queue with confirmation (delete)
```

### 2. Queue Monitoring Flow
```
1. View queue-specific calls with search (calls)
2. View queue-specific customers with search (customers)
3. Monitor real-time queue statistics (realTime)
4. Export queue customer data to files (customersDownload)
```

### 3. Inbound Call Management Flow
```
1. List inbound calls with search (inboundCall)
2. Create inbound call configurations (inboundCallCreate)
3. Edit inbound call settings (inboundCallEdit)
4. Delete inbound call configurations (inboundCallDelete)
5. View inbound call statistics (inboundCallStatistics)
```

---

## Security Features

### 1. Company Validation
- Most endpoints validate company selection
- Automatic redirect to `/companies/index` if no company set
- Company-based data isolation throughout

### 2. AJAX Security
- Action endpoint validates AJAX requests only
- JSON response format for AJAX operations
- Output buffer cleaning for clean responses

### 3. UUID Validation
- All entity-specific endpoints require valid UUIDs
- Queue existence validation before operations
- Proper error handling for missing entities

### 4. Access Control
- User identity and company context validation
- Redirect mechanisms for unauthorized access
- Flash message system for user feedback

---

## Integration Points

### Dependencies
- `CompanyQueuesAdvanced` - main queue model
- Multiple search and CRUD form classes
- Yii Framework flash message system
- File download functionality

### Related Components
- **Companies Management** - company selection requirements
- **Call Management** - queue call associations
- **Customer Management** - queue customer assignments
- **Statistics System** - real-time monitoring and reporting

---

## Usage Examples

### List queues
```http
GET /queues/index
```

### Create queue
```http
POST /queues/create
Content-Type: application/x-www-form-urlencoded

CreateQueueForm[name]=Support Queue&CreateQueueForm[description]=Customer support queue
```

### Queue status action (AJAX)
```http
POST /queues/action
Content-Type: application/x-www-form-urlencoded
X-Requested-With: XMLHttpRequest

uuid=550e8400-e29b-41d4-a716-446655440000&action=start
```

### View queue calls
```http
GET /queues/calls?uuid=550e8400-e29b-41d4-a716-446655440000
```

### Download queue customers
```http
GET /queues/customers-download?uuid=550e8400-e29b-41d4-a716-446655440000&type=csv
```

### Create inbound call
```http
POST /queues/inbound-call-create
Content-Type: application/x-www-form-urlencoded

CreateInboundCallForm[phone_number]=+1234567890&CreateInboundCallForm[queue_uuid]=550e8400-e29b-41d4-a716-446655440000
```

---

## Performance Considerations

### Database Optimization
- Company-based indexing for all queue queries
- Queue-specific filtering for calls and customers
- Search optimization for large datasets

### Real-time Features
- Real-time monitoring may require periodic updates
- Statistics calculations for large call volumes
- Efficient queue status management

### File Operations
- Customer export functionality for large datasets
- File generation and download optimization
- Temporary file management

---

## Error Handling

### Common Errors
- **NotFoundHttpException** - Non-AJAX requests to AJAX endpoints
- **Queue not found** - Invalid UUID or missing queue
- **Company not selected** - Missing company context
- **Export failures** - File generation or download issues

### Redirect Patterns
- Missing company → `/companies/index`
- Missing queue → `/queues/index`
- Successful operations → appropriate success pages

### Flash Messages
- Success operations → `toast_success` flash messages
- Failed operations → `toast_error` flash messages
- User-friendly feedback system

---

## HTTP Status Codes
- **200 OK** - Successful operations
- **302 Found** - Redirects after operations
- **404 Not Found** - Missing resources or invalid AJAX
- **File Download** - Appropriate file response headers

---

*QueuesController is a comprehensive component for managing call center queue operations with extensive monitoring, customer management, real-time statistics, and inbound call handling capabilities.*