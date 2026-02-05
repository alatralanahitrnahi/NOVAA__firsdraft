# Advanced Developer Guide: API Versioning, Webhooks, Git Best Practices & Mobile APIs

**For**: Senior Backend Developers, Team Leads  
**Version**: 1.0  
**Date**: February 5, 2026  

---

## 📋 TABLE OF CONTENTS

1. [API Versioning Strategy](#api-versioning-strategy)
2. [Webhooks Implementation](#webhooks-implementation)
3. [Git & GitHub Best Practices](#git--github-best-practices)
4. [Mobile API Development](#mobile-api-development)
5. [API Documentation Standards](#api-documentation-standards)
6. [Clear Goals & Examples](#clear-goals--examples)
7. [NOVAA-Specific Implementation](#novaa-specific-implementation)

---

## API VERSIONING STRATEGY

### Why Version APIs?

In NOVAA, as we scale from 5 pilot colleges to 500+ institutions, we need to maintain backward compatibility while introducing new features. API versioning allows us to:

- Support legacy clients during migration
- Introduce breaking changes safely
- Maintain multiple API versions simultaneously
- Ensure zero downtime during updates

### Versioning Approach: URI Path Versioning

**Recommended for NOVAA**: URI path versioning (`/api/v1/`, `/api/v2/`)

```javascript
// Current NOVAA API (v1)
GET /api/admissions/:id
POST /api/admissions/apply

// Future NOVAA API (v2) - with enhanced features
GET /api/v2/admissions/:id
POST /api/v2/admissions/apply
```

### Implementation Pattern

```javascript
// backend/src/app.js
const express = require('express');
const v1Routes = require('./routes/v1');
const v2Routes = require('./routes/v2');

const app = express();

// Versioned routes
app.use('/api/v1', v1Routes);
app.use('/api/v2', v2Routes);

// Default to v1 for backward compatibility
app.use('/api', v1Routes);
```

### Version Migration Strategy

**Example: Upgrading NOVAA's Admission API**

**v1 (Current)**:
```javascript
// GET /api/v1/admissions/:id
{
  "_id": "65a3b2c1d4e5f6g7h8i9j0k1",
  "studentId": "65a3b2c1d4e5f6g7h8i9j0k2",
  "courseId": "BSC_CS",
  "status": "SUBMITTED",
  "documents": [...],
  "createdAt": "2026-01-15T10:30:00Z"
}
```

**v2 (Enhanced)**:
```javascript
// GET /api/v2/admissions/:id
{
  "_id": "65a3b2c1d4e5f6g7h8i9j0k1",
  "studentId": "65a3b2c1d4e5f6g7h8i9j0k2",
  "studentInfo": {
    "name": "Aditya Sharma",
    "email": "aditya@example.com",
    "phone": "+919876543210"
  },
  "courseId": "BSC_CS",
  "courseInfo": {
    "name": "Bachelor of Science in Computer Science",
    "duration": "3 years",
    "fees": 61800
  },
  "status": "SUBMITTED",
  "statusHistory": [
    {
      "status": "SUBMITTED",
      "timestamp": "2026-01-15T10:30:00Z",
      "by": "student"
    },
    {
      "status": "PENDING_VERIFICATION",
      "timestamp": "2026-01-15T11:00:00Z",
      "by": "admin"
    }
  ],
  "documents": [...],
  "notifications": {
    "smsSent": true,
    "emailSent": true
  },
  "createdAt": "2026-01-15T10:30:00Z",
  "updatedAt": "2026-01-15T11:00:00Z"
}
```

### Version Deprecation Timeline

```
v1 API: Created Jan 2026
v2 API: Released Mar 2026
v1 Deprecation Notice: Feb 2026 (6 months notice)
v1 Sunset: Sep 2026
```

### Version Header Alternative (For Mobile Apps)

For mobile applications where URI path changes are harder to implement:

```javascript
// Request header versioning
headers: {
  'X-API-Version': '2.1',
  'X-Client-Type': 'mobile-android',
  'X-Client-Version': '1.2.3'
}
```

### NOVAA Versioning Best Practices

1. **Semantic Versioning**: MAJOR.MINOR.PATCH (v1.2.3)
   - MAJOR: Breaking changes (v1 → v2)
   - MINOR: New features (v1.1 → v1.2)
   - PATCH: Bug fixes (v1.2.0 → v1.2.1)

2. **Feature Flags for Gradual Rollout**:
```javascript
// backend/src/config/features.js
const FEATURE_FLAGS = {
  'admissions_v2_enabled': process.env.ADMISSIONS_V2_ENABLED === 'true',
  'mobile_api_optimizations': true,
  'enhanced_notifications': false // Beta feature
};

module.exports = { FEATURE_FLAGS };
```

---

## WEBHOOKS IMPLEMENTATION

### What Are Webhooks?

Webhooks are HTTP callbacks that allow external services to notify your application about events in real-time. In NOVAA, we use webhooks for:

- Payment gateway notifications (Razorpay)
- Document verification status updates
- Attendance system integrations
- Third-party service notifications

### NOVAA Webhook Architecture

```
External Service (Razorpay) → Webhook Endpoint → NOVAA Backend → Process Event → Respond
```

### Webhook Security

**Critical Security Measures**:

```javascript
// backend/src/webhooks/razorpay.js
const crypto = require('crypto');
const { verifyRazorpaySignature } = require('../utils/security');

const handleRazorpayWebhook = async (req, res) => {
  try {
    // 1. Verify webhook signature
    const isValid = verifyRazorpaySignature(
      req.body,
      req.headers['x-razorpay-signature'],
      process.env.RAZORPAY_WEBHOOK_SECRET
    );

    if (!isValid) {
      console.error('Invalid webhook signature:', {
        ip: req.ip,
        userAgent: req.headers['user-agent']
      });
      return res.status(401).json({ error: 'Unauthorized' });
    }

    // 2. Parse event
    const event = req.body.event;
    const payload = req.body.payload;

    // 3. Process based on event type
    switch (event) {
      case 'payment.captured':
        await handlePaymentCaptured(payload.payment.entity);
        break;
      case 'payment.failed':
        await handlePaymentFailed(payload.payment.entity);
        break;
      case 'order.paid':
        await handleOrderPaid(payload.order.entity);
        break;
      default:
        console.warn('Unhandled webhook event:', event);
        break;
    }

    // 4. Respond with 200 OK (important for webhook reliability)
    res.status(200).json({ received: true });
  } catch (error) {
    console.error('Webhook processing error:', error);
    res.status(500).json({ error: 'Internal server error' });
  }
};

// Security utility
const verifyRazorpaySignature = (payload, signature, secret) => {
  const expectedSignature = crypto
    .createHmac('sha256', secret)
    .update(JSON.stringify(payload))
    .digest('hex');

  return crypto.timingSafeEqual(
    Buffer.from(signature),
    Buffer.from(expectedSignature)
  );
};
```

### Webhook Reliability Patterns

**Idempotency for Webhooks**:

```javascript
// backend/src/services/webhookService.js
const processWebhookEvent = async (eventData) => {
  // Use event ID as idempotency key
  const idempotencyKey = eventData.id || eventData.razorpay_payment_id;
  
  // Check if already processed
  const existingRecord = await WebhookLog.findOne({
    idempotencyKey,
    eventType: eventData.event
  });

  if (existingRecord) {
    console.log('Duplicate webhook event, skipping:', idempotencyKey);
    return { processed: false, reason: 'duplicate' };
  }

  // Process event
  const result = await processEvent(eventData);

  // Log the event
  await WebhookLog.create({
    idempotencyKey,
    eventType: eventData.event,
    payload: eventData,
    processedAt: new Date(),
    status: 'success'
  });

  return { processed: true, result };
};
```

### Webhook Retry Logic

```javascript
// backend/src/utils/webhookRetry.js
const processWithRetry = async (handler, maxRetries = 3) => {
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      return await handler();
    } catch (error) {
      if (attempt === maxRetries) {
        console.error('Webhook failed after all retries:', error);
        
        // Log failure for manual processing
        await WebhookFailureLog.create({
          handler: handler.name,
          error: error.message,
          attemptCount: attempt,
          timestamp: new Date()
        });
        
        throw error;
      }

      // Exponential backoff: 1s, 2s, 4s
      await new Promise(resolve => 
        setTimeout(resolve, Math.pow(2, attempt) * 1000)
      );
    }
  }
};
```

### NOVAA Webhook Examples

**Payment Webhook Handler**:

```javascript
// backend/src/webhooks/payment.js
const handlePaymentWebhook = async (paymentData) => {
  const transaction = await Transaction.findOne({
    razorpayOrderId: paymentData.order_id
  });

  if (!transaction) {
    throw new Error(`Transaction not found for order: ${paymentData.order_id}`);
  }

  // Update transaction status
  await Transaction.findByIdAndUpdate(transaction._id, {
    status: paymentData.status === 'captured' ? 'SUCCESS' : 'FAILED',
    razorpayPaymentId: paymentData.id,
    amount: paymentData.amount / 100, // Convert from paise to rupees
    failureReason: paymentData.status === 'failed' ? paymentData.error_reason : null
  });

  if (paymentData.status === 'captured') {
    // Generate receipt
    await generateAndSendReceipt(transaction._id);
    
    // Update admission status if payment completed
    await updateAdmissionStatus(transaction.admissionId, 'PAYMENT_COMPLETED');
  }
};
```

**Attendance Webhook Handler**:

```javascript
// backend/src/webhooks/attendance.js
const handleAttendanceWebhook = async (attendanceData) => {
  const { studentId, courseId, date, status } = attendanceData;

  // Validate student belongs to correct college
  const attendance = await Attendance.findOne({
    studentId,
    courseId,
    date,
    collegeId: attendanceData.collegeId
  });

  if (attendance) {
    // Update existing record
    await Attendance.findByIdAndUpdate(attendance._id, {
      status,
      qrScannedAt: new Date(),
      markedBy: attendanceData.markedBy
    });
  } else {
    // Create new record
    await Attendance.create({
      collegeId: attendanceData.collegeId,
      studentId,
      courseId,
      date,
      status,
      qrScannedAt: new Date(),
      markedBy: attendanceData.markedBy
    });
  }

  // Trigger notifications
  await sendAttendanceNotification(studentId, courseId, date, status);
};
```

---

## GIT & GITHUB BEST PRACTICES

### Branch Strategy for NOVAA

**GitFlow-inspired approach adapted for NOVAA**:

```
main (production-ready)
├── develop (integration branch)
├── feature/* (individual features)
├── hotfix/* (urgent fixes)
└── release/* (pre-release)
```

### Feature Branch Guidelines

**Naming Convention**:
```
feature/admission-enhancements
bugfix/payment-processing-error
hotfix/critical-security-fix
docs/api-documentation-update
refactor/database-optimization
```

**Feature Branch Lifecycle**:

```bash
# 1. Create feature branch from develop
git checkout develop
git pull origin develop
git checkout -b feature/admission-enhancements

# 2. Work on feature (commit frequently)
git add .
git commit -m "feat: add student info to admission response"

# 3. Sync with develop regularly
git checkout develop
git pull origin develop
git checkout feature/admission-enhancements
git rebase develop  # Keep feature branch updated

# 4. Push feature branch
git push -u origin feature/admission-enhancements

# 5. Create PR to develop (not main!)
# 6. After merge, delete feature branch
git branch -d feature/admission-enhancements
git push origin --delete feature/admission-enhancements
```

### Pull Request Best Practices

**PR Size Guidelines**:
- Small PR: 50-200 lines (review in 15 minutes)
- Medium PR: 200-500 lines (review in 30 minutes)
- Large PR: >500 lines (split into smaller PRs)

**PR Template for NOVAA**:

```markdown
## Description
Brief description of the changes made

## Type of Change
- [ ] New feature (non-breaking change which adds functionality)
- [ ] Bug fix (non-breaking change which fixes an issue)
- [ ] Breaking change (fix or feature that would cause existing functionality to not work as expected)
- [ ] Documentation update

## Testing Done
- [ ] Unit tests added/updated
- [ ] Integration tests passed
- [ ] Manual testing performed
- [ ] Tested on staging environment

## Checklist
- [ ] Code follows NOVAA conventions
- [ ] Multi-tenancy enforced (collegeId in all queries)
- [ ] Security considerations addressed
- [ ] Performance implications considered
- [ ] Documentation updated

## Related Issues
Closes #123
Related to #456

## Screenshots (if applicable)
[Add screenshots here]
```

### Commit Message Best Practices

**Conventional Commits for NOVAA**:

```
<type>(<scope>): <description>

<body>

<footer>
```

**Types for NOVAA**:
- `feat`: New feature (admissions, payments, attendance)
- `fix`: Bug fix
- `docs`: Documentation only changes
- `style`: Changes that do not affect meaning (white-space, formatting)
- `refactor`: Code change that neither fixes a bug nor adds a feature
- `perf`: Performance improvement
- `test`: Adding missing tests or correcting existing tests
- `build`: Changes that affect the build system
- `ci`: Changes to CI configuration files and scripts
- `chore`: Other changes that don't modify src or test files

**Examples**:
```bash
git commit -m "feat(admissions): add student info to admission response"
git commit -m "fix(payments): resolve GST calculation error"
git commit -m "docs(api): update payment webhook documentation"
git commit -m "refactor(auth): extract JWT validation logic"
git commit -m "test(admissions): add unit tests for document verification"
```

### Pull Request Review Process

**Reviewer Checklist**:
```markdown
## Code Quality
- [ ] Follows NOVAA coding standards
- [ ] Multi-tenancy properly implemented
- [ ] Error handling comprehensive
- [ ] Security vulnerabilities addressed
- [ ] Performance considerations made

## Functionality
- [ ] Feature works as described
- [ ] Edge cases handled
- [ ] No breaking changes introduced
- [ ] Backward compatibility maintained

## Testing
- [ ] Unit tests added/updated
- [ ] Integration tests pass
- [ ] Manual testing performed

## Documentation
- [ ] API documentation updated
- [ ] Code comments added where necessary
- [ ] README updated if needed
```

### Git Hygiene Practices

**Commit Frequency**:
- Commit early and often
- Each commit should represent a logical unit of work
- Don't commit broken code (but WIP commits are acceptable with clear messages)

**Example workflow**:
```bash
# Start working on feature
git checkout -b feature/payment-enhancements

# Initial setup
git add .
git commit -m "feat: initial payment enhancement setup"

# Add validation
git add .
git commit -m "feat: add payment validation middleware"

# Add business logic
git add .
git commit -m "feat: implement enhanced payment processing"

# Add tests
git add .
git commit -m "test: add payment enhancement test suite"

# Fix any issues found during testing
git add .
git commit -m "fix: resolve payment validation issues"

# Push to GitHub
git push -u origin feature/payment-enhancements
```

### Handling Merge Conflicts

**Prevention**:
- Keep feature branches updated with develop
- Communicate with team about overlapping work
- Break large features into smaller, independent PRs

**Resolution Process**:
```bash
# 1. Fetch latest changes
git fetch origin

# 2. Update your feature branch
git checkout develop
git pull origin develop
git checkout feature/my-feature
git rebase develop

# 3. If conflicts occur, resolve them
# Edit conflicted files to resolve <<<<<<<, =======, >>>>>>> markers

# 4. Stage resolved files
git add .

# 5. Continue rebase
git rebase --continue

# 6. Push updated branch
git push --force-with-lease origin feature/my-feature
```

### Pull Request Lifespan

**Time Limits**:
- Feature branches: Maximum 1 week open
- Hotfix branches: Maximum 2 days open
- PRs: Maximum 3 days for review and merge

**Cleanup Process**:
- Delete branches after merging
- Clean up old, unused branches regularly
- Use `git branch --merged` to find branches that can be deleted

---

## MOBILE API DEVELOPMENT

### Mobile-Specific Considerations for NOVAA

**Network Constraints**:
- Poor connectivity in rural colleges
- High latency on mobile networks
- Limited bandwidth considerations

**Device Constraints**:
- Varying screen sizes and resolutions
- Different processing power capabilities
- Battery life considerations

### Optimized Mobile Endpoints

**Reduced Payload Example**:

```javascript
// Mobile-optimized admission response
// GET /api/mobile/admissions/:id
{
  "id": "65a3b2c1d4e5f6g7h8i9j0k1",
  "status": "SUBMITTED",
  "statusText": "Application submitted, awaiting verification",
  "progress": 25,
  "lastUpdated": "2026-01-15T11:00:00Z",
  "nextAction": "Wait for document verification",
  "estimatedDays": 2
}

// vs Full desktop response (more detailed)
// GET /api/admissions/:id
{
  "id": "65a3b2c1d4e5f6g7h8i9j0k1",
  "studentId": "65a3b2c1d4e5f6g7h8i9j0k2",
  "studentInfo": {...},
  "courseId": "BSC_CS",
  "courseInfo": {...},
  "status": "SUBMITTED",
  "statusHistory": [...],
  "documents": [...],
  "notes": "...",
  "createdAt": "2026-01-15T10:30:00Z",
  "updatedAt": "2026-01-15T11:00:00Z"
}
```

### Mobile API Route Structure

```javascript
// backend/src/routes/mobile.js
const express = require('express');
const router = express.Router();

// Mobile-optimized routes
router.get('/admissions/:id', authenticate, async (req, res) => {
  const admission = await Admission.findOne({
    _id: req.params.id,
    collegeId: req.college._id
  });

  // Return minimal data for mobile
  res.json({
    id: admission._id,
    status: admission.status,
    statusText: getStatusText(admission.status),
    progress: calculateProgress(admission.status),
    lastUpdated: admission.updatedAt,
    nextAction: getNextAction(admission.status),
    estimatedDays: getEstimatedDays(admission.status)
  });
});

router.post('/attendance/mark', authenticate, async (req, res) => {
  // Mobile-optimized attendance marking
  const { studentId, courseId, qrCode } = req.body;
  
  // Validate QR code (mobile-specific)
  const isValidQR = await validateQRCode(qrCode, req.college._id);
  
  if (!isValidQR) {
    return res.status(400).json({ error: 'Invalid QR code' });
  }

  // Mark attendance
  const attendance = await Attendance.create({
    collegeId: req.college._id,
    studentId,
    courseId,
    date: new Date().toISOString().split('T')[0],
    status: 'PRESENT',
    qrScannedAt: new Date(),
    markedBy: req.user._id
  });

  res.json({ success: true, attendanceId: attendance._id });
});

module.exports = router;
```

### Mobile API Performance Optimization

**Caching Strategy**:

```javascript
// backend/src/middleware/mobileCache.js
const redis = require('redis');
const client = redis.createClient();

const mobileCache = async (req, res, next) => {
  if (req.headers['x-client-type'] === 'mobile') {
    const cacheKey = `mobile:${req.originalUrl}`;
    const cached = await client.get(cacheKey);
    
    if (cached) {
      return res.json(JSON.parse(cached));
    }
    
    // Override res.json to cache response
    const originalJson = res.json;
    res.json = function(data) {
      client.setex(cacheKey, 300, JSON.stringify(data)); // Cache for 5 minutes
      originalJson.call(this, data);
    };
  }
  
  next();
};

module.exports = mobileCache;
```

**Pagination for Mobile**:

```javascript
// Mobile-optimized pagination
router.get('/admissions', authenticate, async (req, res) => {
  const { page = 1, limit = 10, mobile = false } = req.query;
  
  const options = {
    page: parseInt(page),
    limit: parseInt(limit),
    sort: { createdAt: -1 }
  };
  
  if (mobile === 'true') {
    // Mobile-optimized response with minimal data
    const admissions = await Admission.paginate(
      { collegeId: req.college._id },
      {
        ...options,
        select: 'id status createdAt updatedAt'
      }
    );
    
    res.json({
      data: admissions.docs.map(adm => ({
        id: adm._id,
        status: adm.status,
        updatedAt: adm.updatedAt
      })),
      pagination: {
        page: admissions.page,
        totalPages: admissions.totalPages,
        totalDocs: admissions.totalDocs
      }
    });
  } else {
    // Full desktop response
    const admissions = await Admission.paginate(
      { collegeId: req.college._id },
      options
    );
    
    res.json(admissions);
  }
});
```

### Mobile-Specific Error Handling

```javascript
// backend/src/middleware/mobileErrors.js
const mobileErrorHandler = (err, req, res, next) => {
  const isMobile = req.headers['x-client-type'] === 'mobile';
  
  if (isMobile) {
    // Simplified error messages for mobile
    const mobileError = {
      error: true,
      message: getMobileErrorMessage(err.message),
      code: err.code || 'INTERNAL_ERROR',
      timestamp: new Date().toISOString()
    };
    
    res.status(err.statusCode || 500).json(mobileError);
  } else {
    // Full error details for desktop
    next(err);
  }
};

const getMobileErrorMessage = (originalMessage) => {
  // Map technical errors to user-friendly messages
  if (originalMessage.includes('duplicate')) {
    return 'This action has already been performed';
  }
  if (originalMessage.includes('authentication')) {
    return 'Session expired. Please log in again';
  }
  if (originalMessage.includes('network')) {
    return 'Network connection lost. Please check your internet';
  }
  
  return 'Something went wrong. Please try again';
};

module.exports = mobileErrorHandler;
```

### Mobile API Security

**Additional Security Headers**:

```javascript
// backend/src/middleware/mobileSecurity.js
const mobileSecurityHeaders = (req, res, next) => {
  if (req.headers['x-client-type'] === 'mobile') {
    // Add mobile-specific security headers
    res.setHeader('X-Content-Type-Options', 'nosniff');
    res.setHeader('X-Frame-Options', 'DENY');
    res.setHeader('X-XSS-Protection', '1; mode=block');
    
    // Mobile-specific rate limiting
    res.setHeader('X-RateLimit-Limit', '100');
    res.setHeader('X-RateLimit-Remaining', '99');
  }
  
  next();
};

module.exports = mobileSecurityHeaders;
```

---

## API DOCUMENTATION STANDARDS

### API Documentation Structure for NOVAA

**Documentation Template**:

```markdown
# API Endpoint: [Endpoint Name]

## Overview
Brief description of what this endpoint does and when to use it.

## HTTP Method & URL
```
[METHOD] /api/[version]/[resource]/[action]
```

## Authentication
- Required: [Yes/No]
- Type: [Bearer Token, API Key, etc.]
- Headers: `Authorization: Bearer {token}`

## Request Parameters

### Path Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| id | string | Yes | Unique identifier for the resource |

### Query Parameters
| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| page | integer | No | 1 | Page number for pagination |
| limit | integer | No | 20 | Items per page |

### Request Body
```json
{
  "field1": "value1",
  "field2": "value2"
}
```

## Response Format

### Success Response (200/201)
```json
{
  "success": true,
  "data": {
    // Response data structure
  }
}
```

### Error Response (4xx/5xx)
```json
{
  "success": false,
  "message": "Error description",
  "error": "ERROR_CODE"
}
```

## HTTP Status Codes
- 200: Success
- 201: Created
- 400: Bad Request
- 401: Unauthorized
- 403: Forbidden
- 404: Not Found
- 500: Internal Server Error

## Example Usage
```javascript
fetch('/api/v1/admissions/123', {
  method: 'GET',
  headers: {
    'Authorization': 'Bearer token123',
    'Content-Type': 'application/json'
  }
})
.then(response => response.json())
.then(data => console.log(data));
```

## Rate Limits
- Requests per minute: 100
- Burst limit: 200

## Notes
- Multi-tenancy enforced: Only data from authenticated college returned
- Caching: Response cached for 5 minutes
```

### NOVAA API Documentation Example

**Admission Status API Documentation**:

```markdown
# API Endpoint: Get Admission Status

## Overview
Retrieve the current status of a student's admission application. This endpoint returns minimal information suitable for both mobile and desktop clients.

## HTTP Method & URL
```
GET /api/v1/admissions/:id
```

## Authentication
- Required: Yes
- Type: Bearer Token
- Headers: `Authorization: Bearer {jwt_token}`
- Additional: `X-College-Code: {college_code}`

## Request Parameters

### Path Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| id | string | Yes | Unique admission ID |

### Headers
| Header | Required | Description |
|--------|----------|-------------|
| Authorization | Yes | Bearer token from authentication |
| X-College-Code | Yes | College code for multi-tenancy |

## Response Format

### Success Response (200)
```json
{
  "success": true,
  "data": {
    "id": "65a3b2c1d4e5f6g7h8i9j0k1",
    "status": "SUBMITTED",
    "statusText": "Application submitted, awaiting verification",
    "progress": 25,
    "lastUpdated": "2026-01-15T11:00:00Z",
    "nextAction": "Wait for document verification",
    "estimatedDays": 2,
    "documents": [
      {
        "category": "MARKSHEET",
        "status": "PENDING_VERIFICATION",
        "uploadedAt": "2026-01-15T10:30:00Z"
      }
    ]
  }
}
```

### Error Response (404)
```json
{
  "success": false,
  "message": "Admission not found",
  "error": "ADMISSION_NOT_FOUND"
}
```

### Error Response (403)
```json
{
  "success": false,
  "message": "Access denied - admission belongs to another college",
  "error": "ACCESS_DENIED"
}
```

## HTTP Status Codes
- 200: Admission found and returned successfully
- 400: Invalid admission ID format
- 401: Authentication token missing or invalid
- 403: Access denied (wrong college context)
- 404: Admission not found
- 500: Internal server error

## Example Usage
```javascript
// JavaScript example
fetch('/api/v1/admissions/65a3b2c1d4e5f6g7h8i9j0k1', {
  method: 'GET',
  headers: {
    'Authorization': 'Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...',
    'X-College-Code': 'STX_MUMBAI',
    'Content-Type': 'application/json'
  }
})
.then(response => response.json())
.then(result => {
  if (result.success) {
    console.log('Admission status:', result.data.status);
    console.log('Progress:', result.data.progress + '%');
  } else {
    console.error('Error:', result.message);
  }
});
```

## Rate Limits
- Requests per minute: 100 per IP
- Burst limit: 200 per IP

## Notes
- Multi-tenancy enforced: Only returns admissions from the authenticated college
- Caching: Response cached for 30 seconds to reduce database load
- Security: College isolation verified server-side, not client-side
```

### Interactive Documentation with Swagger/OpenAPI

```javascript
// backend/src/docs/admissionDoc.js
/**
 * @swagger
 * /api/v1/admissions/{id}:
 *   get:
 *     summary: Get admission status
 *     description: Retrieve the current status of a student's admission application
 *     tags: [Admissions]
 *     security:
 *       - bearerAuth: []
 *     parameters:
 *       - in: path
 *         name: id
 *         required: true
 *         schema:
 *           type: string
 *         description: Unique admission ID
 *       - in: header
 *         name: X-College-Code
 *         required: true
 *         schema:
 *           type: string
 *         description: College code for multi-tenancy
 *     responses:
 *       200:
 *         description: Admission status retrieved successfully
 *         content:
 *           application/json:
 *             schema:
 *               type: object
 *               properties:
 *                 success:
 *                   type: boolean
 *                   example: true
 *                 data:
 *                   $ref: '#/components/schemas/AdmissionStatus'
 *       404:
 *         description: Admission not found
 *         content:
 *           application/json:
 *             schema:
 *               $ref: '#/components/schemas/ErrorResponse'
 */

/**
 * @swagger
 * components:
 *   schemas:
 *     AdmissionStatus:
 *       type: object
 *       properties:
 *         id:
 *           type: string
 *           description: Unique admission ID
 *         status:
 *           type: string
 *           description: Current status of the admission
 *           enum: [DRAFT, SUBMITTED, PENDING_VERIFICATION, VERIFIED, REJECTED, APPROVED]
 *         statusText:
 *           type: string
 *           description: Human-readable status description
 *         progress:
 *           type: integer
 *           description: Progress percentage (0-100)
 *         lastUpdated:
 *           type: string
 *           format: date-time
 *           description: Last update timestamp
 *         nextAction:
 *           type: string
 *           description: Recommended next action
 *         estimatedDays:
 *           type: integer
 *           description: Estimated days for next status update
 */
```

### API Version Documentation

**Version Migration Guide**:

```markdown
# API Version Migration Guide: v1 to v2

## Overview
This guide explains the differences between NOVAA API v1 and v2, and how to migrate your applications.

## Breaking Changes

### 1. Enhanced Response Format
**v1 Response**:
```json
{
  "id": "123",
  "status": "SUBMITTED"
}
```

**v2 Response**:
```json
{
  "id": "123",
  "status": "SUBMITTED",
  "statusHistory": [...],
  "studentInfo": {...},
  "courseInfo": {...}
}
```

### 2. New Required Fields
- `X-Client-Type` header now required for all requests
- `client_version` parameter required in some endpoints

### 3. Changed Field Names
- `student_id` → `studentId` (camelCase)
- `created_at` → `createdAt` (camelCase)

## Migration Steps

### Step 1: Update API Endpoints
Change all `/api/v1/` to `/api/v2/`

### Step 2: Update Request Headers
Add required headers:
```
X-Client-Type: web|mobile-ios|mobile-android
X-Client-Version: 1.0.0
```

### Step 3: Handle New Response Fields
Update your code to handle additional fields in responses.

### Step 4: Test Thoroughly
Test all API calls with v2 endpoints before going live.

## Timeline
- v2 Release: March 1, 2026
- v1 Deprecation Notice: February 1, 2026
- v1 Sunset: September 1, 2026
```

---

## CLEAR GOALS & EXAMPLES

### Setting Clear Development Goals for NOVAA

**SMART Goals Framework**:
- **Specific**: Clearly defined outcomes
- **Measurable**: Quantifiable metrics
- **Achievable**: Realistic with available resources
- **Relevant**: Aligned with NOVAA objectives
- **Time-bound**: Clear deadlines

### Example Goals for NOVAA Modules

#### Goal 1: Admission Module Enhancement
**Objective**: Improve admission processing speed and user experience

**Specific**:
- Reduce admission status update time from 48 hours to 24 hours
- Implement real-time status notifications
- Add mobile-optimized admission tracking

**Measurable**:
- 95% of applications processed within 24 hours
- 90% of students receive status updates within 1 hour
- Mobile app adoption rate: 70% of students

**Achievable**:
- Leverage existing NOVAA infrastructure
- Use webhook-based notifications
- Implement caching for faster responses

**Relevant**:
- Aligns with NOVAA's mission to reduce administrative burden
- Improves student satisfaction
- Supports scalability goals

**Time-bound**:
- Complete by March 15, 2026
- Milestone 1: Notification system (Week 1)
- Milestone 2: Mobile optimization (Week 2)
- Milestone 3: Performance improvements (Week 3)

#### Goal 2: Payment Processing Reliability
**Objective**: Achieve 99.5% payment processing success rate

**Specific**:
- Implement robust webhook handling
- Add payment retry mechanisms
- Enhance error reporting

**Measurable**:
- Payment success rate: 99.5%
- Failed payment recovery rate: 95%
- Average time to detect failed payments: <1 hour

**Achievable**:
- Use proven payment gateway integrations
- Implement comprehensive monitoring
- Create manual override processes

**Relevant**:
- Critical for NOVAA's revenue model
- Ensures GST compliance
- Builds trust with colleges and students

**Time-bound**:
- Complete by February 28, 2026
- Milestone 1: Webhook reliability (Week 1)
- Milestone 2: Error handling (Week 2)
- Milestone 3: Monitoring dashboard (Week 3)

### NOVAA Development Examples

#### Example 1: Feature Implementation Plan

**Feature**: Enhanced Admission Status Tracking

**Planning Phase**:
1. **Requirements Gathering** (Day 1)
   - Interview 3 college admins about current pain points
   - Survey 50 students about status notification preferences
   - Document findings in requirements document

2. **Technical Design** (Day 2)
   - Create API specification
   - Design database schema changes
   - Plan security considerations
   - Review with team lead

3. **Implementation** (Days 3-7)
   - Create database migrations
   - Implement API endpoints
   - Add webhook notifications
   - Write unit tests
   - Perform code review

4. **Testing** (Days 8-9)
   - Integration testing
   - Load testing
   - Security testing
   - User acceptance testing

5. **Deployment** (Day 10)
   - Deploy to staging
   - Final testing
   - Deploy to production
   - Monitor for issues

**Success Metrics**:
- 95% of status updates delivered within 1 hour
- Zero data integrity issues
- Positive feedback from pilot colleges

#### Example 2: Bug Fix Process

**Bug**: Payment webhook occasionally fails to update transaction status

**Investigation**:
1. **Reproduce**: Simulate high-volume payment scenario
2. **Log Analysis**: Check webhook processing logs
3. **Root Cause**: Identify race condition in status update logic
4. **Impact Assessment**: Determine affected transactions

**Fix Implementation**:
1. **Code Review**: Examine current webhook handler
2. **Solution Design**: Implement atomic status updates
3. **Testing**: Create stress test for webhook processing
4. **Rollback Plan**: Prepare in case of issues

**Verification**:
1. **Unit Tests**: Verify fix handles edge cases
2. **Integration Tests**: Test with actual payment gateway
3. **Staging**: Deploy and monitor for 24 hours
4. **Production**: Deploy with monitoring alerts enabled

#### Example 3: Performance Optimization

**Challenge**: Admission list endpoint slow for colleges with 1000+ applications

**Analysis**:
1. **Profiling**: Identify slow database queries
2. **Monitoring**: Track response times in production
3. **Bottleneck**: Find inefficient joins and filters
4. **Impact**: Document effect on user experience

**Optimization Strategy**:
1. **Database Indexes**: Add composite indexes for common queries
2. **Caching**: Implement Redis caching for frequent requests
3. **Pagination**: Optimize pagination queries
4. **Response Size**: Minimize data transfer

**Implementation**:
```javascript
// Optimized admission list query
const getAdmissionsOptimized = async (collegeId, filters, pagination) => {
  // Use indexed fields for filtering
  const query = { collegeId };
  
  if (filters.status) {
    query.status = filters.status;
  }
  
  // Efficient pagination with cursor-based approach
  const admissions = await Admission.find(query)
    .select('_id status createdAt') // Minimal fields for list view
    .sort({ createdAt: -1 })
    .skip((pagination.page - 1) * pagination.limit)
    .limit(pagination.limit)
    .lean(); // Use lean() for better performance
  
  return admissions;
};
```

**Results**:
- Response time reduced from 3.2s to 0.4s
- Database CPU usage decreased by 40%
- User satisfaction scores improved

---

## NOVAA-SPECIFIC IMPLEMENTATION

### Multi-Tenancy Enforcement

**Critical Security Pattern**:

```javascript
// backend/src/middleware/collegeContext.js
const enforceCollegeContext = async (req, res, next) => {
  const collegeCode = req.headers['x-college-code'];
  
  if (!collegeCode) {
    return res.status(400).json({
      success: false,
      message: 'Missing college context',
      error: 'MISSING_COLLEGE_CODE'
    });
  }

  const college = await College.findOne({
    code: collegeCode.toUpperCase(),
    isActive: true
  });

  if (!college) {
    return res.status(403).json({
      success: false,
      message: 'Invalid or inactive college',
      error: 'INVALID_COLLEGE'
    });
  }

  req.college = college;
  next();
};

// Monkey-patch Mongoose queries to enforce college isolation
const originalFind = mongoose.Query.prototype.find;
mongoose.Query.prototype.find = function(query = {}) {
  if (!query.collegeId && !query._id && this.model.modelName !== 'College') {
    throw new Error(
      `SECURITY VIOLATION: Query missing collegeId filter. ` +
      `Model: ${this.model.modelName}. ` +
      `Query: ${JSON.stringify(query)}`
    );
  }
  
  if (this.model.modelName !== 'College') {
    query.collegeId = req.college._id;
  }
  
  return originalFind.call(this, query);
};
```

### GST Calculation Engine

```javascript
// backend/src/utils/gstCalculator.js
const calculateGST = (feeComponents) => {
  let taxableAmount = 0;
  let exemptAmount = 0;
  const gstDetails = [];

  feeComponents.forEach(component => {
    if (component.gstEligible) {
      const gstAmount = component.amount * 0.18; // 18% GST
      taxableAmount += component.amount;
      
      gstDetails.push({
        component: component.name,
        amount: component.amount,
        gstRate: 18,
        gstAmount: parseFloat(gstAmount.toFixed(2)),
        total: component.amount + parseFloat(gstAmount.toFixed(2))
      });
    } else {
      exemptAmount += component.amount;
    }
  });

  const totalGst = parseFloat(gstDetails.reduce((sum, item) => sum + item.gstAmount, 0).toFixed(2));
  const totalAmount = taxableAmount + totalGst + exemptAmount;

  return {
    taxableAmount,
    exemptAmount,
    gstDetails,
    totalGst,
    totalAmount,
    breakdown: feeComponents.map(comp => ({
      name: comp.name,
      amount: comp.amount,
      gstEligible: comp.gstEligible,
      gstAmount: comp.gstEligible ? parseFloat((comp.amount * 0.18).toFixed(2)) : 0
    }))
  };
};
```

### QR Code Attendance System

```javascript
// backend/src/services/qrAttendance.js
const generateAttendanceQR = async (studentId, collegeId, date) => {
  const uniqueCode = crypto.randomBytes(8).toString('hex');
  const qrData = {
    studentId,
    collegeId,
    date,
    uniqueCode,
    timestamp: Date.now()
  };

  // Store in Redis for validation (expires in 24 hours)
  await redis.setex(
    `attendance_qr:${studentId}:${date}`,
    86400, // 24 hours
    JSON.stringify(qrData)
  );

  // Generate QR code
  const qrCode = await QRCode.toDataURL(JSON.stringify(qrData));
  return qrCode;
};

const validateAttendanceQR = async (qrCode, collegeId) => {
  try {
    const qrData = JSON.parse(qrCode);
    const storedData = await redis.get(`attendance_qr:${qrData.studentId}:${qrData.date}`);
    
    if (!storedData) {
      return { valid: false, reason: 'QR code expired or invalid' };
    }

    const parsedStored = JSON.parse(storedData);
    
    if (parsedStored.uniqueCode !== qrData.uniqueCode) {
      return { valid: false, reason: 'QR code tampered' };
    }

    // Verify student belongs to correct college
    const student = await Student.findOne({
      _id: qrData.studentId,
      collegeId
    });

    if (!student) {
      return { valid: false, reason: 'Student does not belong to this college' };
    }

    return { 
      valid: true, 
      studentId: qrData.studentId,
      date: qrData.date 
    };
  } catch (error) {
    return { valid: false, reason: 'Invalid QR code format' };
  }
};
```

### Webhook Monitoring Dashboard

```javascript
// backend/src/controllers/webhookMonitor.js
const getWebhookHealth = async (req, res) => {
  const thirtyMinutesAgo = new Date(Date.now() - 30 * 60 * 1000);
  
  // Get recent webhook statistics
  const stats = await WebhookLog.aggregate([
    {
      $match: {
        collegeId: req.college._id,
        processedAt: { $gte: thirtyMinutesAgo }
      }
    },
    {
      $group: {
        _id: '$eventType',
        count: { $sum: 1 },
        successRate: {
          $avg: { $cond: [{ $eq: ['$status', 'success'] }, 1, 0] }
        }
      }
    }
  ]);

  // Check for failed webhooks
  const failedCount = await WebhookLog.countDocuments({
    collegeId: req.college._id,
    status: 'failed',
    processedAt: { $gte: thirtyMinutesAgo }
  });

  // Get recent failures
  const recentFailures = await WebhookLog.find({
    collegeId: req.college._id,
    status: 'failed',
    processedAt: { $gte: thirtyMinutesAgo }
  }).sort({ processedAt: -1 }).limit(10);

  res.json({
    success: true,
    data: {
      stats,
      failedCount,
      recentFailures,
      timestamp: new Date(),
      health: failedCount === 0 ? 'healthy' : 'degraded'
    }
  });
};
```

---

## CONCLUSION

This advanced developer guide provides comprehensive coverage of essential topics for NOVAA development:

1. **API Versioning**: Strategies for maintaining backward compatibility while introducing new features
2. **Webhooks**: Secure and reliable implementation for real-time event processing
3. **Git/GitHub**: Best practices for collaborative development and code quality
4. **Mobile APIs**: Optimization techniques for mobile applications
5. **Documentation**: Standards for clear and comprehensive API documentation
6. **Goal Setting**: Framework for defining and achieving development objectives

Following these practices will ensure NOVAA continues to scale effectively while maintaining high code quality, security, and performance standards.

---

**Document Owner**: Technical Lead  
**Last Updated**: February 5, 2026  
**Next Review**: Monthly with development team