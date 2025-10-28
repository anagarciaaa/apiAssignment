# Use Cases and Example Workflows

This document provides detailed use cases and example API workflows for the Library Management System.

## Table of Contents
1. [Member Registration and Book Borrowing](#1-member-registration-and-book-borrowing)
2. [Book Search and Details](#2-book-search-and-details)
3. [Loan Renewal](#3-loan-renewal)
4. [Book Return with Late Fee](#4-book-return-with-late-fee)
5. [Book Reservation](#5-book-reservation)
6. [Handling Overdue Books](#6-handling-overdue-books)
7. [Librarian Adding New Books](#7-librarian-adding-new-books)
8. [Member Profile Management](#8-member-profile-management)
9. [Error Scenarios](#9-error-scenarios)

---

## 1. Member Registration and Book Borrowing

### Scenario
A new user wants to register as a library member and borrow their first book.

### Workflow

#### Step 1: Member Registration
**Request:**
```http
POST /v1/members
Content-Type: application/json

{
  "firstName": "Alice",
  "lastName": "Johnson",
  "email": "alice.johnson@example.com",
  "phone": "+1-555-0199",
  "address": {
    "street": "456 Oak Avenue",
    "city": "Springfield",
    "state": "IL",
    "zipCode": "62702",
    "country": "USA"
  },
  "dateOfBirth": "1995-08-20",
  "password": "SecurePassword123!"
}
```

**Response: 201 Created**
```json
{
  "id": "member_890",
  "memberNumber": "LIB2024-001235",
  "firstName": "Alice",
  "lastName": "Johnson",
  "email": "alice.johnson@example.com",
  "phone": "+1-555-0199",
  "address": {
    "street": "456 Oak Avenue",
    "city": "Springfield",
    "state": "IL",
    "zipCode": "62702",
    "country": "USA"
  },
  "dateOfBirth": "1995-08-20",
  "membershipDate": "2024-10-28",
  "membershipExpiry": "2025-10-28",
  "status": "active",
  "currentLoans": 0,
  "maxLoans": 5,
  "overdueBooks": 0,
  "totalFines": 0.00,
  "createdAt": "2024-10-28T10:00:00Z",
  "updatedAt": "2024-10-28T10:00:00Z",
  "_links": {
    "self": "/v1/members/member_890",
    "loans": "/v1/members/member_890/loans"
  }
}
```

#### Step 2: Member Login
**Request:**
```http
POST /v1/auth/login
Content-Type: application/json

{
  "email": "alice.johnson@example.com",
  "password": "SecurePassword123!"
}
```

**Response: 200 OK**
```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJtZW1iZXJfODkwIiwicm9sZSI6Im1lbWJlciIsImlhdCI6MTYzMDQ5MDQwMCwiZXhwIjoxNjMwNDk0MDAwfQ.signature",
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "tokenType": "Bearer",
  "expiresIn": 3600,
  "user": {
    "id": "member_890",
    "email": "alice.johnson@example.com",
    "role": "member"
  }
}
```

#### Step 3: Search for a Book
**Request:**
```http
GET /v1/books?search=1984&available=true
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**Response: 200 OK**
```json
{
  "data": [
    {
      "id": "book_456",
      "isbn": "978-0-452-28423-4",
      "title": "1984",
      "authors": [
        {
          "id": "author_789",
          "name": "George Orwell"
        }
      ],
      "publisher": "Penguin Books",
      "publishedDate": "1949-06-08",
      "categories": ["Fiction", "Dystopian", "Political Fiction"],
      "totalCopies": 8,
      "availableCopies": 3,
      "description": "A dystopian social science fiction novel...",
      "language": "en",
      "pages": 328,
      "_links": {
        "self": "/v1/books/book_456"
      }
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "totalPages": 1,
    "totalItems": 1
  }
}
```

#### Step 4: View Book Details
**Request:**
```http
GET /v1/books/book_456
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**Response: 200 OK** (Full book details including cover image, etc.)

#### Step 5: Librarian Creates Loan (Checkout)
**Request:**
```http
POST /v1/loans
Authorization: Bearer {librarian-token}
Content-Type: application/json

{
  "bookId": "book_456",
  "memberId": "member_890"
}
```

**Response: 201 Created**
```json
{
  "id": "loan_555",
  "book": {
    "id": "book_456",
    "title": "1984",
    "_links": {
      "self": "/v1/books/book_456"
    }
  },
  "member": {
    "id": "member_890",
    "name": "Alice Johnson",
    "_links": {
      "self": "/v1/members/member_890"
    }
  },
  "checkoutDate": "2024-10-28T10:30:00Z",
  "dueDate": "2024-11-11T23:59:59Z",
  "returnDate": null,
  "renewalCount": 0,
  "maxRenewals": 1,
  "status": "active",
  "fine": 0.00,
  "createdAt": "2024-10-28T10:30:00Z",
  "updatedAt": "2024-10-28T10:30:00Z",
  "_links": {
    "self": "/v1/loans/loan_555",
    "renew": "/v1/loans/loan_555/renew",
    "return": "/v1/loans/loan_555/return"
  }
}
```

---

## 2. Book Search and Details

### Scenario
A member wants to search for books by a specific author and view detailed information.

### Workflow

#### Step 1: Search by Author
**Request:**
```http
GET /v1/books?search=Stephen%20King&sort=title&order=asc&limit=10
Authorization: Bearer {token}
```

**Response: 200 OK** (List of books by Stephen King, sorted by title)

#### Step 2: Filter by Category
**Request:**
```http
GET /v1/books?category=Horror&available=true
Authorization: Bearer {token}
```

#### Step 3: View Author Details
**Request:**
```http
GET /v1/authors/author_789
```

**Response: 200 OK**
```json
{
  "id": "author_789",
  "name": "George Orwell",
  "biography": "Eric Arthur Blair, known by his pen name George Orwell, was an English novelist, essayist, journalist and critic...",
  "birthDate": "1903-06-25",
  "deathDate": "1950-01-21",
  "nationality": "British",
  "bookCount": 12,
  "_links": {
    "self": "/v1/authors/author_789",
    "books": "/v1/books?author=author_789"
  }
}
```

---

## 3. Loan Renewal

### Scenario
A member wants to renew a book they currently have checked out.

### Workflow

#### Step 1: View Current Loans
**Request:**
```http
GET /v1/members/member_890/loans?status=active
Authorization: Bearer {token}
```

**Response: 200 OK**
```json
{
  "data": [
    {
      "id": "loan_555",
      "book": {
        "id": "book_456",
        "title": "1984"
      },
      "checkoutDate": "2024-10-28T10:30:00Z",
      "dueDate": "2024-11-11T23:59:59Z",
      "renewalCount": 0,
      "maxRenewals": 1,
      "status": "active",
      "_links": {
        "self": "/v1/loans/loan_555",
        "renew": "/v1/loans/loan_555/renew"
      }
    }
  ]
}
```

#### Step 2: Renew Loan
**Request:**
```http
POST /v1/loans/loan_555/renew
Authorization: Bearer {token}
```

**Response: 200 OK**
```json
{
  "id": "loan_555",
  "newDueDate": "2024-11-18T23:59:59Z",
  "renewalCount": 1,
  "message": "Loan renewed successfully"
}
```

#### Step 3: Attempt Second Renewal (Error)
**Request:**
```http
POST /v1/loans/loan_555/renew
Authorization: Bearer {token}
```

**Response: 400 Bad Request**
```json
{
  "error": {
    "code": "MAX_RENEWALS_REACHED",
    "message": "This loan has reached the maximum number of renewals",
    "details": [
      {
        "field": "renewalCount",
        "issue": "Maximum of 1 renewal allowed per loan"
      }
    ],
    "requestId": "req_xyz789",
    "timestamp": "2024-10-29T10:00:00Z",
    "documentation": "https://api.library.example.com/docs/errors/MAX_RENEWALS_REACHED"
  }
}
```

---

## 4. Book Return with Late Fee

### Scenario
A member returns a book 3 days past the due date.

### Workflow

#### Step 1: Check Loan Status
**Request:**
```http
GET /v1/loans/loan_555
Authorization: Bearer {librarian-token}
```

**Response: 200 OK**
```json
{
  "id": "loan_555",
  "book": {
    "id": "book_456",
    "title": "1984"
  },
  "member": {
    "id": "member_890",
    "name": "Alice Johnson"
  },
  "checkoutDate": "2024-10-28T10:30:00Z",
  "dueDate": "2024-11-11T23:59:59Z",
  "returnDate": null,
  "renewalCount": 0,
  "status": "overdue",
  "fine": 1.50,
  "_links": {
    "self": "/v1/loans/loan_555",
    "return": "/v1/loans/loan_555/return"
  }
}
```

#### Step 2: Process Return
**Request:**
```http
POST /v1/loans/loan_555/return
Authorization: Bearer {librarian-token}
Content-Type: application/json

{
  "returnDate": "2024-11-14T14:30:00Z",
  "condition": "good",
  "notes": "Minor wear on spine"
}
```

**Response: 200 OK**
```json
{
  "id": "loan_555",
  "status": "returned",
  "returnDate": "2024-11-14T14:30:00Z",
  "fine": 1.50,
  "message": "Book returned successfully. Fine of $1.50 has been applied to the member's account."
}
```

#### Step 3: View Member's Fine
**Request:**
```http
GET /v1/members/member_890
Authorization: Bearer {token}
```

**Response: 200 OK**
```json
{
  "id": "member_890",
  "memberNumber": "LIB2024-001235",
  "firstName": "Alice",
  "lastName": "Johnson",
  "email": "alice.johnson@example.com",
  "status": "active",
  "currentLoans": 0,
  "overdueBooks": 0,
  "totalFines": 1.50,
  "_links": {
    "self": "/v1/members/member_890",
    "fines": "/v1/members/member_890/fines"
  }
}
```

---

## 5. Book Reservation

### Scenario
All copies of a popular book are checked out. A member wants to reserve it.

### Workflow

#### Step 1: Check Book Availability
**Request:**
```http
GET /v1/books/book_789
```

**Response: 200 OK**
```json
{
  "id": "book_789",
  "isbn": "978-0-7432-7356-5",
  "title": "The Da Vinci Code",
  "authors": [
    {
      "id": "author_234",
      "name": "Dan Brown"
    }
  ],
  "totalCopies": 5,
  "availableCopies": 0,
  "status": "All copies checked out",
  "_links": {
    "self": "/v1/books/book_789",
    "reservations": "/v1/books/book_789/reservations"
  }
}
```

#### Step 2: Create Reservation
**Request:**
```http
POST /v1/reservations
Authorization: Bearer {token}
Content-Type: application/json

{
  "bookId": "book_789",
  "memberId": "member_890"
}
```

**Response: 201 Created**
```json
{
  "id": "reservation_111",
  "book": {
    "id": "book_789",
    "title": "The Da Vinci Code"
  },
  "member": {
    "id": "member_890",
    "name": "Alice Johnson"
  },
  "reservationDate": "2024-10-28T11:00:00Z",
  "expiryDate": "2024-11-04T23:59:59Z",
  "status": "pending",
  "queuePosition": 3,
  "_links": {
    "self": "/v1/reservations/reservation_111",
    "cancel": "/v1/reservations/reservation_111"
  }
}
```

#### Step 3: Check Reservation Status
**Request:**
```http
GET /v1/reservations/reservation_111
Authorization: Bearer {token}
```

**Response: 200 OK** (Updated when book becomes available)
```json
{
  "id": "reservation_111",
  "book": {
    "id": "book_789",
    "title": "The Da Vinci Code"
  },
  "member": {
    "id": "member_890",
    "name": "Alice Johnson"
  },
  "reservationDate": "2024-10-28T11:00:00Z",
  "expiryDate": "2024-11-04T23:59:59Z",
  "status": "ready",
  "queuePosition": 1,
  "message": "Book is ready for pickup. Please collect within 7 days.",
  "_links": {
    "self": "/v1/reservations/reservation_111"
  }
}
```

---

## 6. Handling Overdue Books

### Scenario
A member tries to borrow a new book but has overdue items.

### Workflow

#### Step 1: Attempt to Create Loan
**Request:**
```http
POST /v1/loans
Authorization: Bearer {librarian-token}
Content-Type: application/json

{
  "bookId": "book_999",
  "memberId": "member_890"
}
```

**Response: 409 Conflict**
```json
{
  "error": {
    "code": "MEMBER_HAS_OVERDUE_BOOKS",
    "message": "Cannot create new loan. Member has overdue books.",
    "details": [
      {
        "field": "memberId",
        "issue": "Member member_890 has 1 overdue book(s) and owes $1.50 in fines"
      }
    ],
    "requestId": "req_abc456",
    "timestamp": "2024-10-28T12:00:00Z",
    "documentation": "https://api.library.example.com/docs/errors/MEMBER_HAS_OVERDUE_BOOKS"
  }
}
```

#### Step 2: View Member's Overdue Books
**Request:**
```http
GET /v1/members/member_890/loans?status=overdue
Authorization: Bearer {librarian-token}
```

**Response: 200 OK** (List of overdue loans)

---

## 7. Librarian Adding New Books

### Scenario
A librarian receives new books and adds them to the catalog.

### Workflow

#### Step 1: Check if Book Already Exists
**Request:**
```http
GET /v1/books?search=978-1-234567-89-0
Authorization: Bearer {librarian-token}
```

**Response: 200 OK** (Empty list if not exists)

#### Step 2: Create Author if Needed
**Request:**
```http
POST /v1/authors
Authorization: Bearer {librarian-token}
Content-Type: application/json

{
  "name": "Jane Smith",
  "biography": "Contemporary fiction author...",
  "birthDate": "1980-03-15",
  "nationality": "American"
}
```

**Response: 201 Created**
```json
{
  "id": "author_new_123",
  "name": "Jane Smith",
  "biography": "Contemporary fiction author...",
  "birthDate": "1980-03-15",
  "nationality": "American",
  "bookCount": 0
}
```

#### Step 3: Add New Book
**Request:**
```http
POST /v1/books
Authorization: Bearer {librarian-token}
Content-Type: application/json

{
  "isbn": "978-1-234567-89-0",
  "title": "The New Adventure",
  "authorIds": ["author_new_123"],
  "publisher": "Modern Press",
  "publishedDate": "2024-10-01",
  "categories": ["Fiction", "Adventure"],
  "totalCopies": 3,
  "description": "An exciting new adventure story...",
  "language": "en",
  "pages": 425
}
```

**Response: 201 Created**
```json
{
  "id": "book_new_456",
  "isbn": "978-1-234567-89-0",
  "title": "The New Adventure",
  "authors": [
    {
      "id": "author_new_123",
      "name": "Jane Smith"
    }
  ],
  "publisher": "Modern Press",
  "publishedDate": "2024-10-01",
  "categories": ["Fiction", "Adventure"],
  "totalCopies": 3,
  "availableCopies": 3,
  "description": "An exciting new adventure story...",
  "language": "en",
  "pages": 425,
  "createdAt": "2024-10-28T13:00:00Z",
  "updatedAt": "2024-10-28T13:00:00Z",
  "_links": {
    "self": "/v1/books/book_new_456"
  }
}
```

---

## 8. Member Profile Management

### Scenario
A member updates their contact information.

### Workflow

#### Step 1: View Current Profile
**Request:**
```http
GET /v1/members/member_890
Authorization: Bearer {token}
```

#### Step 2: Update Profile
**Request:**
```http
PATCH /v1/members/member_890
Authorization: Bearer {token}
Content-Type: application/json

{
  "phone": "+1-555-0200",
  "address": {
    "street": "789 Maple Street",
    "city": "Springfield",
    "state": "IL",
    "zipCode": "62703",
    "country": "USA"
  }
}
```

**Response: 200 OK** (Updated member object)

---

## 9. Error Scenarios

### Scenario A: Book Not Found
**Request:**
```http
GET /v1/books/book_nonexistent
```

**Response: 404 Not Found**
```json
{
  "error": {
    "code": "RESOURCE_NOT_FOUND",
    "message": "The requested book was not found",
    "details": [
      {
        "field": "bookId",
        "issue": "Book with ID 'book_nonexistent' does not exist"
      }
    ],
    "requestId": "req_error_001",
    "timestamp": "2024-10-28T14:00:00Z",
    "documentation": "https://api.library.example.com/docs/errors/RESOURCE_NOT_FOUND"
  }
}
```

### Scenario B: Insufficient Permissions
**Request:**
```http
POST /v1/books
Authorization: Bearer {member-token}
Content-Type: application/json

{
  "isbn": "978-1-234567-89-0",
  "title": "Test Book"
}
```

**Response: 403 Forbidden**
```json
{
  "error": {
    "code": "INSUFFICIENT_PERMISSIONS",
    "message": "You do not have permission to perform this action",
    "details": [
      {
        "field": "role",
        "issue": "Creating books requires 'librarian' or 'admin' role"
      }
    ],
    "requestId": "req_error_002",
    "timestamp": "2024-10-28T14:05:00Z"
  }
}
```

### Scenario C: Validation Error
**Request:**
```http
POST /v1/members
Content-Type: application/json

{
  "firstName": "John",
  "email": "invalid-email",
  "password": "short"
}
```

**Response: 422 Unprocessable Entity**
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "The request contains validation errors",
    "details": [
      {
        "field": "lastName",
        "issue": "lastName is required"
      },
      {
        "field": "email",
        "issue": "Invalid email format"
      },
      {
        "field": "password",
        "issue": "Password must be at least 8 characters"
      },
      {
        "field": "dateOfBirth",
        "issue": "dateOfBirth is required"
      }
    ],
    "requestId": "req_error_003",
    "timestamp": "2024-10-28T14:10:00Z"
  }
}
```

### Scenario D: Loan Limit Exceeded
**Request:**
```http
POST /v1/loans
Authorization: Bearer {librarian-token}
Content-Type: application/json

{
  "bookId": "book_456",
  "memberId": "member_with_5_loans"
}
```

**Response: 409 Conflict**
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
    "requestId": "req_error_004",
    "timestamp": "2024-10-28T14:15:00Z"
  }
}
```

### Scenario E: Rate Limit Exceeded
**Request:**
```http
GET /v1/books
```

**Response: 429 Too Many Requests**
```json
{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Rate limit exceeded. Please try again later.",
    "retryAfter": 3600,
    "requestId": "req_error_005",
    "timestamp": "2024-10-28T14:20:00Z"
  }
}
```

**Headers:**
```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1698508800
Retry-After: 3600
```

---

## Summary

These use cases demonstrate:

1. ✅ **Complete user workflows** - From registration to borrowing
2. ✅ **Business logic enforcement** - Loan limits, overdue handling
3. ✅ **Error handling** - Comprehensive error scenarios
4. ✅ **Authentication & authorization** - Role-based access
5. ✅ **Real-world scenarios** - Reservations, renewals, late fees
6. ✅ **API best practices** - RESTful design, proper status codes
7. ✅ **Validation** - Input validation and business rule validation

The API design handles both happy paths and error scenarios gracefully, providing clear feedback to API consumers while enforcing business rules and security policies.
