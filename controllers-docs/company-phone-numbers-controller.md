# CompanyPhoneNumbersController Documentation

## Overview

CompanyPhoneNumbersController manages company phone numbers in the Jobix Dashboard admin panel, including listing, creating, deleting, and assigning numbers to agents.

Base path: `/company-phone-numbers`  
Controller type: WebController (Admin Application)  
Namespace: `apps\\admin\\controllers\\CompanyPhoneNumbersController`

---

## Endpoints

### 1. Index — Numbers List

| Param | Value |
|------|-------|
| URL | `/company-phone-numbers/index?company_uuid={company_uuid}` |
| Method | GET |
| Description | List all phone numbers for the company with filters |

### 2. Create — Add Number

| Param | Value |
|------|-------|
| URL | `/company-phone-numbers/create?company_uuid={company_uuid}` |
| Methods | GET, POST, AJAX |
| Description | Add a new phone number for the company |

Input (CreatePhoneNumberForm):
- Provider, number, country, capabilities, status

### 3. Delete — Remove Number

| Param | Value |
|------|-------|
| URL | `/company-phone-numbers/delete?company_uuid={company_uuid}&uuid={number_uuid}` |
| Method | POST |
| Description | Remove a phone number from the company |

### 4. Assign — Assign Number to Agent

| Param | Value |
|------|-------|
| URL | `/company-phone-numbers/assign` |
| Method | POST |
| Description | Assign a phone number to a specific agent |

Input (AssignPhoneNumberForm):
- `company_uuid`, `number_uuid`, `agent_uuid`

---

## Form Classes

### CreatePhoneNumberForm
- Validates provider, number format, and uniqueness within a company

### DeletePhoneNumberForm
- Validates existence and safe removal

### AssignPhoneNumberForm
- Validates relations: number ↔ company, agent ↔ company

---

## Security Features
- Company-based access control
- UUID validation for all identifiers
- Provider capability checks

---

## Usage Examples

Create number
```http
POST /company-phone-numbers/create?company_uuid=c-uuid
Content-Type: application/x-www-form-urlencoded

provider=twilio&number=+12025550123&country=US&status=1
```

Assign number
```http
POST /company-phone-numbers/assign
Content-Type: application/json

{"company_uuid":"c-uuid","number_uuid":"n-uuid","agent_uuid":"a-uuid"}
```

