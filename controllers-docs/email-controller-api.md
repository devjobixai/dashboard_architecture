# EmailController API Documentation

## Overview

EmailController manages email template rendering and test email sending functionality for the Jobix Dashboard API. The controller provides specialized endpoints for email template preview generation and sending test emails through the Brevo email service integration with advanced template engine support.

**Base Path:** `/api/v1/email`  
**Controller Type:** ApiController (API Application)  
**Namespace:** `apps\api\versions\v1\controllers\EmailController`  
**Authentication:** Requires `company-key` header

---

## Endpoints

### 1. Preview - Email Template Preview

| Parameter | Value |
|-----------|-------|
| **URL** | `/api/v1/email/preview` |
| **Method** | POST |
| **Description** | Generate HTML preview of email template with customer data substitution |

**Input Data (PreviewEmail bodyParams):**

| Field | Type | Required | Validation | Description |
|-------|------|----------|-----------|-------------|
| `html_template` | string | ✓ | required, string | HTML email template with Twig syntax |
| `customer_uuid` | string | ✗ | UUID, exists in customers | Customer UUID for data substitution |

**Detailed Validation PreviewEmail:**
```php
// Required fields
[['html_template'], 'required']

// UUID validation with customer existence check
[['customer_uuid'], UuidHelper::class]
[['customer_uuid'], 'exist', 'targetClass' => CompanyCustomersAdvanced::class,
 'targetAttribute' => ['customer_uuid' => 'uuid']]

// String validation
[['html_template'], 'string']
```

**Business Logic:**
1. Validate HTML template format and customer UUID
2. Load customer data from MongoDB if customer UUID provided
3. Generate MD5 hash for template name identification
4. Render template using Twig template engine with customer variables
5. Return rendered HTML or error message

**Response:**
- **Success (200):** Rendered HTML template as string
- **Validation Error (422):** PreviewEmail form object with validation errors
- **Template Error (200):** Error message string with template processing details

**Template Engine Features:**
- Twig syntax support for variable substitution
- Customer data integration from MongoDB
- Dynamic template compilation
- Error handling with user-friendly messages

---

### 2. Send Test - Send Test Email

| Parameter | Value |
|-----------|-------|
| **URL** | `/api/v1/email/send-test` |
| **Method** | POST |
| **Description** | Send test emails using template with Brevo API integration |

**Input Data (SendTestEmail bodyParams):**

| Field | Type | Required | Validation | Description |
|-------|------|----------|-----------|-------------|
| `html_template` | string | ✓ | required, string | HTML email template with Twig syntax |
| `subject` | string | ✓ | required, string | Email subject line template |
| `sender_email` | string | ✓ | required, string | Sender email address |
| `sender_name` | string | ✓ | required, string | Sender display name |
| `brevo_api_key` | string | ✓ | required, string | Brevo API key for email service |
| `emails` | array | ✓ | required, each email validated | Array of recipient email addresses |
| `pre_head` | string | ✗ | string | Email preheader text |
| `reply_to_email` | string | ✗ | string | Reply-to email address |
| `customer_uuid` | string | ✗ | UUID, exists in customers | Customer UUID for data substitution |

**Detailed Validation SendTestEmail:**
```php
// Required fields
[['html_template', 'subject', 'sender_email', 'sender_name', 'brevo_api_key', 'emails'], 'required']

// String validation
[['html_template', 'subject', 'sender_name', 'brevo_api_key', 'pre_head'], 'string']
[['sender_email', 'reply_to_email'], 'string']

// Email array validation
[['emails'], 'each', 'rule' => ['email']]

// UUID validation with customer existence check
[['customer_uuid'], UuidHelper::class]
[['customer_uuid'], 'exist', 'targetClass' => CompanyCustomersAdvanced::class,
 'targetAttribute' => ['customer_uuid' => 'uuid']]
```

**Business Logic:**
1. Validate all email sending parameters
2. Load customer data from MongoDB if customer UUID provided
3. Generate MD5 hash for template identification
4. Render email template, subject, and preheader using Twig engine
5. Create email payload for each recipient using EmailHelper
6. Send emails through Brevo API integration
7. Track successful and failed email deliveries
8. Return success status or detailed error information

**Email Processing Flow:**
```php
1. Template Rendering:
   - HTML template → Rendered email body
   - Subject template → Dynamic subject line
   - Preheader template → Email preview text

2. Email Payload Generation:
   - Recipient formatting with test name
   - Company API key integration
   - Brevo system configuration
   - Reply-to handling with domain extraction

3. Batch Email Sending:
   - Individual email processing
   - Success/failure tracking
   - Error aggregation and reporting
```

**Response:**
- **Success (200):** `true` boolean when all emails sent successfully
- **Validation Error (422):** SendTestEmail form object with validation errors
- **Partial Failure (422):** SendTestEmail form object with failed email details
- **Template Error (422):** SendTestEmail form object with template rendering errors

---

## Form Classes

### 1. PreviewEmail

**File:** `src/apps/api/versions/v1/forms/email/PreviewEmail.php`  
**Extends:** `customs\Model` (85 lines)

**Public Properties:**
```php
public mixed $html_template = null;        // HTML email template (required)
public mixed $customer_uuid = null;       // Customer UUID for data substitution
```

**Private Properties:**
```php
private ?CompanyCustomersAdvanced $_customerModel; // Cached customer model
```

**Key Methods:**
- `preview()` - Generate HTML preview with customer data substitution
- `getNodular()` - Get Twig template engine component
- `getCustomerModel()` - Lazy load customer model with UUID lookup

**Template Processing:**
- MD5 hash generation for template identification
- Twig template engine integration with NodularComponent
- Customer data loading from MongoDB through MongoHelper
- Exception handling with user-friendly error messages

**Dependencies:**
- NodularComponent for template engine management
- TwigEngine for HTML template processing
- MongoHelper for customer data retrieval
- CompanyCustomersAdvanced for customer model access

---

### 2. SendTestEmail

**File:** `src/apps/api/versions/v1/forms/email/SendTestEmail.php`  
**Extends:** `customs\Model` (157 lines)

**Public Properties:**
```php
public mixed $html_template = null;        // HTML email template (required)
public mixed $subject = null;              // Email subject template (required)
public mixed $sender_email = null;         // Sender email address (required)
public mixed $sender_name = null;          // Sender display name (required)
public mixed $brevo_api_key = null;        // Brevo API key (required)
public mixed $pre_head = null;             // Email preheader text
public mixed $reply_to_email = null;       // Reply-to email address
public mixed $customer_uuid = null;        // Customer UUID for data substitution
public mixed $emails = null;               // Array of recipient emails (required)
```

**Private Properties:**
```php
private ?CompanyCustomersAdvanced $_customerModel; // Cached customer model
```

**Key Methods:**
- `send()` - Process and send test emails with full error handling
- `getNodular()` - Get Twig template engine component
- `getCustomerModel()` - Lazy load customer model with UUID lookup

**Email Processing Features:**
- Multi-template rendering (HTML, subject, preheader)
- Batch email processing with individual tracking
- Brevo API integration through EmailHelper
- Success/failure reporting with detailed error messages
- Reply-to domain extraction and validation

**Integration Dependencies:**
- EmailHelper for Brevo API communication
- CompanyEmailsAdvanced for system configuration
- NodularComponent and TwigEngine for template processing
- MongoHelper for customer data retrieval

---

## Business Logic

### 1. Email Template Preview Flow
```
1. Validate HTML template format
2. Validate customer UUID if provided
3. Load customer data from MongoDB
4. Generate unique template identifier (MD5)
5. Render template using Twig engine
6. Apply customer variable substitution
7. Return rendered HTML or error message
8. Handle template syntax errors gracefully
```

### 2. Test Email Sending Flow
```
1. Validate all required email parameters
2. Validate recipient email addresses format
3. Load customer data if UUID provided
4. Render all template components:
   - HTML body with customer data
   - Subject line with variables
   - Preheader text with substitution
5. Generate email payloads for each recipient
6. Send emails through Brevo API integration
7. Track successful and failed deliveries
8. Return success status or detailed errors
9. Provide batch processing results
```

### 3. Template Engine Integration
```
1. NodularComponent manages Twig template engine
2. Template identification using MD5 hashes
3. Dynamic compilation and caching
4. Customer data integration from MongoDB
5. Variable substitution with error handling
6. Multiple template types (HTML, subject, preheader)
```

---

## Security Features

### 1. Input Validation
- Required field validation for critical email parameters
- Email format validation for all email addresses
- UUID validation with customer existence checks
- String length and format validation
- API key validation for Brevo integration

### 2. Template Security
- Safe template rendering with exception handling
- Template compilation isolation
- Customer data access validation
- Error message sanitization

### 3. API Integration Security
- Brevo API key validation
- Company-based access control
- Email payload sanitization
- Reply-to domain validation

---

## Integration Points

### Dependencies
- `NodularComponent` - Template engine management
- `TwigEngine` - HTML template processing
- `EmailHelper` - Brevo API integration
- `MongoHelper` - Customer data retrieval
- `CompanyCustomersAdvanced` - Customer model access
- `CompanyEmailsAdvanced` - Email system configuration

### External Services
- **Brevo API** - Email delivery service
- **MongoDB** - Customer data storage
- **Twig Template Engine** - Template processing

### Related Components
- **Customer Management** - Customer data integration
- **Company Management** - API key and configuration
- **Template System** - Email template management
- **Notification System** - Email delivery tracking

---

## Usage Examples

### Preview email template
```http
POST /api/v1/email/preview
Content-Type: application/json
company-key: your-company-api-key

{
  "html_template": "<h1>Hello {{customer.name}}!</h1><p>Your order #{{customer.order_id}} is ready.</p>",
  "customer_uuid": "550e8400-e29b-41d4-a716-446655440000"
}
```

### Send test email
```http
POST /api/v1/email/send-test
Content-Type: application/json
company-key: your-company-api-key

{
  "html_template": "<h1>Hello {{customer.name}}!</h1><p>Thank you for your order!</p>",
  "subject": "Order Confirmation for {{customer.name}}",
  "sender_email": "orders@company.com",
  "sender_name": "Company Orders",
  "brevo_api_key": "xkeysib-abc123...",
  "pre_head": "Your order has been confirmed",
  "reply_to_email": "support@company.com",
  "customer_uuid": "550e8400-e29b-41d4-a716-446655440000",
  "emails": ["test1@example.com", "test2@example.com"]
}
```

---

## Error Handling

### Common Errors
- **"html_template cannot be blank"** - Missing required template
- **"Invalid customer UUID"** - Customer not found or invalid format
- **"Cannot render email template"** - Twig syntax errors
- **"The following emails could not be sent"** - Brevo API delivery failures

### Template Processing Errors
- Twig syntax validation with detailed error messages
- Template compilation error handling
- Customer data substitution failures
- Variable reference validation

### Email Delivery Errors
- Brevo API authentication failures
- Invalid email address handling
- Batch processing error reporting
- Network connectivity issues

### HTTP Status Codes
- **200 OK** - Successful operations (preview/send)
- **422 Unprocessable Entity** - Validation errors or processing failures

---

*EmailController API provides comprehensive email template management with advanced Twig template processing, customer data integration, and reliable Brevo email delivery service integration.*