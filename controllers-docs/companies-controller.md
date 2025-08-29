# CompaniesController Documentation

## Overview

CompaniesController manages companies in the Jobix Dashboard admin panel, including creating, editing, deleting, and activating companies.

Base path: `/companies`  
Controller type: WebController (Admin Application)  
Namespace: `apps\\admin\\controllers\\CompaniesController`

---

## Endpoints

### 1. Index — Companies List

| Param | Value |
|-------|-------|
| URL | `/companies/index` |
| Method | GET |
| Description | Display a list of all companies with search |

Input:
- Query parameters for search (via CompaniesSearch)

Response:
- Success: HTML view with `dataProvider` and `searchModel`
- Data: `dataProvider` (companies list), `searchModel` (search form)

---

### 2. Active List — AJAX list of active companies

| Param | Value |
|-------|-------|
| URL | `/companies/active-list` |
| Method | GET (AJAX only) |
| Description | Fetch active companies for AJAX requests |

Input:
- Query parameters for search
- `filter[status]` — automatically set to `active`

Validation:
- AJAX request required (otherwise 404)

Response:
- Success: JSON with active companies
- Error: 404 NotFoundHttpException (if not AJAX)
- Format: Partial view with serialized data

---

### 3. Create — Create a Company

| Param | Value |
|-------|-------|
| URL | `/companies/create` |
| Methods | GET, POST |
| Description | Create a new company |

#### GET `/companies/create`
Purpose: Render the creation form

Response:
- Success: HTML view with an empty CreateCompanyForm

#### POST `/companies/create`
Purpose: Handle company creation

Input (CreateCompanyForm):

| Field | Type | Required | Validation | Description |
|-------|------|----------|------------|-------------|
| `name` | string | ✓ | required, string, min:5, max:255, trim, unique validation | Company name |
| `description` | string | ✗ | string, max:1024, trim | Company description |
| `status` | boolean | ✗ | boolean | Active status (default: true) |
| `settings` | array | ✗ | array | Company settings |
| `additional_info` | array | ✗ | array | Additional info |

CreateCompanyForm validation:
```php
[['name'], 'required']
[['name', 'description'], 'filter', 'filter' => 'trim']
[['name'], 'string', 'min' => 5, 'max' => 255]
[['description'], 'string', 'max' => 1024]
[['status'], 'boolean']
[['name'], 'validateName'] // Custom: uniqueness by slug
```

Custom validation:
- `validateName`: Checks uniqueness of the company name via slug for the current user

Creation business logic:
1. Generate slug from company name
2. Generate UUID for the company
3. Generate API key (40 chars)
4. Set current user's `user_id`
5. Convert status to `Constants::STATUS_ACTIVE/INACTIVE`

Auto-populated fields:
- `slug` — generated from name
- `uuid` — unique identifier
- `settings.api_key` — random 40-char key
- `user_id` — current user ID
- `status` — converted to constant

Responses:
- Success: Redirect to `/companies/edit/{uuid}` of the new company
- Validation Error: HTML view with form and errors

---

### 4. Edit — Edit a Company

| Param | Value |
|-------|-------|
| URL | `/companies/edit/{uuid}` |
| Methods | GET, POST |
| Description | Edit an existing company |

URL Parameters:
- `uuid` (string) — unique company identifier

#### GET `/companies/edit/{uuid}`
Purpose: Render the edit form

Behavior:
- If the company is not found — redirect to index
- Otherwise — load company data into the form

#### POST `/companies/edit/{uuid}`
Purpose: Handle company update

Input (EditCompanyForm):

| Field | Type | Required | Validation | Description |
|-------|------|----------|------------|-------------|
| `uuid` | string | ✓ | required | Company UUID |
| `name` | string | ✓ | inherited from CreateCompanyForm + modified unique check | Company name |
| `description` | string | ✗ | inherited from CreateCompanyForm | Company description |
| `status` | boolean | ✗ | inherited from CreateCompanyForm | Active status |
| `settings` | array | ✗ | inherited from CreateCompanyForm | Settings |
| `additional_info` | array | ✗ | inherited from CreateCompanyForm | Additional info |

EditCompanyForm specifics:
- Extends CreateCompanyForm
- Adds UUID field as required
- Modifies validateName to exclude the current company
- Automatically loads existing company data

Modified Name Validation:
```php
// Excludes the current company from uniqueness check
['!=', 'uuid', $this->uuid]
```

Update business logic:
1. Update slug from the new name
2. Convert status
3. Set updated_at timestamp
4. Call `CompaniesAdvanced::updateCompany()`

Additional methods:
- `getCompany()` — returns the company object
- `getCompanyApiKey()` — returns the company's API key

Responses:
- Success: Redirect to `/companies/edit/{uuid}`
- Not Found: Redirect to `/companies/index`
- Validation Error: HTML view with form and errors

---

### 5. Delete — Delete a Company

| Param | Value |
|-------|-------|
| URL | `/companies/delete/{uuid}` |
| Method | POST |
| Description | Delete a company |

URL Parameters:
- `uuid` (string) — unique company identifier

Input (DeleteCompanyForm):

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `uuid` | string | ✓ (constructor) | UUID of the company to delete |

DeleteCompanyForm specifics:
- Does not extend Model (simple class)
- UUID is passed via constructor
- Automatically finds company by UUID + `user_id`
- `process()` returns a boolean result

Deletion business logic:
1. Find company by UUID and current user's `user_id`
2. Call `CompaniesAdvanced::deleteCompany()`
3. Return boolean result (`deleteCount >= 1`)

Security:
- Only deletes companies belonging to the current user (`user_id` check)

Response:
- Always: Redirect to `/companies/index` (regardless of result)

---

### 6. Activate — Activate a Company

| Param | Value |
|-------|-------|
| URL | `/companies/activate/{uuid}` |
| Methods | GET, AJAX |
| Description | Activate a company as the current one for the user |

URL Parameters:
- `uuid` (string) — unique company identifier

Request validation:
- Only GET or AJAX requests are allowed
- Other methods return 404

Input (ActivateCompanyForm):

| Field | Type | Required | Validation | Description |
|-------|------|----------|------------|-------------|
| `company_uuid` | string | ✓ | required, custom validation | Company UUID to activate |

ActivateCompanyForm validation:
```php
[['company_uuid'], 'required']
[['company_uuid'], 'validateCompanyUuid'] // Custom validation
```

Custom Company Validation:
1. Company existence: check via `CompaniesAdvanced::findByParams()`
2. User existence: check via `User::find()->whereId()`
3. Permissions:
   - If user is admin — access to all companies
   - Otherwise — check via CompanyUsersAdvanced

Permission Logic:
```php
if (!$user->is_admin) {
    // Check access through CompanyUsersAdvanced
    $companyAccess = CompanyUsersAdvanced::find()
        ->whereCompanyId($company->id)
        ->whereUserId($user->id)
        ->exists();
}
```

Activation business logic:
1. Validate permissions
2. Update user's `additional_info` (company_uuid)
3. Save the user
4. Refresh Yii::$app->user identity

Error Messages:
- "Company UUID is required." — company not found
- "User not Found." — user not found
- "You dont have permission to access this company." — no access rights

Responses:

#### AJAX request:
```json
{
  "status": true,
  "errors": {}
}
```

or

```json
{
  "status": false,
  "errors": {
    "company_uuid": ["Error message"]
  }
}
```

#### GET request:
- Success/Error: Redirect to `/companies/index`

#### Wrong method:
- Error: 404 NotFoundHttpException

---

## Form Classes

### 1. CreateCompanyForm

File: `src/apps/admin/forms/company/CreateCompanyForm.php`  
Extends: `yii\\base\\Model`

Public properties:
```php
public string|null $name = null;          // Company name
public mixed $description = null;         // Description
public bool $status = true;               // Status (default: active)
public array $settings = [];              // Settings
public array $additional_info = [];       // Additional information
```

Attribute Labels:
```php
'name' => 'Company Name'
'status' => 'Status'
'description' => 'Company Description'
```

Key methods:
- `rules()` — validation rules
- `validateName()` — custom uniqueness validation
- `process()` — create a company

---

### 2. EditCompanyForm

File: `src/apps/admin/forms/company/EditCompanyForm.php`  
Extends: `CreateCompanyForm`

Additional properties:
```php
public string $uuid = '';                 // Company UUID
protected object|null $_company = null;   // Cached company
```

Constructor: Takes UUID, loads company data

Additional methods:
- `getCompany()` — returns the company object
- `getCompanyApiKey()` — returns the API key
- Modified `validateName()` — excludes the current company
- Modified `process()` — updates instead of creating

---

### 3. DeleteCompanyForm

File: `src/apps/admin/forms/company/DeleteCompanyForm.php`  
Extends: (simple class)

Public properties:
```php
public string $uuid = '';                 // Company UUID
protected object|null $_company = null;   // Cached company
```

Constructor: Finds company by UUID + user_id

Methods:
- `process()` — delete company (returns boolean)

---

### 4. ActivateCompanyForm

File: `src/apps/admin/forms/company/ActivateCompanyForm.php`  
Extends: `yii\\base\\Model`

Public properties:
```php
public string $company_uuid = '';         // Company UUID
protected User|null $_user = null;        // Cached user
```

Methods:
- `rules()` — validation rules
- `validateCompanyUuid()` — complex permission validation
- `process()` — activate the company for the user

---

## Security Features

### 1. User Isolation
- All operations are tied to the current user's `user_id`
- DeleteCompanyForm verifies company ownership

### 2. Permission Checking
- ActivateCompanyForm validates access rights to the company
- Admin privileges supported
- For non-admin users, checks via CompanyUsersAdvanced

### 3. UUID Validation
- All operations use UUID instead of ID
- Company existence is validated before operations

### 4. AJAX Protection
- active-list is available only via AJAX
- activate supports both AJAX and regular requests

---

## Business Logic

### 1. Company Creation Flow
```
1. Validate form (name, description, status)
2. Check name uniqueness (by slug)
3. Generate slug, UUID, API key
4. Save with user_id
5. Redirect to edit page
```

### 2. Company Activation Flow
```
1. Check company existence
2. Validate user permissions
3. Update user's additional_info
4. Refresh Yii identity
5. Respond (JSON/redirect)
```

### 3. Slug Generation
- Automatically generated from name via SlugHelper
- Used for uniqueness checks
- Updated when the name changes

---

## Integration Points

### Dependencies
- `CompaniesAdvanced` — core company model
- `CompanyUsersAdvanced` — user access rights
- `User` — users model
- `SlugHelper` — slug generation
- `UuidHelper` — UUID generation
- `NotificationHelper` — user notifications

### Related Models
- `CompaniesSearch` — search and filtering of companies
- Various company-related advanced models

---

## Usage Examples

### Create a company
```http
POST /companies/create
Content-Type: application/x-www-form-urlencoded

name=Test Company&description=Company description&status=1
```

### Activate a company (AJAX)
```http
GET /companies/activate/uuid-here
X-Requested-With: XMLHttpRequest
```

Response:
```json
{
  "status": true,
  "errors": {}
}
```

