# AuthController Documentation

## Overview

AuthController handles user authentication for the Jobix Dashboard admin panel.

Base path: `/auth`  
Controller type: WebController (Admin Application)  
Namespace: `apps\admin\controllers\AuthController`

---

## Endpoints

### 1. Login — User Sign-in

| Param | Value |
|-------|-------|
| URL | `/auth/login` |
| Methods | GET, POST |
| Description | Authenticate a user in the system |
| Layout | `auth` |

#### GET `/auth/login`
Purpose: Display the login form

Behavior:
- If the user is already authenticated — redirect to the homepage
- Otherwise — render the login form

Response:
- Success: HTML view with the login form
- Redirect: To the homepage (if the user is already authenticated)

#### POST `/auth/login`
Purpose: Process login data

Input (LoginForm):

| Field | Type | Required | Validation | Description |
|-------|------|----------|------------|-------------|
| `email` | string | ✓ | required, string, max:255, trim | User email |
| `password` | string | ✓ | required, string, max:255, custom validation | User password |
| `remember_me` | boolean | ✗ | boolean | Remember user (default: true) |

Detailed validation:

```php
// Validation Rules
[['email', 'password'], 'required']
[['email', 'password'], 'string', 'max' => 255]
[['email'], 'trim']
[['remember_me'], 'boolean']
[['password'], 'validatePassword'] // Custom validation
```

Custom Password Validation:
- Checks if a user exists by email
- Validates the password via the User model
- On error adds: "Incorrect email or password"

Authentication business logic:
1. Check user tokens
2. Generate/extend token
3. Set `access_token` cookie
4. Log in the user to Yii session

Token Management:
- remember_me = true: `User::TOKEN_ADVANCED_EXPIRE_HOURS`
- remember_me = false: `User::TOKEN_BASE_EXPIRE_HOURS`

Cookie Settings:
```php
[
    'name' => 'access_token',
    'value' => $user->access_token,
    'httpOnly' => false,
    'secure' => false,
    'expire' => $user->access_token_expired_at
]
```

Responses:
- Success: Redirect to `/dashboard/index`
- Validation Error: HTML view with form and errors
- System Error: "Something happened when you try login. Please contact with administrator!"

---

### 2. Logout — User Sign-out

| Param | Value |
|-------|-------|
| URL | `/auth/logout` |
| Method | GET |
| Description | Terminate the user session |

Input: None

Business logic:
1. Call `Yii::$app->user->logout()`
2. Remove the `access_token` cookie
3. Redirect to the login form

Response:
- Success: Redirect to `/auth/login`

---

## Form Classes

### LoginForm

File: `src/apps/admin/forms/auth/LoginForm.php`  
Extends: `yii\base\Model`

Public properties:
```php
public null|string $email = null;
public null|string $password = null;
public bool $remember_me = true;
```

Private properties:
```php
private User|null $_user = null; // Cached user instance
```

Methods:
- `rules()`: Validation rules
- `validatePassword()`: Custom password validation
- `login()`: Authentication logic
- `getUser()`: Retrieve User model

Labels:
```php
'email' => 'Email',
'password' => 'Password',
'remember_me' => 'Remember Me'
```

---

## Error Handling

### Validation Errors
- Required fields: Standard Yii validation messages
- Invalid credentials: "Incorrect email or password"
- System error: "Something happened when you try login. Please contact with administrator!"

### System Errors
- Database exceptions are logged to the `system_error` category
- User-friendly error messages

---

## Security Features

1. Token Management: Automatic token generation and extension
2. Cookie Security: Access token stored in a cookie
3. Password Validation: Via User model with protected logic
4. Session Management: Integrated with Yii User component
5. Error Logging: System errors logged for monitoring

---

## Integration Points

### Dependencies
- `User` model — authentication and tokens
- `TimeHelper` — token expiry calculations
- Yii User component — session management
- Cookie component — token storage

### Related Controllers
- `DashboardController` — redirect destination after login
- All other admin controllers — depend on successful authentication

---

## Usage Examples

### Successful login
```http
POST /auth/login
Content-Type: application/x-www-form-urlencoded

email=user@example.com&password=userpassword&remember_me=1
```

Response: `302 Redirect` to `/dashboard/index`

### Invalid credentials
```http
POST /auth/login
Content-Type: application/x-www-form-urlencoded

email=user@example.com&password=wrongpassword
```

Response: `200 OK` with HTML form and validation error

