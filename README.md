# REST API Design Assignment - Library Management System

## Overview

This repository contains a comprehensive REST API design for a **Library Management System**. The design demonstrates strong understanding of:

- ✅ Business processes and domain modeling
- ✅ RESTful API principles and best practices
- ✅ Multiple use cases and user workflows
- ✅ Error scenarios and comprehensive error handling
- ✅ Security and authentication considerations
- ✅ Scalability and performance design

## 📚 Business Domain

The Library Management System manages:
- **Books** - Library catalog with ISBN, titles, authors, and availability
- **Members** - Library patrons with membership details
- **Loans** - Book checkouts with due dates and renewals
- **Reservations** - Book holds for unavailable items
- **Authors** - Book author information
- **Fines** - Late fee calculation and tracking

## 🗂️ Documentation

### Main Design Document
**[API_DESIGN.md](API_DESIGN.md)** - Complete REST API design documentation including:
- Detailed endpoint specifications
- Request/response formats
- Business rules and constraints
- Use cases and workflows
- Error handling strategies
- Authentication and authorization
- Rate limiting and pagination
- Design considerations

### OpenAPI Specification
**[openapi.yaml](openapi.yaml)** - Machine-readable API specification:
- Complete OpenAPI 3.0 definition
- All endpoints with parameters
- Request/response schemas
- Error responses
- Can be used with Swagger UI, Postman, or code generation tools

## 🎯 Key Features

### RESTful Design Principles
- **Resource-based URLs** - URLs represent resources, not actions
- **HTTP Methods** - Proper use of GET, POST, PATCH, DELETE
- **Stateless** - Each request contains all necessary information
- **HATEOAS** - Hypermedia links to related resources
- **Standard Status Codes** - Meaningful HTTP response codes

### API Capabilities
- 📖 **Book Management** - CRUD operations for library catalog
- 👥 **Member Management** - Registration and profile management
- 🔄 **Loan Operations** - Checkout, return, and renewal workflows
- 📌 **Reservations** - Queue system for unavailable books
- 🔐 **Authentication** - JWT-based security
- 🔍 **Search & Filter** - Flexible querying with pagination
- ⚠️ **Error Handling** - Comprehensive error responses

### Business Logic
- Members can borrow up to 5 books at a time
- 14-day loan period with one 7-day renewal
- Late fees: $0.50 per day per book
- Reservation system when books are unavailable
- Cannot borrow with overdue books or unpaid fines

## 📋 API Endpoints

### Books
```
GET    /v1/books              - List books (paginated)
GET    /v1/books/{id}         - Get book details
POST   /v1/books              - Create new book
PATCH  /v1/books/{id}         - Update book
DELETE /v1/books/{id}         - Delete book
```

### Members
```
GET    /v1/members            - List members
GET    /v1/members/{id}       - Get member details
POST   /v1/members            - Register new member
PATCH  /v1/members/{id}       - Update member
DELETE /v1/members/{id}       - Deactivate member
```

### Loans
```
GET    /v1/loans              - List loans
GET    /v1/loans/{id}         - Get loan details
POST   /v1/loans              - Create loan (checkout)
POST   /v1/loans/{id}/return  - Return book
POST   /v1/loans/{id}/renew   - Renew loan
```

### Reservations
```
GET    /v1/reservations       - List reservations
POST   /v1/reservations       - Create reservation
DELETE /v1/reservations/{id}  - Cancel reservation
```

### Authors
```
GET    /v1/authors            - List authors
GET    /v1/authors/{id}       - Get author details
```

### Authentication
```
POST   /v1/auth/login         - User login (get JWT token)
```

## 🔧 Technical Details

### Authentication
- JWT (JSON Web Token) based authentication
- Role-based access control (Admin, Librarian, Member)
- Token expiry: 1 hour with refresh tokens

### Pagination
- Query parameters: `page` (default: 1) and `limit` (default: 20, max: 100)
- Response includes pagination metadata and navigation links

### Error Handling
- Standard HTTP status codes
- Consistent error response format
- Detailed error codes and messages
- Field-level validation errors

### Rate Limiting
- Authenticated users: 1000 requests/hour
- Unauthenticated: 100 requests/hour
- Admins/Librarians: 5000 requests/hour

## 📝 Example Use Cases

### Use Case 1: Member Borrows a Book
1. Member searches for books: `GET /v1/books?search=gatsby`
2. Member views details: `GET /v1/books/book_123`
3. Librarian creates loan: `POST /v1/loans`
4. System validates availability and member status
5. Book checked out, availability updated

### Use Case 2: Renewing a Loan
1. Member views loans: `GET /v1/members/member_789/loans`
2. Member requests renewal: `POST /v1/loans/loan_321/renew`
3. System validates renewal eligibility
4. Due date extended by 7 days

### Use Case 3: Reserving an Unavailable Book
1. Member searches book: `GET /v1/books/book_123`
2. All copies checked out (availableCopies: 0)
3. Member creates reservation: `POST /v1/reservations`
4. System queues reservation
5. Member notified when book becomes available

## 🎨 Design Highlights

### Scalability Considerations
- Database indexing on frequently queried fields
- Response compression
- Caching with ETags
- Partial responses with field selection
- Batch operations support

### Security Measures
- HTTPS only
- Input validation and sanitization
- SQL injection prevention
- Rate limiting
- CORS configuration
- Security headers

### Error Scenarios Covered
- Resource not found (404)
- Validation errors (422)
- Authentication failures (401)
- Permission denied (403)
- Conflicts (409) - e.g., book unavailable, loan limit exceeded
- Rate limit exceeded (429)

## 🚀 Future Enhancements

- Digital content (e-books, audiobooks)
- Event management (library events)
- Recommendation engine
- User reviews and ratings
- Reading lists
- Notifications (email/SMS)
- Analytics and reporting
- Multi-branch support
- Inter-library loans

## 📖 How to Use This Design

1. **Review the Design Document**: Read [API_DESIGN.md](API_DESIGN.md) for complete specifications
2. **Explore the OpenAPI Spec**: Use [openapi.yaml](openapi.yaml) with:
   - Swagger UI for interactive documentation
   - Postman for API testing
   - Code generators for client SDKs
3. **Understand Use Cases**: Review example workflows and error scenarios
4. **Implement**: Use this design as a blueprint for implementation

## 🔗 Related Resources

- [OpenAPI Specification](https://swagger.io/specification/)
- [REST API Best Practices](https://restfulapi.net/)
- [HTTP Status Codes](https://httpstatuses.com/)
- [JWT Authentication](https://jwt.io/)

## 👨‍💻 Author

This REST API design demonstrates comprehensive understanding of:
- RESTful architecture principles
- Business domain modeling
- Error handling and security
- Scalability and performance
- API documentation and standards

---

**Note**: This is a design assignment demonstrating REST API design principles. For implementation details, see the comprehensive documentation in [API_DESIGN.md](API_DESIGN.md) and the OpenAPI specification in [openapi.yaml](openapi.yaml).