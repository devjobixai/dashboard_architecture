# SmsController API Documentation

## Overview

SmsController manages SMS messaging functionality for the Jobix Dashboard API. The controller provides comprehensive SMS capabilities including sending messages to customers, testing SMS functionality, and previewing message content. It serves as the primary interface for SMS communication with customers and testing SMS delivery systems.

**Base Path:** `/api/v1/sms`  
**Controller Type:** ApiController (API Application)  
**Namespace:** `apps\api\versions\v1\controllers\SmsController`  
**Authentication:** Requires `company-key` header

---

## Endpoints

### 1. Send - Send SMS to Customer

| Parameter | Value |
|-----------|-------|
| **URL** | `/api/v1/sms/send` |
| **Method** | POST |
| **Description** | Send SMS messages to customers with dynamic content and template support |

**Input Data (SendSMSToCustomer bodyParams):**
- Customer identification and contact information
- SMS message content and template data
- Sending configuration and provider settings
- Message scheduling and delivery options

**Business Logic:**
1. Load SMS sending parameters from request
2. Validate customer information and permissions
3. Process message content and template variables
4. Configure SMS provider and delivery settings
5. Send SMS message and track delivery status

**Response:**
- **Success (200):** String confirmation of SMS delivery
- **Error (422):** SendSMSToCustomer form object with validation errors

**SMS Sending Features:**
- Customer targeting and validation
- Template-based message content
- Provider integration and configuration
- Delivery tracking and confirmation
- Error handling and retry logic

---

### 2. Send Test - Send Test SMS

| Parameter | Value |
|-----------|-------|
| **URL** | `/api/v1/sms/send-test` |
| **Method** | POST |
| **Description** | Send test SMS messages for validation and debugging purposes |

**Input Data (SendTestSms bodyParams):**
- Test recipient phone numbers
- Test message content and configuration
- Provider settings and testing parameters
- Delivery and tracking options

**Business Logic:**
1. Load test SMS parameters from request
2. Validate test recipient information
3. Process test message content
4. Configure SMS provider for testing
5. Send test SMS and return delivery results

**Response:**
- **Success (200):** String confirmation of test SMS delivery
- **Error (422):** SendTestSms form object with validation errors

**Test SMS Features:**
- Test recipient validation
- Test message customization
- Provider testing and validation
- Delivery confirmation and debugging
- Error analysis and troubleshooting

---

### 3. Preview - Preview SMS Content

| Parameter | Value |
|-----------|-------|
| **URL** | `/api/v1/sms/preview` |
| **Method** | POST |
| **Description** | Preview SMS message content with template rendering and variable substitution |

**Input Data (PreviewSms bodyParams):**
- SMS message template and content
- Template variable data and context
- Customer information for personalization
- Preview configuration options

**Business Logic:**
1. Load preview parameters from request
2. Process SMS template and variables
3. Render message content with substitutions
4. Apply formatting and length validation
5. Return rendered preview content

**Response:**
- **Success (200):** String with rendered SMS preview content
- **Error (422):** PreviewSms form object with validation errors

**Preview Features:**
- Template rendering and variable substitution
- Message length validation
- Content formatting and optimization
- Personalization with customer data
- Real-time preview generation

---

## Form Classes

### 1. SendSMSToCustomer

**File:** `src/apps/api/versions/v1/forms/sms/SendSMSToCustomer.php`  
**Extends:** `customs\Model`

**Purpose:** Handle SMS delivery to customers with full messaging capabilities
**Key Methods:**
- `sendSms()` - Process and send SMS to customer

**Features:**
- Customer identification and validation
- Message template processing
- SMS provider integration
- Delivery tracking and confirmation
- Error handling and retry logic

---

### 2. SendTestSms

**File:** `src/apps/api/versions/v1/forms/sms/SendTestSms.php`  
**Extends:** `customs\Model`

**Purpose:** Handle test SMS sending for validation and debugging
**Key Methods:**
- `sendSms()` - Process and send test SMS

**Features:**
- Test recipient management
- Test message configuration
- Provider testing capabilities
- Delivery confirmation and debugging
- Error analysis and reporting

---

### 3. PreviewSms

**File:** `src/apps/api/versions/v1/forms/sms/PreviewSms.php`  
**Extends:** `customs\Model`

**Purpose:** Generate SMS message previews with template rendering
**Key Methods:**
- `preview()` - Generate and return SMS preview

**Features:**
- Template processing and rendering
- Variable substitution and personalization
- Message length and format validation
- Customer data integration
- Real-time preview generation

---

## Business Logic

### 1. SMS Delivery Flow
```
1. Customer SMS Sending
   - Validate customer information
   - Process message template and content
   - Configure SMS provider settings
   - Send message and track delivery
   - Handle errors and retry logic

2. Message Processing
   - Template variable substitution
   - Content formatting and optimization
   - Length validation and truncation
   - Personalization with customer data
   - Provider-specific formatting

3. Delivery Tracking
   - Send message through provider API
   - Track delivery status and confirmation
   - Handle failed deliveries and retries
   - Update delivery statistics and logs
```

### 2. Testing and Preview Flow
```
1. Test SMS Processing
   - Validate test recipient information
   - Process test message content
   - Configure provider for testing
   - Send test message and confirm delivery
   - Provide debugging and error information

2. Preview Generation
   - Load message template and variables
   - Render content with substitutions
   - Validate message format and length
   - Return formatted preview content
   - Handle template errors and validation

3. Validation and Debugging
   - Template syntax validation
   - Variable substitution testing
   - Provider configuration verification
   - Error reporting and troubleshooting
```

### 3. Provider Integration Flow
```
1. SMS Provider Management
   - Configure provider-specific settings
   - Handle authentication and credentials
   - Manage rate limiting and quotas
   - Process provider responses and errors

2. Message Formatting
   - Apply provider-specific formatting
   - Handle character encoding and limits
   - Optimize content for delivery
   - Validate message structure and format

3. Delivery Management
   - Route messages through appropriate providers
   - Handle delivery confirmations and webhooks
   - Process delivery failures and retries
   - Update delivery status and tracking
```

---

## Security Features

### 1. Company-based Access Control
- All SMS operations filtered by company context
- Customer validation and access permissions
- Phone number privacy and protection
- Message content security and validation

### 2. Input Validation
- Phone number format validation
- Message content sanitization
- Template variable validation
- Provider configuration security

### 3. SMS Security
- Rate limiting and abuse prevention
- Content filtering and compliance
- Delivery tracking and audit logging
- Error handling with security considerations

---

## Integration Points

### Dependencies
- `SendSMSToCustomer` - Customer SMS delivery form
- `SendTestSms` - Test SMS functionality form
- `PreviewSms` - Message preview generation form
- Company authentication system
- SMS provider integrations

### External Services
- **SMS Providers** - Twilio, AWS SNS, and other SMS services
- **Template Engines** - Message template processing
- **Customer Database** - Customer information and preferences
- **Analytics Systems** - SMS delivery tracking and reporting

### Related Components
- **Customer Management** - Customer data and communication preferences
- **Template System** - Message template management
- **Analytics Dashboard** - SMS delivery statistics and reporting
- **Queue System** - Bulk SMS sending and processing

---

## Usage Examples

### Send SMS to customer
```http
POST /api/v1/sms/send
Content-Type: application/json
company-key: your-company-api-key

{
  "customer_id": "550e8400-e29b-41d4-a716-446655440000",
  "phone_number": "+1234567890",
  "message": "Hi {{customer.name}}, your order #{{order.id}} is ready for pickup!",
  "template_variables": {
    "customer": {
      "name": "John Doe"
    },
    "order": {
      "id": "ORD-123"
    }
  },
  "provider": "twilio",
  "schedule_time": null
}
```

### Send test SMS
```http
POST /api/v1/sms/send-test
Content-Type: application/json
company-key: your-company-api-key

{
  "test_numbers": ["+1234567890", "+0987654321"],
  "message": "Test message: {{test.variable}}",
  "template_variables": {
    "test": {
      "variable": "Hello World"
    }
  },
  "provider": "twilio",
  "test_mode": true
}
```

### Preview SMS content
```http
POST /api/v1/sms/preview
Content-Type: application/json
company-key: your-company-api-key

{
  "message_template": "Hello {{customer.name}}, your appointment on {{appointment.date}} is confirmed!",
  "template_variables": {
    "customer": {
      "name": "Jane Smith"
    },
    "appointment": {
      "date": "2024-01-15 10:00 AM"
    }
  },
  "customer_id": "550e8400-e29b-41d4-a716-446655440000"
}
```

### Send promotional SMS
```http
POST /api/v1/sms/send
Content-Type: application/json
company-key: your-company-api-key

{
  "customer_id": "550e8400-e29b-41d4-a716-446655440000",
  "phone_number": "+1234567890",
  "message": "🎉 Special offer for {{customer.name}}! Get 20% off your next purchase. Use code: SAVE20. Valid until {{offer.expires}}.",
  "template_variables": {
    "customer": {
      "name": "John Doe"
    },
    "offer": {
      "expires": "Jan 31, 2024"
    }
  },
  "provider": "twilio",
  "tracking_campaign": "winter_promo"
}
```

---

## Error Handling

### Common Errors
- **"Invalid phone number"** - Phone number format validation failure
- **"Customer not found"** - Invalid customer ID or access denied
- **"Message too long"** - SMS content exceeds character limits
- **"Provider error"** - SMS provider integration failure

### SMS Delivery Errors
- Phone number validation failures
- Message content formatting errors
- Provider authentication issues
- Rate limiting exceeded
- Network connectivity problems

### Template Processing Errors
- Invalid template syntax
- Missing template variables
- Variable substitution failures
- Content rendering errors

### HTTP Status Codes
- **200 OK** - Successful SMS operations
- **422 Unprocessable Entity** - Validation errors or business logic failures

---

## Performance Considerations

### Message Processing
- Efficient template rendering
- Optimized variable substitution
- Minimal processing latency
- Scalable message handling

### Provider Integration
- Connection pooling and reuse
- Rate limiting compliance
- Bulk sending optimization
- Error handling and retries

### Delivery Tracking
- Asynchronous delivery confirmation
- Efficient status updates
- Performance metrics collection
- Scalable tracking infrastructure

---

*SmsController API provides comprehensive SMS messaging capabilities with template support, testing functionality, and preview generation for enterprise customer communication.*