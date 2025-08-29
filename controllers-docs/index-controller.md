# IndexController Documentation

## Overview

IndexController serves as the main entry point and error handler for the Jobix Dashboard administrative panel. The controller provides routing functionality to redirect users to the main dashboard and handles application-wide error responses with appropriate layouts and user-friendly messages.

**Base Path:** `/` (root)  
**Controller Type:** WebController (Admin Application)  
**Namespace:** `apps\admin\controllers\IndexController`

---

## Endpoints

### 1. Index - Main Entry Point

| Parameter | Value |
|-----------|-------|
| **URL** | `/` or `/index/index` |
| **Method** | GET |
| **Description** | Main application entry point that redirects to dashboard |

**Business Logic:**
1. Receive request to application root
2. Redirect to `/dashboard/index` using Yii URL routing
3. No data processing or validation required
4. Simple routing mechanism

**Implementation:**
```php
public function actionIndex()
{
    return $this->redirect(Url::toRoute(['/dashboard/index']));
}
```

**Response:**
- **Type:** HTTP Redirect (302)
- **Location:** `/dashboard/index`
- **Purpose:** Ensures users accessing root URL are directed to main dashboard

---

### 2. Error - Application Error Handler

| Parameter | Value |
|-----------|-------|
| **URL** | `/index/error` |
| **Method** | GET |
| **Description** | Handle application errors with user-friendly error pages |

**Business Logic:**
1. Set layout to 'guest' for clean error display
2. Retrieve exception from Yii error handler
3. Check exception type for specific handling
4. Render appropriate error view with custom messages
5. Provide user-friendly error information

**Implementation:**
```php
public function actionError()
{
    $this->layout = 'guest';
    
    $exception = Yii::$app->errorHandler->exception;
    
    if ($exception instanceof ForbiddenHttpException) {
        return $this->render('error', [
            'name' => 'STOP! No Access!',
            'message' => $exception->getMessage()
        ]);
    }
    
    return $this->render('error', [
        'name' => 'Page not found',
        'message' => $exception->getMessage()
    ]);
}
```

**Error Types Handled:**

#### ForbiddenHttpException (403 Errors)
- **Name:** "STOP! No Access!"
- **Message:** Exception message from the system
- **Use Case:** Access denied, insufficient permissions
- **Layout:** Guest layout (minimal UI)

#### General Exceptions (404 and Other Errors)
- **Name:** "Page not found"
- **Message:** Exception message from the system
- **Use Case:** Page not found, general application errors
- **Layout:** Guest layout (minimal UI)

**Response:**
- **Success:** HTML error page with appropriate layout
- **Layout:** 'guest' layout for clean, minimal error display
- **Data:** `name` (error title), `message` (error description)

---

## Layout Management

### Guest Layout Usage
- **Purpose:** Provides clean, minimal interface for error pages
- **Features:** No navigation, simplified styling, focused on error message
- **Benefits:** Reduces visual clutter during error states
- **Usage:** Applied automatically for all error responses

---

## Business Logic

### 1. Root Access Flow
```
1. User accesses application root URL (/)
2. IndexController actionIndex() receives request
3. Generate redirect URL to /dashboard/index
4. Return HTTP 302 redirect response
5. Browser automatically navigates to dashboard
```

### 2. Error Handling Flow
```
1. Application encounters error/exception
2. Yii error handler routes to /index/error
3. Set guest layout for minimal error display
4. Retrieve exception from Yii::$app->errorHandler
5. Check exception type (ForbiddenHttpException vs others)
6. Set appropriate error title and message
7. Render error view with exception details
8. Display user-friendly error page
```

---

## Security Features

### 1. Error Information Control
- Controlled error message exposure
- User-friendly error titles instead of technical details
- Exception message filtering through Yii framework
- No sensitive system information leaked

### 2. Access Control Integration
- Special handling for ForbiddenHttpException
- Clear access denied messages
- Integration with application-wide security
- Proper error classification

### 3. Layout Security
- Guest layout prevents access to admin navigation
- Minimal interface reduces attack surface
- Clean error display without system details
- Safe error handling for unauthenticated users

---

## Integration Points

### Dependencies
- Yii Framework routing system (`Url::toRoute()`)
- Yii error handling system (`Yii::$app->errorHandler`)
- WebController base class functionality
- Exception handling classes (`ForbiddenHttpException`)

### Related Components
- **DashboardController** - main dashboard destination
- **Authentication System** - generates ForbiddenHttpException
- **Error Handler** - provides exception details
- **Layout System** - manages guest layout rendering

### Framework Integration
- Integrates with Yii2 error handling pipeline
- Uses standard Yii routing mechanisms
- Follows Yii MVC patterns for error display
- Compatible with Yii exception system

---

## Usage Examples

### Access application root
```http
GET /
```
**Response:** 302 Redirect to `/dashboard/index`

### Handle 403 Forbidden error
```http
GET /index/error
```
**Response:** HTML error page with "STOP! No Access!" title

### Handle 404 Not Found error
```http
GET /index/error
```
**Response:** HTML error page with "Page not found" title

### Direct error page access
```http
GET /index/error
```
**Response:** Error page based on current exception context

---

## Error Page Structure

### Error View Data
```php
[
    'name' => 'Error Title',      // User-friendly error title
    'message' => 'Error Details'  // Exception message
]
```

### Layout Features
- **Guest Layout:** Clean, minimal design
- **No Navigation:** Removes admin menu and controls
- **Focused Content:** Emphasizes error message
- **User-Friendly:** Non-technical error presentation

---

## Performance Considerations

### Routing Efficiency
- Simple redirect with minimal processing
- No database queries or complex logic
- Efficient URL generation using Yii routing
- Fast response for root access requests

### Error Handling Performance
- Minimal processing overhead for error pages
- Efficient exception type checking
- Fast layout switching to guest mode
- No unnecessary resource loading for errors

### Caching Considerations
- Error pages should not be cached
- Redirect responses can be cached safely
- Guest layout optimized for quick rendering
- Minimal resource requirements

---

## HTTP Status Codes

### Successful Operations
- **302 Found** - Redirect from root to dashboard

### Error Responses
- **403 Forbidden** - ForbiddenHttpException handling
- **404 Not Found** - General error handling
- **500 Internal Server Error** - System errors

---

## Configuration

### URL Rules
- Root URL (`/`) maps to `index/index`
- Error handling URL (`/index/error`) configured in error handler
- Redirect target configurable through route parameters

### Layout Configuration
- Guest layout path: `@app/views/layouts/guest.php`
- Error view path: `@app/views/index/error.php`
- Layout switching handled automatically

---

*IndexController is a fundamental component providing essential routing and error handling functionality with clean, user-friendly error pages and efficient application entry point management.*