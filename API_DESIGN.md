# REST API Design: Library Management System

## Overview
This document describes the REST API design for a Library Management System. The system manages books, members, loans, and reservations, demonstrating core REST principles, proper resource modeling, and comprehensive error handling.

## Business Domain

### Key Entities
1. **Books** - Physical items in the library collection
2. **Members** - Library patrons who can borrow books
3. **Loans** - Active book checkouts
4. **Reservations** - Book holds for future borrowing
5. **Authors** - Book authors (related to books)
6. **Categories** - Book classifications

### Business Rules
- Members can borrow up to 5 books at a time
- Loan period is 14 days with one 7-day renewal allowed
- Members cannot borrow books if they have overdue items
- Books can be reserved when all copies are checked out
- Late fees: $0.50 per day per book

## API Design Principles

### REST Principles Applied
1. **Resource-Based URLs** - URLs represent resources, not actions
2. **HTTP Methods** - Use appropriate verbs (GET, POST, PUT, PATCH, DELETE)
3. **Stateless** - Each request contains all necessary information
4. **HATEOAS** - Include links to related resources
5. **Standard Status Codes** - Meaningful HTTP status codes

### Versioning Strategy
- URL-based versioning: `/api/v1/`
- Major version in URL path
- Backward compatibility maintained within major versions

### Authentication & Authorization
- JWT (JSON Web Token) based authentication
- Role-based access control (Admin, Librarian, Member)
- API key for public read-only endpoints

## API Endpoints

### Base URL
```
https://api.library.example.com/v1
```

---

## 1. Books

### 1.1 List Books
**Endpoint:** `GET /books`

**Description:** Retrieve a paginated list of books

**Query Parameters:**
- `page` (integer, default: 1) - Page number
- `limit` (integer, default: 20, max: 100) - Items per page
- `search` (string) - Search in title, author, ISBN
- `category` (string) - Filter by category ID
- `available` (boolean) - Filter by availability
- `sort` (string) - Sort field (title, author, publishedDate)
- `order` (string: asc|desc, default: asc) - Sort order

**Response:** 200 OK
```json
{
  "data": [
    {
      "id": "book_123",
      "isbn": "978-0-123456-78-9",
      "title": "The Great Gatsby",
      "authors": [
        {
          "id": "author_456",
          "name": "F. Scott Fitzgerald"
        }
      ],
      "publisher": "Scribner",
      "publishedDate": "1925-04-10",
      "categories": ["Fiction", "Classic Literature"],
      "totalCopies": 5,
      "availableCopies": 2,
      "description": "A story of decadence and excess...",
      "language": "en",
      "pages": 180,
      "createdAt": "2024-01-15T10:30:00Z",
      "updatedAt": "2024-10-20T14:22:00Z",
      "_links": {
        "self": "/v1/books/book_123",
        "loans": "/v1/books/book_123/loans",
        "reservations": "/v1/books/book_123/reservations"
      }
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "totalPages": 15,
    "totalItems": 289
  },
  "_links": {
    "self": "/v1/books?page=1&limit=20",
    "next": "/v1/books?page=2&limit=20",
    "last": "/v1/books?page=15&limit=20"
  }
}
```

**Error Responses:**
- `400 Bad Request` - Invalid query parameters
- `401 Unauthorized` - Missing or invalid authentication token

---

### 1.2 Get Book by ID
**Endpoint:** `GET /books/{bookId}`

**Description:** Retrieve detailed information about a specific book

**Path Parameters:**
- `bookId` (string, required) - Unique book identifier

**Response:** 200 OK
```json
{
  "id": "book_123",
  "isbn": "978-0-123456-78-9",
  "title": "The Great Gatsby",
  "authors": [
    {
      "id": "author_456",
      "name": "F. Scott Fitzgerald",
      "_links": {
        "self": "/v1/authors/author_456"
      }
    }
  ],
  "publisher": "Scribner",
  "publishedDate": "1925-04-10",
  "categories": ["Fiction", "Classic Literature"],
  "totalCopies": 5,
  "availableCopies": 2,
  "description": "A story of decadence and excess...",
  "language": "en",
  "pages": 180,
  "coverImage": "https://cdn.library.example.com/covers/book_123.jpg",
  "createdAt": "2024-01-15T10:30:00Z",
  "updatedAt": "2024-10-20T14:22:00Z",
  "_links": {
    "self": "/v1/books/book_123",
    "loans": "/v1/books/book_123/loans",
    "reservations": "/v1/books/book_123/reservations"
  }
}
```

**Error Responses:**
- `404 Not Found` - Book does not exist

---

### 1.3 Create Book
**Endpoint:** `POST /books`

**Description:** Add a new book to the library collection

**Authorization:** Requires `librarian` or `admin` role

**Request Body:**
```json
{
  "isbn": "978-0-123456-78-9",
  "title": "The Great Gatsby",
  "authorIds": ["author_456"],
  "publisher": "Scribner",
  "publishedDate": "1925-04-10",
  "categories": ["Fiction", "Classic Literature"],
  "totalCopies": 5,
  "description": "A story of decadence and excess...",
  "language": "en",
  "pages": 180
}
```

**Response:** 201 Created
```json
{
  "id": "book_123",
  "isbn": "978-0-123456-78-9",
  "title": "The Great Gatsby",
  "authors": [
    {
      "id": "author_456",
      "name": "F. Scott Fitzgerald"
    }
  ],
  "publisher": "Scribner",
  "publishedDate": "1925-04-10",
  "categories": ["Fiction", "Classic Literature"],
  "totalCopies": 5,
  "availableCopies": 5,
  "description": "A story of decadence and excess...",
  "language": "en",
  "pages": 180,
  "createdAt": "2024-10-28T10:30:00Z",
  "updatedAt": "2024-10-28T10:30:00Z",
  "_links": {
    "self": "/v1/books/book_123"
  }
}
```

**Error Responses:**
- `400 Bad Request` - Invalid request body or duplicate ISBN
- `401 Unauthorized` - Missing or invalid authentication token
- `403 Forbidden` - Insufficient permissions
- `422 Unprocessable Entity` - Validation errors

---

### 1.4 Update Book
**Endpoint:** `PATCH /books/{bookId}`

**Description:** Update book information (partial update)

**Authorization:** Requires `librarian` or `admin` role

**Request Body:**
```json
{
  "totalCopies": 7,
  "description": "Updated description..."
}
```

**Response:** 200 OK (returns updated book)

**Error Responses:**
- `400 Bad Request` - Invalid request body
- `401 Unauthorized` - Missing or invalid authentication token
- `403 Forbidden` - Insufficient permissions
- `404 Not Found` - Book does not exist

---

### 1.5 Delete Book
**Endpoint:** `DELETE /books/{bookId}`

**Description:** Remove a book from the collection (soft delete)

**Authorization:** Requires `admin` role

**Response:** 204 No Content

**Error Responses:**
- `401 Unauthorized` - Missing or invalid authentication token
- `403 Forbidden` - Insufficient permissions
- `404 Not Found` - Book does not exist
- `409 Conflict` - Book has active loans or reservations

---

## 2. Members

### 2.1 List Members
**Endpoint:** `GET /members`

**Authorization:** Requires `librarian` or `admin` role

**Query Parameters:**
- `page`, `limit` - Pagination
- `search` - Search by name, email, member number
- `status` (active|inactive|suspended) - Filter by status

**Response:** 200 OK (paginated list of members)

---

### 2.2 Get Member by ID
**Endpoint:** `GET /members/{memberId}`

**Authorization:** Member can view own profile, librarians can view all

**Response:** 200 OK
```json
{
  "id": "member_789",
  "memberNumber": "LIB2024-001234",
  "firstName": "John",
  "lastName": "Doe",
  "email": "john.doe@example.com",
  "phone": "+1-555-0123",
  "address": {
    "street": "123 Main St",
    "city": "Springfield",
    "state": "IL",
    "zipCode": "62701",
    "country": "USA"
  },
  "dateOfBirth": "1990-05-15",
  "membershipDate": "2024-01-15",
  "membershipExpiry": "2025-01-15",
  "status": "active",
  "currentLoans": 2,
  "maxLoans": 5,
  "overdueBooks": 0,
  "totalFines": 0.00,
  "createdAt": "2024-01-15T10:30:00Z",
  "updatedAt": "2024-10-20T14:22:00Z",
  "_links": {
    "self": "/v1/members/member_789",
    "loans": "/v1/members/member_789/loans",
    "reservations": "/v1/members/member_789/reservations",
    "fines": "/v1/members/member_789/fines"
  }
}
```

**Error Responses:**
- `401 Unauthorized` - Missing or invalid authentication token
- `403 Forbidden` - Cannot view other members' profiles
- `404 Not Found` - Member does not exist

---

### 2.3 Create Member
**Endpoint:** `POST /members`

**Authorization:** Requires `librarian` or `admin` role (or public registration)

**Request Body:**
```json
{
  "firstName": "John",
  "lastName": "Doe",
  "email": "john.doe@example.com",
  "phone": "+1-555-0123",
  "address": {
    "street": "123 Main St",
    "city": "Springfield",
    "state": "IL",
    "zipCode": "62701",
    "country": "USA"
  },
  "dateOfBirth": "1990-05-15",
  "password": "SecurePass123!"
}
```

**Response:** 201 Created

**Error Responses:**
- `400 Bad Request` - Invalid request body or duplicate email
- `422 Unprocessable Entity` - Validation errors (e.g., invalid email format)

---

### 2.4 Update Member
**Endpoint:** `PATCH /members/{memberId}`

**Authorization:** Member can update own profile, librarians can update all

**Response:** 200 OK

---

### 2.5 Deactivate Member
**Endpoint:** `DELETE /members/{memberId}`

**Authorization:** Requires `admin` role

**Response:** 204 No Content

**Error Responses:**
- `409 Conflict` - Member has active loans

---

## 3. Loans

### 3.1 List Loans
**Endpoint:** `GET /loans`

**Authorization:** Requires `librarian` or `admin` role

**Query Parameters:**
- `page`, `limit` - Pagination
- `memberId` - Filter by member
- `bookId` - Filter by book
- `status` (active|returned|overdue) - Filter by status

**Response:** 200 OK (paginated list of loans)

---

### 3.2 Get Loan by ID
**Endpoint:** `GET /loans/{loanId}`

**Response:** 200 OK
```json
{
  "id": "loan_321",
  "book": {
    "id": "book_123",
    "title": "The Great Gatsby",
    "_links": {
      "self": "/v1/books/book_123"
    }
  },
  "member": {
    "id": "member_789",
    "name": "John Doe",
    "_links": {
      "self": "/v1/members/member_789"
    }
  },
  "checkoutDate": "2024-10-14T10:00:00Z",
  "dueDate": "2024-10-28T23:59:59Z",
  "returnDate": null,
  "renewalCount": 0,
  "maxRenewals": 1,
  "status": "active",
  "fine": 0.00,
  "createdAt": "2024-10-14T10:00:00Z",
  "updatedAt": "2024-10-14T10:00:00Z",
  "_links": {
    "self": "/v1/loans/loan_321",
    "renew": "/v1/loans/loan_321/renew",
    "return": "/v1/loans/loan_321/return"
  }
}
```

---

### 3.3 Create Loan (Checkout Book)
**Endpoint:** `POST /loans`

**Authorization:** Requires `librarian` or `admin` role

**Request Body:**
```json
{
  "bookId": "book_123",
  "memberId": "member_789"
}
```

**Response:** 201 Created

**Error Responses:**
- `400 Bad Request` - Invalid request
- `404 Not Found` - Book or member not found
- `409 Conflict` - Book not available, member has reached loan limit, or member has overdue books

---

### 3.4 Return Book
**Endpoint:** `POST /loans/{loanId}/return`

**Authorization:** Requires `librarian` or `admin` role

**Request Body:**
```json
{
  "returnDate": "2024-10-27T14:30:00Z",
  "condition": "good",
  "notes": "Minor wear on cover"
}
```

**Response:** 200 OK
```json
{
  "id": "loan_321",
  "status": "returned",
  "returnDate": "2024-10-27T14:30:00Z",
  "fine": 0.00,
  "message": "Book returned successfully"
}
```

**Error Responses:**
- `400 Bad Request` - Book already returned
- `404 Not Found` - Loan not found

---

### 3.5 Renew Loan
**Endpoint:** `POST /loans/{loanId}/renew`

**Authorization:** Member can renew own loans, librarians can renew all

**Response:** 200 OK
```json
{
  "id": "loan_321",
  "newDueDate": "2024-11-04T23:59:59Z",
  "renewalCount": 1,
  "message": "Loan renewed successfully"
}
```

**Error Responses:**
- `400 Bad Request` - Maximum renewals reached
- `404 Not Found` - Loan not found
- `409 Conflict` - Book is reserved by another member

---

## 4. Reservations

### 4.1 List Reservations
**Endpoint:** `GET /reservations`

**Query Parameters:**
- `memberId` - Filter by member
- `bookId` - Filter by book
- `status` (pending|ready|fulfilled|cancelled) - Filter by status

**Response:** 200 OK (paginated list)

---

### 4.2 Create Reservation
**Endpoint:** `POST /reservations`

**Request Body:**
```json
{
  "bookId": "book_123",
  "memberId": "member_789"
}
```

**Response:** 201 Created
```json
{
  "id": "reservation_654",
  "book": {
    "id": "book_123",
    "title": "The Great Gatsby"
  },
  "member": {
    "id": "member_789",
    "name": "John Doe"
  },
  "reservationDate": "2024-10-28T10:00:00Z",
  "expiryDate": "2024-11-04T23:59:59Z",
  "status": "pending",
  "queuePosition": 3,
  "_links": {
    "self": "/v1/reservations/reservation_654",
    "cancel": "/v1/reservations/reservation_654"
  }
}
```

**Error Responses:**
- `400 Bad Request` - Book is available (no need to reserve)
- `409 Conflict` - Member already has a reservation for this book

---

### 4.3 Cancel Reservation
**Endpoint:** `DELETE /reservations/{reservationId}`

**Response:** 204 No Content

---

## 5. Authors

### 5.1 List Authors
**Endpoint:** `GET /authors`

**Query Parameters:**
- `page`, `limit` - Pagination
- `search` - Search by name

**Response:** 200 OK (paginated list)

---

### 5.2 Get Author by ID
**Endpoint:** `GET /authors/{authorId}`

**Response:** 200 OK
```json
{
  "id": "author_456",
  "name": "F. Scott Fitzgerald",
  "biography": "American novelist...",
  "birthDate": "1896-09-24",
  "deathDate": "1940-12-21",
  "nationality": "American",
  "bookCount": 12,
  "_links": {
    "self": "/v1/authors/author_456",
    "books": "/v1/books?author=author_456"
  }
}
```

---

## Error Handling

### Standard Error Response Format
All error responses follow this structure:

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
    "requestId": "req_abc123",
    "timestamp": "2024-10-28T10:30:00Z",
    "documentation": "https://api.library.example.com/docs/errors/RESOURCE_NOT_FOUND"
  }
}
```

### HTTP Status Codes

#### Success Codes
- `200 OK` - Successful GET, PATCH, PUT requests
- `201 Created` - Successful POST creating a resource
- `204 No Content` - Successful DELETE

#### Client Error Codes
- `400 Bad Request` - Malformed request or invalid parameters
- `401 Unauthorized` - Missing or invalid authentication
- `403 Forbidden` - Authenticated but insufficient permissions
- `404 Not Found` - Resource does not exist
- `409 Conflict` - Request conflicts with current state
- `422 Unprocessable Entity` - Validation errors
- `429 Too Many Requests` - Rate limit exceeded

#### Server Error Codes
- `500 Internal Server Error` - Unexpected server error
- `503 Service Unavailable` - Temporary unavailability

### Error Code Categories

**Authentication Errors:**
- `AUTH_TOKEN_MISSING` - No authentication token provided
- `AUTH_TOKEN_INVALID` - Token is invalid or expired
- `AUTH_TOKEN_EXPIRED` - Token has expired

**Authorization Errors:**
- `INSUFFICIENT_PERMISSIONS` - User lacks required permissions
- `RESOURCE_ACCESS_DENIED` - Cannot access this specific resource

**Resource Errors:**
- `RESOURCE_NOT_FOUND` - Requested resource doesn't exist
- `RESOURCE_ALREADY_EXISTS` - Duplicate resource (e.g., ISBN)
- `RESOURCE_CONFLICT` - Operation conflicts with resource state

**Validation Errors:**
- `VALIDATION_ERROR` - One or more fields failed validation
- `INVALID_INPUT_FORMAT` - Incorrect data format
- `MISSING_REQUIRED_FIELD` - Required field not provided

**Business Logic Errors:**
- `LOAN_LIMIT_EXCEEDED` - Member has reached maximum loans
- `BOOK_NOT_AVAILABLE` - No copies available for checkout
- `MEMBER_HAS_OVERDUE_BOOKS` - Cannot borrow with overdue items
- `MAX_RENEWALS_REACHED` - Cannot renew loan anymore
- `BOOK_RESERVED_BY_OTHER` - Book reserved by another member

**Rate Limiting:**
- `RATE_LIMIT_EXCEEDED` - Too many requests in time window

---

## Rate Limiting

### Rate Limit Policy
- **Authenticated users:** 1000 requests per hour
- **Unauthenticated (public endpoints):** 100 requests per hour
- **Admin/Librarian roles:** 5000 requests per hour

### Rate Limit Headers
```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 847
X-RateLimit-Reset: 1698508800
```

### Rate Limit Exceeded Response
**Status:** 429 Too Many Requests
```json
{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Rate limit exceeded. Please try again later.",
    "retryAfter": 3600,
    "requestId": "req_abc123",
    "timestamp": "2024-10-28T10:30:00Z"
  }
}
```

---

## Pagination

### Standard Pagination
- Query parameters: `page` (default: 1) and `limit` (default: 20, max: 100)
- Response includes pagination metadata and HATEOAS links

### Pagination Response
```json
{
  "data": [...],
  "pagination": {
    "page": 2,
    "limit": 20,
    "totalPages": 15,
    "totalItems": 289,
    "hasNext": true,
    "hasPrevious": true
  },
  "_links": {
    "self": "/v1/books?page=2&limit=20",
    "first": "/v1/books?page=1&limit=20",
    "previous": "/v1/books?page=1&limit=20",
    "next": "/v1/books?page=3&limit=20",
    "last": "/v1/books?page=15&limit=20"
  }
}
```

---

## Filtering and Sorting

### Filtering
Use query parameters matching field names:
- `/v1/books?category=Fiction&language=en`
- `/v1/loans?status=overdue&memberId=member_789`

### Sorting
- `sort` parameter: field name
- `order` parameter: `asc` or `desc`
- Multiple fields: `/v1/books?sort=author,title&order=asc,asc`

### Searching
- `search` parameter for text search across relevant fields
- `/v1/books?search=gatsby`

---

## Authentication

### JWT Authentication
1. Obtain token via `/v1/auth/login`
2. Include in Authorization header: `Authorization: Bearer {token}`
3. Token expiry: 1 hour
4. Refresh tokens available for extended sessions

### Login Endpoint
**Endpoint:** `POST /v1/auth/login`

**Request:**
```json
{
  "email": "john.doe@example.com",
  "password": "SecurePass123!"
}
```

**Response:** 200 OK
```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "tokenType": "Bearer",
  "expiresIn": 3600,
  "user": {
    "id": "member_789",
    "email": "john.doe@example.com",
    "role": "member"
  }
}
```

---

## Use Cases

### Use Case 1: Member Borrows a Book
1. Member searches for books: `GET /v1/books?search=gatsby`
2. Member views book details: `GET /v1/books/book_123`
3. Librarian creates loan: `POST /v1/loans` with bookId and memberId
4. System validates:
   - Book is available
   - Member hasn't exceeded loan limit
   - Member has no overdue books
5. Loan created, book availability decremented

### Use Case 2: Member Renews a Loan
1. Member views their loans: `GET /v1/members/member_789/loans`
2. Member requests renewal: `POST /v1/loans/loan_321/renew`
3. System validates:
   - Loan exists and is active
   - Maximum renewals not reached
   - Book not reserved by another member
4. Due date extended by 7 days

### Use Case 3: Member Reserves a Book
1. Member searches for book: `GET /v1/books?search=gatsby`
2. Book shows 0 available copies
3. Member creates reservation: `POST /v1/reservations`
4. System queues reservation
5. When book available, reservation status changes to "ready"
6. Member notified and has 7 days to pick up

### Use Case 4: Handling Overdue Books
1. System runs daily job to identify overdue loans
2. For each overdue loan:
   - Calculate fine: days overdue × $0.50
   - Update loan status to "overdue"
   - Send notification to member
3. Member cannot borrow new books until:
   - Book is returned
   - Fine is paid

### Use Case 5: Librarian Adds New Book
1. Librarian authenticates: `POST /v1/auth/login`
2. Librarian creates book: `POST /v1/books`
3. System validates:
   - ISBN not already in system
   - All required fields present
   - Author IDs exist
4. Book created and available for checkout

---

## Design Considerations

### 1. Idempotency
- All GET, PUT, DELETE operations are idempotent
- POST operations use idempotency keys for safe retries
- Header: `Idempotency-Key: {unique-key}`

### 2. Caching
- ETags for conditional requests
- Cache-Control headers
- `If-None-Match` and `If-Modified-Since` support

### 3. Concurrency
- Optimistic locking using version fields
- `If-Match` header for updates
- Conflict detection for simultaneous checkouts

### 4. Performance
- Database indexing on frequently queried fields
- Response compression (gzip)
- Partial responses: `fields` query parameter
- Batch operations where appropriate

### 5. Security
- HTTPS only
- Input validation and sanitization
- SQL injection prevention
- Rate limiting
- CORS configuration
- Security headers (HSTS, CSP, etc.)

### 6. Monitoring & Logging
- Request ID tracking
- Structured logging
- API metrics (response times, error rates)
- Health check endpoint: `GET /v1/health`

### 7. Documentation
- Interactive API documentation (Swagger UI)
- Code examples in multiple languages
- Postman collection
- Changelog for API updates

---

## Future Enhancements

1. **Digital Content** - E-books and audiobooks
2. **Events Management** - Library events and registrations
3. **Recommendation Engine** - Personalized book suggestions
4. **Reading Lists** - User-created collections
5. **Reviews & Ratings** - Member book reviews
6. **Notifications** - Email/SMS for due dates, holds ready
7. **Analytics** - Reading statistics, popular books
8. **Multi-branch Support** - Multiple library locations
9. **Inter-library Loans** - Borrowing from other libraries
10. **Mobile App Integration** - Dedicated mobile endpoints

---

## Conclusion

This REST API design demonstrates:
- ✅ Strong understanding of the business domain
- ✅ Comprehensive resource modeling
- ✅ Multiple use cases and user flows
- ✅ Robust error handling strategies
- ✅ Security and authentication considerations
- ✅ Performance and scalability design
- ✅ RESTful best practices and conventions
- ✅ Clear documentation and examples

The API is designed to be intuitive, maintainable, and scalable, following industry standards and best practices for REST API design.
