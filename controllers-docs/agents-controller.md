# AgentsController Documentation

## Overview

AgentsController manages AI agents in the Jobix Dashboard administrative panel. This is one of the most complex controllers, including creation, editing, copying, deletion of agents, knowledge management, widget configuration, and phone number management.

**Base Path:** `/agents`  
**Controller Type:** WebController (Admin Application)  
**Namespace:** `apps\admin\controllers\AgentsController`

---

## Endpoints

### 1. Index - Agents List

| Parameter | Value |
|-----------|-------|
| **URL** | `/agents/index` |
| **Method** | GET |
| **Description** | Display list of all company AI agents with search functionality |

**Input Data:**
- Query parameters for search (through AgentsSearch)

**Business Logic:**
1. Create AgentsSearch model
2. Load search parameters from query string
3. Execute search and create DataProvider

**Response:**
- **Success:** HTML view with agents list and search form
- **Data:** `pageName`, `dataProvider`, `searchModel`

---

### 2. Save Call Widget Settings - Call Widget Configuration

| Parameter | Value |
|-----------|-------|
| **URL** | `/agents/save-call-widget-settings?uuid={agent_uuid}` |
| **Method** | POST |
| **Description** | Save call widget settings for an agent |

**URL Parameters:**
- `uuid` (string) - Agent UUID

**Input Data (SaveCallWidgetForm bodyParams):**
- Field details require SaveCallWidgetForm analysis

**Business Logic:**
1. Create SaveCallWidgetForm with agent UUID
2. Load data from bodyParams
3. Save widget settings
4. Show notification about result

**Responses:**
- **Success:** Redirect to `/agents/profile?uuid={uuid}&showTab=embed-tab&showSubTab=call-widget`
- **Error:** Redirect with error notification

---

### 3. Save Chat Widget Settings - Chat Widget Configuration

| Parameter | Value |
|-----------|-------|
| **URL** | `/agents/save-chat-widget-settings?uuid={agent_uuid}` |
| **Method** | POST |
| **Description** | Save chat widget settings for an agent |

**URL Parameters:**
- `uuid` (string) - Agent UUID

**Input Data (SaveChatWidgetForm bodyParams):**
- Field details require SaveChatWidgetForm analysis

**Business Logic:**
- Similar to call widget, but for chat settings

**Responses:**
- **Success:** Redirect to `/agents/profile?uuid={uuid}&showTab=embed-tab&showSubTab=chat-widget`
- **Error:** Redirect with error notification

---

### 4. Create - New Agent Creation

| Parameter | Value |
|-----------|-------|
| **URL** | `/agents/create` |
| **Methods** | GET, POST, AJAX |
| **Description** | Create new AI agent |

#### GET `/agents/create`
**Purpose:** Display agent creation form

#### AJAX Validation
**Purpose:** Real-time form validation

**Response Format:** JSON with validation results

#### POST `/agents/create`
**Purpose:** Handle agent creation

**Input Data (SaveAgentForm):**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `avatar` | file | ✗ | Agent avatar file |
| + other fields | mixed | varies | Depends on scenario |

**Creation Business Logic:**
1. Instantiate SaveAgentForm with scenario = SCENARIO_CREATE
2. AJAX validation (if AJAX request)
3. Upload avatar via UploadedFile
4. Call `model->create()`
5. Show notifications and redirect

**Responses:**
- **AJAX:** JSON validation results
- **Success:** Redirect to `/agents/profile/{new_agent_uuid}`
- **Error:** HTML view with form and errors
- **GET:** HTML view with empty form

---

### 5. Copy - Agent Copying

| Parameter | Value |
|-----------|-------|
| **URL** | `/agents/copy?uuid={agent_uuid}` |
| **Method** | GET |
| **Description** | Create a copy of existing agent |

**URL Parameters:**
- `uuid` (string) - Agent UUID to copy

**Input Data (CopyAgentForm):**
- UUID via GET parameter

**Business Logic:**
1. Create CopyAgentForm with UUID
2. Call `copy()`
3. Create new agent copy
4. Show result via notifications

**Responses:**
- **Success:** Redirect to `/agents/profile/{new_agent_uuid}` with success notification
- **Error:** Redirect to referrer with error notification

---

### 6. Profile - Agent Profile (Edit)

| Parameter | Value |
|-----------|-------|
| **URL** | `/agents/profile?uuid={agent_uuid}` |
| **Methods** | GET, POST, AJAX |
| **Description** | View and edit agent profile |

**URL Parameters:**
- `uuid` (string) - Agent UUID

**Query Parameters:**
- `showTab` (string, optional) - Tab to display (general, knowledge, embed-tab, etc.)
- `showSubTab` (string, optional) - Sub-tab to display within main tab

#### GET `/agents/profile`
**Purpose:** Display agent profile with tabs

**Behavior:**
- Automatic data loading through SaveAgentForm populate()
- Tab management for different agent configuration sections
- Sub-tab support for detailed configurations

#### AJAX Validation
**Purpose:** Real-time form validation for profile updates

**Response Format:** JSON with validation results

#### POST `/agents/profile`
**Purpose:** Handle agent profile updates

**Input Data (SaveAgentForm bodyParams):**
- Fields depend on scenario (UPDATE)
- Avatar file upload support
- Various agent configuration parameters

**Update Business Logic:**
1. Load existing agent data
2. Validate submitted changes
3. Handle avatar upload if provided
4. Call `model->update()`
5. Show notifications and maintain current tab/sub-tab state

**Responses:**
- **AJAX:** JSON validation results
- **Success:** HTML view with updated profile and success notification
- **Error:** HTML view with form and validation errors
- **GET:** HTML view with populated agent profile form

---

### 7. Delete - Agent Deletion

| Parameter | Value |
|-----------|-------|
| **URL** | `/agents/delete?uuid={agent_uuid}` |
| **Method** | POST |
| **Description** | Delete an agent |

**URL Parameters:**
- `uuid` (string) - Agent UUID for deletion

**Business Logic:**
1. Create SaveAgentForm with SCENARIO_DELETE
2. Set agent UUID
3. Call form delete() method
4. Handle associated data cleanup
5. Show notification about result

**Delete Operation Flow:**
```php
$model = new SaveAgentForm();
$model->scenario = SaveAgentForm::SCENARIO_DELETE;
$model->uuid = $this->request->get('uuid');
$isDeleted = $model->delete();
```

**Responses:**
- **Success:** Redirect to `/agents/index` with success notification
- **Error:** Redirect to referrer with error notification
- **Not Found:** Error if agent doesn't exist or no access

---

### 8. Delete Knowledge - Knowledge Item Deletion

| Parameter | Value |
|-----------|-------|
| **URL** | `/agents/delete-knowledge?uuid={agent_uuid}&knowledge_uuid={knowledge_uuid}` |
| **Method** | POST |
| **Description** | Delete specific knowledge item from agent |

**URL Parameters:**
- `uuid` (string) - Agent UUID
- `knowledge_uuid` (string) - Knowledge item UUID for deletion

**Business Logic:**
1. Validate agent and knowledge UUIDs
2. Check permissions for knowledge deletion
3. Delete knowledge item
4. Update agent knowledge index if needed

**Responses:**
- **Success:** Redirect to agent profile knowledge tab
- **Error:** Redirect with error notification

---

### 9. Attach Phone Number - Phone Number Assignment

| Parameter | Value |
|-----------|-------|
| **URL** | `/agents/attach-phone-number?uuid={agent_uuid}` |
| **Method** | POST |
| **Description** | Attach phone number to agent |

**URL Parameters:**
- `uuid` (string) - Agent UUID

**Input Data:**
- Phone number selection parameters
- Assignment configuration data

**Business Logic:**
1. Validate agent UUID and phone number availability
2. Create association between agent and phone number
3. Update agent phone number configuration
4. Show notification about assignment result

**Responses:**
- **Success:** Redirect to agent profile with success notification
- **Error:** Redirect with error notification

---

### 10. Detach Phone Number - Phone Number Removal

| Parameter | Value |
|-----------|-------|
| **URL** | `/agents/detach-phone-number?uuid={agent_uuid}&phone_uuid={phone_uuid}` |
| **Method** | POST |
| **Description** | Remove phone number from agent |

**URL Parameters:**
- `uuid` (string) - Agent UUID
- `phone_uuid` (string) - Phone number UUID to detach

**Business Logic:**
1. Validate agent and phone number UUIDs
2. Remove association between agent and phone number
3. Update agent configuration
4. Show notification about removal result

**Responses:**
- **Success:** Redirect to agent profile with success notification
- **Error:** Redirect with error notification

---

## Form Classes

### 1. SaveAgentForm

**Purpose:** Main form for agent CRUD operations

**Scenarios:**
- `SCENARIO_CREATE` - create new agent
- `SCENARIO_UPDATE` - edit existing agent
- `SCENARIO_DELETE` - delete agent
- `SCENARIO_POPULATE` - load existing agent data

**Key Methods:**
- `populate()` - load existing agent data for editing
- `create()` - create new agent with all associated data
- `update()` - update existing agent
- `delete()` - delete agent and cleanup associated data

**Features:**
- Avatar file upload handling
- Multi-tab configuration support
- Validation for different scenarios
- Integration with knowledge management
- Phone number assignment support

---

### 2. CopyAgentForm

**Purpose:** Handle agent copying functionality

**Features:**
- Deep copy of agent configuration
- Knowledge base duplication
- Widget settings copying
- Phone number handling for copied agents

**Key Methods:**
- `copy()` - perform complete agent duplication

---

### 3. SaveCallWidgetForm

**Purpose:** Manage call widget settings for agents

**Features:**
- Call widget appearance configuration
- Integration settings
- Display options
- Functionality parameters

---

### 4. SaveChatWidgetForm

**Purpose:** Manage chat widget settings for agents

**Features:**
- Chat widget appearance configuration
- Integration settings
- Display options
- Functionality parameters

---

### 5. AgentsSearch

**Purpose:** Handle agent search and filtering

**Features:**
- Text search across agent names and descriptions
- Status filtering
- Company-based isolation
- Pagination support
- Sorting options

---

## Security Features

### 1. Company-based Access Control
- All operations isolated by selectedUserCompany
- Agent access limited to company scope
- Knowledge management within company boundaries
- Phone number assignments restricted to company resources

### 2. UUID Validation
- UuidHelper validation for all UUID parameters
- Existence checking for agents, knowledge, and phone numbers
- Secure parameter handling

### 3. File Upload Security
- Avatar upload validation
- File type restrictions
- Size limitations
- Secure file storage

### 4. Knowledge Management Security
- Access control for knowledge items
- Validation of knowledge ownership
- Secure knowledge deletion

---

## Integration Points

### Dependencies
- `AgentsAdvanced` - main agents model
- `AgentKnowledgeAdvanced` - knowledge management model
- `CompanyPhoneNumbersAdvanced` - phone number integration
- `UuidHelper` - UUID validation and generation
- Various widget configuration models

### Related Functionality
- Integration with knowledge base system
- Phone number management integration
- Widget embedding system
- File upload and storage system

### External Integrations
- AI/ML model integration for agent functionality
- Communication provider integrations
- Analytics and tracking systems

---

## Usage Examples

### Create new agent
```http
POST /agents/create
Content-Type: multipart/form-data

SaveAgentForm[name]=Customer Support Agent&SaveAgentForm[description]=AI agent for customer support&avatar=@avatar.jpg
```

### Update agent profile
```http
POST /agents/profile?uuid=550e8400-e29b-41d4-a716-446655440000
Content-Type: application/x-www-form-urlencoded

SaveAgentForm[name]=Updated Agent Name&SaveAgentForm[description]=Updated description
```

### Copy existing agent
```http
GET /agents/copy?uuid=550e8400-e29b-41d4-a716-446655440000
```

### Configure call widget
```http
POST /agents/save-call-widget-settings?uuid=550e8400-e29b-41d4-a716-446655440000
Content-Type: application/x-www-form-urlencoded

SaveCallWidgetForm[enabled]=1&SaveCallWidgetForm[theme]=modern&SaveCallWidgetForm[position]=bottom-right
```

### Attach phone number
```http
POST /agents/attach-phone-number?uuid=550e8400-e29b-41d4-a716-446655440000
Content-Type: application/x-www-form-urlencoded

phone_uuid=550e8400-e29b-41d4-a716-446655440001
```

---

## Error Handling

### Common Errors
- **"Agent not found"** - Invalid agent UUID or access denied
- **"Knowledge not found"** - Invalid knowledge UUID or access denied
- **"Phone number not available"** - Phone number already assigned or not accessible
- **Avatar upload failures** - File validation or storage errors

### File Upload Errors
- Invalid file format for avatar
- File size exceeds limits
- Storage system errors
- Permission issues

### Knowledge Management Errors
- Knowledge item conflicts
- Invalid knowledge format
- Access permission errors
- Storage limitations

### Widget Configuration Errors
- Invalid widget parameters
- Integration configuration failures
- Display setting conflicts

### HTTP Status Codes
- **200** - Success responses
- **404** - Agent, knowledge, or phone number not found
- **400** - Invalid input parameters or validation failures
- **500** - System errors with user-friendly messages
- **Redirect** - Success operations redirect to appropriate pages

---

*AgentsController is a comprehensive system for managing AI agents with advanced features including knowledge management, widget configuration, phone number integration, and multi-tab profile management capabilities.*