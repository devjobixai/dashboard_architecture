# CallsController Documentation

## Overview

CallsController manages calls in the Jobix Dashboard admin panel, including listing calls, viewing call details, related sales, and call transcriptions.

Base path: `/calls`  
Controller type: WebController (Admin Application)  
Namespace: `apps\\admin\\controllers\\CallsController`

---

## Endpoints

### 1. Index — Calls List

| Param | Value |
|------|-------|
| URL | `/calls/index` |
| Method | GET |
| Description | Display all company calls with search and filtering |

Input (CallsSearch queryParams):
- Query parameters for searching and filtering calls
- Processed via CallsSearch form

Business logic:
1. Instantiate CallsSearch model
2. Load search params from query string
3. Validate search parameters
4. Execute search and build DataProvider

Response:
- Success: HTML view with calls list
- Data: `dataProvider` (search results), `searchModel` (search form)

---

### 2. Details — Call Details

| Param | Value |
|------|-------|
| URL | `/calls/details/{uuid}` |
| Method | GET |
| Description | Detailed view of a specific call with system information |

URL Parameters:
- `uuid` (string) — call UUID

Input (CallModelDetails):

| Field | Type | Required | Validation | Description |
|------|------|----------|------------|-------------|
| `uuid` | string | ✓ | required, custom validation | Call UUID |

CallModelDetails validation:
```php
[['uuid'], 'required']
[['uuid'], 'setCallDetails'] // Custom: find call by UUID and validate existence
```

Business logic:
1. Load call UUID from URL
2. Find call via CompanyCallsAdvanced
3. Validate call existence
4. Process system-specific details by provider
5. Fetch call recordings from various systems

Provider Integration:
- Call Widget: Widget ID and recording URL from additional_info
- Twilio: Integration with Twilio API for recordings
- Jobix/Vonage/Telico/IDT/Didlogic: JobixHelper for recordings
- Recording Management: Automatic download and caching

Call Details Processing:
```php
return match ($call->system) {
    CompanyCallsAdvanced::INTERNET_CALL_WIDGET => $this->getCallWidgetDetails(),
    TwilioProvider::slug() => $this->getTwilioDetails(),
    JobixProvider::slug(), JobixVonageProvider::slug() => $this->getJobixDetails(),
    default => [],
};
```

Responses:
- Success: HTML view with call details
- Not Found: Redirect to `/calls/index`
- Data: `callDetails` (call, agent, customer, system info)

---

### 3. Sales — Sales Related to Call

| Param | Value |
|------|-------|
| URL | `/calls/sales/{uuid}` |
| Method | GET (AJAX/PJAX only) |
| Description | AJAX endpoint to load sales related to a specific call |

URL Parameters:
- `uuid` (string) — call UUID (call_sid_id)

Request validation:
- AJAX or PJAX request required
- Other request types return 404

Input (CallSalesSearch):

| Field | Type | Required | Validation | Description |
|------|------|----------|------------|-------------|
| `call_sid_id` | string | ✓ | required, string | Call ID to search related sales |
| `status` | string | ✗ | string | Optional sale status filter |

CallSalesSearch validation:
```php
[['call_sid_id'], 'required']
[['call_sid_id', 'status'], 'string']
```

Business logic:
1. Validate AJAX/PJAX request
2. Create CallSalesSearch with call UUID
3. Merge additional query parameters
4. Search sales via CompanySalesAdvanced
5. Filter by company_id and call_sid_id
6. Pagination (50 per page)

Search Features:
- Company-based filtering by selectedUserCompany
- Optional status filter
- Sorting by status and created_at
- Default sorting: created_at DESC

Responses:
- Success: HTML partial view with sales
- Invalid Request: 404 NotFoundHttpException
- Template: `partials/sales-template`

---

### 4. Transcription — Call Transcription

| Param | Value |
|------|-------|
| URL | `/calls/transcription/{uuid}` |
| Method | GET (AJAX only) |
| Description | AJAX endpoint to load a call's transcription |

URL Parameters:
- `uuid` (string) — call UUID

Request validation:
- AJAX request required (otherwise 404)
- Output buffer cleaned for a pure JSON response

Input (TranscriptionSearch):
- UUID passed via constructor
- GET params for pagination and filtering

Business logic:
1. Validate AJAX request
2. Instantiate TranscriptionSearch with UUID
3. Load extra GET parameters
4. Search transcriptions via CompanyCallTranscriptionsAdvanced
5. Serialize results via Serializer
6. Determine if next page available

Advanced Features:
- BaseSearch integration
- Widget integrations (AgentNameWidget, CallStatusWidget, DurationWidget)
- Custom rules via DetailsRule
- Pagination meta information
- Template rendering with serialized data

Responses:

Success:
```json
{
  "nextPageAvailable": true,
  "content": "HTML content of transcription partial"
}
```

No More Pages:
```json
{
  "nextPageAvailable": false,
  "content": "HTML content of transcription partial"
}
```

Invalid Request:
- Error: 404 NotFoundHttpException

Template: `partials/transcription`

---

## Form Classes

### 1. CallsSearch

File: `src/apps/admin/forms/calls/CallsSearch.php`  
Extends: `customs\\Model` (332 lines)

Purpose: Advanced search and filtering of company calls.

Key methods:
- `rules()` — validation rules for search fields
- `search()` — main search with filtering and pagination
- `getSort()` — sorting configuration
- `getSystems()` — phone systems list
- `customersDropdown()` — customers dropdown for filtering
- `agentsDropdown()` — agents dropdown for filtering
- `statusesDropdown()` — call statuses dropdown
- `durationDropdown()` — duration range dropdown

Search Features:
- Advanced filtering by multiple criteria
- Phone number providers integration
- Carbon date parsing for ranges
- Company-based data isolation
- Complex multi-field sorting
- Pagination support

Integration Points:
- CompanyCallsAdvanced for primary data
- CompanyCustomersAdvanced for customer info
- PhoneNumberProviders for system integration
- Constants for statuses and settings

---

### 2. CallModelDetails

File: `src/apps/admin/forms/calls/CallModelDetails.php`  
Extends: `customs\\Model` (183 lines)

Public properties:
```php
public mixed $uuid = null;           // Call UUID
```

Private properties:
```php
protected mixed $_call = null;       // CompanyCallsAdvanced instance
```

Key methods:
- `rules()` — UUID validation and call lookup
- `setCallDetails()` — custom validation to find call
- `process()` — process and return call details
- `getCallDetails()` — system-specific details
- Provider-specific methods for different systems

Provider Integration Methods:
- `getCallWidgetDetails()` — Internet Call Widget
- `getTwilioDetails()` — Twilio integration
- `getJobixDetails()` — Jobix and related providers
- `getTwilioRecordingUrl()` — Twilio recording retrieval
- `getJobixRecordingUrl()` — Jobix recording retrieval

Recording Management:
- Automatic file downloading and caching
- Multiple provider support (Twilio, Jobix, etc.)
- Local file storage at `@adminRecords` alias
- Error handling for provider failures

Business Logic:
- Match-based provider selection
- Recording URL construction
- File existence checking
- Automatic file downloading from external APIs
- Error-resilient processing

---

### 3. CallSalesSearch

File: `src/apps/admin/forms/calls/CallSalesSearch.php`  
Extends: `customs\\Model` (85 lines)

Public properties:
```php
public mixed $status = null;         // Sale status filter
public mixed $call_sid_id = null;    // Call ID (required)
```

Validation Rules:
```php
[['call_sid_id'], 'required']
[['call_sid_id', 'status'], 'string']
```

Key methods:
- `rules()` — validation rules
- `search()` — search sales by call
- `getSort()` — sorting configuration

Search Features:
- Company-based filtering via selectedUserCompany
- Call-specific filtering by call_sid_id
- Optional status filtering
- Pagination (50 per page)
- Sorting by status and created_at

Default Sorting:
- Primary: created_at DESC
- Secondary: status ASC

---

### 4. TranscriptionSearch

File: `src/apps/admin/forms/calls/TranscriptionSearch.php`  
Extends: `apps\\admin\\forms\\base\\BaseSearch` (247 lines)

Advanced Search Form Features:
- BaseSearch inheritance with extended capabilities
- Constructor-based configuration
- Advanced filtering and sorting setup
- Widgets integration for UI components

Integration Components:
- AgentNameWidget for agent display
- CallStatusWidget for call statuses
- DurationWidget for call duration
- DetailsRule for custom validation

Key methods:
- `__construct()` — search form configuration
- `rules()` — validation rules
- `setSortFields()` — sorting fields setup
- `setFilterFields()` — filter fields setup
- `getBaseQuery()` — base query to CompanyCallTranscriptionsAdvanced

Advanced Features:
- Widget-based UI component integration
- Custom validation rules
- Complex query building
- Serialization support for AJAX responses
- Template-based rendering

---

## Business Logic

### 1. Calls Listing Flow
```
1. Load calls index page
2. Initialize CallsSearch with query parameters
3. Validate search parameters
4. Execute search with company-based filtering
5. Apply sorting and pagination
6. Render results with search form
```

### 2. Call Details Flow
```
1. Extract UUID from URL parameter
2. Validate call existence through CallModelDetails
3. Determine call provider/system type
4. Fetch provider-specific details (recordings, etc.)
5. Download and cache recording files if needed
6. Return comprehensive call information
```

### 3. Sales Data Flow
```
1. Validate AJAX/PJAX request
2. Extract call UUID and create CallSalesSearch
3. Filter sales by call_sid_id and company
4. Apply optional status filtering
5. Paginate results (50 per page)
6. Render partial template with sales data
```

### 4. Transcription Flow
```
1. Validate AJAX request and clean output buffer
2. Create TranscriptionSearch with call UUID
3. Load additional query parameters for filtering
4. Search transcriptions with BaseSearch functionality
5. Serialize results and determine pagination status
6. Return JSON with content and pagination info
```

---

## Security Features

### 1. Company-based Access Control
- All searches are isolated by selectedUserCompany
- Company ID filtering in all queries
- No cross-company data access

### 2. Request Type Validation
- AJAX-only endpoints for sales and transcription
- PJAX support for sales endpoint
- 404 responses for invalid request types

### 3. UUID Validation
- Required UUID validation for call details
- Existence checking via database lookups
- Safe UUID parameter handling

### 4. Provider Security
- Error handling for external API calls
- Safe file downloading with authentication
- Local file caching with proper paths

---

## Integration Points

### Dependencies
- `CompanyCallsAdvanced` — calls model
- `CompanySalesAdvanced` — sales model
- `CompanyCallTranscriptionsAdvanced` — transcriptions model
- `PhoneNumberProviders` — providers integration
- `JobixHelper` — Jobix provider helper
- `Twilio\\Rest\\Client` — Twilio API integration
- `Serializer` — data serialization for AJAX

### Provider Integrations
- Twilio: Recording retrieval with authentication
- Jobix/Vonage/Telico/IDT/Didlogic: JobixHelper integration
- Call Widget: Widget-based call handling
- Recording Management: Multi-provider recording support

### Widget Components
- AgentNameWidget — agent display
- CallStatusWidget — status display
- DurationWidget — duration formatting
- CallCountsWidget — statistics display

---

## Usage Examples

### View calls list
```http
GET /calls/index?status=completed&agent_id=123&date_from=2023-01-01&date_to=2023-12-31
```

Response: HTML view with calls and filters

### View call details
```http
GET /calls/details/550e8400-e29b-41d4-a716-446655440000
```

Response: HTML view with call details, recording, agent and customer

### AJAX load sales
```http
GET /calls/sales/call_sid_123456
X-Requested-With: XMLHttpRequest

Response: HTML partial with sales related to the call
```

### AJAX load transcription
```http
GET /calls/transcription/550e8400-e29b-41d4-a716-446655440000?page=2
X-Requested-With: XMLHttpRequest
```

Response:
```json
{
  "nextPageAvailable": false,
  "content": "<div class=\"transcription-item\">...</div>"
}
```

