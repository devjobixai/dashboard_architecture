# QueueController API Documentation

## Overview

QueueController manages queue operations and webhook processing for the Jobix Dashboard API. The controller provides comprehensive functionality for queue management including webhook processors for calls, SMS, and emails, queue control operations (run, stop, pause, resume), speed management, and queue information retrieval. It serves as the core API for asynchronous job processing and external system integration.

**Base Path:** `/api/v1/queue`  
**Controller Type:** ApiController (API Application)  
**Namespace:** `apps\api\versions\v1\controllers\QueueController`  
**Authentication:** Requires `company-key` header

---

## Endpoints

### 1. Call Hook - Process Call Webhooks

| Parameter | Value |
|-----------|-------|
| **URL** | `/api/v1/queue/call-hook` |
| **Method** | POST |
| **Description** | Process incoming call webhook events and queue call-related jobs |

**Input Data (CallHookProcessorForm bodyParams):**
- Call webhook payload data
- Call event information and metadata
- Provider-specific call data

**Business Logic:**
1. Parse incoming call webhook data
2. Validate call event information
3. Process call-specific webhook logic
4. Queue call-related jobs for processing
5. Return webhook processing confirmation

**Response:**
- **Success (200):** Webhook processing confirmation
- **Error (422):** CallHookProcessorForm object with validation errors

**Webhook Processing:**
- Call event validation and parsing
- Provider-specific data handling
- Job queue integration for async processing
- Response formatting for webhook providers

---

### 2. SMS Hook - Process SMS Webhooks

| Parameter | Value |
|-----------|-------|
| **URL** | `/api/v1/queue/sms-hook` |
| **Method** | POST |
| **Description** | Process incoming SMS webhook events and queue SMS-related jobs |

**Input Data (SmsHookProcessorForm bodyParams):**
- SMS webhook payload data
- Message delivery status information
- SMS provider event data

**Business Logic:**
1. Parse incoming SMS webhook data
2. Validate SMS event information
3. Process SMS-specific webhook logic
4. Queue SMS-related jobs for processing
5. Return webhook processing confirmation

**Response:**
- **Success (200):** SMS webhook processing confirmation
- **Error (422):** SmsHookProcessorForm object with validation errors

**SMS Processing Features:**
- Delivery status tracking
- Provider webhook validation
- Message processing queue integration
- Error handling for failed deliveries

---

### 3. Email Hook - Process Email Webhooks

| Parameter | Value |
|-----------|-------|
| **URL** | `/api/v1/queue/email-hook` |
| **Method** | POST |
| **Description** | Process incoming email webhook events and queue email-related jobs |

**Input Data (EmailHookProcessorForm bodyParams):**
- Email webhook payload data
- Email delivery status information
- Email provider event data

**Business Logic:**
1. Parse incoming email webhook data
2. Validate email event information
3. Process email-specific webhook logic
4. Queue email-related jobs for processing
5. Return webhook processing confirmation

**Response:**
- **Success (200):** Email webhook processing confirmation
- **Error (422):** EmailHookProcessorForm object with validation errors

**Email Processing Features:**
- Delivery tracking and status updates
- Bounce and complaint handling
- Provider-specific webhook processing
- Queue integration for email analytics

---

### 4. Info - Get Queue Information

| Parameter | Value |
|-----------|-------|
| **URL** | `/api/v1/queue/info` |
| **Method** | POST |
| **Description** | Retrieve detailed information about queue status and performance |

**Input Data (QueueInfoForm bodyParams):**
- Queue identification parameters
- Information request specifications
- Performance metric filters

**Business Logic:**
1. Validate queue access permissions
2. Retrieve queue status and statistics
3. Calculate performance metrics
4. Format queue information response
5. Return comprehensive queue data

**Response:**
- **Success (200):** Array with detailed queue information
- **Error (422):** QueueInfoForm object with validation errors

**Queue Information Includes:**
- Queue status and health metrics
- Job processing statistics
- Performance and throughput data
- Error rates and failure analysis

---

### 5. Run - Start Queue Processing

| Parameter | Value |
|-----------|-------|
| **URL** | `/api/v1/queue/run` |
| **Method** | POST |
| **Description** | Start queue processing and job execution |

**Input Data (QueueActionForm bodyParams):**
- Queue execution parameters
- Processing configuration settings
- Job priority and filtering options

**Business Logic:**
1. Validate queue operation permissions
2. Initialize queue processing system
3. Start job execution with configured parameters
4. Monitor queue startup process
5. Return execution status confirmation

**Response:**
- **Success (200):** Array with queue start confirmation and status
- **Error (422):** Array with error details and failure reasons

**Queue Start Features:**
- Configurable processing parameters
- Job priority management
- Resource allocation control
- Startup validation and monitoring

---

### 6. Stop - Stop Queue Processing

| Parameter | Value |
|-----------|-------|
| **URL** | `/api/v1/queue/stop` |
| **Method** | POST |
| **Description** | Stop queue processing and halt job execution |

**Input Data (QueueActionForm bodyParams):**
- Queue stop parameters
- Graceful shutdown configuration
- Job completion handling options

**Business Logic:**
1. Validate queue stop permissions
2. Initiate graceful queue shutdown
3. Handle running job completion
4. Clean up queue resources
5. Return stop confirmation status

**Response:**
- **Success (200):** Array with queue stop confirmation and status
- **Error (422):** Array with error details and failure reasons

**Queue Stop Features:**
- Graceful shutdown process
- Running job completion handling
- Resource cleanup and deallocation
- Status monitoring during shutdown

---

### 7. Pause - Pause Queue Processing

| Parameter | Value |
|-----------|-------|
| **URL** | `/api/v1/queue/pause` |
| **Method** | POST |
| **Description** | Pause queue processing temporarily without stopping |

**Input Data (QueueActionForm bodyParams):**
- Queue pause parameters
- Temporary suspension configuration
- Job handling during pause

**Business Logic:**
1. Validate queue pause permissions
2. Pause new job processing
3. Maintain running job execution
4. Update queue status to paused
5. Return pause confirmation

**Response:**
- **Success (200):** Array with queue pause confirmation and status
- **Error (422):** Array with error details and failure reasons

**Queue Pause Features:**
- Temporary processing suspension
- Running job preservation
- Quick resume capability
- Status tracking during pause

---

### 8. Resume - Resume Queue Processing

| Parameter | Value |
|-----------|-------|
| **URL** | `/api/v1/queue/resume` |
| **Method** | POST |
| **Description** | Resume paused queue processing and job execution |

**Input Data (QueueActionForm bodyParams):**
- Queue resume parameters
- Processing restart configuration
- Job priority adjustments

**Business Logic:**
1. Validate queue resume permissions
2. Resume paused job processing
3. Restore queue processing parameters
4. Update queue status to active
5. Return resume confirmation

**Response:**
- **Success (200):** Array with queue resume confirmation and status
- **Error (422):** Array with error details and failure reasons

**Queue Resume Features:**
- Quick processing restart
- Parameter restoration
- Priority queue handling
- Seamless operation continuation

---

### 9. Slow Down - Reduce Queue Speed

| Parameter | Value |
|-----------|-------|
| **URL** | `/api/v1/queue/slow-down` |
| **Method** | POST |
| **Description** | Reduce queue processing speed and job execution rate |

**Input Data (QueueActionForm bodyParams):**
- Speed reduction parameters
- New processing rate configuration
- Performance adjustment settings

**Business Logic:**
1. Validate speed adjustment permissions
2. Calculate new processing rate
3. Apply speed reduction configuration
4. Monitor performance impact
5. Return speed adjustment confirmation

**Response:**
- **Success (200):** Array with speed adjustment confirmation and new rate
- **Error (422):** Array with error details and failure reasons

**Speed Control Features:**
- Dynamic rate adjustment
- Performance impact monitoring
- Configurable reduction parameters
- Real-time speed modification

---

### 10. Speed Up - Increase Queue Speed

| Parameter | Value |
|-----------|-------|
| **URL** | `/api/v1/queue/speed-up` |
| **Method** | POST |
| **Description** | Increase queue processing speed and job execution rate |

**Input Data (QueueActionForm bodyParams):**
- Speed increase parameters
- New processing rate configuration
- Performance optimization settings

**Business Logic:**
1. Validate speed adjustment permissions
2. Calculate optimal processing rate
3. Apply speed increase configuration
4. Monitor resource utilization
5. Return speed adjustment confirmation

**Response:**
- **Success (200):** Array with speed adjustment confirmation and new rate
- **Error (422):** Array with error details and failure reasons

**Speed Enhancement Features:**
- Dynamic performance scaling
- Resource optimization
- Intelligent rate calculation
- Real-time throughput monitoring

---

### 11. Inbound Call Hook - Process Inbound Call Webhooks

| Parameter | Value |
|-----------|-------|
| **URL** | `/api/v1/queue/inbound-call-hook` |
| **Method** | POST |
| **Description** | Process incoming inbound call webhook events and queue related jobs |

**Input Data (InboundCallHookProcessorForm bodyParams):**
- Inbound call webhook payload
- Call routing and handling data
- Provider-specific inbound call information

**Business Logic:**
1. Parse inbound call webhook data
2. Validate call routing information
3. Process inbound call logic
4. Queue inbound call jobs
5. Return processing confirmation

**Response:**
- **Success (200):** Inbound call webhook processing confirmation
- **Error (422):** InboundCallHookProcessorForm object with validation errors

**Inbound Call Processing:**
- Call routing validation
- Queue integration for call handling
- Provider webhook processing
- Real-time call event handling

---

## Form Classes

### 1. CallHookProcessorForm

**File:** `src/apps/api/versions/v1/forms/queues/CallHookProcessorForm.php`  
**Extends:** `customs\Model`

**Purpose:** Process call webhook events and queue related jobs
**Features:**
- Call webhook data validation
- Provider-specific processing
- Job queue integration
- Call event handling

---

### 2. SmsHookProcessorForm

**File:** `src/apps/api/versions/v1/forms/queues/SmsHookProcessorForm.php`  
**Extends:** `customs\Model`

**Purpose:** Process SMS webhook events and delivery tracking
**Features:**
- SMS webhook validation
- Delivery status processing
- Provider integration
- Message queue management

---

### 3. EmailHookProcessorForm

**File:** `src/apps/api/versions/v1/forms/queues/EmailHookProcessorForm.php`  
**Extends:** `customs\Model`

**Purpose:** Process email webhook events and delivery management
**Features:**
- Email webhook processing
- Delivery status tracking
- Bounce and complaint handling
- Email analytics integration

---

### 4. QueueActionForm

**File:** `src/apps/api/versions/v1/forms/queues/QueueActionForm.php`  
**Extends:** `customs\Model`

**Purpose:** Handle queue control operations (run, stop, pause, resume, speed control)
**Features:**
- Queue operation validation
- Processing parameter management
- Performance control
- Status monitoring

---

### 5. QueueInfoForm

**File:** `src/apps/api/versions/v1/forms/queues/QueueInfoForm.php`  
**Extends:** `customs\Model`

**Purpose:** Retrieve and format queue information and statistics
**Features:**
- Queue status retrieval
- Performance metrics calculation
- Information formatting
- Access control validation

---

### 6. InboundCallHookProcessorForm

**File:** `src/apps/api/versions/v1/forms/queues/InboundCallHookProcessorForm.php`  
**Extends:** `customs\Model`

**Purpose:** Process inbound call webhook events
**Features:**
- Inbound call validation
- Call routing processing
- Provider webhook handling
- Queue integration

---

## Business Logic

### 1. Webhook Processing Flow
```
1. Webhook Reception and Validation
   - Parse incoming webhook payload
   - Validate provider signatures
   - Extract event information
   - Perform data sanitization

2. Event Processing
   - Route to appropriate processor
   - Apply business logic rules
   - Update relevant data models
   - Generate response events

3. Job Queue Integration
   - Create processing jobs
   - Set job priorities and delays
   - Queue jobs for execution
   - Monitor job processing
```

### 2. Queue Management Flow
```
1. Queue Control Operations
   - Validate operation permissions
   - Execute control commands
   - Monitor status changes
   - Return operation confirmations

2. Performance Management
   - Dynamic speed adjustment
   - Resource allocation optimization
   - Throughput monitoring
   - Load balancing control

3. Status and Information
   - Real-time status tracking
   - Performance metrics calculation
   - Error rate monitoring
   - Historical data analysis
```

### 3. Job Processing Flow
```
1. Job Creation and Queuing
   - Create ExecuteModelJob instances
   - Set job parameters and priorities
   - Add jobs to processing queue
   - Track job lifecycle

2. Processing and Execution
   - Job pickup and execution
   - Error handling and retry logic
   - Progress monitoring
   - Result processing

3. Completion and Cleanup
   - Job completion confirmation
   - Resource cleanup
   - Result storage and reporting
   - Performance metrics updates
```

---

## Security Features

### 1. Company-based Access Control
- All queue operations filtered by company context
- Webhook validation with company authentication
- Permission-based queue control access

### 2. Webhook Security
- Provider signature validation
- Payload sanitization and validation
- Rate limiting and abuse protection
- Secure webhook endpoint handling

### 3. Queue Security
- Operation permission validation
- Resource access control
- Job execution isolation
- Error handling and logging

---

## Integration Points

### Dependencies
- `ExecuteModelJob` - Job execution framework
- `Queue` (Yii DB Queue) - Queue processing engine
- `Constants` - System configuration constants
- Company authentication system
- Webhook provider integrations

### External Services
- **Call Providers** - Twilio, Vonage, etc.
- **SMS Providers** - Various SMS service providers
- **Email Providers** - Brevo and other email services
- **Queue Systems** - Database-backed queue processing

### Related Components
- **Call Management** - Call processing and routing
- **SMS Management** - Message delivery and tracking
- **Email Management** - Email delivery and analytics
- **Job Processing** - Asynchronous job execution

---

## Usage Examples

### Process call webhook
```http
POST /api/v1/queue/call-hook
Content-Type: application/json
company-key: your-company-api-key

{
  "CallSid": "CAxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
  "CallStatus": "completed",
  "Duration": "120",
  "From": "+1234567890",
  "To": "+0987654321",
  "RecordingUrl": "https://api.twilio.com/recording.mp3"
}
```

### Get queue information
```http
POST /api/v1/queue/info
Content-Type: application/json
company-key: your-company-api-key

{
  "queue_id": "main_queue",
  "metrics": ["status", "throughput", "error_rate"],
  "time_range": "last_hour"
}
```

### Start queue processing
```http
POST /api/v1/queue/run
Content-Type: application/json
company-key: your-company-api-key

{
  "queue_name": "main_processing",
  "max_workers": 5,
  "priority_threshold": 10
}
```

### Pause queue
```http
POST /api/v1/queue/pause
Content-Type: application/json
company-key: your-company-api-key

{
  "queue_name": "main_processing",
  "graceful": true,
  "timeout": 30
}
```

### Adjust queue speed
```http
POST /api/v1/queue/slow-down
Content-Type: application/json
company-key: your-company-api-key

{
  "queue_name": "main_processing",
  "reduction_factor": 0.5,
  "apply_immediately": true
}
```

### Process SMS webhook
```http
POST /api/v1/queue/sms-hook
Content-Type: application/json
company-key: your-company-api-key

{
  "MessageSid": "SMxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
  "MessageStatus": "delivered",
  "To": "+1234567890",
  "From": "+0987654321",
  "Body": "Your message has been delivered"
}
```

---

## Error Handling

### Common Errors
- **"Invalid queue operation"** - Unauthorized queue control action
- **"Webhook validation failed"** - Invalid webhook signature or payload
- **"Queue not found"** - Specified queue doesn't exist
- **"Permission denied"** - Insufficient permissions for operation

### Webhook Processing Errors
- Invalid webhook payload structure
- Provider signature validation failures
- Rate limiting exceeded
- Processing job creation failures

### Queue Operation Errors
- Queue already in requested state
- Resource allocation failures
- Performance adjustment out of bounds
- Job processing system errors

### HTTP Status Codes
- **200 OK** - Successful operations and webhook processing
- **422 Unprocessable Entity** - Validation errors or business logic failures

---

*QueueController API provides comprehensive queue management with webhook processing capabilities, performance control, and robust job processing integration for enterprise-level asynchronous operations.*