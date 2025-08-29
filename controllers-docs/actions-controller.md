# ActionsController Documentation

## Overview

ActionsController manages company actions for integration and automation workflows in the Jobix Dashboard administrative panel. The controller provides comprehensive CRUD functionality for managing actions with support for platform integrations, HTTP method configurations, custom headers, JSON content management, and activity history tracking.

**Base Path:** `/actions`  
**Controller Type:** WebController (Admin Application)  
**Namespace:** `apps\admin\controllers\ActionsController`

---

## Endpoints

### 1. Index - Actions List

| Parameter | Value |
|-----------|-------|
| **URL** | `/actions/index` |
| **Method** | GET |
| **Description** | Display list of all company actions with search, filtering, and role-based access controls |

**Input Data (ActionsSearch queryParams):**
- Search parameters loaded from query string through BaseSearch
- Company-based filtering applied automatically

**Business Logic:**
1. Create ActionsSearch form extending BaseSearch
2. Load search parameters with company isolation
3. Execute search with JOIN to companies table
4. Apply role-based access control (ProfileRule, DeleteRule)
5. Display actions with entity status widgets
6. Provide edit and delete action buttons based on permissions

**Response:**
- **Success:** HTML view with actions list
- **Data:** `dataProvider`, `searchModel`

**Available Columns:**
- **Name:** Linked to general action page
- **Description:** Action description text
- **Status:** Entity status widget display
- **Updated At:** Formatted timestamp with timezone
- **Actions:** Profile/Delete buttons based on user permissions

---

### 2. Create - Create New Action

| Parameter | Value |
|-----------|-------|
| **URL** | `/actions/create` |
| **Methods** | GET, POST |
| **Description** | Create new company action |

#### GET `/actions/create`
**Response:** HTML view with empty creation form

#### POST `/actions/create`
**Input Data (SaveActionForm bodyParams):**

| Field | Type | Required | Validation | Description |
|-------|------|----------|-----------|-------------|
| `name` | string | ✓ | required, string, min:5, max:255, trim | Action name |
| `description` | string | ✗ | string, max:1024 | Action description |
| `find_prompt` | string | ✓ | required, string, max:10000 | AI prompt for action finding |
| `status` | boolean | ✗ | boolean | Action status |

**Detailed Validation SaveActionForm (CREATE scenario):**
```php
// CREATE scenario fields
[['name', 'description', 'status', 'find_prompt'], 'required']

// String validation with trimming
[['name', 'description'], 'filter', 'filter' => 'trim']
[['name'], 'string', 'min' => 5, 'max' => 255]
[['description'], 'string', 'max' => 1024]
[['find_prompt'], 'string', 'max' => 10000]

// Boolean validation
[['status'], 'boolean']
```

**Business Logic:**
1. Validate form with CREATE scenario
2. Create new CompanyActionsAdvanced record
3. Set company_id from selectedUserCompany
4. Generate UUID for new action
5. Update AI connection through _updateAiConnection()
6. Redirect to general page for further configuration

**Responses:**
- **Success:** Redirect to `/actions/general?uuid={action_uuid}` with success notification
- **Validation Error:** HTML view with form and errors using NotificationHelper
- **GET:** HTML view with empty form

---

### 3. General - Update General Information

| Parameter | Value |
|-----------|-------|
| **URL** | `/actions/general?uuid={action_uuid}` |
| **Methods** | GET, POST |
| **Description** | Update action general information (name, description, prompt, status) |

**URL Parameters:**
- `uuid` (string, required) - UUID of the action

#### GET `/actions/general`
**Purpose:** Display general information edit form

**Behavior:**
- Automatic data loading through `populate()`
- Load current action data

#### POST `/actions/general`
**Purpose:** Process general information update

**Input Data (SaveActionForm bodyParams):**

| Field | Type | Required | Validation | Description |
|-------|------|----------|-----------|-------------|
| `uuid` | string | ✓ | required, UUID, exists | Action UUID |
| `name` | string | ✓ | required, string, min:5, max:255, trim | Action name |
| `description` | string | ✗ | string, max:1024 | Action description |
| `find_prompt` | string | ✓ | required, string, max:10000 | AI prompt for action finding |
| `status` | boolean | ✗ | boolean | Action status |

**Detailed Validation SaveActionForm (UPDATE_GENERAL scenario):**
```php
// UPDATE_GENERAL scenario includes uuid
[['uuid', 'name', 'description', 'status', 'find_prompt'], 'required']

// UUID validation with company filtering
[['uuid'], UuidHelper::class]
[['uuid'], 'exist', 'targetClass' => CompanyActionsAdvanced::class,
 'filter' => function($query) {
     return $query->whereCompanyId($this->selectedUserCompany->id);
 }]
```

**Business Logic:**
1. Load existing action data on GET request
2. Validate form with UPDATE_GENERAL scenario
3. Update general information fields
4. Maintain AI connection consistency
5. Provide user feedback through notifications

**Responses:**
- **Success:** Redirect to `/actions/general?uuid={action_uuid}` with success notification
- **Error:** HTML view with form and error notifications
- **GET:** HTML view with populated form

---

### 4. Integration - Update Platform Integration Settings

| Parameter | Value |
|-----------|-------|
| **URL** | `/actions/integration?uuid={action_uuid}` |
| **Methods** | GET, POST |
| **Description** | Update action platform integration settings (HTTP method, URL, headers, content) |

**URL Parameters:**
- `uuid` (string, required) - UUID of the action

#### GET `/actions/integration`
**Purpose:** Display integration settings form

**Behavior:**
- Automatic data loading through `populate()`
- Load current integration configuration

#### POST `/actions/integration`
**Purpose:** Process integration settings update

**Input Data (SaveActionForm bodyParams):**

| Field | Type | Required | Validation | Description |
|-------|------|----------|-----------|-------------|
| `uuid` | string | ✓ | required, UUID, exists | Action UUID |
| `platform` | string | ✓ | required, in range of available platforms | Integration platform |
| `method` | string | ✓ | required, in range of HTTP methods | HTTP method |
| `url` | string | ✓ | required, URL with HTTPS scheme | API endpoint URL |
| `content_type` | string | ✓ | required, in range of content types | Request content type |
| `headers` | array | ✗ | custom validation | HTTP headers array |
| `content` | string | ✗ | JSON validation, max:1024 | Request body content |

**Detailed Validation SaveActionForm (UPDATE_UPDATE_INTEGRATION scenario):**
```php
// Integration-specific validation
[['platform'], 'in', 'range' => array_keys(CompanyActionsAdvanced::getPlatformsList())]
[['content_type'], 'in', 'range' => array_keys(CompanyActionsAdvanced::getContentType())]
[['method'], 'in', 'range' => array_keys(CompanyActionsAdvanced::getMethods())]
[['url'], 'url', 'validSchemes' => ['https'], 'defaultScheme' => 'https']
[['headers'], 'validateHeader'] // Custom header validation
[['content'], JsonValidator::class]
```

**Custom Header Validation:**
```php
public function validateHeader($attribute): void
{
    $headers = $this->$attribute;
    if (!empty($headers)) {
        foreach ($headers as $index => $header) {
            // Validate header format and content
            if (empty($header['key']) || empty($header['value'])) {
                $this->addError($attribute, "Header at index $index is invalid");
            }
        }
    }
}
```

**Business Logic:**
1. Load existing integration settings on GET request
2. Validate integration configuration
3. Update platform-specific settings
4. Validate HTTP method and URL scheme compatibility
5. Process custom headers and JSON content
6. Update AI connection integration

**Responses:**
- **Success:** Redirect to `/actions/integration?uuid={action_uuid}` with success notification
- **Error:** HTML view with form and detailed validation errors
- **GET:** HTML view with populated integration form

---

### 5. History - Action Activities History

| Parameter | Value |
|-----------|-------|
| **URL** | `/actions/history?uuid={action_uuid}` |
| **Method** | GET |
| **Description** | Display history of action activities and usage logs |

**URL Parameters:**
- `uuid` (string, required) - UUID of the action

**Input Data (ActionActivitiesSearch):**
- Action UUID context for filtering
- Company validation through user identity

**Business Logic:**
1. Validate company selection (redirect if not set)
2. Create ActionActivitiesSearch with action UUID context
3. Load action activities history
4. Display chronological activity log

**Response:**
- **Success:** HTML view with action activities history
- **Data:** `dataProvider`, `searchModel`, `actionUuid`
- **No Company:** Redirect to `/companies/index`

**Features:**
- Company context validation
- Action-specific activity filtering
- Chronological activity display
- Integration activity tracking

---

### 6. Delete - Delete Action

| Parameter | Value |
|-----------|-------|
| **URL** | `/actions/delete?uuid={action_uuid}` |
| **Method** | POST |
| **Description** | Delete company action |

**URL Parameters:**
- `uuid` (string, required) - UUID of action to delete

**Business Logic:**
1. Create SaveActionForm with DELETE scenario
2. Set action UUID
3. Execute delete operation
4. Provide user feedback through notifications
5. Redirect to actions index

**Delete Operation Flow:**
```php
$model = new SaveActionForm();
$model->uuid = $this->request->get('uuid');
$model->delete();

if ($model->hasErrors()) {
    NotificationHelper::errorDeleteModel($model, 'Company Action');
} else {
    NotificationHelper::successDelete('Company Action');
}
```

**Responses:**
- **Always:** Redirect to `/actions/index`
- **Success:** Success notification with NotificationHelper
- **Error:** Error notification with detailed messages

---

## Form Classes

### 1. SaveActionForm

**File:** `src/apps/admin/forms/action/SaveActionForm.php`  
**Extends:** `customs\Model` (346 lines)

**Scenarios:**
- `SCENARIO_CREATE` - creating new action
- `SCENARIO_DELETE` - deleting existing action
- `SCENARIO_UPDATE_GENERAL` - updating general information
- `SCENARIO_UPDATE_UPDATE_INTEGRATION` - updating integration settings

**Public Properties:**
```php
public mixed $uuid = null;                // Action UUID (required for update/delete)
public mixed $name = null;                // Action name (required)
public mixed $description = null;         // Action description
public mixed $find_prompt = null;         // AI prompt for action finding (required)
public mixed $status = null;              // Action status (boolean)
public mixed $platform = null;           // Integration platform
public mixed $method = null;              // HTTP method
public mixed $url = null;                 // API endpoint URL
public mixed $content_type = null;        // Request content type
public mixed $headers = [];               // HTTP headers array
public mixed $content = '{}';             // Request body content (JSON)
```

**Private Properties:**
```php
private null|CompanyActionsAdvanced $_companyActionModel = null; // Action model
```

**Scenario Fields:**
- **CREATE:** `['name', 'description', 'status', 'find_prompt']`
- **DELETE:** `['uuid']`
- **UPDATE_GENERAL:** `['uuid', 'name', 'description', 'status', 'find_prompt']`
- **UPDATE_UPDATE_INTEGRATION:** `['uuid', 'platform', 'method', 'url', 'content_type', 'headers', 'content']`

**Key Methods:**
- `populate()` - load existing action data
- `create()` - create new action with AI integration
- `updateGeneral()` - update general information
- `updateIntegration()` - update platform integration settings
- `delete()` - delete action
- `_updateAiConnection()` - maintain AI connection consistency
- `validateHeader()` - custom header validation
- `getCompanyActionModel()` - lazy load action model

**Validation Features:**
- Platform validation against available platforms list
- HTTP method validation against supported methods
- HTTPS-only URL validation with scheme enforcement
- JSON content validation
- Custom header structure validation
- Company-based access control through UUID filtering

**AI Integration:**
- ActionAIConnectorHelper integration
- AI connection updates on action changes
- Find prompt management for AI functionality

---

### 2. ActionsSearch

**File:** `src/apps/admin/forms/action/ActionsSearch.php`  
**Extends:** `BaseSearch` (196 lines)

**Public Properties:**
```php
public array $columns = [];               // Grid columns configuration
public int $status = 1;                   // Default status filter
```

**Search Features:**
- **Company Isolation:** Automatic filtering by selectedUserCompany through JOIN
- **Role-based Access:** ProfileRule and DeleteRule integration
- **Status Filtering:** Active/Inactive status filtering
- **Grid Columns:** Name, Description, Status, Updated At, Actions
- **Entity Status Widget:** Visual status representation
- **Time Zone Support:** Localized timestamp display

**Column Configuration:**
```php
// Name column with link to general page
'value' => function ($model) {
    return '<a href="' . Url::toRoute(['/actions/general', 'uuid' => $model['uuid']]) . '" 
            class="client-link" data-pjax="0">' . $model['name'] . '</a>';
}

// Status column with widget
'value' => function ($model) {
    return EntityStatusWidget::widget(['status' => $model['status']]);
}

// Action buttons with role-based display
'template' => (function(){
    $buttons = '';
    if(ProfileRule::availableForUser()) $buttons .= '{profile}';
    if(DeleteRule::availableForUser()) $buttons .= '{delete}';
    return '<div class="flex justify-center items-center">' . $buttons . '</div>';
})()
```

**Base Query:**
```php
protected function getBaseQuery(): Query
{
    return CompanyActionsAdvanced::find()
        ->select(['uuid', 'name', 'description', 'updated_at', 'status'])
        ->innerJoin(CompaniesAdvanced::tableName(), 
                   'companies.id = company_actions.company_id')
        ->where(['companies.uuid' => \Yii::$app->user->identity->getSelectedCompanyUuid()]);
}
```

**Filter Fields:**
- **Name:** Custom filtering with ILIKE support
- **Status:** Default filtering for active/inactive

**Sort Fields:**
- **Created At:** Ascending/Descending with custom labels

---

### 3. ActionActivitiesSearch

**File:** `src/apps/admin/forms/action/ActionActivitiesSearch.php`  
**Purpose:** Search and display action usage activities and history

**Key Features:**
- Action-specific activity filtering
- Company context validation
- Activity chronological display
- Integration activity tracking

---

## Business Logic

### 1. Action Creation Flow
```
1. Display creation form (GET)
2. Validate action data (name, description, find_prompt)
3. Generate UUID for new action
4. Set company_id from selectedUserCompany
5. Create CompanyActionsAdvanced record
6. Update AI connection through ActionAIConnectorHelper
7. Redirect to general page for further configuration
8. Provide success notification
```

### 2. General Information Update Flow
```
1. Load existing action data and populate form (GET)
2. Display form with current general information
3. Validate updated data with UPDATE_GENERAL scenario
4. Update action record with new information
5. Maintain AI connection consistency
6. Show success notification and redirect
```

### 3. Integration Settings Update Flow
```
1. Load existing integration settings (GET)
2. Display integration form with platform options
3. Validate integration configuration
4. Check platform, method, URL scheme compatibility
5. Validate custom headers structure
6. Validate JSON content format
7. Update integration settings
8. Update AI connection integration
9. Provide detailed feedback on success/errors
```

### 4. Action History Flow
```
1. Validate company selection and redirect if needed
2. Create ActionActivitiesSearch with action UUID
3. Load action activities history
4. Display chronological activity log
5. Show integration usage patterns
```

### 5. Action Deletion Flow
```
1. Validate action existence and company access
2. Execute delete operation with proper cleanup
3. Update AI connection status
4. Provide user feedback through notifications
5. Redirect to actions index
```

---

## Security Features

### 1. Company-based Access Control
- All operations isolated by selectedUserCompany
- Action existence validation within company
- JOIN queries ensure company data isolation
- UUID validation with company filtering

### 2. Role-based Permissions
- ProfileRule integration for edit access
- DeleteRule integration for delete operations
- Dynamic action button display based on permissions
- Proper authorization checks

### 3. Input Validation
- String length limits for all text fields
- HTTPS-only URL validation with scheme enforcement
- JSON structure validation for content
- Custom header validation with format checking
- Platform and method validation against allowed lists

### 4. AI Integration Security
- Secure ActionAIConnectorHelper integration
- AI connection consistency maintenance
- Find prompt validation and sanitization
- Integration state management

---

## Integration Points

### Dependencies
- `CompanyActionsAdvanced` - main action model
- `ActionAIConnectorHelper` - AI integration functionality
- `UuidHelper` - UUID validation and generation
- `NotificationHelper` - user notifications
- `EntityStatusWidget` - status display
- `JsonValidator` - JSON content validation
- Role-based access rules (ProfileRule, DeleteRule)

### AI Integration
- ActionAIConnectorHelper for AI functionality
- Find prompt management for action discovery
- AI connection state synchronization
- Integration with AI workflow systems

### Platform Integration
- Multiple platform support through configuration
- HTTP method standardization
- HTTPS enforcement for security
- Custom header and content management
- JSON-based configuration storage

### Related Components
- **Companies Management** - company selection and validation
- **User Management** - role-based access control
- **AI Systems** - action discovery and execution
- **Platform APIs** - external system integration

---

## Usage Examples

### Create action
```http
POST /actions/create
Content-Type: application/x-www-form-urlencoded

SaveActionForm[name]=Customer Notification&SaveActionForm[description]=Send notification to customer&SaveActionForm[find_prompt]=Send customer notification when order is complete&SaveActionForm[status]=1
```

### Update general information
```http
POST /actions/general?uuid=550e8400-e29b-41d4-a716-446655440000
Content-Type: application/x-www-form-urlencoded

SaveActionForm[name]=Updated Customer Notification&SaveActionForm[description]=Updated description&SaveActionForm[find_prompt]=Updated AI prompt&SaveActionForm[status]=1
```

### Update integration settings
```http
POST /actions/integration?uuid=550e8400-e29b-41d4-a716-446655440000
Content-Type: application/x-www-form-urlencoded

SaveActionForm[platform]=webhook&SaveActionForm[method]=POST&SaveActionForm[url]=https://api.example.com/webhook&SaveActionForm[content_type]=application/json&SaveActionForm[headers][0][key]=Authorization&SaveActionForm[headers][0][value]=Bearer token123&SaveActionForm[content]={"message":"Customer notification"}
```

### View action history
```http
GET /actions/history?uuid=550e8400-e29b-41d4-a716-446655440000
```

### Delete action
```http
POST /actions/delete?uuid=550e8400-e29b-41d4-a716-446655440000
```

---

## Performance Considerations

### Database Optimization
- Company-based indexing for all action queries
- JOIN optimization with companies table
- Proper indexing for search and filter operations
- UUID-based lookups with company filtering

### AI Integration Performance
- Efficient AI connection updates
- Batched AI operation processing
- Connection state caching
- Asynchronous AI integration where possible

### Search Performance
- BaseSearch optimization with proper filtering
- Role-based query optimization
- Grid column data optimization
- Pagination for large action lists

---

## Error Handling

### Common Errors
- **"Company Action not found"** - Invalid UUID or no access
- **Platform validation errors** - Invalid platform selection
- **URL scheme errors** - Non-HTTPS URLs rejected
- **JSON validation errors** - Invalid content format
- **Header validation errors** - Invalid header structure

### Integration Errors
- AI connection failures
- Platform API validation errors
- HTTP method compatibility issues
- Content type validation failures

### Notification System
- NotificationHelper integration for user feedback
- Success notifications for all operations
- Detailed error messages for validation failures
- Model error aggregation for complex forms

### HTTP Status Codes
- **200 OK** - Successful operations
- **302 Found** - Redirects after operations
- **500 Internal Server Error** - System errors with user-friendly messages

---

*ActionsController is a comprehensive component for managing company automation actions with AI integration, platform connectivity, and sophisticated validation systems for enterprise workflow automation.*