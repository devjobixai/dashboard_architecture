# Jobix Dashboard — Routing Documentation

## Project Overview

This project uses the Yii framework with a modular architecture consisting of:
- Admin Application — administration web UI
- API Application — REST API (version v1)

## Routing Structure

### Admin Application Routes

#### AuthController (`/auth`)

| Method | Endpoint | Input | Output | Description |
|--------|----------|-------|--------|-------------|
| GET/POST | `/auth/login` | POST: LoginForm (username, password) | HTML view or redirect | User authentication |
| GET | `/auth/logout` | — | Redirect to `/auth/login` | User logout |

Login input:
- `username` (string)
- `password` (string)
- Uses `LoginForm` for validation

#### CompaniesController (`/companies`)

| Method | Endpoint | Input | Output | Description |
|--------|----------|-------|--------|-------------|
| GET | `/companies/index` | Query params (search) | HTML view with DataProvider | Companies list |
| AJAX GET | `/companies/active-list` | Query params + `filter[status]` | JSON with active companies | AJAX list of active companies |
| GET/POST | `/companies/create` | POST: CreateCompanyForm | HTML view or redirect | Create a company |
| GET/POST | `/companies/edit/{uuid}` | POST: EditCompanyForm | HTML view or redirect | Edit a company |
| POST | `/companies/delete/{uuid}` | DeleteCompanyForm | Redirect to index | Delete a company |
| GET/AJAX | `/companies/activate/{uuid}` | `company_uuid` | JSON or redirect | Activate a company |

Notes:
- Form classes: `CreateCompanyForm`, `EditCompanyForm`
- `uuid` URL param for company-specific actions

#### ActionsController (`/actions`)
- Defined; requires implementation analysis

#### AgentsController (`/agents`)

| Method | Endpoint | Input | Output | Description |
|--------|----------|-------|--------|-------------|
| GET | `/agents/index` | Query params (search) | HTML view with DataProvider | AI agents list |
| GET/POST/AJAX | `/agents/create` | SaveAgentForm + avatar file | HTML view or redirect | Create agent |
| GET/POST/AJAX | `/agents/profile` | SaveAgentForm (scenarios) | HTML view or redirect | Edit agent profile |
| GET | `/agents/copy` | Agent UUID | Redirect with notification | Copy agent |
| GET | `/agents/delete` | Agent UUID | Redirect to index | Delete agent |
| POST | `/agents/upload-knowledge-files` | File + Agent UUID | Text or error | Upload knowledge files |
| DELETE | `/agents/delete-knowledge-file` | Agent UUID + file UUID | JSON | Delete knowledge file |
| POST | `/agents/save-call-widget-settings` | SaveCallWidgetForm | Redirect to profile | Call-widget settings |
| POST | `/agents/save-chat-widget-settings` | SaveChatWidgetForm | Redirect to profile | Chat-widget settings |
| POST | `/agents/create-phone-number` | CreatePhoneNumberForm | JSON array | Create phone number |
| POST | `/agents/delete-phone-number` | DeletePhoneNumberForm | JSON array | Delete phone number |
| POST | `/agents/run-test` | CreateTestForm | JSON array | Test agent |
| GET | `/agents/numbers-list` | — | JSON array | Available numbers |

Highlights:
- Scenario-based editing (GENERAL, DETAILS, PROMPTS, KNOWLEDGE_TEXT, etc.)
- File uploads for avatar and knowledge files
- AJAX validation and JSON endpoints
- Special node layout support
- Widget management for call/chat

#### BillingController (`/billing`)

| Method | Endpoint | Input | Output | Description |
|--------|----------|-------|--------|-------------|
| GET | `/billing/index` | BillingCompanyDashboard (uuid, init) | HTML billing dashboard | Company billing statistics and charts |
| GET/POST | `/billing/billing-preference` | BillingDetailsForm (8 required fields) | HTML view or redirect | Configure billing details |
| GET | `/billing/bills` | — | HTML view | User bills list |
| GET | `/billing/bill/{uuid}` | Bill UUID | HTML view | Single bill view |

Highlights:
- Company selection with fallback logic
- BillingComponent integration for stats/graphs
- Comprehensive validation with HTML purification
- Auto-loading of existing billing details
- Multi-layer input sanitization (trim, strip_tags, HtmlPurifier)
- JSON storage in `additional_info`

#### CallsController (`/calls`)

| Method | Endpoint | Input | Output | Description |
|--------|----------|-------|--------|-------------|
| GET | `/calls/index` | CallsSearch query params | HTML view with DataProvider | Calls list with search/filters |
| GET | `/calls/details/{uuid}` | CallModelDetails (uuid) | HTML view or redirect | Call details with recordings |
| GET AJAX/PJAX | `/calls/sales/{uuid}` | CallSalesSearch (call_sid_id, status) | HTML partial | Sales related to the call |
| GET AJAX | `/calls/transcription/{uuid}` | TranscriptionSearch (uuid + GET params) | JSON with content/pagination | Call transcription with pagination |

Highlights:
- Multi-provider integration (Twilio, Jobix, Vonage, Telico, IDT, Didlogic)
- Recording management with auto download and caching
- AJAX-only endpoints (sales/transcription)
- BaseSearch integration with UI widgets
- Provider-specific recording retrieval and file management
- Complex search with dropdown helpers and advanced filtering

#### CompanyPhoneNumbersController (`/company-phone-numbers`)
- Defined; requires implementation analysis

#### CompanyRolesController (`/company-roles`)
- Defined; requires implementation analysis

#### CompanyUsersController (`/company-users`)
- Defined; requires implementation analysis

#### CustomerCategoriesController (`/customer-categories`)

| Method | Endpoint | Input | Output | Description |
|--------|----------|-------|--------|-------------|
| GET | `/customer-categories/index` | SearchCustomerCategoryForm | HTML view with DataProvider | Customer categories list |
| GET/POST | `/customer-categories/fields/{uuid}` | SaveCustomerCategoryForm (FIELDS) | HTML view or redirect | Manage category fields |
| GET/POST/AJAX | `/customer-categories/create` | SaveCustomerCategoryForm (CREATE) | HTML view or redirect | Create category |
| GET/POST/AJAX | `/customer-categories/update/{uuid}` | SaveCustomerCategoryForm (UPDATE) | HTML view with notification | Edit category |
| POST | `/customer-categories/delete/{uuid}` | DeleteForm widget | JSON | Delete category |
| POST | `/customer-categories/copy/{uuid}` | CopyForm widget | JSON | Copy category |

Highlights:
- Scenario-based editing (CREATE, UPDATE, FIELDS)
- Widgets for delete/copy
- Custom fields management
- Company-based access control with UUID validation

#### CustomerListsController (`/customer-lists`)

| Method | Endpoint | Input | Output | Description |
|--------|----------|-------|--------|-------------|
| GET | `/customer-lists/index` | CustomerListSearch | HTML view with DataProvider | Customer lists |
| GET/POST | `/customer-lists/create` | CreateCustomerListForm | HTML view or redirect | Create list |
| GET/POST | `/customer-lists/edit/{uuid}` | EditCustomerListForm | HTML view or redirect | Edit list |
| GET/POST/PJAX | `/customer-lists/filters/{uuid}` | FilterCustomerListForm (JSON filters) | HTML view with MongoDB integration | Manage list filters |
| GET | `/customer-lists/customers/{uuid}` | SearchCustomersListForm | HTML view with DataProvider | Customers within the list |
| POST | `/customer-lists/delete/{uuid}` | DeleteCustomerListForm | Redirect | Delete list |

Highlights:
- MongoDB integration for complex filtering
- JSON-based filter conditions with QueryBuilder validation
- PJAX support for dynamic filter UI

#### FlowsController (Admin) (`/flows`)

| Method | Endpoint | Input | Output | Description |
|--------|----------|-------|--------|-------------|
| GET | `/flows/index` | SearchFlowForm | HTML view with DataProvider | Flows list with search |
| GET/POST/AJAX | `/flows/create` | SaveFlowForm (CREATE) | HTML view, JSON, or redirect | Create flow |
| GET/POST/AJAX | `/flows/update` | SaveFlowForm (UPDATE) | HTML view or JSON | Edit flow |
| GET | `/flows/builder` | Flow UUID (query) | HTML view | Visual flow builder |
| POST | `/flows/delete` | DeleteForm widget | JSON | Soft delete flow |
| POST | `/flows/copy` | CopyForm widget | JSON | Copy flow |

Highlights:
- AJAX validation and JSON endpoints (copy/delete)
- Visual flow builder (drag-and-drop)
- Search/filtering with pagination (25 items)
- Transaction-safe operations with rollback
- Company-based access control with UUID validation

#### IndexController (`/`)

| Method | Endpoint | Input | Output | Description |
|--------|----------|-------|--------|-------------|
| GET | `/` or `/index/index` | — | Redirect to `/dashboard/index` | Main application entry point |
| GET | `/index/error` | Exception (error handler) | HTML error view (guest layout) | Application error handler |

Highlights:
- Simple root redirect to dashboard
- Error handling for various exception types
- ForbiddenHttpException → "STOP! No Access!"
- Other exceptions → "Page not found"
- Guest layout for clean error display

#### ManagementController (`/management`)

| Method | Endpoint | Input | Output | Description |
|--------|----------|-------|--------|-------------|
| GET | `/management/customer-fields` | CustomerFieldSearchForm | HTML view with list + form | Manage customer field definitions |
| POST AJAX | `/management/customer-fields` | UpdateOrCreateCustomerFieldForm (fields array) | JSON success/error | Create/Update customer fields |

Highlights:
- Batch processing of multiple fields
- Create/Update logic based on UUID presence
- FieldsValidator with company context validation
- Transaction safety for data consistency
- AJAX-only operations for field management

#### QueuesController (`/queues`)

| Method | Endpoint | Input | Output | Description |
|--------|----------|-------|--------|-------------|
| GET | `/queues/index` | QueuesSearch | HTML view with DataProvider | Queues list |
| POST AJAX | `/queues/action` | ActionQueueForm | JSON array | Control queue status |
| GET/POST | `/queues/create` | CreateQueueForm | HTML view or redirect | Create queue |
| GET/POST | `/queues/edit/{uuid}` | EditQueueForm | HTML view or redirect | Edit queue |
| POST | `/queues/delete/{uuid}` | DeleteQueueForm | Redirect to index | Delete queue |
| GET | `/queues/calls/{uuid}` | QueueCallSearch | HTML view with DataProvider | Queue calls |
| GET | `/queues/customers/{uuid}` | QueueCustomersSearch | HTML view with DataProvider | Queue customers |
| GET | `/queues/real-time/{uuid}` | — | HTML view with queue model | Real-time monitoring |
| GET | `/queues/customers-download/{uuid}` | QueueCustomersDownloadForm (type) | File download or redirect | Export customer data |
| GET | `/queues/inbound-call` | InboundCallSearch | HTML view with DataProvider | Inbound calls list |
| GET/POST | `/queues/inbound-call-create` | CreateInboundCallForm | HTML view or redirect | Create inbound call |
| GET/POST | `/queues/inbound-call-edit/{uuid}` | EditInboundCallForm | HTML view or redirect | Edit inbound call |
| POST | `/queues/inbound-call-delete/{uuid}` | DeleteInboundCallForm | Redirect with flash | Delete inbound call |
| GET | `/queues/inbound-call-statistics/{uuid}` | InboundCallStatisticsSearch | HTML view with DataProvider | Inbound calls statistics |

Highlights:
- Comprehensive queue management with monitoring
- Real-time statistics and customer management
- File export for queue data
- Inbound call management with statistics
- Company validation with automatic redirects
- Flash messaging for feedback
- AJAX operations for queue control

#### UsersController (`/users`)
- Defined but not implemented

---

### API Application Routes (v1)

Base URL: `/api/v1`  
Authentication: Header `company-key` (company identification)

#### CustomerController (`/api/v1/customer`)

| Method | Endpoint | Input | Output | Description |
|--------|----------|-------|--------|-------------|
| POST | `/customer/save` | SaveForm bodyParams + optional `?useQueue=true` | SaveForm object or array with `queueNumber` | Save customer |
| POST | `/customer/delete` | DeleteForm bodyParams | DeleteForm or bool | Delete customer |
| POST | `/customer/add-to-list` | AddToListForm bodyParams | AddToListForm | Add customer to list |
| POST | `/customer/count` | CountForm bodyParams | CountForm or array | Count customers |
| POST | `/customer/regenerate-list` | RegenerateListForm bodyParams | RegenerateListForm or bool | Regenerate a list |
| POST | `/customer/regenerate-list-progress` | RegenerateListProgressForm bodyParams | RegenerateListProgressForm, int or null | Regeneration progress |
| GET | `/customer/search` | Query params + Header `company-key` | ActiveDataProvider or CustomersListForm | Search customers |

Headers:
- `company-key` (string) — API key for authentication

Search query params:
- Sent via queryParams

#### CallController (`/api/v1/call`)

| Method | Endpoint | Input | Output | Description |
|--------|----------|-------|--------|-------------|
| POST | `/call/transcription` | TranscriptionForm bodyParams | array | Process call transcription |
| POST | `/call/init` | InitCallForm bodyParams | array | Initialize a call |
| POST | `/call/real-call-init` | RealCallInitForm bodyParams | array | Initialize a real call |
| POST | `/call/internet-call-webhook` | InternetCallWebhookForm bodyParams | CompanyAgentCallActivitiesAdvanced or InternetCallWebhookForm | Webhook for internet calls |

#### ActionController (`/api/v1/action`)
- **[⚡ ActionController API](controllers-docs/action-controller-api.md)** - Company actions automation API

#### EmailController (`/api/v1/email`)
- **[📧 EmailController API](controllers-docs/email-controller-api.md)** - Email template rendering and delivery API

#### FieldsController (`/api/v1/fields`)
- **[🏷️ FieldsController API](controllers-docs/fields-controller-api.md)** - Customer field management and query building API

#### FlowsController (`/api/v1/flows`)

| Method | Endpoint | Input | Output | Description |
|--------|----------|-------|--------|-------------|
| POST | `/flows/statistic` | StatisticForm bodyParams | Array with detailed statistics | Generate flow statistics |
| GET | `/flows/nodes` | Flow UUID (query param) | Array of active nodes | Flow nodes list |
| POST | `/flows/activate` | Flow UUID (query param) | Boolean (true/false) | Activate/deactivate flow |
| POST | `/flows/update-name` | UUID + name (query + body) | Boolean or form with errors | Update flow name |
| POST | `/flows/update-description` | UUID + description (query + body) | Boolean or form with errors | Update flow description |
| GET | `/flows/versions` | Flow UUID (query param) | ActiveDataProvider with versions | Flow versions list |

Highlights:
- Complex statistics with nodular system integration
- MongoDB integration for stats and caching
- UUID validation with company-based access control
- Activation toggle protected from deleted flows
- Flow versioning
- ListNode integration for node management

#### NodesController (`/api/v1/nodes`)
- **[🔗 NodesController API](controllers-docs/nodes-controller-api.md)** - Flow builder node management and execution API

#### QueueController (`/api/v1/queue`)
- **[🔄 QueueController API](controllers-docs/queue-controller-api.md)** - Queue management and webhook processing API

#### ResourcesController (`/api/v1/resources`)
- **[🛠️ ResourcesController API](controllers-docs/resources-controller-api.md)** - Resource management and testing utilities API

#### SaleController (`/api/v1/sale`)
- **[💰 SaleController API](controllers-docs/sale-controller-api.md)** - Purchase event processing API

#### SmsController (`/api/v1/sms`)
- **[💬 SmsController API](controllers-docs/sms-controller-api.md)** - SMS messaging and delivery API

#### TestController (`/api/v1/test`)
- **[🔍 TestController API](controllers-docs/test-controller-api.md)** - API connectivity and health check API

---

## Common Patterns

### Admin Controllers
- Base class: `WebController`
- Responses: HTML views and redirects
- AJAX requests return JSON
- Form classes handle validation and data processing
- UUID URL params identify resources

### API Controllers
- Base class: `ApiController`
- Responses: JSON
- Use `bodyParams` for POST payloads
- Use `queryParams` for GET
- Header `company-key` for authentication and data isolation

### Validation and Security
- UUID validation for all entity identifiers
- Company-based access control across admin and API
- Input sanitization (trim, strip_tags, HtmlPurifier where applicable)
- Transaction safety with rollback on errors
- Error handling with user-friendly messages and logging

---

If you add or change routes, please update this document accordingly and ensure the corresponding form/controller documentation under `controllers-docs/` stays in sync.

