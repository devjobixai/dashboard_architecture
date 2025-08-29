# BillingController Documentation

## Overview

BillingController manages the billing system in the Jobix Dashboard admin panel, including billing statistics, billing details configuration, bills list, and individual bill view.

Base path: `/billing`  
Controller type: WebController (Admin Application)  
Namespace: `apps\\admin\\controllers\\BillingController`

---

## Endpoints

### 1. Index — Billing Dashboard

| Param | Value |
|------|-------|
| URL | `/billing/index?uuid={company_uuid}` |
| Method | GET |
| Description | Billing homepage with statistics and charts for the selected company |

URL Parameters:
- `uuid` (string, optional) — company UUID to display billing

Input (BillingCompanyDashboard):

| Field | Type | Required | Validation | Description |
|------|------|----------|------------|-------------|
| `uuid` | string | ✗ | string, HtmlPurifier, trim | Company UUID |
| `init` | string | ✓ | required, string, custom validation | Initialization parameter |

BillingCompanyDashboard validation:
```php
[['init'], 'required']
[['uuid', 'init'], 'string']
[['uuid'], 'filter', 'filter' => [HtmlPurifier::class, 'process']]
[['uuid'], 'trim']
[['init'], 'initCompany'] // Custom company initialization
```

Business logic:
1. Auto-load params (uuid, init = 'init')
2. Initialize company via `initCompany()`
3. Fallback company selection:
   - If UUID provided: find a specific user company
   - If UUID missing or company not found: use the user's first company
   - If no companies: validation error

Company Selection Logic:
```php
if (!empty($uuid)) {
    $company = CompaniesAdvanced::getCustomerCompany(userId, uuid);
}
if ($company === null) {
    $company = CompaniesAdvanced::getCustomerFirstCompany(userId);
}
```

Statistics and data:
- `billingCounts` — current month stats via BillingComponent
- `billingGraph` — current month billing graph
- `company` — selected company
- `companies` — all user's companies list

Responses:
- Success: HTML view with the billing dashboard
- No Company: Redirect to `/dashboard/index`
- Data: `billingCounts`, `billingGraph`, `company`, `companies`

---

### 2. Billing Preference — Billing Details

| Param | Value |
|------|-------|
| URL | `/billing/billing-preference` |
| Methods | GET, POST |
| Description | Manage user billing details |

#### GET `/billing/billing-preference`
Purpose: Render billing details form

#### POST `/billing/billing-preference`
Purpose: Save billing details

Input (BillingDetailsForm bodyParams):

| Field | Type | Required | Validation | Description |
|------|------|----------|------------|-------------|
| `first_name` | string | ✓ | required, string, trim, strip_tags, HtmlPurifier | Card holder first name |
| `last_name` | string | ✓ | required, string, trim, strip_tags, HtmlPurifier | Card holder last name |
| `company_name` | string | ✓ | required, string, trim, strip_tags, HtmlPurifier | Company name |
| `address` | string | ✓ | required, string, trim, strip_tags, HtmlPurifier | Address |
| `city` | string | ✓ | required, string, trim, strip_tags, HtmlPurifier | City |
| `post_code` | string | ✓ | required, string, trim, strip_tags, HtmlPurifier | Post code |
| `country` | string | ✓ | required, string, trim, strip_tags, HtmlPurifier | Country |
| `vat_id` | string | ✓ | required, string, trim, strip_tags, HtmlPurifier | VAT ID |

BillingDetailsForm validation:
```php
// All fields are required
[['first_name', 'last_name', 'company_name', 'address', 'city', 'post_code', 'country', 'vat_id'], 'required']

// String validation
[['first_name', 'last_name', 'company_name', 'address', 'city', 'post_code', 'country', 'vat_id'], 'string']

// Sanitization
[['first_name', 'last_name', 'company_name', 'address', 'city', 'post_code', 'country', 'vat_id'], 'trim']
[['first_name', 'last_name', 'company_name', 'address', 'city', 'post_code', 'country', 'vat_id'], 'filter', 'filter' => 'strip_tags']
[['first_name', 'last_name', 'company_name', 'address', 'city', 'post_code', 'country', 'vat_id'], 'filter', 'filter' => [HtmlPurifier::class, 'process']]
```

Save business logic:
1. Find existing user billing details
2. If not found — create a new record
3. Save into `additional_info` of BillingDetailsAdvanced
4. Auto-load existing data when rendering the form

Data Loading Logic:
```php
$billingDetails = BillingDetailsAdvanced::findBillingDetailsByUser(userId);
if ($billingDetails === null) {
    $billingDetails = new BillingDetailsAdvanced();
    $billingDetails->user_id = userId;
}
// Auto-load existing data if record exists
if (!$billingDetails->isNewRecord) {
    $form->load((array)$billingDetails->additional_info);
}
```

Responses:
- Success: Redirect to `/billing/billing-preference` with success notification
- Validation Error: HTML view with form and errors
- GET: HTML view with form (pre-filled if data exists)

---

### 3. Bills — Bills List

| Param | Value |
|------|-------|
| URL | `/billing/bills` |
| Method | GET |
| Description | Display list of user's bills |

Input: None

Business logic:
- Simple view render without extra logic
- Data expected to be loaded in the view layer

Response:
- Success: HTML view with bills list

---

### 4. Bill — Individual Bill View

| Param | Value |
|------|-------|
| URL | `/billing/bill/{uuid}` |
| Method | GET |
| Description | Detailed view of a single bill |

URL Parameters:
- `uuid` (string) — bill UUID

Input: UUID via URL

Business logic:
- Simple view render with the given UUID
- Bill loading logic is expected in the view layer

Response:
- Success: HTML view with bill details

---

## Form Classes

### 1. BillingCompanyDashboard

File: `src/apps/admin/forms/billing/BillingCompanyDashboard.php`  
Extends: `customs\\Model`

Public properties:
```php
public mixed $uuid = null;           // Company UUID
public mixed $init = null;           // Initialization parameter
```

Private properties:
```php
protected mixed $_company = null;    // Selected company
protected mixed $_companies = null;  // User companies list
```

Key methods:
- `rules()` — validation rules
- `initCompany()` — initialization and company selection logic
- `process()` — processing and billing data generation

Company Selection Features:
- Automatic company selection by UUID
- Fallback to user's first company
- Multiple company support
- Error handling for missing companies

Business Logic:
- BillingComponent integration for statistics
- Current month billing stats generation
- Billing graph generation
- Company access validation

---

### 2. BillingDetailsForm

File: `src/apps/admin/forms/billing/BillingDetailsForm.php`  
Extends: `customs\\Model`

Public properties:
```php
public mixed $first_name = null;     // Card holder first name
public mixed $last_name = null;      // Card holder last name
public mixed $company_name = null;   // Company name
public mixed $address = null;        // Address
public mixed $city = null;           // City
public mixed $post_code = null;      // Post code
public mixed $country = null;        // Country
public mixed $vat_id = null;         // VAT ID
```

Private properties:
```php
protected mixed $_billingDetails = null; // BillingDetailsAdvanced instance
```

Attribute Labels:
```php
'first_name' => 'Card Holder\'s First Name'
'last_name' => 'Card Holder\'s Last Name'
'company_name' => 'Company Name'
'address' => 'Address'
'city' => 'City'
'post_code' => 'Post Code'
'country' => 'Country'
'vat_id' => 'VAT ID'
```

Key methods:
- `__construct()` — auto-load existing data
- `rules()` — comprehensive validation with sanitization
- `attributeLabels()` — UI labels
- `save()` — persist to additional_info

Security Features:
- HTML purification via HtmlPurifier
- Strip tags sanitization
- Trim whitespace
- Exception handling with error propagation

Data Persistence:
- Auto-load existing billing details
- Store in additional_info (JSON)
- User-specific billing details
- Create new record if not exists

---

## Business Logic

### 1. Billing Dashboard Flow
```
1. Load dashboard with optional company UUID
2. Initialize company selection (specific or first available)
3. Generate billing statistics for current month
4. Generate billing graph data
5. Load user's companies list
6. Render dashboard with all data
```

### 2. Billing Details Management Flow
```
1. Load existing billing details for user
2. Auto-populate form if data exists
3. Display form with current values
4. On submit: validate all required fields
5. Sanitize and purify all inputs
6. Save to additional_info field
7. Redirect with success notification
```

### 3. Company Selection Logic
```
1. IF UUID provided → find specific company for user
2. IF not found or no UUID → use user's first company
3. IF no companies → show error and redirect
4. Load all user companies for selection
```

---

## Security Features

### 1. Input Sanitization
- HTML purification for all text fields
- Strip tags to remove unsafe content
- Trim whitespace for clean data
- Multiple layers of filtering

### 2. User-based Access Control
- Billing details isolated by user_id
- Company access validated via user ownership
- No cross-user data access

### 3. Data Validation
- Required validation for all billing fields
- String type validation
- HTML purifier for XSS protection

### 4. Error Handling
- Exception handling with user-friendly messages
- Validation error propagation
- Graceful fallbacks for missing data

---

## Integration Points

### Dependencies
- `BillingComponent` — billing statistics and graphs
- `BillingDetailsAdvanced` — billing details model
- `CompaniesAdvanced` — user companies management
- `NotificationHelper` — user notifications
- `HtmlPurifier` — input sanitization

### Related Models
- `BillingDetails` (basic) — base model
- User model — for user_id associations
- Companies — for company selection and validation

### External Integrations
- Billing statistics system
- Company management system
- Notification system
- HTML purification system

---

## Usage Examples

### View billing dashboard
```http
GET /billing/index?uuid=550e8400-e29b-41d4-a716-446655440000
```

Response: HTML view with company billing statistics

### Update billing details
```http
POST /billing/billing-preference
Content-Type: application/x-www-form-urlencoded

first_name=John&last_name=Doe&company_name=Test Company&address=123 Main St&city=Kyiv&post_code=01001&country=Ukraine&vat_id=UA123456789
```

Response: Redirect to `/billing/billing-preference` with success notification

### View bills list
```http
GET /billing/bills
```

Response: HTML view with bills list

### View a bill
```http
GET /billing/bill/550e8400-e29b-41d4-a716-446655440000
```

Response: HTML view with bill details

---

## Performance Considerations

### Dashboard Optimization
- Billing statistics can be resource-intensive
- Current month calculations on large datasets
- Graph generation may benefit from caching

### Data Loading
- Auto-loading billing details on each request
- Company lists can be large for some users
- JSON operations on additional_info fields

---

## Error Handling

### Common Errors
- "Company not found" — user has no companies
- Billing details validation errors — invalid fields
- Database save errors — persistence issues

### Redirect Logic
- Redirect to dashboard if no companies
- Redirect after successful save of billing details
- Graceful handling of missing data

---

BillingController provides comprehensive management of billing information with secure data persistence and detailed usage statistics.

