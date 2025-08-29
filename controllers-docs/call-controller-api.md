# CallController (API) Documentation

## Overview

CallController manages calls via REST API v1, including transcription, call initialization, real-call initialization, and internet-call webhooks.

Base path: `/api/v1/call`  
Controller type: ApiController (API Application v1)  
Namespace: `apps\\api\\versions\\v1\\controllers\\CallController`  
Authentication: Uses `company_key` in bodyParams to identify the company

---

## Endpoints

### 1. Transcription — Process Call Transcription

| Param | Value |
|------|-------|
| URL | `/api/v1/call/transcription` |
| Method | POST |
| Description | Process call transcription with AI/ML integration |

Input (TranscriptionForm bodyParams):

| Field | Type | Required | Validation | Description |
|------|------|----------|------------|-------------|
| `company_key` | string | ✓ | required, string, custom validation | Company API key |
| `call_id` | string | ✓ | required, string, call existence validation | Call ID |
| `transcription` | object | ✓ | required, complex validation through partial form | Transcription data |
| `end_by` | object | ✗ | complex validation through EndByForm | Call end data |
| `total_tokens_llm` | number | ✗ | number | Total LLM tokens |

Transcription payload:
- Validated via TranscriptionForm partial
- Contains transcription text and metadata
- Structure and format validated

End By payload:
- Validated via EndByForm partial
- Contains how the call ended
- Includes reason and additional details

TranscriptionForm Validation:
```php
[['company_key', 'transcription', 'call_id'], 'required']
[['company_key', 'call_id'], 'string']
[['total_tokens_llm'], 'number']
[['company_key'], 'validateCompanyApiKey']
[['call_id'], 'setCallDetails']
[['transcription'], 'validateTranscription']
[['end_by'], 'validateEndBy']
```

Transcription processing business logic:
1. Validation:
   - Validate company by API key
   - Validate call existence by call_id
   - Validate transcription and end_by structure
2. Save transcription:
   - Register in `CompanyCallTranscriptionsAdvanced`
   - Generate transcription UUID
   - Link with company_id, customer_id, agent_id
3. Trigger Processing:
   - Execute `AgentCallTrigger::executeTrigger()`
   - Automated business logic
4. Statistics Processing:
   - Save LLM stats (`saveTheAvgLLMTime()`)
   - Save STT stats (`saveTheAvgSTTTime()`)
   - Save TTS stats (`saveTheAvgTTSTime()`)
5. Call Record Update:
   - Update `additional_info` in `CompanyCallsAdvanced`
   - Persist all statistics and end_by data
   - Save total_tokens_llm
6. WebSocket Integration:
   - Get queue ID for real-time updates
   - Send STT/TTS statistics via WebSocket

Responses:

Success:
```json
{
  "status": true,
  "errors": {},
  "object": {
    "transcription_id": "uuid-here",
    "call_id": "call_sid_123",
    "saved": true
  }
}
```

Validation Error:
```json
{
  "status": false,
  "errors": {
    "company_key": ["Company not found"],
    "call_id": ["Call not found!"],
    "transcription": ["Invalid transcription format"]
  },
  "object": null
}
```

---

### 2. Init Call

| Param | Value |
|------|-------|
| URL | `/api/v1/call/init` |
| Method | POST |
| Description | Initialize a new call |

Input (InitCallForm bodyParams):
- See InitCallForm for details

Business logic:
- Load data from `Yii::$app->request->bodyParams`
- Process via `InitCallForm->process()`

Responses:
- Always: `array` with initialization result

---

### 3. Real Call Init

| Param | Value |
|------|-------|
| URL | `/api/v1/call/real-call-init` |
| Method | POST |
| Description | Initialize a real (non-test) call |

Input (RealCallInitForm bodyParams):
- See RealCallInitForm for details

Business logic:
- Load data from `Yii::$app->request->bodyParams`
- Process via `RealCallInitForm->process()`

Responses:
- Always: `array` with initialization result

---

### 4. Internet Call Webhook

| Param | Value |
|------|-------|
| URL | `/api/v1/call/internet-call-webhook` |
| Method | POST |
| Description | Webhook for internet calls |

Input (InternetCallWebhookForm bodyParams):
- Attributes set via `attributes = Yii::$app->request->bodyParams`

Business logic:
- Create `InternetCallWebhookForm`
- Set all attributes from bodyParams
- Call `save()`

Responses:
- Success: `CompanyAgentCallActivitiesAdvanced` object
- Error: `InternetCallWebhookForm` object with errors

---

## Form Classes

### 1. TranscriptionForm

File: `src/apps/api/versions/v1/forms/call/TranscriptionForm.php`  
Extends: `customs\\Model` (347 lines)

Public properties:
```php
public string $company_key = '';           // Company API key
public string $call_id = '';               // Call ID
public mixed $transcription = null;        // Transcription data
public mixed $end_by = null;               // End-by data
public int $total_tokens_llm = 0;          // Total LLM tokens
```

Private properties:
```php
private CompaniesAdvanced $_companyModel = null;        // Company model
private CompanyCallsAdvanced $_callModel = null;        // Call model
```

Key methods:
- `rules()` — validation rules
- `validateCompanyApiKey()` — validate company API key
- `setCallDetails()` — find and validate call
- `validateTranscription()` — validate transcription via partial
- `validateEndBy()` — validate end_by payload
- `process()` — main business logic
- `saveTheAvgSTTTime()` — save STT stats
- `saveTheAvgTTSTime()` — save TTS stats
- `saveTheAvgLLMTime()` — save LLM stats
- `getQueueId()` — retrieve queue ID for WebSocket
- `setSocket()` — send data via WebSocket

Partial Forms Integration:
- `EndByForm` — validate end_by data
- `TranscriptionForm` (partial) — validate transcription data

Statistics Features:
- LLM processing time tracking
- STT (Speech-to-Text) metrics
- TTS (Text-to-Speech) metrics
- Average calculation and storage

WebSocket Integration:
- Real-time updates via WebSocket Client
- STT channel updates for live statistics
- TTS channel updates for live statistics
- Queue-based channel identification

---

### 2. InitCallForm

File: `src/apps/api/versions/v1/forms/call/InitCallForm.php`  
Extends: `yii\\base\\Model`

Details require further analysis for full documentation.

---

### 3. RealCallInitForm

File: `src/apps/api/versions/v1/forms/call/RealCallInitForm.php`  
Extends: `yii\\base\\Model`

Details require further analysis for full documentation.

---

### 4. InternetCallWebhookForm

File: `src/apps/api/versions/v1/forms/call/InternetCallWebhookForm.php`  
Extends: `yii\\base\\Model`

Details require further analysis for full documentation.

---

## Business Logic

### 1. Transcription Processing Flow
```
1. Validate company_key and call_id
2. Validate transcription structure via partial form
3. Validate end_by data (if present)
4. Register transcription in the DB
5. Execute AgentCallTrigger for automation
6. Save statistics (LLM, STT, TTS)
7. Update call record with additional info
8. Send real-time updates via WebSocket
```

### 2. Statistics Tracking
- LLM Stats: processing time and tokens count
- STT Stats: speech recognition time
- TTS Stats: speech synthesis time
- Average Calculations: averages over time
- Real-time Updates: via WebSocket

### 3. WebSocket Integration
- Channel-based communication
- Queue ID identification
- Real-time statistics updates
- STT/TTS performance monitoring

---

## Security Features

### 1. Company-based Access Control
- Validate company_key for all operations
- Cross-company data isolation
- API key validation via CompaniesAdvanced

### 2. Call Validation
- Check call existence before processing
- Company relation via call validation
- call_sid_id based lookup

### 3. Data Integrity
- Partial form validation for complex payloads
- Transaction safety for DB operations
- Error handling with detailed logging

### 4. Real-time Security
- WebSocket channel validation
- Queue-based access control
- Secure channel identification

---

## Integration Points

### Dependencies
- `CompaniesAdvanced` — company management and API keys
- `CompanyCallsAdvanced` — company calls
- `CompanyCallTranscriptionsAdvanced` — call transcriptions
- `CompanyAgentCallActivitiesAdvanced` — agent activities
- `AgentCallTrigger` — automation triggers
- `WebSocket\\Client` — real-time communication
- `SttWidget` / `TtsWidget` — real-time widgets

### Related Services
- Speech-to-Text (STT) processing
- Text-to-Speech (TTS) synthesis
- Large Language Model (LLM) integration
- WebSocket server for real-time updates
- Queue system for call management
- Trigger system for automation

### External Integrations
- AI/ML services for transcription
- VoIP providers for call management
- Real-time communication infrastructure
- Analytics and monitoring systems

---

## Usage Examples

### Process call transcription
```http
POST /api/v1/call/transcription
Content-Type: application/json

{
  "company_key": "your-api-key-here",
  "call_id": "call_sid_123456",
  "transcription": {
    "text": "Hello, this is a transcribed call...",
    "confidence": 0.95,
    "language": "en",
    "segments": [
      {"speaker": "agent", "text": "Hello, how can I help you?", "start_time": 0.0, "end_time": 3.2},
      {"speaker": "customer", "text": "I need help with my account", "start_time": 3.5, "end_time": 7.1}
    ]
  },
  "end_by": {"type": "customer", "reason": "completed", "duration": 120},
  "total_tokens_llm": 450
}
```

Response:
```json
{
  "status": true,
  "errors": {},
  "object": {
    "transcription_id": "550e8400-e29b-41d4-a716-446655440000",
    "call_id": "call_sid_123456",
    "saved": true,
    "statistics": {
      "llm": {"avg": 1.2, "total": 450},
      "stt": {"avg": 0.8},
      "tts": {"avg": 0.6}
    }
  }
}
```

### Initialize a call
```http
POST /api/v1/call/init
Content-Type: application/json

{
  "company_key": "your-api-key-here",
  "customer_phone": "+1234567890",
  "agent_id": "agent_123",
  "queue_id": "queue_456"
}
```

### Webhook for internet call
```http
POST /api/v1/call/internet-call-webhook
Content-Type: application/json

{
  "call_sid": "call_789",
  "status": "completed",
  "duration": 180,
  "from": "+1234567890",
  "to": "+0987654321",
  "recording_url": "https://example.com/recording.wav"
}
```

---

## Error Handling

### Common Errors
- "Company not found" — Invalid company_key
- "Call not found!" — Invalid call_id
- Transcription validation errors — invalid transcription structure
- EndBy validation errors — invalid end_by data format

### System Errors
- Database transaction failures
- WebSocket connection issues
- External service unavailability
- Statistics calculation errors

### Logging
- Request/response logging with unique UUID
- Error details in `transcription` category
- Performance metrics logging
- WebSocket communication logs

