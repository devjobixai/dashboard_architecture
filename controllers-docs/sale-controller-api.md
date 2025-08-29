# SaleController API Documentation

## Overview

SaleController manages purchase event processing for the Jobix Dashboard API. The controller provides focused functionality for handling sales transactions and purchase events within the system. It serves as the primary endpoint for recording and processing customer purchases and sales-related activities.

**Base Path:** `/api/v1/sale`  
**Controller Type:** ApiController (API Application)  
**Namespace:** `apps\api\versions\v1\controllers\SaleController`  
**Authentication:** Requires `company-key` header

---

## Endpoints

### 1. Purchase - Process Purchase Event

| Parameter | Value |
|-----------|-------|
| **URL** | `/api/v1/sale/purchase` |
| **Method** | POST |
| **Description** | Process customer purchase events and record sales transactions |

**Input Data (PurchaseEventForm bodyParams):**
- Purchase transaction details and metadata
- Customer information and context
- Product or service purchase data
- Payment and billing information

**Business Logic:**
1. Load purchase event data from request body
2. Validate purchase transaction parameters
3. Process purchase event through business logic
4. Record transaction and update relevant systems
5. Return purchase processing results

**Response:**
- **Success (200):** Array with purchase processing results and transaction details
- **Error (422):** PurchaseEventForm object with validation errors

**Purchase Processing Features:**
- Transaction validation and recording
- Customer context integration
- Purchase event tracking
- Sales analytics integration

---

## Form Classes

### 1. PurchaseEventForm

**File:** `src/apps/api/versions/v1/forms/sales/PurchaseEventForm.php`  
**Extends:** `customs\Model`

**Purpose:** Handle purchase event processing and validation
**Key Methods:**
- `process()` - Process purchase event and return results

**Features:**
- Purchase transaction validation
- Customer data integration
- Event recording and tracking
- Business logic processing
- Error handling and validation

**Load Method:**
- Uses `load()` with empty prefix for direct parameter mapping
- Processes bodyParams directly without form prefix

---

## Business Logic

### 1. Purchase Event Processing Flow
```
1. Purchase Event Reception
   - Load purchase data from request
   - Validate transaction parameters
   - Check customer context and permissions
   - Verify purchase data integrity

2. Transaction Processing
   - Apply business rules and validation
   - Process payment and billing information
   - Update customer records and history
   - Record transaction in sales system

3. Event Recording and Analytics
   - Log purchase event for tracking
   - Update sales statistics and metrics
   - Trigger related workflows or notifications
   - Return processing confirmation
```

### 2. Integration Flow
```
1. Customer Integration
   - Link purchase to customer record
   - Update customer purchase history
   - Apply customer-specific business rules
   - Handle customer context validation

2. Sales System Integration
   - Record transaction in sales database
   - Update inventory and product data
   - Process commission and analytics
   - Generate sales reports and metrics

3. Event Tracking
   - Log purchase event for audit trail
   - Update system analytics and dashboards
   - Trigger automated workflows
   - Send notifications and confirmations
```

---

## Security Features

### 1. Company-based Access Control
- All purchase operations filtered by company context
- Customer validation and access control
- Transaction security and integrity
- Purchase data privacy protection

### 2. Input Validation
- Purchase parameter validation and sanitization
- Customer data verification
- Transaction amount and detail validation
- Fraud detection and prevention measures

### 3. Transaction Security
- Secure purchase data handling
- Payment information protection
- Audit logging for all transactions
- Error handling with security considerations

---

## Integration Points

### Dependencies
- `PurchaseEventForm` - Purchase event processing form
- Company authentication system
- Customer management system
- Sales and analytics tracking
- Payment processing integration

### Related Components
- **Customer Management** - Customer data and purchase history
- **Sales Analytics** - Transaction tracking and reporting
- **Payment Systems** - Payment processing and validation
- **Workflow Engine** - Automated purchase-related workflows

### External Integrations
- Payment processing services
- Sales analytics platforms
- Customer relationship management (CRM) systems
- Inventory management systems

---

## Usage Examples

### Process purchase event
```http
POST /api/v1/sale/purchase
Content-Type: application/json
company-key: your-company-api-key

{
  "customer_id": "550e8400-e29b-41d4-a716-446655440000",
  "product_id": "product-123",
  "quantity": 2,
  "amount": 199.99,
  "currency": "USD",
  "transaction_id": "txn-789",
  "payment_method": "credit_card",
  "metadata": {
    "source": "website",
    "campaign": "summer_sale"
  }
}
```

### Process subscription purchase
```http
POST /api/v1/sale/purchase
Content-Type: application/json
company-key: your-company-api-key

{
  "customer_id": "550e8400-e29b-41d4-a716-446655440000",
  "subscription_plan": "premium",
  "billing_cycle": "monthly",
  "amount": 29.99,
  "currency": "USD",
  "payment_method": "stripe",
  "start_date": "2024-01-01",
  "metadata": {
    "upgrade_from": "basic",
    "promo_code": "SAVE20"
  }
}
```

### Process service purchase
```http
POST /api/v1/sale/purchase
Content-Type: application/json
company-key: your-company-api-key

{
  "customer_id": "550e8400-e29b-41d4-a716-446655440000",
  "service_type": "consulting",
  "hours": 10,
  "rate": 150.00,
  "amount": 1500.00,
  "currency": "USD",
  "project_id": "proj-456",
  "metadata": {
    "consultant": "John Doe",
    "contract": "CON-2024-001"
  }
}
```

---

## Error Handling

### Common Errors
- **"Invalid customer ID"** - Customer not found or access denied
- **"Purchase validation failed"** - Invalid purchase parameters
- **"Payment processing error"** - Payment system integration failure
- **"Insufficient permissions"** - Company access or customer permissions

### Purchase Processing Errors
- Transaction amount validation failures
- Customer context validation errors
- Payment method verification issues
- Business rule validation failures

### Integration Errors
- Payment processor connectivity issues
- Customer data synchronization failures
- Sales system recording errors
- Analytics tracking failures

### HTTP Status Codes
- **200 OK** - Successful purchase processing
- **422 Unprocessable Entity** - Validation errors or business logic failures

---

## Performance Considerations

### Transaction Processing
- Efficient purchase event handling
- Optimized database operations
- Minimal processing latency
- Scalable transaction recording

### Integration Optimization
- Asynchronous analytics processing
- Cached customer data lookup
- Batched sales system updates
- Optimized external API calls

### Monitoring and Analytics
- Purchase event tracking
- Performance metrics collection
- Error rate monitoring
- Transaction success rates

---

*SaleController API provides streamlined purchase event processing with comprehensive transaction handling, customer integration, and sales analytics support for enterprise commerce operations.*