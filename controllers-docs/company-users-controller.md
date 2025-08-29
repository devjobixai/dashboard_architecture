# CompanyUsersController Documentation

## Overview

CompanyUsersController manages company users in the Jobix Dashboard administrative panel. The controller provides full CRUD functionality for user management with support for roles, personal settings, and unique functionality for user migration between companies.

**Base Path:** `/company-users`  
**Controller Type:** WebController (Admin Application)  
**Namespace:** `apps\admin\controllers\CompanyUsersController`

---

## Endpoints

### 1. Index - Company Users List

| Parameter | Value |
|-----------|-------|
| **URL** | `/company-users/index?company_uuid={company_uuid}` |
| **Method** | GET |
| **Description** | Display list of all company users with search, filtering, and migration capabilities |

**URL Parameters:**
- `company_uuid` (string, required) - Company UUID

**Input Data (CompanyUserSearchForm queryParams):**

| Field | Type | Required | Validation | Description |
|-------|------|----------|------------|-------------|
| `company_uuid` | string | ✓ | required, UUID, exists | Company UUID |
| `email` | string | ✗ | string | Search by user email (ILIKE) |
| `name` | string | ✗ | string | Search by user name (CONCAT first_name + last_name) |
| `role` | string | ✗ | string | Filter by user role |
| `status` | integer | ✗ | integer | Filter by status (active/inactive) |
| `language` | string | ✗ | string | Filter by user language |
| `timezone` | string | ✗ | string | Filter by timezone |
| `is_admin` | boolean | ✗ | boolean | Administrator filter |
| `updated_at` | array | ✗ | timestamp array validation | Filter by update date [start, end] |

**Business Logic:**
1. Validate company_uuid and check user access to company
2. Create CompanyUserSearchForm and MigrateCompanyUserForm
3. Load search parameters from query string
4. Execute search with JOIN operations (user, role tables)
5. Pagination (50 elements per page)
6. Default sorting: users.created_at DESC

**Response:**
- **Success:** HTML view with users list and migration form
- **Data:** `pageName`, `searchModel`, `dataProvider`, `migrateModel`
- **Error:** NotFoundHttpException if company not found

**Available Sorting:**
- `email`, `name` (CONCAT expression), `role`, `language`, `timezone`, `status`, `updated_at`

---

### 2. Create - New User Creation

| Parameter | Value |
|-----------|-------|
| **URL** | `/company-users/create?company_uuid={company_uuid}` |
| **Methods** | GET, POST, AJAX |
| **Description** | Create new company user with role and personal data configuration |

**URL Parameters:**
- `company_uuid` (string, required) - Company UUID

#### GET `/company-users/create`
**Purpose:** Display creation form

#### AJAX Validation
**Response Format:** JSON with validation results

#### POST `/company-users/create`
**Purpose:** Process user creation

**Input Data (SaveCompanyUserForm bodyParams):**

| Field | Type | Required | Validation | Description |
|-------|------|----------|------------|-------------|
| `company_uuid` | string | ✓ | required, UUID, exists | Company UUID |
| `first_name` | string | ✓ | required, string, max:64 | User first name |
| `last_name` | string | ✗ | string, max:64 | User last name |
| `email` | string | ✓ | required, email, unique globally | User email |
| `password` | string | ✓ | required, PasswordValidator | User password (CREATE only) |
| `role_id` | integer | ✓ | required, exists within company | User role ID |
| `status` | boolean | ✗ | boolean, default: true | Activity status |
| `language` | string | ✗ | LanguageValidator | User language |
| `timezone` | string | ✗ | TimezoneValidator | User timezone |

**Detailed Validation SaveCompanyUserForm (CREATE scenario):**
```php
// Required fields
[['uuid', 'company_uuid', 'first_name', 'email', 'status', 'role_id'], 'required']
[['password'], 'required', 'on' => self::SCENARIO_CREATE] // Only for CREATE

// String validation
[['first_name', 'last_name', 'password'], 'string', 'max' => 64]

// Email validation with global uniqueness
[['email'], 'email']
[['email'], 'unique', 'targetClass' => UsersAdvanced::class]

// UUID validation
[['uuid', 'company_uuid'], UuidHelper::class]

// Custom validators
[['password'], PasswordValidator::class]
[['language'], LanguageValidator::class]
[['timezone'], TimezoneValidator::class]

// Role validation within company
[['role_id'], 'exist', 'targetClass' => CompanyRolesAdvanced::class,
 'filter' => function($query) {
     return $query->whereCompanyId($this->company->id);
 }]

// Company validation
[['company_uuid'], 'exist', 'targetClass' => CompaniesAdvanced::class,
 'filter' => function($query) {
     return $query->whereUserId($this->selectedUserCompany->user_id);
 }]
```

**Creation Business Logic:**
1. Form validation with CREATE scenario
2. Check email uniqueness globally (not just within company)
3. Start database transaction
4. Create UsersAdvanced record with personal data
5. Create CompanyUsersAdvanced record linking user to company and role
6. Generate UUIDs for both records
7. Activate company if needed
8. Transaction commit/rollback

**Responses:**
- **AJAX:** JSON validation results
- **Success:** Redirect to `/company-users/update?uuid={user_uuid}&company_uuid={company_uuid}`
- **Validation Error:** HTML view with form and errors
- **GET:** HTML view with empty form

---

### 3. Update - User Editing

| Parameter | Value |
|-----------|-------|
| **URL** | `/company-users/update?uuid={user_uuid}&company_uuid={company_uuid}` |
| **Methods** | GET, POST, AJAX |
| **Description** | Edit existing company user |

**URL Parameters:**
- `uuid` (string, required) - Company user UUID
- `company_uuid` (string, required) - Company UUID

#### GET `/company-users/update`
**Purpose:** Display edit form

**Behavior:**
- Automatic user data loading through `populate()`
- Load personal data and role settings

#### AJAX Validation
**Response Format:** JSON with validation results

#### POST `/company-users/update`
**Purpose:** Process user update

**Input Data (SaveCompanyUserForm bodyParams):**

| Field | Type | Required | Validation | Description |
|-------|------|----------|------------|-------------|
| `uuid` | string | ✓ | required, UUID, exists | Company user UUID |
| `company_uuid` | string | ✓ | required, UUID, exists | Company UUID |
| `first_name` | string | ✓ | required, string, max:64 | User first name |
| `last_name` | string | ✗ | string, max:64 | User last name |
| `email` | string | ✓ | required, email, unique (exclude current) | User email |
| `password` | string | ✗ | PasswordValidator | New password (optional for UPDATE) |
| `role_id` | integer | ✓ | required, exists within company | User role ID |
| `status` | boolean | ✗ | boolean | Activity status |
| `language` | string | ✗ | LanguageValidator | User language |
| `timezone` | string | ✗ | TimezoneValidator | User timezone |

**Data Population Logic:**
```php
public function populate(): static
{
    if ($this->uuid && $this->company_uuid) {
        $companyUser = $this->getCompanyUser(); // CompanyUsersAdvanced model
        if ($companyUser && $companyUser->user) {
            // Load user personal data
            $this->first_name = $companyUser->user->first_name;
            $this->last_name = $companyUser->user->last_name;
            $this->email = $companyUser->user->email;
            $this->language = $companyUser->user->language;
            $this->timezone = $companyUser->user->timezone;
            $this->status = $companyUser->user->status;
            $this->role_id = $companyUser->role_id;
        }
    }
    return $this;
}
```

**Responses:**
- **AJAX:** JSON validation results
- **Success:** HTML view with updated form and success notification
- **Error:** HTML view with form and errors
- **GET:** HTML view with populated form

---

### 4. Delete - User Deletion

| Parameter | Value |
|-----------|-------|
| **URL** | `/company-users/delete?uuid={user_uuid}&company_uuid={company_uuid}` |
| **Method** | POST |
| **Description** | Remove user from company |

**URL Parameters:**
- `uuid` (string, required) - Company user UUID for deletion
- `company_uuid` (string, required) - Company UUID

**Business Logic:**
1. Create SaveCompanyUserForm with SCENARIO_DELETE
2. Set uuid and company_uuid
3. Call form delete() method
4. Soft delete company user (set deleted_at)
5. Keep main user record in system

**Delete Operation Flow:**
```php
$model = new SaveCompanyUserForm();
$model->scenario = SaveCompanyUserForm::SCENARIO_DELETE;
$model->uuid = $this->request->get('uuid');
$model->company_uuid = $this->request->get('company_uuid');
$model->delete();
```

**Responses:**
- **Success:** Redirect to `/company-users/index?company_uuid={company_uuid}`
- **Error:** Redirect with error notification
- **Not Found:** Error if user doesn't exist or no access

---

### 5. Migrate - User Migration

| Parameter | Value |
|-----------|-------|
| **URL** | `/company-users/migrate?company_uuid={company_uuid}` |
| **Methods** | POST, AJAX |
| **Description** | Migrate existing user from another company to current company |

**URL Parameters:**
- `company_uuid` (string, required) - Target company UUID

#### AJAX Validation
**Response Format:** JSON with validation results

#### POST `/company-users/migrate`
**Purpose:** Process user migration

**Input Data (MigrateCompanyUserForm bodyParams):**

| Field | Type | Required | Validation | Description |
|-------|------|----------|------------|-------------|
| `user_uuid` | string | ✓ | required, UUID, exists in CompanyUsersAdvanced | User UUID for migration |
| `company_uuid` | string | ✓ | required, UUID, exists | Target company UUID |
| `role_slug` | string | ✓ | required, string, exists within target company | Role slug in target company |

**Migration Business Logic:**
```php
public function migrate(): CompanyUsersAdvanced|static
{
    // 1. Start database transaction
    $transaction = Yii::$app->db->beginTransaction();
    
    try {
        // 2. Find source company user
        $migrateCompanyUser = CompanyUsersAdvanced::find()->whereUuid($this->user_uuid)->one();
        
        // 3. Find target role in target company
        $companyRole = CompanyRolesAdvanced::find()
            ->whereSlug($this->role_slug)
            ->whereCompanyId($this->company->id)
            ->one();
        
        // 4. Create new company user record
        $companyUser = new CompanyUsersAdvanced();
        $companyUser->uuid = UuidHelper::generate();
        $companyUser->user_id = $migrateCompanyUser->user_id; // Same user, different company
        $companyUser->role_id = $companyRole->id;
        $companyUser->company_id = $this->company->id;
        
        // 5. Save and commit
        if (!$companyUser->save()) {
            $transaction->rollBack();
            return $this;
        }
        
        $transaction->commit();
        return $companyUser;
    } catch (\Exception $e) {
        $transaction->rollBack();
        // Error handling
    }
}
```

**Responses:**
- **AJAX:** JSON validation results
- **Success:** Redirect to `/company-users/index?company_uuid={company_uuid}`
- **Validation Error:** Redirect with error notification
- **System Error:** Error with administrator contact message

---

## Form Classes

### 1. SaveCompanyUserForm

**File:** `src/apps/admin/forms/company_users/SaveCompanyUserForm.php`  
**Extends:** `customs\Model` (390 lines)

**Scenarios:**
- `SCENARIO_POPULATE` - load existing user data
- `SCENARIO_CREATE` - create new user
- `SCENARIO_UPDATE` - edit existing user
- `SCENARIO_DELETE` - delete user

**Public Properties:**
```php
public mixed $uuid = null;                // Company user UUID (required for UPDATE/DELETE)
public mixed $company_uuid = null;        // Company UUID (required)
public mixed $role_id = null;             // User role ID (required)
public mixed $first_name = null;          // User first name (required)
public mixed $last_name = null;           // User last name
public mixed $email = null;               // User email (required, unique globally)
public mixed $password = null;            // Password (required only for CREATE)
public mixed $status = true;              // Activity status (default: true)
public mixed $language = null;            // User language
public mixed $timezone = null;            // User timezone
```

**Key Methods:**
- `populate()` - load existing user data and personal settings
- `create()` - create new user and link to company
- `update()` - update user personal data
- `delete()` - soft delete user from company
- `getRolesDropdownItems()` - company roles dropdown list

**Validation Features:**
- Global email uniqueness (not just within company)
- Password validation only on creation
- Custom validators for language, timezone, password
- Role validation within company
- Company access control through selectedUserCompany

---

### 2. CompanyUserSearchForm

**File:** `src/apps/admin/forms/company_users/CompanyUserSearchForm.php`  
**Extends:** `customs\Model` (185 lines)

**Public Properties:**
```php
public mixed $company_uuid = null;    // Company UUID (required)
public mixed $email = null;           // Search by user email
public mixed $name = null;            // Search by name (CONCAT first_name + last_name)
public mixed $role = null;            // Filter by role
public mixed $status = null;          // Filter by status
public mixed $language = null;        // Filter by language
public mixed $timezone = null;        // Filter by timezone
public mixed $is_admin = false;       // Administrator filter
public mixed $updated_at = null;      // Filter by update date (array)
```

**Search Features:**
- **Company Isolation:** Automatic filtering by selectedUserCompany
- **JOIN Operations:** innerJoinWith(['role', 'user']) for related data access
- **Complex Name Search:** CONCAT expressions for full name search
- **Role-based Filtering:** filter by company roles
- **Personal Settings Filtering:** language, timezone, admin status
- **Date Range Filtering:** updated_at with timestamp validation

---

### 3. MigrateCompanyUserForm

**File:** `src/apps/admin/forms/company_users/MigrateCompanyUserForm.php`  
**Extends:** `customs\Model` (181 lines)

**Public Properties:**
```php
public mixed $user_uuid = null;       // User UUID for migration (required)
public mixed $company_uuid = null;    // Target company UUID (required)
public mixed $role_slug = null;       // Role slug in target company (required)
```

**Migration Features:**
- **Cross-Company User Transfer:** transfer user between companies
- **Role Assignment:** assign new role in target company
- **Duplicate Prevention:** automatic filtering of existing company users
- **Transaction Safety:** full transactional security
- **UUID Generation:** generate new UUIDs for company user records

**Key Methods:**
```php
migrate(): CompanyUsersAdvanced|static
// Main migration logic with transaction safety

getUsersForMigration(): array
// Returns users available for migration (not already in target company)

getRolesDropdownItems(): array
// Returns roles available in target company for dropdown
```

---

## Security Features

### 1. Company-based Access Control
- All operations isolated by selectedUserCompany
- User search results limited to user's company
- Role validation only within respective company
- Migration validation with cross-company access control

### 2. Global Email Uniqueness
- Email validation at UsersAdvanced level (globally)
- Prevention of duplicate emails in system
- Proper exclusion of current user in UPDATE operations
- Email format validation

### 3. Role-based Security
- Role validation within company
- Role assignment through role_id with existence validation
- Migration role validation in target company
- Proper role dropdown generation with access control

### 4. Password Security
- PasswordValidator for complexity requirements
- Password required only for CREATE operations
- Optional password updates for UPDATE operations
- Secure password handling

### 5. Migration Security
- Source user validation
- Target company access validation
- Duplicate prevention (user already in target company)
- Transaction safety for consistency
- UUID validation for all migration parameters

---

## Usage Examples

### Create company user
```http
POST /company-users/create?company_uuid=550e8400-e29b-41d4-a716-446655440000
Content-Type: application/x-www-form-urlencoded

SaveCompanyUserForm[first_name]=John&SaveCompanyUserForm[last_name]=Doe&SaveCompanyUserForm[email]=john.doe@example.com&SaveCompanyUserForm[password]=SecurePass123&SaveCompanyUserForm[role_id]=5&SaveCompanyUserForm[language]=en&SaveCompanyUserForm[timezone]=UTC
```

### Search users with filtering
```http
GET /company-users/index?company_uuid=550e8400-e29b-41d4-a716-446655440000&name=John&role=manager&status=1&language=en
```

### Migrate user between companies
```http
POST /company-users/migrate?company_uuid=550e8400-e29b-41d4-a716-446655440000
Content-Type: application/x-www-form-urlencoded

MigrateCompanyUserForm[user_uuid]=550e8400-e29b-41d4-a716-446655440001&MigrateCompanyUserForm[role_slug]=manager
```

### Update user
```http
POST /company-users/update?uuid=550e8400-e29b-41d4-a716-446655440002&company_uuid=550e8400-e29b-41d4-a716-446655440000
Content-Type: application/x-www-form-urlencoded

SaveCompanyUserForm[first_name]=Jane&SaveCompanyUserForm[last_name]=Smith&SaveCompanyUserForm[email]=jane.smith@example.com&SaveCompanyUserForm[role_id]=3&SaveCompanyUserForm[timezone]=Europe/Kiev
```

---

## Error Handling

### Common Errors
- **"User not found"** - Invalid user UUID or access denied
- **"Role not found"** - Invalid role_id or role not in company
- **"Company not found"** - Invalid company_uuid or access denied
- **Email uniqueness violations** - Duplicate email in system

### Migration Errors
- **"Cannot attach user to company"** - Migration save failure
- **"System error, please contact with Administrator!"** - Transaction or system failures
- Source user doesn't exist or unavailable
- Target role doesn't exist in target company

### HTTP Status Codes
- **200** - Success responses
- **404** - NotFoundHttpException for invalid company_uuid
- **500** - System errors with user-friendly messages
- **Redirect** - Success operations redirect to appropriate pages

---

*CompanyUsersController is a central component of user management system with support for multi-company architecture, role-based access control, comprehensive personal settings management, and unique cross-company migration capabilities.*