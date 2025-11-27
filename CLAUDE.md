# CLAUDE.md - AI Assistant Guide for HotelShoma

## Project Overview

**HotelShoma** is a hotel management system repository. This document provides comprehensive guidance for AI assistants working on this codebase.

**Repository:** bjncrypto/hotelshoma
**Current Status:** Initial setup phase
**Last Updated:** 2025-11-27

---

## Repository Structure

As this is a new repository, the structure will evolve. Common patterns for hotel management systems include:

```
hotelshoma/
├── src/                    # Source code
│   ├── api/               # API endpoints and routes
│   ├── components/        # Reusable UI components (if frontend)
│   ├── models/            # Data models and schemas
│   ├── services/          # Business logic and services
│   ├── controllers/       # Request handlers
│   ├── middleware/        # Custom middleware
│   ├── utils/             # Utility functions
│   └── config/            # Configuration files
├── tests/                 # Test files
│   ├── unit/             # Unit tests
│   ├── integration/      # Integration tests
│   └── e2e/              # End-to-end tests
├── docs/                  # Documentation
├── scripts/               # Build and deployment scripts
├── public/                # Static assets (if applicable)
└── config/                # Environment-specific configs
```

---

## Tech Stack Expectations

Based on typical hotel management systems, expect:

### Backend Possibilities
- **Node.js** with Express, Fastify, or NestJS
- **Python** with Django, Flask, or FastAPI
- **Java** with Spring Boot
- **Go** with Gin or Echo

### Frontend Possibilities
- **React** / **Next.js**
- **Vue.js** / **Nuxt.js**
- **Angular**
- **Svelte** / **SvelteKit**

### Database Possibilities
- **PostgreSQL** (recommended for relational data)
- **MongoDB** (for document storage)
- **MySQL** / **MariaDB**
- **Redis** (for caching and sessions)

### Common Features to Expect
- Room booking and reservation management
- Guest check-in/check-out
- Inventory management
- Billing and payment processing
- Staff management
- Reporting and analytics
- Multi-property support
- Calendar and availability tracking

---

## Development Workflow

### 1. Before Making Changes

**ALWAYS:**
- Read existing code before proposing changes
- Check for similar implementations in the codebase
- Understand the current architecture and patterns
- Look for configuration files to understand the setup

**NEVER:**
- Assume functionality without verification
- Make changes to code you haven't read
- Introduce breaking changes without discussion
- Skip reading tests that cover the area you're modifying

### 2. Code Analysis Approach

When exploring the codebase:
1. Start with configuration files (package.json, requirements.txt, etc.)
2. Review the main entry point (index.js, main.py, etc.)
3. Examine the routing/API structure
4. Understand the data models
5. Check existing tests for expected behavior
6. Look for documentation in comments and docs/

### 3. Making Changes

**Principles:**
- **Minimal changes**: Only modify what's necessary
- **Consistency**: Follow existing patterns and conventions
- **Security first**: Prevent SQL injection, XSS, CSRF, and other vulnerabilities
- **No over-engineering**: Avoid premature optimization and abstraction
- **Test coverage**: Maintain or improve test coverage

**Security Checklist:**
- [ ] Input validation on all user inputs
- [ ] SQL/NoSQL injection prevention (use parameterized queries)
- [ ] XSS prevention (sanitize outputs)
- [ ] Authentication and authorization checks
- [ ] Secure password handling (hashing with bcrypt/argon2)
- [ ] Rate limiting on sensitive endpoints
- [ ] HTTPS enforcement
- [ ] Sensitive data encryption
- [ ] Proper error handling (no stack traces to users)
- [ ] CORS configuration
- [ ] Content Security Policy headers

### 4. Testing Strategy

**Test Levels:**
1. **Unit Tests**: Individual functions and methods
2. **Integration Tests**: API endpoints and service interactions
3. **E2E Tests**: Full user workflows

**Coverage Targets:**
- Minimum 70% code coverage
- 100% coverage for critical paths (payments, bookings, auth)
- All edge cases and error conditions

### 5. Git Workflow

**Branch Naming:**
- `feature/description` - New features
- `fix/description` - Bug fixes
- `refactor/description` - Code refactoring
- `docs/description` - Documentation updates
- `test/description` - Test additions/updates

**Commit Messages:**
Follow conventional commits:
```
<type>(<scope>): <subject>

<body>

<footer>
```

Types: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`

**Examples:**
```
feat(booking): add room availability check endpoint

Implements real-time availability checking with Redis caching
for improved performance on high-traffic periods.

Closes #123
```

```
fix(auth): prevent JWT token expiration edge case

Fixed race condition where tokens could be used immediately
after expiration due to clock skew.
```

**Commit Guidelines:**
- Keep commits atomic and focused
- Write clear, descriptive messages
- Reference issue numbers when applicable
- Don't commit secrets or sensitive data
- Run tests before committing

### 6. Code Review Checklist

Before submitting changes:
- [ ] Code follows existing style and conventions
- [ ] All tests pass
- [ ] New tests added for new functionality
- [ ] No console.log or debug statements
- [ ] No commented-out code
- [ ] Error handling is comprehensive
- [ ] Security vulnerabilities addressed
- [ ] Performance considerations reviewed
- [ ] Documentation updated if needed
- [ ] Breaking changes are documented

---

## Coding Conventions

### General Principles

1. **Readability over cleverness**: Code should be self-documenting
2. **DRY (Don't Repeat Yourself)**: But avoid premature abstraction
3. **KISS (Keep It Simple, Stupid)**: Simple solutions are better
4. **YAGNI (You Aren't Gonna Need It)**: Don't build for hypothetical futures
5. **Separation of Concerns**: Each module has a single responsibility

### Naming Conventions

**Variables and Functions:**
- Use descriptive names: `getUserById` not `getUser`
- Boolean variables: `isAvailable`, `hasPermission`, `shouldRedirect`
- Avoid abbreviations unless standard: `req`, `res`, `id`, `url`

**Constants:**
- UPPERCASE_WITH_UNDERSCORES: `MAX_ROOM_CAPACITY`, `API_BASE_URL`

**Classes and Types:**
- PascalCase: `BookingService`, `RoomModel`, `PaymentController`

**Files:**
- Lowercase with hyphens or underscores: `booking-service.js`, `room_model.py`
- Match the primary export: `BookingService` → `booking-service.js`

### Code Organization

**File Size:**
- Keep files under 300 lines when possible
- Split large files by responsibility

**Function Length:**
- Aim for functions under 50 lines
- Extract complex logic into helper functions

**Import Order:**
1. External dependencies
2. Internal modules (absolute imports)
3. Relative imports
4. Types/interfaces (if TypeScript)

### Error Handling

**DO:**
```javascript
// Proper error handling with context
try {
  const booking = await bookingService.create(data);
  return booking;
} catch (error) {
  logger.error('Failed to create booking', { error, data });
  throw new BookingError('Unable to create booking', { cause: error });
}
```

**DON'T:**
```javascript
// Silent failures or generic errors
try {
  await bookingService.create(data);
} catch (error) {
  console.log(error); // Never use console.log
  throw new Error('Error'); // Too generic
}
```

### Comments and Documentation

**When to Comment:**
- Complex business logic that isn't obvious
- Workarounds for third-party bugs
- Performance optimizations
- Security considerations

**When NOT to Comment:**
- Obvious code: `// increment counter` for `counter++`
- What the code does (code should be self-explanatory)
- Commented-out code (delete it, git remembers)

**Documentation:**
- JSDoc/docstrings for public APIs
- README for each major module
- API documentation (OpenAPI/Swagger)
- Architecture decision records (ADRs) for major decisions

---

## Database Guidelines

### Schema Design

**Conventions:**
- Table names: plural lowercase (`bookings`, `rooms`, `guests`)
- Column names: snake_case (`created_at`, `room_number`, `guest_id`)
- Primary keys: `id` (auto-increment or UUID)
- Foreign keys: `<table>_id` (`booking_id`, `user_id`)
- Timestamps: `created_at`, `updated_at`, `deleted_at` (if soft delete)

### Migrations

- Always create migrations for schema changes
- Never modify existing migrations that are in production
- Test migrations on development data first
- Include both up and down migrations
- Document breaking changes

### Queries

**DO:**
```sql
-- Use indexes for frequently queried columns
CREATE INDEX idx_bookings_check_in_date ON bookings(check_in_date);

-- Use prepared statements to prevent injection
SELECT * FROM bookings WHERE guest_id = $1 AND status = $2
```

**DON'T:**
```sql
-- String concatenation (SQL injection risk!)
SELECT * FROM bookings WHERE guest_id = '" + guestId + "'
```

---

## API Design

### RESTful Conventions

**Resource Naming:**
- Use nouns, not verbs: `/bookings` not `/getBookings`
- Plural for collections: `/rooms`, `/guests`
- Nested resources: `/hotels/:hotelId/rooms`

**HTTP Methods:**
- `GET`: Retrieve resources (idempotent, no side effects)
- `POST`: Create new resources
- `PUT`: Full update of existing resource
- `PATCH`: Partial update of existing resource
- `DELETE`: Remove resource

**Status Codes:**
- `200 OK`: Successful GET, PUT, PATCH, DELETE
- `201 Created`: Successful POST
- `204 No Content`: Successful DELETE with no response body
- `400 Bad Request`: Invalid input
- `401 Unauthorized`: Missing or invalid authentication
- `403 Forbidden`: Authenticated but not authorized
- `404 Not Found`: Resource doesn't exist
- `409 Conflict`: Resource conflict (e.g., double booking)
- `422 Unprocessable Entity`: Validation errors
- `500 Internal Server Error`: Server-side error

**Response Format:**
```json
{
  "success": true,
  "data": {
    "id": "123",
    "roomNumber": "101",
    "status": "available"
  },
  "meta": {
    "timestamp": "2025-11-27T19:00:00Z",
    "version": "1.0"
  }
}
```

**Error Format:**
```json
{
  "success": false,
  "error": {
    "code": "ROOM_NOT_AVAILABLE",
    "message": "Room 101 is not available for selected dates",
    "details": {
      "roomId": "101",
      "requestedDates": ["2025-12-01", "2025-12-05"],
      "nextAvailable": "2025-12-10"
    }
  }
}
```

### Pagination

```json
{
  "data": [...],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 150,
    "totalPages": 8,
    "hasNext": true,
    "hasPrev": false
  }
}
```

---

## Security Best Practices

### Authentication & Authorization

1. **Password Security:**
   - Hash with bcrypt (cost factor 10+) or argon2
   - Minimum 8 characters, recommend 12+
   - Never log or display passwords

2. **JWT Tokens:**
   - Short expiration (15-30 minutes for access tokens)
   - Refresh token rotation
   - Store secrets in environment variables
   - Validate on every request

3. **Session Management:**
   - Secure, HttpOnly, SameSite cookies
   - Session timeout after inactivity
   - Logout clears all session data

4. **Authorization:**
   - Role-based access control (RBAC)
   - Verify permissions on every protected endpoint
   - Principle of least privilege

### Input Validation

```javascript
// Example validation schema
const bookingSchema = {
  checkIn: {
    type: 'date',
    required: true,
    min: new Date()
  },
  checkOut: {
    type: 'date',
    required: true,
    validate: (value, data) => value > data.checkIn
  },
  roomId: {
    type: 'string',
    required: true,
    pattern: /^[0-9a-f]{24}$/
  },
  guests: {
    type: 'number',
    required: true,
    min: 1,
    max: 10
  }
};
```

### Data Protection

- **Encryption at rest**: Sensitive data in database
- **Encryption in transit**: HTTPS/TLS only
- **PII handling**: Comply with GDPR/CCPA
- **Audit logging**: Track sensitive operations
- **Data retention**: Implement proper deletion policies

---

## Performance Optimization

### Caching Strategy

1. **Redis/Memcached** for:
   - Session data
   - Frequently accessed room availability
   - Rate limiting counters
   - Temporary booking holds

2. **Database Query Optimization:**
   - Use indexes appropriately
   - Avoid N+1 queries
   - Use connection pooling
   - Implement query timeouts

3. **API Response Caching:**
   - Cache-Control headers
   - ETag for conditional requests
   - CDN for static assets

### Database Performance

- **Indexes**: On foreign keys and frequently queried columns
- **Query optimization**: Use EXPLAIN to analyze slow queries
- **Pagination**: Always paginate large result sets
- **Batch operations**: Bulk inserts/updates when possible
- **Read replicas**: For read-heavy workloads

### Frontend Performance (if applicable)

- Code splitting and lazy loading
- Image optimization
- Minification and compression
- Service workers for offline support
- Debounce/throttle user inputs

---

## Testing Guidelines

### Unit Tests

**Test Structure:**
```javascript
describe('BookingService', () => {
  describe('createBooking', () => {
    it('should create a booking with valid data', async () => {
      // Arrange
      const bookingData = { ... };

      // Act
      const result = await bookingService.createBooking(bookingData);

      // Assert
      expect(result.id).toBeDefined();
      expect(result.status).toBe('confirmed');
    });

    it('should throw error when room is unavailable', async () => {
      // Test error cases
    });
  });
});
```

**What to Test:**
- Happy path scenarios
- Edge cases and boundaries
- Error conditions
- Input validation
- Business logic rules

**Mocking:**
- Mock external services
- Mock database calls
- Use test fixtures for data

### Integration Tests

Test API endpoints:
```javascript
describe('POST /api/bookings', () => {
  it('should create booking and return 201', async () => {
    const response = await request(app)
      .post('/api/bookings')
      .send(validBookingData)
      .expect(201);

    expect(response.body.data.id).toBeDefined();
  });
});
```

### E2E Tests

Test complete user workflows:
- Search for available rooms
- Select room and dates
- Complete booking
- Verify confirmation email
- Check-in process
- Check-out and payment

---

## Deployment and DevOps

### Environment Variables

**Required Variables:**
```bash
# Application
NODE_ENV=production
PORT=3000
API_VERSION=v1

# Database
DATABASE_URL=postgresql://user:pass@host:5432/dbname
REDIS_URL=redis://host:6379

# Authentication
JWT_SECRET=your-secret-key
JWT_EXPIRATION=15m
REFRESH_TOKEN_SECRET=your-refresh-secret
REFRESH_TOKEN_EXPIRATION=7d

# Email (if applicable)
SMTP_HOST=smtp.example.com
SMTP_PORT=587
SMTP_USER=user
SMTP_PASS=pass

# Payment Gateway (if applicable)
STRIPE_SECRET_KEY=sk_test_...
STRIPE_PUBLISHABLE_KEY=pk_test_...

# Logging
LOG_LEVEL=info
```

### CI/CD Pipeline

**Stages:**
1. **Lint**: Code style checks
2. **Test**: Run all tests
3. **Build**: Create production build
4. **Security Scan**: Check for vulnerabilities
5. **Deploy**: Deploy to staging/production

**Pre-deployment Checks:**
- All tests passing
- No security vulnerabilities
- Database migrations ready
- Environment variables configured
- Rollback plan prepared

---

## Monitoring and Logging

### Logging Levels

- **ERROR**: Errors that need immediate attention
- **WARN**: Potential issues or deprecated usage
- **INFO**: General application flow
- **DEBUG**: Detailed debugging information

### What to Log

**DO Log:**
- API requests and responses (sanitize sensitive data)
- Database query errors
- Authentication attempts (success and failure)
- Business logic errors
- Performance metrics

**DON'T Log:**
- Passwords or tokens
- Credit card numbers
- Personal identification numbers
- Full user objects (log IDs instead)

### Monitoring Metrics

- **Application**: Response times, error rates, throughput
- **Infrastructure**: CPU, memory, disk usage
- **Database**: Query performance, connection pool
- **Business**: Booking conversion rate, revenue, occupancy

---

## Common Patterns and Anti-Patterns

### ✅ Good Patterns

**1. Service Layer Pattern:**
```javascript
// Separate business logic from controllers
class BookingService {
  async createBooking(data) {
    // Validation
    // Business logic
    // Database operations
    // Return result
  }
}
```

**2. Repository Pattern:**
```javascript
// Abstract database operations
class BookingRepository {
  async findById(id) { }
  async create(data) { }
  async update(id, data) { }
  async delete(id) { }
}
```

**3. Middleware for Cross-Cutting Concerns:**
```javascript
app.use(authMiddleware);
app.use(rateLimitMiddleware);
app.use(loggingMiddleware);
```

### ❌ Anti-Patterns to Avoid

**1. God Objects:**
Don't create classes that do everything

**2. Callback Hell:**
Use async/await instead of nested callbacks

**3. Global State:**
Avoid global variables, use dependency injection

**4. Tight Coupling:**
Depend on abstractions, not concrete implementations

**5. Premature Optimization:**
Make it work, make it right, then make it fast

---

## Domain-Specific Concepts

### Hotel Management Terminology

- **Booking/Reservation**: Customer's request for accommodation
- **Check-in**: Guest arrival and room assignment
- **Check-out**: Guest departure and final billing
- **Occupancy Rate**: Percentage of rooms occupied
- **ADR (Average Daily Rate)**: Average revenue per occupied room
- **RevPAR (Revenue Per Available Room)**: Total room revenue / available rooms
- **Overbooking**: Accepting more reservations than available rooms
- **Room Types**: Standard, Deluxe, Suite, etc.
- **Rate Plans**: Pricing strategies (rack rate, corporate, package deals)
- **Inventory**: Available rooms for sale
- **Channel Manager**: Distribution across booking platforms
- **PMS (Property Management System)**: Core hotel operations software

### Business Rules to Consider

1. **Booking Rules:**
   - Minimum/maximum stay requirements
   - Advance booking windows
   - Cancellation policies
   - No-show policies
   - Deposit requirements

2. **Pricing Rules:**
   - Dynamic pricing based on demand
   - Seasonal rates
   - Special event pricing
   - Early bird/last-minute discounts
   - Group booking rates

3. **Room Assignment:**
   - Room preferences and requests
   - Accessibility requirements
   - VIP guest handling
   - Room blocking for maintenance

4. **Inventory Management:**
   - Room availability calculation
   - Overbooking thresholds
   - Channel allocation
   - Block bookings for groups

---

## Troubleshooting Guide

### Common Issues

**1. Database Connection Errors:**
- Check DATABASE_URL environment variable
- Verify database server is running
- Check connection pool settings
- Review firewall rules

**2. Authentication Failures:**
- Verify JWT_SECRET is set
- Check token expiration settings
- Validate CORS configuration
- Review authentication middleware

**3. Payment Processing Errors:**
- Verify API keys are correct
- Check webhook endpoints
- Review error logs for specific errors
- Test with sandbox/test mode first

**4. Performance Issues:**
- Profile slow database queries
- Check for N+1 query problems
- Review caching strategy
- Analyze API response times

### Debug Mode

Enable detailed logging:
```bash
LOG_LEVEL=debug npm start
```

---

## AI Assistant Specific Guidelines

### When Analyzing Code

1. **Read before suggesting**: Always read files before proposing changes
2. **Understand patterns**: Identify and follow existing code patterns
3. **Check tests**: Review tests to understand expected behavior
4. **Security first**: Always consider security implications
5. **Ask when uncertain**: Request clarification rather than assuming

### When Writing Code

1. **Follow conventions**: Match existing code style
2. **Keep it simple**: Avoid over-engineering
3. **Test coverage**: Write tests for new code
4. **Document decisions**: Explain non-obvious choices
5. **No placeholders**: Complete implementations, no TODOs

### When Making Changes

1. **Atomic commits**: One logical change per commit
2. **Clear messages**: Descriptive commit messages
3. **Run tests**: Ensure all tests pass
4. **No debug code**: Remove console.logs and debug statements
5. **Update docs**: Keep documentation in sync

### Communication

1. **Be specific**: Reference exact file paths and line numbers
2. **Explain reasoning**: Why, not just what
3. **Acknowledge uncertainty**: Say when you don't know
4. **Provide context**: Help understand the bigger picture
5. **Suggest alternatives**: When multiple approaches exist

---

## Resources and References

### Documentation to Create

As the project grows, maintain:

- **API Documentation**: OpenAPI/Swagger specs
- **Database Schema**: ER diagrams and schema docs
- **Architecture Docs**: System design and component diagrams
- **Deployment Guide**: Step-by-step deployment instructions
- **Onboarding Guide**: New developer setup guide
- **Runbooks**: Operational procedures and troubleshooting

### External Resources

- OWASP Top 10: https://owasp.org/www-project-top-ten/
- REST API Guidelines: https://restfulapi.net/
- Semantic Versioning: https://semver.org/
- Conventional Commits: https://www.conventionalcommits.org/

---

## Changelog

### 2025-11-27
- Initial CLAUDE.md creation
- Established repository structure and conventions
- Defined development workflows
- Set security and testing guidelines

---

## Contributing

This document should be updated as the project evolves:

- When new patterns are established
- When technology choices are made
- When conventions change
- When new team members join
- After significant architectural decisions

**Last Review:** 2025-11-27
**Next Review:** When first major component is implemented

---

## Quick Reference

### Before Starting Work
- [ ] Pull latest changes from main branch
- [ ] Read relevant code sections
- [ ] Understand existing patterns
- [ ] Check for related tests

### Before Committing
- [ ] All tests pass
- [ ] No debug code
- [ ] Code follows conventions
- [ ] Security considerations addressed
- [ ] Documentation updated if needed

### Before Pushing
- [ ] Commits are atomic and well-described
- [ ] No secrets in code
- [ ] Ready for code review
- [ ] CI/CD will pass

---

**This is a living document. Update it as the project evolves.**
