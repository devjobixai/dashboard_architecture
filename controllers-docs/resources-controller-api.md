# ResourcesController API Documentation

## Overview

ResourcesController provides utility resources and testing capabilities for the Jobix Dashboard API. The controller offers comprehensive functionality for managing external resources including phone number providers, LLM (Large Language Model) integrations, dropdown data sources, API testing utilities, and node documentation services. It serves as a centralized resource hub for various system components.

**Base Path:** `/api/v1/resources`  
**Controller Type:** ApiController (API Application)  
**Namespace:** `apps\api\versions\v1\controllers\ResourcesController`  
**Authentication:** Requires `company-key` header

---

## Endpoints

### 1. Phone Numbers - Manage Phone Number Resources

| Parameter | Value |
|-----------|-------|
| **URL** | `/api/v1/resources/phone-numbers` |
| **Method** | POST |
| **Description** | Manage phone number resources through provider integrations |

**Input Data (bodyParams via PhoneNumberProviders component):**
- Phone number provider parameters
- Operation type and configuration
- Provider-specific settings and credentials

**Business Logic:**
1. Access PhoneNumberProviders component
2. Parse request parameters for provider operations
3. Execute provider-specific API calls
4. Process provider responses and format results
5. Return formatted phone number data or operation results

**Response:**
- **Success (200):** Array with phone number operation results
- **Error (422):** ApiForm object with provider errors
- **Exception:** Error details from provider integration

**Provider Integration Features:**
- Multi-provider phone number management
- Provider API abstraction layer
- Configuration-based provider selection
- Error handling across different providers

---

### 2. LLM Test Request - Test Language Model Requests

| Parameter | Value |
|-----------|-------|
| **URL** | `/api/v1/resources/llm-test-request` |
| **Method** | POST |
| **Description** | Send test requests to Language Model providers for testing and validation |

**Input Data (LlmTestRequestForm bodyParams):**
- LLM provider configuration
- Test request parameters and prompts
- Model-specific settings and options

**Business Logic:**
1. Validate LLM test request parameters
2. Configure LLM provider connection
3. Send test request to specified model
4. Process LLM response and format results
5. Return test results with performance metrics

**Response:**
- **Success (200):** Array with LLM test response and metrics
- **Error (422):** LlmTestRequestForm object with validation errors

**LLM Testing Features:**
- Multi-provider LLM testing support
- Request/response validation
- Performance metrics collection
- Error handling and debugging information

---

### 3. LLM Providers Dropdown - Get Available LLM Providers

| Parameter | Value |
|-----------|-------|
| **URL** | `/api/v1/resources/llm-providers-dropdown` |
| **Method** | GET |
| **Description** | Retrieve list of available Language Model providers for dropdown selection |

**Input Data:** None (static provider list)

**Business Logic:**
1. Access LLM provider configuration
2. Format provider list for dropdown consumption
3. Include provider capabilities and status
4. Return formatted dropdown data

**Response:**
- **Success (200):** Array of LLM provider options for dropdown

**Dropdown Features:**
- Provider name and display labels
- Provider capability information
- Status indicators (available/unavailable)
- Configuration requirements

---

### 4. Node Documentation - Get Node Documentation

| Parameter | Value |
|-----------|-------|
| **URL** | `/api/v1/resources/node-documentation` |
| **Method** | POST |
| **Description** | Generate or retrieve documentation for workflow nodes |

**Input Data (NodeDocumentationForm bodyParams):**
- Node type and configuration parameters
- Documentation generation options
- Context and formatting preferences

**Business Logic:**
1. Parse node documentation request
2. Load node type definitions and templates
3. Generate comprehensive node documentation
4. Format documentation based on request preferences
5. Return formatted documentation content

**Response:**
- **Success (200):** Generated node documentation content
- **Error (422):** NodeDocumentationForm object with validation errors

**Documentation Features:**
- Dynamic node documentation generation
- Template-based documentation formatting
- Context-aware content generation
- Multiple output format support

---

### 5. Send Test API Request - Test External API Calls

| Parameter | Value |
|-----------|-------|
| **URL** | `/api/v1/resources/send-test-api-request` |
| **Method** | POST |
| **Description** | Send test requests to external APIs for validation and debugging |

**Input Data (ApiTestRequestForm bodyParams):**
- Target API endpoint and configuration
- Request method, headers, and payload
- Authentication and security parameters

**Business Logic:**
1. Validate API test request parameters
2. Configure HTTP client with specified settings
3. Execute API request to external endpoint
4. Capture response data and performance metrics
5. Return test results with debugging information

**Response:**
- **Success (200):** Array with API test results and response data
- **Error (422):** ApiTestRequestForm object with validation errors

**API Testing Features:**
- Comprehensive HTTP method support
- Header and authentication management
- Response validation and analysis
- Performance timing and debugging data

---

### 6. Agents Dropdown - Get Available Agents

| Parameter | Value |
|-----------|-------|
| **URL** | `/api/v1/resources/agents-dropdown?agent_name={search_term}` |
| **Method** | GET |
| **Description** | Retrieve list of available AI agents for dropdown selection with optional filtering |

**URL Parameters:**
- `agent_name` (string, optional) - Search term for agent filtering

**Input Data (AgentsDropdownForm):**
- Agent search and filtering parameters loaded from query

**Business Logic:**
1. Load available AI agents from company context
2. Apply name-based filtering if search term provided
3. Format agent data for dropdown consumption
4. Include agent status and availability information
5. Return filtered and formatted agent list

**Response:**
- **Success (200):** Array of agent options for dropdown
- **Error (422):** AgentsDropdownForm object with validation errors

---

### 7. Actions Dropdown - Get Available Actions

| Parameter | Value |
|-----------|-------|
| **URL** | `/api/v1/resources/actions-dropdown?action_name={search_term}` |
| **Method** | GET |
| **Description** | Retrieve list of available company actions for dropdown selection with optional filtering |

**URL Parameters:**
- `action_name` (string, optional) - Search term for action filtering

**Input Data (ActionsDropdownForm):**
- Action search and filtering parameters loaded from query

**Business Logic:**
1. Load available company actions from database
2. Apply name-based filtering if search term provided
3. Format action data for dropdown consumption
4. Include action status and configuration details
5. Return filtered and formatted action list

**Response:**
- **Success (200):** Array of action options for dropdown
- **Error (422):** ActionsDropdownForm object with validation errors

---

### 8. Customers Emails Dropdown - Get Customer Email Options

| Parameter | Value |
|-----------|-------|
| **URL** | `/api/v1/resources/customers-emails-dropdown?customer_email={search_term}` |
| **Method** | GET |
| **Description** | Retrieve list of customer emails for dropdown selection with optional filtering |

**URL Parameters:**
- `customer_email` (string, optional) - Search term for email filtering

**Input Data (CustomersEmailDropdownForm):**
- Customer email search and filtering parameters

**Business Logic:**
1. Load customer email addresses from company database
2. Apply email-based filtering if search term provided
3. Format email data for dropdown consumption
4. Include customer context and validation status
5. Return filtered and formatted email list

**Response:**
- **Success (200):** Array of customer email options for dropdown
- **Error (422):** ActionsDropdownForm object with validation errors (Note: appears to be type mismatch in controller)

---

### 9. SMS Phone Numbers Dropdown - Get SMS Phone Number Options

| Parameter | Value |
|-----------|-------|
| **URL** | `/api/v1/resources/sms-phone-numbers-dropdown?phone_title={search_term}` |
| **Method** | GET |
| **Description** | Retrieve list of SMS-enabled phone numbers for dropdown selection with optional filtering |

**URL Parameters:**
- `phone_title` (string, optional) - Search term for phone number filtering

**Input Data (SMSPhoneNumbersDropdownForm):**
- Phone number search and filtering parameters

**Business Logic:**
1. Load SMS-enabled phone numbers from company configuration
2. Apply title-based filtering if search term provided
3. Format phone number data for dropdown consumption
4. Include provider and capability information
5. Return filtered and formatted phone number list

**Response:**
- **Success (200):** Array of SMS phone number options for dropdown
- **Error (422):** SMSPhoneNumbersDropdownForm object with validation errors

---

## Form Classes

### 1. LlmTestRequestForm

**File:** `src/apps/api/versions/v1/forms/resources/LlmTestRequestForm.php`  
**Extends:** `customs\Model`

**Purpose:** Handle LLM provider testing and validation
**Key Methods:**
- `sendLlmRequest()` - Execute LLM test request and return results

**Features:**
- Multi-provider LLM integration
- Request validation and formatting
- Response processing and metrics
- Error handling and debugging

---

### 2. LlmProvidersDropdownForm

**File:** `src/apps/api/versions/v1/forms/resources/LlmProvidersDropdownForm.php`  
**Extends:** `customs\Model`

**Purpose:** Provide LLM provider options for dropdown selection
**Features:**
- Static provider configuration
- Provider capability enumeration
- Status and availability tracking
- Dropdown-optimized formatting

---

### 3. NodeDocumentationForm

**File:** `src/apps/api/versions/v1/forms/resources/NodeDocumentationForm.php`  
**Extends:** `customs\Model`

**Purpose:** Generate documentation for workflow nodes
**Key Methods:**
- `documentation()` - Generate and return node documentation

**Features:**
- Dynamic documentation generation
- Template-based formatting
- Context-aware content creation
- Multiple output formats

---

### 4. ApiTestRequestForm

**File:** `src/apps/api/versions/v1/forms/resources/ApiTestRequestForm.php`  
**Extends:** `customs\Model`

**Purpose:** Handle external API testing and validation
**Key Methods:**
- `sendApiRequest()` - Execute API test request and return results

**Features:**
- HTTP method configuration
- Authentication and header management
- Response validation and analysis
- Performance metrics collection

---

### 5. AgentsDropdownForm

**File:** `src/apps/api/versions/v1/forms/resources/AgentsDropdownForm.php`  
**Extends:** `customs\Model`

**Public Properties:**
```php
public mixed $agent_name = null;       // Agent search filter
```

**Purpose:** Provide agent options for dropdown selection
**Features:**
- Agent search and filtering
- Company-based agent access
- Status and availability information
- Dropdown-optimized data structure

---

### 6. ActionsDropdownForm

**File:** `src/apps/api/versions/v1/forms/resources/ActionsDropdownForm.php`  
**Extends:** `customs\Model`

**Public Properties:**
```php
public mixed $action_name = null;      // Action search filter
```

**Purpose:** Provide action options for dropdown selection
**Features:**
- Action search and filtering
- Company-based action access
- Configuration and status details
- Dropdown-optimized formatting

---

### 7. CustomersEmailDropdownForm

**File:** `src/apps/api/versions/v1/forms/resources/CustomersEmailDropdownForm.php`  
**Extends:** `customs\Model`

**Public Properties:**
```php
public mixed $customer_email = null;   // Email search filter
```

**Purpose:** Provide customer email options for dropdown selection
**Features:**
- Email search and filtering
- Customer context integration
- Validation status tracking
- Privacy-aware data handling

---

### 8. SMSPhoneNumbersDropdownForm

**File:** `src/apps/api/versions/v1/forms/resources/SMSPhoneNumbersDropdownForm.php`  
**Extends:** `customs\Model`

**Public Properties:**
```php
public mixed $phone_title = null;      // Phone number search filter
```

**Purpose:** Provide SMS phone number options for dropdown selection
**Features:**
- Phone number search and filtering
- SMS capability validation
- Provider integration
- Configuration status tracking

---

## Business Logic

### 1. Resource Management Flow
```
1. External Resource Integration
   - Phone number provider management
   - LLM provider configuration
   - API endpoint testing and validation
   - Provider capability discovery

2. Data Source Management
   - Company-specific resource filtering
   - Search and filtering capabilities
   - Status and availability tracking
   - Performance optimization

3. Dropdown Data Provision
   - Formatted data for UI components
   - Search and filtering support
   - Context-aware data selection
   - Real-time availability updates
```

### 2. Testing and Validation Flow
```
1. LLM Testing Process
   - Provider configuration validation
   - Test request formatting
   - Response processing and analysis
   - Performance metrics collection

2. API Testing Process
   - External endpoint validation
   - Request configuration and execution
   - Response analysis and debugging
   - Security and authentication testing

3. Documentation Generation
   - Node type analysis
   - Template-based content creation
   - Context-aware documentation
   - Multi-format output support
```

### 3. Provider Integration Flow
```
1. Phone Number Management
   - Multi-provider abstraction
   - Provider-specific API handling
   - Configuration management
   - Error handling and fallback

2. Component Integration
   - PhoneNumberProviders component usage
   - Configuration-based provider selection
   - API request routing and processing
   - Response formatting and validation
```

---

## Security Features

### 1. Company-based Access Control
- All resource access filtered by company context
- Agent and action access validation
- Customer data privacy protection
- Resource ownership verification

### 2. External Integration Security
- Provider authentication management
- API key and credential protection
- Request validation and sanitization
- Rate limiting and abuse protection

### 3. Testing Security
- Sandbox environment for API testing
- Credential isolation for test requests
- Response data sanitization
- Audit logging for test activities

---

## Integration Points

### Dependencies
- `PhoneNumberProviders` - Phone number provider management component
- `ApiForm` - Phone number provider form abstraction
- Company authentication system
- External LLM providers (OpenAI, etc.)
- SMS and email service providers

### External Services
- **Phone Providers** - Twilio, Vonage, and other telephony services
- **LLM Providers** - OpenAI, Anthropic, and other AI services
- **Email Services** - Various email delivery providers
- **SMS Services** - SMS delivery and management providers

### Related Components
- **Agent Management** - AI agent configuration and deployment
- **Action Management** - Company action automation
- **Customer Management** - Customer data and communication
- **Flow Builder** - Node documentation and workflow management

---

## Usage Examples

### Test LLM request
```http
POST /api/v1/resources/llm-test-request
Content-Type: application/json
company-key: your-company-api-key

{
  "provider": "openai",
  "model": "gpt-4",
  "prompt": "Generate a customer service response",
  "parameters": {
    "max_tokens": 150,
    "temperature": 0.7
  }
}
```

### Get LLM providers dropdown
```http
GET /api/v1/resources/llm-providers-dropdown
company-key: your-company-api-key
```

### Test external API
```http
POST /api/v1/resources/send-test-api-request
Content-Type: application/json
company-key: your-company-api-key

{
  "url": "https://api.example.com/webhook",
  "method": "POST",
  "headers": {
    "Authorization": "Bearer token123",
    "Content-Type": "application/json"
  },
  "body": {
    "test": "data"
  }
}
```

### Get agents dropdown with search
```http
GET /api/v1/resources/agents-dropdown?agent_name=customer
company-key: your-company-api-key
```

### Get actions dropdown with search
```http
GET /api/v1/resources/actions-dropdown?action_name=notification
company-key: your-company-api-key
```

### Generate node documentation
```http
POST /api/v1/resources/node-documentation
Content-Type: application/json
company-key: your-company-api-key

{
  "node_type": "condition",
  "format": "markdown",
  "include_examples": true,
  "context": {
    "flow_id": "uuid-here"
  }
}
```

### Manage phone numbers
```http
POST /api/v1/resources/phone-numbers
Content-Type: application/json
company-key: your-company-api-key

{
  "operation": "list",
  "provider": "twilio",
  "filters": {
    "country": "US",
    "capabilities": ["voice", "sms"]
  }
}
```

---

## Error Handling

### Common Errors
- **"Provider not available"** - External service unavailable
- **"Invalid configuration"** - Provider or service configuration errors
- **"Search term too short"** - Insufficient search criteria
- **"Resource not found"** - Requested resource doesn't exist

### Provider Integration Errors
- External API authentication failures
- Provider service outages
- Rate limiting exceeded
- Invalid provider responses

### Testing Errors
- LLM request validation failures
- API endpoint connectivity issues
- Authentication and authorization errors
- Response parsing and validation failures

### HTTP Status Codes
- **200 OK** - Successful operations
- **422 Unprocessable Entity** - Validation errors or business logic failures

---

*ResourcesController API provides comprehensive utility resources with external service integrations, testing capabilities, and optimized data sources for enterprise application components.*