# DashboardController Documentation

## Overview

DashboardController is responsible for the main control panel in the Jobix Dashboard admin area. It is the primary controller that provides access to the main page after authentication.

Base path: `/dashboard`  
Controller type: WebController (Admin Application)  
Namespace: `apps\\admin\\controllers\\DashboardController`

---

## Endpoints

### 1. Index — Dashboard

| Param | Value |
|-------|-------|
| URL | `/dashboard/index` |
| Method | GET |
| Description | Display the main dashboard with company analytics |

Input: None (GET request without parameters)

Business logic:

1. Company selection check:
   - Calls `\\Yii::$app->user->identity->isCompanySet()`
   - Verifies that the user has selected an active company to work with

2. Conditional routing:
   - If no company is selected: Redirect to `/companies/index`
   - If a company is selected: Render the dashboard view

3. Data passing:
   - Passes `company_id` of the currently selected company to the view
   - Uses `$this->selectedUserCompany->id` to get the ID

Responses:

#### Success (Company Selected):
- HTTP Status: 200 OK
- Content-Type: text/html
- Body: HTML view with the dashboard interface
- Data: `company_id` — the active company ID of the user

#### Redirect (No Company Selected):
- HTTP Status: 302 Found
- Location: `/companies/index`
- Purpose: The user must select a company first

---

## Business Logic

### Company Selection Flow
```
1. User navigates to /dashboard/index
2. System checks if the user has selected a company
3a. IF company selected → Show dashboard with company data
3b. IF no company selected → Redirect to company selection page
```

### User Identity Integration
- Uses `Yii::$app->user->identity` to check user status
- Method `isCompanySet()` checks the presence of a selected company
- Accesses `selectedUserCompany` to get current company data

---

## Security Features

### 1. Authentication Check
- Inherits base authentication from `WebController`
- Requires an authenticated user for access

### 2. Company Access Control
- Automatic check for selected company
- Redirect to company selection if none is active
- Data isolation by `company_id`

### 3. User Session Validation
- Integrated with the Yii User component
- Validates the user identity

---

## Integration Points

### Dependencies
- `WebController` — base class with authentication logic
- `Yii::$app->user` — user management component
- `Url::toRoute()` — helper for URL generation

### Related Controllers
- `CompaniesController` — for selecting/managing companies
- `AuthController` — for initial authentication

### View Integration
- `index` view — main dashboard UI
- Passes `company_id` for content personalization

---

## Usage Examples

### Successful dashboard access
```http
GET /dashboard/index
Cookie: session_cookie=value

Response: 200 OK
Content-Type: text/html

[HTML content of dashboard with company-specific data]
```

### Redirect when no company selected
```http
GET /dashboard/index
Cookie: session_cookie=value

Response: 302 Found
Location: /companies/index
```

---

## Error Handling

### Automatic Redirects
- No Company Selected: Automatic redirect to `/companies/index`
- Not Authenticated: Handled by WebController (redirect to login)

### Potential Issues
- Company Access Rights: May require additional permission checks
- Company Deletion: If the selected company was deleted

---

## Performance Considerations

### Lightweight Controller
- Minimal business logic
- Quick company status check
- Efficient usage of Yii caching mechanisms

### View Optimization
- Company-specific data loading in the view layer
- Potential for lazy loading heavy analytics

---

## Maintenance Notes

### Extension Points
- Add analytics widgets to the dashboard view
- Integrate with reporting systems
- Real-time data updates via WebSocket

