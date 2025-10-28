# Error Handling Guide

This document provides a comprehensive guide to error handling in the Library Management System API.

## Table of Contents
1. [Error Response Format](#error-response-format)
2. [HTTP Status Codes](#http-status-codes)
3. [Error Codes](#error-codes)
4. [Error Scenarios by Category](#error-scenarios-by-category)
5. [Best Practices for Error Handling](#best-practices-for-error-handling)

---

## Error Response Format

All error responses follow a consistent JSON structure:

```json
{
  "error": {
    "code": "ERROR_CODE",
    "message": "Human-readable error message",
    "details": [
      {
        "field": "fieldName",
        "issue": "Description of the issue with this field"
      }
    ],
    "requestId": "req_unique_id",
    "timestamp": "2024-10-28T10:30:00Z",
    "documentation": "https://api.library.example.com/docs/errors/ERROR_CODE"
  }
}
```

### Field Descriptions

- **code** (string, required): Machine-readable error code for programmatic handling
- **message** (string, required): Human-readable error description
- **details** (array, optional): Detailed information about specific field errors
- **requestId** (string, required): Unique identifier for this request (for debugging)
- **timestamp** (string, required): ISO 8601 timestamp of when the error occurred
- **documentation** (string, optional): URL to error documentation

---

## HTTP Status Codes

### 2xx Success
| Code | Description | Usage |
|------|-------------|-------|
| 200 OK | Success | Successful GET, PATCH, PUT requests |
| 201 Created | Resource created | Successful POST creating a resource |
| 204 No Content | Success, no content | Successful DELETE operations |

### 4xx Client Errors
| Code | Description | Usage |
|------|-------------|-------|
| 400 Bad Request | Invalid request | Malformed request, invalid parameters |
| 401 Unauthorized | Authentication required | Missing or invalid authentication token |
| 403 Forbidden | Access denied | Authenticated but insufficient permissions |
| 404 Not Found | Resource not found | Requested resource doesn't exist |
| 409 Conflict | Request conflict | Operation conflicts with current state |
| 422 Unprocessable Entity | Validation error | Request semantically invalid |
| 429 Too Many Requests | Rate limit exceeded | Too many requests in time window |

### 5xx Server Errors
| Code | Description | Usage |
|------|-------------|-------|
| 500 Internal Server Error | Server error | Unexpected server-side error |
| 503 Service Unavailable | Service unavailable | Temporary unavailability (maintenance, etc.) |

---

## Error Codes

### Authentication Errors (AUTH_*)

#### AUTH_TOKEN_MISSING
**Status:** 401 Unauthorized

No authentication token was provided in the request.

**Example:**
```json
{
  "error": {
    "code": "AUTH_TOKEN_MISSING",
    "message": "Authentication token is required",
    "details": [
      {
        "field": "Authorization",
        "issue": "Missing Authorization header"
      }
    ],
    "requestId": "req_001",
    "timestamp": "2024-10-28T10:30:00Z"
  }
}
```

**Resolution:** Include the Authorization header: `Authorization: Bearer {token}`

---

#### AUTH_TOKEN_INVALID
**Status:** 401 Unauthorized

The provided authentication token is invalid or malformed.

**Example:**
```json
{
  "error": {
    "code": "AUTH_TOKEN_INVALID",
    "message": "The provided authentication token is invalid",
    "details": [
      {
        "field": "Authorization",
        "issue": "Token signature verification failed"
      }
    ],
    "requestId": "req_002",
    "timestamp": "2024-10-28T10:35:00Z"
  }
}
```

**Resolution:** Obtain a new token via `/v1/auth/login`

---

#### AUTH_TOKEN_EXPIRED
**Status:** 401 Unauthorized

The authentication token has expired.

**Example:**
```json
{
  "error": {
    "code": "AUTH_TOKEN_EXPIRED",
    "message": "Authentication token has expired",
    "details": [
      {
        "field": "Authorization",
        "issue": "Token expired at 2024-10-28T09:30:00Z"
      }
    ],
    "requestId": "req_003",
    "timestamp": "2024-10-28T10:40:00Z"
  }
}
```

**Resolution:** Use refresh token to obtain a new access token

---

### Authorization Errors (PERMISSION_*)

#### INSUFFICIENT_PERMISSIONS
**Status:** 403 Forbidden

The authenticated user lacks the required permissions.

**Example:**
```json
{
  "error": {
    "code": "INSUFFICIENT_PERMISSIONS",
    "message": "You do not have permission to perform this action",
    "details": [
      {
        "field": "role",
        "issue": "This operation requires 'librarian' or 'admin' role. Current role: 'member'"
      }
    ],
    "requestId": "req_004",
    "timestamp": "2024-10-28T10:45:00Z",
    "documentation": "https://api.library.example.com/docs/permissions"
  }
}
```

**Resolution:** Contact an administrator to adjust permissions

---

#### RESOURCE_ACCESS_DENIED
**Status:** 403 Forbidden

Access to this specific resource is denied.

**Example:**
```json
{
  "error": {
    "code": "RESOURCE_ACCESS_DENIED",
    "message": "You cannot access this resource",
    "details": [
      {
        "field": "memberId",
        "issue": "You can only access your own member profile"
      }
    ],
    "requestId": "req_005",
    "timestamp": "2024-10-28T10:50:00Z"
  }
}
```

---

### Resource Errors (RESOURCE_*)

#### RESOURCE_NOT_FOUND
**Status:** 404 Not Found

The requested resource does not exist.

**Example:**
```json
{
  "error": {
    "code": "RESOURCE_NOT_FOUND",
    "message": "The requested book was not found",
    "details": [
      {
        "field": "bookId",
        "issue": "Book with ID 'book_999' does not exist"
      }
    ],
    "requestId": "req_006",
    "timestamp": "2024-10-28T11:00:00Z",
    "documentation": "https://api.library.example.com/docs/errors/RESOURCE_NOT_FOUND"
  }
}
```

**Resolution:** Verify the resource ID is correct

---

#### RESOURCE_ALREADY_EXISTS
**Status:** 400 Bad Request

Cannot create resource because it already exists.

**Example:**
```json
{
  "error": {
    "code": "RESOURCE_ALREADY_EXISTS",
    "message": "A book with this ISBN already exists",
    "details": [
      {
        "field": "isbn",
        "issue": "ISBN '978-0-123456-78-9' is already in the system (book_123)"
      }
    ],
    "requestId": "req_007",
    "timestamp": "2024-10-28T11:05:00Z"
  }
}
```

**Resolution:** Update the existing resource instead of creating a new one

---

#### RESOURCE_CONFLICT
**Status:** 409 Conflict

The operation conflicts with the current state of the resource.

**Example:**
```json
{
  "error": {
    "code": "RESOURCE_CONFLICT",
    "message": "Cannot delete book with active loans",
    "details": [
      {
        "field": "bookId",
        "issue": "Book has 3 active loans and 2 pending reservations"
      }
    ],
    "requestId": "req_008",
    "timestamp": "2024-10-28T11:10:00Z"
  }
}
```

**Resolution:** Resolve the conflict (e.g., wait for loans to be returned)

---

### Validation Errors (VALIDATION_*)

#### VALIDATION_ERROR
**Status:** 422 Unprocessable Entity

One or more fields failed validation.

**Example:**
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "The request contains validation errors",
    "details": [
      {
        "field": "email",
        "issue": "Invalid email format"
      },
      {
        "field": "phone",
        "issue": "Phone number must be in format +1-XXX-XXXX"
      },
      {
        "field": "totalCopies",
        "issue": "Must be a positive integer"
      }
    ],
    "requestId": "req_009",
    "timestamp": "2024-10-28T11:15:00Z"
  }
}
```

**Resolution:** Fix the invalid fields according to the details

---

#### MISSING_REQUIRED_FIELD
**Status:** 422 Unprocessable Entity

A required field is missing from the request.

**Example:**
```json
{
  "error": {
    "code": "MISSING_REQUIRED_FIELD",
    "message": "Required field is missing",
    "details": [
      {
        "field": "title",
        "issue": "title is required"
      },
      {
        "field": "isbn",
        "issue": "isbn is required"
      }
    ],
    "requestId": "req_010",
    "timestamp": "2024-10-28T11:20:00Z"
  }
}
```

**Resolution:** Include all required fields in the request

---

#### INVALID_INPUT_FORMAT
**Status:** 400 Bad Request

Input data format is incorrect.

**Example:**
```json
{
  "error": {
    "code": "INVALID_INPUT_FORMAT",
    "message": "Invalid data format",
    "details": [
      {
        "field": "publishedDate",
        "issue": "Date must be in ISO 8601 format (YYYY-MM-DD)"
      },
      {
        "field": "page",
        "issue": "Must be a positive integer"
      }
    ],
    "requestId": "req_011",
    "timestamp": "2024-10-28T11:25:00Z"
  }
}
```

---

### Business Logic Errors (BUSINESS_*)

#### LOAN_LIMIT_EXCEEDED
**Status:** 409 Conflict

Member has reached the maximum number of concurrent loans.

**Example:**
```json
{
  "error": {
    "code": "LOAN_LIMIT_EXCEEDED",
    "message": "Cannot create loan. Member has reached maximum loan limit.",
    "details": [
      {
        "field": "memberId",
        "issue": "Member has 5 active loans (maximum: 5)"
      }
    ],
    "requestId": "req_012",
    "timestamp": "2024-10-28T11:30:00Z",
    "documentation": "https://api.library.example.com/docs/business-rules/loan-limits"
  }
}
```

**Resolution:** Return a book before borrowing another

---

#### BOOK_NOT_AVAILABLE
**Status:** 409 Conflict

No copies of the book are currently available.

**Example:**
```json
{
  "error": {
    "code": "BOOK_NOT_AVAILABLE",
    "message": "Book is not available for checkout",
    "details": [
      {
        "field": "bookId",
        "issue": "All 5 copies are currently checked out. Consider making a reservation."
      }
    ],
    "requestId": "req_013",
    "timestamp": "2024-10-28T11:35:00Z",
    "_links": {
      "reservations": "/v1/reservations"
    }
  }
}
```

**Resolution:** Create a reservation or wait for a copy to become available

---

#### MEMBER_HAS_OVERDUE_BOOKS
**Status:** 409 Conflict

Member cannot borrow because they have overdue items.

**Example:**
```json
{
  "error": {
    "code": "MEMBER_HAS_OVERDUE_BOOKS",
    "message": "Cannot create new loan. Member has overdue books.",
    "details": [
      {
        "field": "memberId",
        "issue": "Member has 2 overdue book(s) and owes $3.50 in fines"
      }
    ],
    "requestId": "req_014",
    "timestamp": "2024-10-28T11:40:00Z",
    "_links": {
      "overdueLoans": "/v1/members/member_789/loans?status=overdue",
      "fines": "/v1/members/member_789/fines"
    }
  }
}
```

**Resolution:** Return overdue books and pay outstanding fines

---

#### MAX_RENEWALS_REACHED
**Status:** 400 Bad Request

Loan has reached the maximum number of renewals.

**Example:**
```json
{
  "error": {
    "code": "MAX_RENEWALS_REACHED",
    "message": "This loan has reached the maximum number of renewals",
    "details": [
      {
        "field": "renewalCount",
        "issue": "Maximum of 1 renewal allowed per loan. Current count: 1"
      }
    ],
    "requestId": "req_015",
    "timestamp": "2024-10-28T11:45:00Z"
  }
}
```

**Resolution:** Return the book and check it out again if still needed

---

#### BOOK_RESERVED_BY_OTHER
**Status:** 409 Conflict

Cannot renew loan because book is reserved by another member.

**Example:**
```json
{
  "error": {
    "code": "BOOK_RESERVED_BY_OTHER",
    "message": "Cannot renew loan. Book is reserved by another member.",
    "details": [
      {
        "field": "loanId",
        "issue": "Book has 2 pending reservations"
      }
    ],
    "requestId": "req_016",
    "timestamp": "2024-10-28T11:50:00Z"
  }
}
```

**Resolution:** Return the book on the due date

---

#### RESERVATION_ALREADY_EXISTS
**Status:** 409 Conflict

Member already has a reservation for this book.

**Example:**
```json
{
  "error": {
    "code": "RESERVATION_ALREADY_EXISTS",
    "message": "Member already has a reservation for this book",
    "details": [
      {
        "field": "bookId",
        "issue": "Active reservation exists (reservation_111)"
      }
    ],
    "requestId": "req_017",
    "timestamp": "2024-10-28T11:55:00Z",
    "_links": {
      "reservation": "/v1/reservations/reservation_111"
    }
  }
}
```

---

### Rate Limiting Errors

#### RATE_LIMIT_EXCEEDED
**Status:** 429 Too Many Requests

API rate limit has been exceeded.

**Example:**
```json
{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Rate limit exceeded. Please try again later.",
    "retryAfter": 3600,
    "requestId": "req_018",
    "timestamp": "2024-10-28T12:00:00Z",
    "documentation": "https://api.library.example.com/docs/rate-limits"
  }
}
```

**Response Headers:**
```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1698508800
Retry-After: 3600
```

**Resolution:** Wait for the time specified in `retryAfter` (seconds) or `Retry-After` header

---

### Server Errors

#### INTERNAL_SERVER_ERROR
**Status:** 500 Internal Server Error

An unexpected error occurred on the server.

**Example:**
```json
{
  "error": {
    "code": "INTERNAL_SERVER_ERROR",
    "message": "An unexpected error occurred. Please try again later.",
    "requestId": "req_019",
    "timestamp": "2024-10-28T12:05:00Z",
    "documentation": "https://api.library.example.com/docs/support"
  }
}
```

**Resolution:** Retry the request. If the problem persists, contact support with the requestId

---

#### SERVICE_UNAVAILABLE
**Status:** 503 Service Unavailable

The service is temporarily unavailable.

**Example:**
```json
{
  "error": {
    "code": "SERVICE_UNAVAILABLE",
    "message": "Service is temporarily unavailable due to maintenance",
    "retryAfter": 1800,
    "requestId": "req_020",
    "timestamp": "2024-10-28T12:10:00Z"
  }
}
```

**Response Headers:**
```
Retry-After: 1800
```

---

## Error Scenarios by Category

### 1. Authentication & Authorization

| Scenario | Status | Error Code |
|----------|--------|------------|
| No token provided | 401 | AUTH_TOKEN_MISSING |
| Invalid token | 401 | AUTH_TOKEN_INVALID |
| Expired token | 401 | AUTH_TOKEN_EXPIRED |
| Insufficient role | 403 | INSUFFICIENT_PERMISSIONS |
| Cannot access resource | 403 | RESOURCE_ACCESS_DENIED |

### 2. Resource Operations

| Scenario | Status | Error Code |
|----------|--------|------------|
| Resource not found | 404 | RESOURCE_NOT_FOUND |
| Duplicate resource | 400 | RESOURCE_ALREADY_EXISTS |
| State conflict | 409 | RESOURCE_CONFLICT |

### 3. Validation

| Scenario | Status | Error Code |
|----------|--------|------------|
| Invalid field values | 422 | VALIDATION_ERROR |
| Missing required field | 422 | MISSING_REQUIRED_FIELD |
| Wrong data format | 400 | INVALID_INPUT_FORMAT |

### 4. Business Rules

| Scenario | Status | Error Code |
|----------|--------|------------|
| Loan limit exceeded | 409 | LOAN_LIMIT_EXCEEDED |
| Book unavailable | 409 | BOOK_NOT_AVAILABLE |
| Overdue books | 409 | MEMBER_HAS_OVERDUE_BOOKS |
| Max renewals reached | 400 | MAX_RENEWALS_REACHED |
| Book reserved | 409 | BOOK_RESERVED_BY_OTHER |
| Duplicate reservation | 409 | RESERVATION_ALREADY_EXISTS |

### 5. System

| Scenario | Status | Error Code |
|----------|--------|------------|
| Rate limit exceeded | 429 | RATE_LIMIT_EXCEEDED |
| Server error | 500 | INTERNAL_SERVER_ERROR |
| Service unavailable | 503 | SERVICE_UNAVAILABLE |

---

## Best Practices for Error Handling

### For API Consumers

1. **Always check HTTP status code first**
   ```javascript
   if (response.status !== 200) {
     const error = await response.json();
     handleError(error);
   }
   ```

2. **Use error codes for programmatic handling**
   ```javascript
   switch (error.error.code) {
     case 'AUTH_TOKEN_EXPIRED':
       await refreshToken();
       return retry();
     case 'RATE_LIMIT_EXCEEDED':
       await sleep(error.error.retryAfter * 1000);
       return retry();
     default:
       showErrorToUser(error.error.message);
   }
   ```

3. **Display user-friendly messages**
   ```javascript
   // Use error.message for display
   alert(error.error.message);
   
   // Show field-specific errors
   error.error.details.forEach(detail => {
     showFieldError(detail.field, detail.issue);
   });
   ```

4. **Log requestId for support**
   ```javascript
   console.error(`Error ${error.error.code}: ${error.error.message}`);
   console.error(`Request ID: ${error.error.requestId}`);
   ```

5. **Implement retry logic with exponential backoff**
   ```javascript
   async function retryWithBackoff(fn, maxRetries = 3) {
     for (let i = 0; i < maxRetries; i++) {
       try {
         return await fn();
       } catch (error) {
         if (error.status >= 500 && i < maxRetries - 1) {
           await sleep(Math.pow(2, i) * 1000);
           continue;
         }
         throw error;
       }
     }
   }
   ```

### For API Developers

1. **Be consistent**: Always use the same error response format
2. **Be specific**: Provide detailed error messages and field-level issues
3. **Be helpful**: Include links to documentation and related resources
4. **Be secure**: Don't expose sensitive information in error messages
5. **Be logged**: Log all errors with requestId for debugging
6. **Be documented**: Document all possible error codes and scenarios

---

## Testing Error Scenarios

### Example: Testing Authentication Error

```bash
# Request without token
curl -X GET https://api.library.example.com/v1/books/book_123

# Expected: 401 Unauthorized with AUTH_TOKEN_MISSING
```

### Example: Testing Validation Error

```bash
# Request with invalid data
curl -X POST https://api.library.example.com/v1/members \
  -H "Content-Type: application/json" \
  -d '{
    "firstName": "John",
    "email": "invalid-email"
  }'

# Expected: 422 Unprocessable Entity with VALIDATION_ERROR
```

### Example: Testing Business Logic Error

```bash
# Try to checkout when limit reached
curl -X POST https://api.library.example.com/v1/loans \
  -H "Authorization: Bearer {token}" \
  -H "Content-Type: application/json" \
  -d '{
    "bookId": "book_123",
    "memberId": "member_with_5_loans"
  }'

# Expected: 409 Conflict with LOAN_LIMIT_EXCEEDED
```

---

## Summary

This error handling guide provides:

- ✅ Consistent error response format
- ✅ Comprehensive error codes
- ✅ Clear HTTP status code usage
- ✅ Detailed error scenarios
- ✅ Best practices for handling errors
- ✅ Examples for common situations

Proper error handling ensures API consumers can:
- Understand what went wrong
- Take appropriate corrective action
- Provide meaningful feedback to users
- Debug issues effectively
