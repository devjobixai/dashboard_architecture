# TestController API Documentation

## Overview

TestController provides basic testing and connectivity functionality for the Jobix Dashboard API. The controller offers minimal endpoints for API health checking, connectivity testing, and basic system status verification. It serves as a simple utility for monitoring API availability and response times.

**Base Path:** `/api/v1/test`  
**Controller Type:** ApiController (API Application)  
**Namespace:** `apps\api\versions\v1\controllers\TestController`  
**Authentication:** Requires `company-key` header

---

## Endpoints

### 1. Ping - API Connectivity Test

| Parameter | Value |
|-----------|-------|
| **URL** | `/api/v1/test/ping` |
| **Method** | GET |
| **Description** | Test API connectivity and get current server timestamp |

**Input Data:** None (simple connectivity test)

**Business Logic:**
1. Generate current timestamp using Carbon library
2. Return timestamp as integer value
3. Provide immediate response for connectivity testing

**Response:**
- **Success (200):** Integer timestamp representing current server time

**Ping Features:**
- Immediate response for connectivity testing
- Current server timestamp for time synchronization
- Minimal processing overhead for quick response
- System availability verification

---

## Dependencies

### 1. Carbon

**Library:** `Carbon\Carbon`  
**Purpose:** Date and time manipulation library
**Usage:** Generate current timestamp for ping response

**Features:**
- Precise timestamp generation
- Time zone handling and conversion
- Date formatting and manipulation
- Cross-platform compatibility

### 2. TimeHelper

**Helper:** `helpers\TimeHelper`  
**Purpose:** Time-related utility functions (imported but not used in current implementation)
**Potential Usage:** Extended time functionality and formatting

---

## Business Logic

### 1. Connectivity Testing Flow
```
1. Ping Request Processing
   - Receive ping request from client
   - Generate current server timestamp
   - Return timestamp immediately
   - Provide connectivity confirmation

2. Health Check Functionality
   - Verify API availability
   - Test response time and latency
   - Confirm server time synchronization
   - Monitor system responsiveness

3. System Status Verification
   - Basic API endpoint accessibility
   - Server time and timezone validation
   - Network connectivity confirmation
   - Service availability testing
```

---

## Security Features

### 1. Company-based Access Control
- Ping endpoint filtered by company authentication
- Basic access validation through company-key header
- Minimal exposure of system information

### 2. Information Security
- Limited system information disclosure
- Timestamp-only response data
- No sensitive system details exposed
- Secure connectivity testing

---

## Integration Points

### Dependencies
- `Carbon` - Date and time manipulation library
- `TimeHelper` - Time utility functions
- Company authentication system
- API controller framework

### Related Components
- **API Monitoring** - Health check and availability monitoring
- **System Diagnostics** - Basic system status verification
- **Load Balancing** - Health check endpoint for load balancers
- **Monitoring Services** - External monitoring system integration

---

## Usage Examples

### Basic ping test
```http
GET /api/v1/test/ping
company-key: your-company-api-key
```

**Response:**
```
1704067200
```

### Connectivity verification with curl
```bash
curl -H "company-key: your-company-api-key" \
     https://api.example.com/api/v1/test/ping
```

### Health check monitoring script
```javascript
// JavaScript example for monitoring
async function checkAPIHealth() {
    try {
        const response = await fetch('/api/v1/test/ping', {
            headers: {
                'company-key': 'your-company-api-key'
            }
        });
        
        if (response.ok) {
            const timestamp = await response.text();
            console.log('API is healthy, server time:', new Date(timestamp * 1000));
            return true;
        }
    } catch (error) {
        console.error('API health check failed:', error);
        return false;
    }
}
```

### Load balancer health check
```yaml
# Example load balancer configuration
health_check:
  path: "/api/v1/test/ping"
  headers:
    company-key: "your-company-api-key"
  interval: 30s
  timeout: 5s
  healthy_threshold: 2
  unhealthy_threshold: 3
```

---

## Error Handling

### Common Errors
- **Authentication failure** - Invalid or missing company-key header
- **Server unavailability** - API service down or unreachable
- **Network connectivity** - Network issues preventing response

### HTTP Status Codes
- **200 OK** - Successful ping response with timestamp
- **401 Unauthorized** - Invalid authentication credentials
- **500 Internal Server Error** - Server-side processing errors

---

## Performance Considerations

### Response Time
- Minimal processing overhead for immediate response
- Lightweight timestamp generation
- No database queries or external API calls
- Optimized for low latency testing

### Monitoring Integration
- Suitable for high-frequency health checks
- Minimal resource consumption
- Fast response for load balancer integration
- Scalable for monitoring systems

### System Load
- Negligible impact on system performance
- No resource-intensive operations
- Efficient timestamp generation
- Minimal memory and CPU usage

---

## Use Cases

### 1. API Health Monitoring
- Automated health check systems
- Load balancer backend verification
- Service availability monitoring
- Response time measurement

### 2. Connectivity Testing
- Network connectivity verification
- API endpoint accessibility testing
- Basic functionality confirmation
- Service discovery validation

### 3. Time Synchronization
- Server time verification
- Timestamp synchronization testing
- Time zone validation
- Clock drift detection

### 4. Development and Debugging
- Quick API connectivity testing during development
- Basic endpoint functionality verification
- Network configuration validation
- Service deployment confirmation

---

*TestController API provides essential connectivity testing and health check functionality with minimal overhead for reliable API monitoring and system verification.*