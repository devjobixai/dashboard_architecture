# AgentsController Documentation

## Overview

AgentsController manages AI agents in the Jobix Dashboard admin panel. It covers creating, editing, copying, deleting agents, knowledge management, widget configuration, and phone numbers.

Base path: `/agents`  
Controller type: WebController (Admin Application)  
Namespace: `apps\\admin\\controllers\\AgentsController`

---

### 3. Save Chat Widget Settings

| Param | Value |
|------|-------|
| URL | `/agents/save-chat-widget-settings?uuid={agent_uuid}` |
| Method | POST |
| Description | Save chat widget settings for an agent |

URL Parameters:
- `uuid` (string) — agent UUID

Input (SaveChatWidgetForm bodyParams):
- See SaveChatWidgetForm for fields

Business logic:
- Same as call widget, but for chat settings

Responses:
- Success: Redirect to `/agents/profile?uuid={uuid}&showTab=embed-tab&showSubTab=chat-widget`
- Error: Redirect with error notification

---

### 4. Create — Create a New Agent

| Param | Value |
|------|-------|
| URL | `/agents/create` |
| Methods | GET, POST, AJAX |
| Description | Create a new AI agent |

#### GET `/agents/create`
Purpose: Render the agent creation form

#### AJAX Validation
Purpose: Real-time form validation

Response Format: JSON with validation results

#### POST `/agents/create`
Purpose: Handle agent creation

Input (SaveAgentForm):

| Field | Type | Required | Description |
|------|------|----------|-------------|
| `avatar` | file | ✗ | Agent avatar file |
| + others | mixed | varies | Depends on scenario |

Creation business logic:
1. Instantiate SaveAgentForm with scenario = SCENARIO_CREATE
2. AJAX validation (if AJAX request)
3. Upload avatar via UploadedFile
4. Call `model->create()`
5. Show notifications and redirect

Responses:
- AJAX: JSON validation results
- Success: Redirect to `/agents/profile/{new_agent_uuid}`
- Error: HTML view with form and errors
- GET: HTML view with empty form

---

### 5. Copy — Copy an Agent

| Param | Value |
|------|-------|
| URL | `/agents/copy?uuid={agent_uuid}` |
| Method | GET |
| Description | Create a copy of an existing agent |

URL Parameters:
- `uuid` (string) — agent UUID to copy

Input (CopyAgentForm):
- UUID via GET parameter

Business logic:
1. Create CopyAgentForm with UUID
2. Call `copy()`
3. Create a new agent copy
4. Show result via notifications

Responses:
- Success: Redirect to `/agents/profile/{new_agent_uuid}` with success notification
- Error: Redirect to referrer with error notification

---

### 6. Profile — Agent Profile (Edit)

| Param | Value |
|------|-------|
| URL | `/agents/profile?uuid={agent_uuid}` |
| Methods | GET, POST, AJAX |
| Description | View and edit agent profile |

URL Parameters:
- `uuid` (string) — agent UUID
- `scenario` (string, optional) — edit scenario
- `layout` (string, optional) — "node" for special rendering

#### GET `/agents/profile`
Purpose: Render the edit form

Special Features:
- Node Layout Detection: Automatically sets layout="node" for special rendering
- Data Population: Automatically loads agent data into the form

#### AJAX Validation
Response Format: JSON with validation results

#### POST `/agents/profile`
Purpose: Handle agent update

Scenarios (SaveAgentForm):

| Scenario | Method | Description | Show Tab |
|----------|--------|-------------|----------|
| `SCENARIO_GENERAL` | `saveGeneral()` | General info + avatar upload | behavior-tab |
| `SCENARIO_DETAILS` | `saveDetails()` | Agent details | behavior-tab |
| `SCENARIO_DESCRIPTION` | `saveDescription()` | Agent description | behavior-tab |
| `SCENARIO_PROMPTS` | `savePrompt()` | AI prompts | behavior-tab |
| `SCENARIO_GREETING` | `saveGreeting()` | Agent greeting | behavior-tab |
| `SCENARIO_PRESET` | `savePreset()` | Agent presets | behavior-tab |
| `SCENARIO_DYNAMIC_PARAMS` | `saveDynamicParam()` | Dynamic parameters | dynamic-params-tab |
| `SCENARIO_KNOWLEDGE_TEXT` | `saveKnowledgeText()` | Text knowledge | knowledge-tab |

Node Layout Support:
- Supports a special layout for node-based UI
- Passes the layout parameter in redirect URLs

Responses:
- AJAX: JSON validation results
- Success: Redirect to `/agents/profile?uuid={uuid}&showTab={appropriate_tab}`
- Error: HTML view with form and errors
- GET: HTML view with populated form

---

### 7. Delete — Remove an Agent

| Param | Value |
|------|-------|
| URL | `/agents/delete?uuid={agent_uuid}` |
| Method | GET |
| Description | Delete an agent |

URL Parameters:
- `uuid` (string) — agent UUID to delete

Input:
- UUID via GET parameter

Business logic:
1. Instantiate SaveAgentForm with UUID
2. Call `delete()`
3. Show notification with the result

Responses:
- Always: Redirect to `/agents/index` with a notification

---

### 8. Upload Knowledge Files

| Param | Value |
|------|-------|
| URL | `/agents/upload-knowledge-files?uuid={agent_uuid}` |
| Method | POST |
| Description | Upload files to the agent's knowledge base |

URL Parameters:
- `uuid` (string) — agent UUID

Input:
- `file` — file via `UploadedFile::getInstanceByName('file')`

Business logic:
1. Receive file via UploadedFile
2. Call `uploadKnowledgeFile()` on SaveAgentForm
3. Handle result and errors

Responses:
- Success: Text "success" (HTTP 200)
- Error: Error message text (HTTP 500)

---

### 9. Delete Knowledge File

| Param | Value |
|------|-------|
| URL | `/agents/delete-knowledge-file?uuid={agent_uuid}&file_uuid={file_uuid}` |
| Method | DELETE |
| Description | Delete a file from the agent's knowledge base |

URL Parameters:
- `uuid` (string) — agent UUID
- `file_uuid` (string) — file UUID to delete

Business logic:
1. Enforce DELETE method
2. Pass `file_uuid` to knowledge_file field
3. Call `deleteKnowledgeFile()`

Responses:

Success:
```json
{
  "success": true
}
```

Error:
```json
{
  "success": false,
  "errors": {
    "field": ["error message"]
  }
}
```

Invalid Method:
```json
{
  "success": false
}
```

---

### 10. Create Phone Number

| Param | Value |
|------|-------|
| URL | `/agents/create-phone-number` |
| Method | POST |
| Description | Add a phone number to an agent |

Input (CreatePhoneNumberForm bodyParams):
- See CreatePhoneNumberForm for fields

Response:
- Format: JSON array with the operation result

---

### 11. Delete Phone Number

| Param | Value |
|------|-------|
| URL | `/agents/delete-phone-number` |
| Method | POST |
| Description | Delete an agent's phone number |

Input (DeletePhoneNumberForm bodyParams):
- See DeletePhoneNumberForm for fields

Response:
- Format: JSON array with the operation result

---

### 12. Run Test — Execute Agent Test

| Param | Value |
|------|-------|
| URL | `/agents/run-test` |
| Method | POST |
| Description | Run agent functionality tests |

Input (CreateTestForm bodyParams):
- See CreateTestForm for fields

Response:
- Format: JSON array with test results

---

### 13. Numbers List

| Param | Value |
|------|-------|
| URL | `/agents/numbers-list` |
| Method | GET |
| Description | Get available phone numbers list |

Response:
- Format: JSON array of available numbers

---

## Form Classes

### SaveAgentForm

File: `src/apps/admin/forms/agent/SaveAgentForm.php`  
Extends: `yii\\base\\Model`

Scenarios:
- `SCENARIO_CREATE` — create a new agent
- `SCENARIO_GENERAL` — general info
- `SCENARIO_DETAILS` — agent details
- `SCENARIO_DESCRIPTION` — description
- `SCENARIO_PROMPTS` — AI prompts
- `SCENARIO_GREETING` — greeting
- `SCENARIO_PRESET` — presets
- `SCENARIO_DYNAMIC_PARAMS` — dynamic parameters
- `SCENARIO_KNOWLEDGE_TEXT` — text knowledge

File Upload Fields:
- `avatar` — agent avatar (UploadedFile)
- `knowledge_file` — knowledge files (UploadedFile)

Key methods:
- `create()` — create a new agent
- `populate()` — load existing data
- `saveGeneral()` — save general info
- `saveDetails()` — save details
- `saveDescription()` — save description
- `savePrompt()` — save prompts
- `saveGreeting()` — save greeting
- `savePreset()` — save presets
- `saveDynamicParam()` — save dynamic params
- `saveKnowledgeText()` — save text knowledge
- `delete()` — delete agent
- `uploadKnowledgeFile()` — upload knowledge file
- `deleteKnowledgeFile()` — delete knowledge file

## Endpoints

### 1. Index — Agents List

| Param | Value |
|------|-------|
| URL | `/agents/index` |
| Method | GET |
| Description | Display all AI agents of the company with search |

Input:
- Query parameters for search (via AgentsSearch)

Business logic:
1. Create AgentsSearch model
2. Load search parameters from query string
3. Execute search and build DataProvider

Response:
- Success: HTML view with agents list and search form
- Data: `pageName`, `dataProvider`, `searchModel`

---

### 2. Save Call Widget Settings

| Param | Value |
|------|-------|
| URL | `/agents/save-call-widget-settings?uuid={agent_uuid}` |
| Method | POST |
| Description | Save call widget settings for an agent |

URL Parameters:
- `uuid` (string) — agent UUID

Input (SaveCallWidgetForm bodyParams):
- See SaveCallWidgetForm for fields

Business logic:
1. Instantiate SaveCallWidgetForm with agent UUID
2. Load data from bodyParams
3. Save widget settings
4. Show notification

Responses:
- Success: Redirect to `/agents/profile?uuid={uuid}&showTab=embed-tab&showSubTab=call-widget`
- Error: Redirect with error notification

---
