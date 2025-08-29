# Controllers Documentation Index

## Overview

This directory contains detailed documentation for all Jobix Dashboard controllers, including routing, forms, validation, and business logic.

## Documentation Structure

### 📁 Admin Controllers (WebController)

#### [AuthController](auth-controller.md)
- Path: `/auth`
- Purpose: User authentication for the admin panel
- Key endpoints:
  - `GET/POST /auth/login` - User login
  - `GET /auth/logout` - User logout
- Form classes: LoginForm
- Highlights: Token management, Cookie handling, Session management

#### [CompaniesController](companies-controller.md)
- Path: `/companies`
- Purpose: Manage companies in the system
- Key endpoints:
  - `GET /companies/index` - Companies list
  - `GET /companies/active-list` - AJAX list of active companies
  - `GET/POST /companies/create` - Create a new company
  - `GET/POST /companies/edit/{uuid}` - Edit a company
  - `POST /companies/delete/{uuid}` - Delete a company
  - `GET/AJAX /companies/activate/{uuid}` - Activate a company
- Form classes: CreateCompanyForm, EditCompanyForm, DeleteCompanyForm, ActivateCompanyForm
- Highlights: UUID-based routing, Permission checking, Slug generation, API key management

#### [DashboardController](dashboard-controller.md)
- Path: `/dashboard`
- Purpose: Main control panel after authentication
- Key endpoints:
  - `GET /dashboard/index` - Dashboard
- Form classes: None (simple controller)
- Highlights: Company selection validation, Automatic redirects, Entry point after authentication

#### [AgentsController](agents-controller.md)
- Path: `/agents`
- Purpose: Manage company AI agents
- Key endpoints:
  - `GET /agents/index` - AI agents list
  - `GET/POST/AJAX /agents/create` - Create a new agent
  - `GET/POST/AJAX /agents/profile` - Edit agent profile
  - `GET /agents/copy` - Copy an agent
  - `GET /agents/delete` - Delete an agent
  - `POST /agents/upload-knowledge-files` - Upload knowledge files
  - `DELETE /agents/delete-knowledge-file` - Delete a knowledge file
  - `POST /agents/save-*-widget-settings` - Configure widgets
  - `POST /agents/create-phone-number` - Manage phone numbers
  - `POST /agents/run-test` - Test an agent
- Form classes: SaveAgentForm (scenario-based), CopyAgentForm, AgentsSearch, SaveCallWidgetForm, SaveChatWidgetForm, CreatePhoneNumberForm, DeletePhoneNumberForm, CreateTestForm
- Highlights: Scenario-based editing, File upload support, AJAX validation, Knowledge management, Widget configuration, Phone number management, Node layout support

#### [FlowsController Admin](flows-controller-admin.md)
- Path: `/flows`
- Purpose: Manage automated workflows
- Key endpoints:
  - `GET /flows/index` - List workflows with search
  - `GET/POST/AJAX /flows/create` - Create a new flow
  - `GET/POST/AJAX /flows/update` - Edit a flow
  - `GET /flows/builder` - Visual flow builder
  - `POST /flows/delete` - Soft delete a flow
  - `POST /flows/copy` - Copy a flow
- Form classes: SaveFlowForm (scenario-based), SearchFlowForm, DeleteForm, CopyForm
- Highlights: Visual flow builder, Transaction-safe operations, Search and filtering, AJAX validation, Company-based access control, UUID validation

#### [CustomersController](customers-controller.md)
- Path: `/customers`
- Purpose: Manage company customers
- Key endpoints:
  - `GET /customers/index` - Customers list with search
  - `GET/POST/AJAX /customers/create` - Create a new customer
  - `GET/POST/AJAX /customers/edit` - Edit a customer + event history
  - `POST /customers/upload` - Bulk upload customers
- Form classes: SaveCustomerForm (scenario-based, 306 lines), CustomerSearchForm, CustomerEventsSearchForm, UploadCustomersForm
- Highlights: API integration, Custom field management, Bulk upload functionality, Customer events integration, Email uniqueness validation, ValueHelper integration

#### [BillingController](billing-controller.md)
- Path: `/billing`
- Purpose: Manage company billing
- Key endpoints:
  - `GET /billing/index` - Billing dashboard with statistics
  - `GET/POST /billing/billing-preference` - Configure billing details
  - `GET /billing/bills` - Bills list
  - `GET /billing/bill/{uuid}` - View an individual bill
- Form classes: BillingCompanyDashboard, BillingDetailsForm (8 required fields)
- Highlights: Company selection fallback logic, BillingComponent integration, Multi-layer input sanitization, HTML purification, Auto-loading billing data, JSON storage

#### [CallsController](calls-controller.md)
- Path: `/calls`
- Purpose: Manage calls and related data
- Key endpoints:
  - `GET /calls/index` - Calls list with search and filters
  - `GET /calls/details/{uuid}` - Call details with recordings
  - `GET AJAX/PJAX /calls/sales/{uuid}` - Sales related to a call
  - `GET AJAX /calls/transcription/{uuid}` - Call transcription
- Form classes: CallsSearch (332 lines), CallModelDetails (183 lines), CallSalesSearch, TranscriptionSearch (247 lines)
- Highlights: Multi-provider integration (Twilio, Jobix, etc.), Recording management with caching, AJAX endpoints, BaseSearch integration, Provider-specific recording retrieval, Advanced filtering

---

### 🚀 API Controllers v1 (ApiController)

#### [CustomerController](customer-controller-api.md)
- Path: `/api/v1/customer`
- Purpose: Manage customers via REST API
- Key endpoints:
  - `POST /customer/save` - Save customer (with queue support)
  - `POST /customer/delete` - Delete customer
  - `POST /customer/add-to-list` - Add customer to a list
  - `POST /customer/count` - Count customers
  - `POST /customer/regenerate-list` - Regenerate customers list
  - `POST /customer/regenerate-list-progress` - Regeneration progress
  - `GET /customer/search` - Search customers
- Form classes: SaveForm (806 lines), DeleteForm, AddToListForm, CountForm, RegenerateListForm, RegenerateListProgressForm, CustomersListForm
- Highlights: Complex validation structures, MongoDB integration, Queue processing, Event handling, Nodular components

#### [CallController](call-controller-api.md)
- Path: `/api/v1/call`
- Purpose: Manage calls and transcriptions
- Key endpoints:
  - `POST /call/transcription` - Process call transcription
  - `POST /call/init` - Initialize a call
  - `POST /call/real-call-init` - Initialize a real call
  - `POST /call/internet-call-webhook` - Webhook for internet calls
- Form classes: TranscriptionForm (347 lines), InitCallForm, RealCallInitForm, InternetCallWebhookForm
- Highlights: AI/ML integration, WebSocket real-time updates, Statistics tracking (LLM/STT/TTS), Partial form validation, Trigger processing

#### [FlowsController API](flows-controller-api.md)
- Path: `/api/v1/flows`
- Purpose: Manage automated workflows via API
- Key endpoints:
  - `POST /flows/statistic` - Generate detailed flow statistics
  - `GET /flows/nodes` - List active flow nodes
  - `POST /flows/activate` - Activate/deactivate a flow
  - `POST /flows/update-name` - Update flow name
  - `POST /flows/update-description` - Update flow description
  - `GET /flows/versions` - List flow versions
- Form classes: StatisticForm (717 lines), ActivateForm, UpdateNameForm, UpdateDescriptionForm, VersionsForm
- Highlights: Complex statistics generation, MongoDB integration, Nodular system integration, UUID validation, Flow versioning, Company-based access control

---

## Common Patterns and Highlights

### Admin Controllers
- Base Class: `WebController`
- Response Types: HTML views, redirects, JSON (for AJAX)
- Authentication: Yii User component with session management
- Data Handling: Form classes with Yii validation
- Routing: Traditional web routes with UUID parameters
- Security: User isolation, CSRF protection

### API Controllers
- Base Class: `ApiController`
- Response Types: JSON objects, arrays
- Authentication: `company-key` header or bodyParams
- Data Handling: Complex form validation with ArrayValidator
- Routing: RESTful API endpoints
- Security: API key validation, company-based isolation

### Validation Patterns

#### Admin Forms
```php
// Simple validation rules
[['field'], 'required']
[['field'], 'string', 'max' => 255]
[['field'], 'customValidation']
```

#### API Forms
```php
// Complex validation with ArrayValidator
[['field'], ArrayValidator::class, 'structure' => [
    'nested_field' => [['required'], ['string']],
    'values' => [[ArrayValidator::class]]
], 'maxSizeInBytes' => 102400]
```

### Business Logic

#### Transaction Management
- Admin: Simple operations with basic error handling
- API: Complex transactions with rollback and cleanup

#### Integration Points
- PostgreSQL - primary database
- MongoDB - customer data storage (API)
- Queue System - asynchronous processing (API)
- WebSocket - real-time communication (API)
- External Services - AI/ML, VoIP, Email

### Security Features

#### General
- Input validation and sanitization
- SQL injection protection
- Cross-company data isolation
- Error logging and monitoring

#### Admin-specific
- Session management
- CSRF protection
- User authentication
- Permission checking

#### API-specific
- API key validation
- Rate limiting readiness
- Payload size limits
- Real-time security

---

## How to Use the Docs

### 1. Frontend Developers
- Check the Endpoints sections for URLs and HTTP methods
- See Input and Responses for request/response formats
- Use Usage Examples for a quick start

### 2. Backend Developers
- Study Form Classes to understand business logic
- Check Business Logic sections for flow understanding
- See Integration Points for dependencies

### 3. QA Engineers
- Use Error Handling sections for test cases
- Check Security Features for security testing
- See Validation rules for edge cases

### 4. DevOps Engineers
- Check Dependencies for infrastructure needs
- Study Integration Points for service configuration
- See Logging information for monitoring setup

---

## Additional Resources

### Main Docs
- [ROUTING_DOCUMENTATION.md](../ROUTING_DOCUMENTATION.md) - Routing overview
- [README.md](../README.md) - Project setup instructions

### API Specification
- [OpenAPI YAML](../src/apps/admin/web/openapi.yaml) - API specification file

### Project Structure
```
src/
├── apps/
│   ├── admin/                 # Admin Application
│   │   ├── controllers/       # Web Controllers
│   │   └── forms/             # Form Classes
│   └── api/                   # API Application
│       └── versions/v1/       # API v1
│           ├── controllers/   # API Controllers
│           └── forms/         # API Form Classes
├── components/                # Shared Components
├── models/                    # Database Models
└── helpers/                   # Helper Classes
```

---

## Maintenance Notes

### Updating the Docs
When making changes to controllers or forms, please update the corresponding documentation:

1. Controller changes: Update the relevant controller MD file
2. Form changes: Update the Form Classes sections
3. New endpoints: Add to the endpoints list and usage examples
4. Breaking changes: Update compatibility notes

### Documentation Standards
- Use English for descriptions
- Include code examples for complex structures
- Document all validation rules
- Describe business logic flows
- Include error handling information

---

Documentation generated automatically based on analysis of project forms and controllers.

