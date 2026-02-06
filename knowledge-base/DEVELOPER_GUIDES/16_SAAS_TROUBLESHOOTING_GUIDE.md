# SaaS Development Troubleshooting Guide: Common Problems & Solutions

**For**: New Developers Working on NOVAA SaaS Platform  
**Version**: 1.0  
**Date**: February 5, 2026  

---

## 📋 TABLE OF CONTENTS

1. [Introduction](#introduction)
2. [Common Development Issues](#common-development-issues)
3. [Multi-Tenancy Gotchas](#multi-tenancy-gotchas)
4. [Performance Bottlenecks](#performance-bottlenecks)
5. [Security Vulnerabilities](#security-vulnerabilities)
6. [Database Troubles](#database-troubles)
7. [API Development Tips](#api-development-tips)
8. [Debugging Strategies](#debugging-strategies)
9. [Testing Challenges](#testing-challenges)
10. [Deployment Issues](#deployment-issues)
11. [Quick Fixes & Workarounds](#quick-fixes--workarounds)
12. [Best Practices Summary](#best-practices-summary)

---

## INTRODUCTION

Welcome to NOVAA SaaS development! This guide addresses the most common challenges new developers face when working on multi-tenant SaaS applications. Each section includes real-world examples from NOVAA, common symptoms, root causes, and proven solutions.

### Why This Guide Exists

SaaS development presents unique challenges that traditional single-tenant applications don't have:
- **Data Isolation**: Ensuring one customer's data never leaks to another
- **Performance at Scale**: Maintaining speed as customer count grows
- **Security**: Protecting against sophisticated multi-tenant attacks
- **Compliance**: Meeting regulatory requirements across different regions

This guide will help you avoid common pitfalls and become productive faster.

---

## COMMON DEVELOPMENT ISSUES

### Issue 1: "My query is returning data from other colleges!"

**Symptoms**:
- Users see data that doesn't belong to their college
- Security audit flags data leakage
- Random test failures in CI/CD

**Root Cause**:
- Missing `collegeId` filter in database queries
- Bypassing tenant enforcement middleware
- Direct database access without proper context

**Solution**:

```javascript
// ❌ WRONG - Missing tenant filter
const students = await Student.find({ status: 'ACTIVE' });

// ✅ CORRECT - Always include tenant filter
const students = await Student.find({
  collegeId: req.college._id,  // Always include this!
  status: 'ACTIVE'
});

// ✅ BETTER - Use tenant-aware repository
class StudentRepository {
  constructor(tenantId) {
    this.tenantId = tenantId;
  }

  async findByStatus(status) {
    return await Student.find({
      collegeId: this.tenantId,  // Encapsulated tenant filter
      status
    });
  }
}
```

**Prevention**:
- Always run tenant enforcement middleware
- Use tenant-aware repositories
- Implement query monkey-patching to enforce tenant filters

### Issue 2: "The application is getting slower as we add more colleges"

**Symptoms**:
- Response times increasing over time
- Database queries taking longer
- Users complaining about slowness

**Root Cause**:
- Missing database indexes
- Inefficient queries without proper filtering
- Lack of caching strategies

**Solution**:

```javascript
// ❌ Inefficient query
const admissions = await Admission.find({
  status: 'SUBMITTED'
}).populate('studentId');

// ✅ Optimized with proper indexing and projection
// First, ensure proper indexes exist
// db.admissions.createIndex({ collegeId: 1, status: 1 })

const admissions = await Admission.find({
  collegeId: req.college._id,  // Tenant filter
  status: 'SUBMITTED'
}).select('_id status createdAt studentId')  // Only needed fields
 .populate('studentId', 'name email');       // Only needed fields from populate
```

### Issue 3: "My feature works locally but fails in production"

**Symptoms**:
- Tests pass locally but fail in CI/CD
- Features work in staging but break in production
- Intermittent failures that are hard to reproduce

**Root Cause**:
- Environment differences (different data, load, configuration)
- Race conditions that only appear under load
- Assumptions about data state that don't hold in production

**Solution**:
- Use identical environments across dev/staging/prod
- Implement proper mocking for external services
- Add comprehensive integration tests
- Use feature flags for gradual rollout

---

## MULTI-TENANCY GOTCHAS

### Gotcha 1: Direct Object Reference Attacks

**Problem**: Manipulating IDs to access other tenants' data

```javascript
// ❌ Vulnerable endpoint
app.get('/api/students/:id', async (req, res) => {
  const student = await Student.findById(req.params.id);  // No tenant check!
  res.json(student);
});

// ✅ Secure endpoint
app.get('/api/students/:id', async (req, res) => {
  const student = await Student.findOne({
    _id: req.params.id,
    collegeId: req.college._id  // Always verify tenant ownership
  });
  
  if (!student) {
    return res.status(404).json({ error: 'Student not found' });
  }
  
  res.json(student);
});
```

### Gotcha 2: Cross-Tenant File Access

**Problem**: Storing files without tenant isolation

```javascript
// ❌ Vulnerable file upload
const uploadFile = async (req, res) => {
  const fileName = req.file.originalname;
  const filePath = `uploads/${fileName}`;  // Same location for all tenants!
  
  // Save file...
  res.json({ url: `/files/${fileName}` });
};

// ✅ Secure file upload
const uploadFile = async (req, res) => {
  const fileName = `${req.college._id}_${Date.now()}_${req.file.originalname}`;
  const filePath = `uploads/${req.college.code}/${fileName}`;  // Tenant-specific path
  
  // Save file to tenant-specific location...
  res.json({ url: `/files/${req.college.code}/${fileName}` });
};
```

### Gotcha 3: Tenant Context Loss

**Problem**: Losing tenant context during async operations

```javascript
// ❌ Context loss during async operations
const processAdmission = async (admissionId) => {
  const admission = await Admission.findById(admissionId);
  // admission.collegeId is available here
  
  // But what if we need to do async work?
  setTimeout(async () => {
    // Context might be lost here!
    const college = await College.findById(admission.collegeId);
    // This might fail if admission.collegeId is not available
  }, 1000);
};

// ✅ Preserve context
const processAdmission = async (admissionId) => {
  const admission = await Admission.findById(admissionId);
  const collegeId = admission.collegeId;  // Store context explicitly
  
  setTimeout(async () => {
    // Use preserved context
    const college = await College.findById(collegeId);
    // This will work reliably
  }, 1000);
};
```

### Gotcha 4: Shared Configuration Issues

**Problem**: Configuration settings affecting all tenants

```javascript
// ❌ Shared configuration
let feeStructure = {}; // Global variable!

const loadFeeStructure = async (collegeId) => {
  feeStructure = await FeeStructure.findOne({ collegeId }); // Modifies global!
  return feeStructure;
};

// ✅ Tenant-specific configuration
const loadFeeStructure = async (collegeId) => {
  // Use tenant-specific cache or local variables
  const cacheKey = `fee_structure_${collegeId}`;
  let structure = await getFromCache(cacheKey);
  
  if (!structure) {
    structure = await FeeStructure.findOne({ collegeId });
    await setCache(cacheKey, structure, 3600); // Cache for 1 hour
  }
  
  return structure;
};
```

---

## PERFORMANCE BOTTLENECKS

### Bottleneck 1: N+1 Query Problem

**Problem**: Making multiple database queries in a loop

```javascript
// ❌ N+1 problem
const admissions = await Admission.find({ collegeId: req.college._id });

// This creates N queries (one for each admission)
const admissionsWithStudents = [];
for (const admission of admissions) {
  const student = await Student.findById(admission.studentId);  // N queries!
  admissionsWithStudents.push({
    ...admission.toObject(),
    student
  });
}

// ✅ Solution 1: Use populate
const admissions = await Admission.find({ collegeId: req.college._id })
  .populate('studentId', 'name email phone');

// ✅ Solution 2: Use aggregation
const admissions = await Admission.aggregate([
  { $match: { collegeId: req.college._id } },
  {
    $lookup: {
      from: 'students',
      localField: 'studentId',
      foreignField: '_id',
      as: 'student'
    }
  },
  { $unwind: '$student' }
]);
```

### Bottleneck 2: Unindexed Queries

**Problem**: Slow queries due to missing indexes

```javascript
// ❌ Slow query without proper indexing
const recentAdmissions = await Admission.find({
  collegeId: req.college._id,
  status: 'SUBMITTED',
  createdAt: { $gte: new Date(Date.now() - 7 * 24 * 60 * 60 * 1000) }
}).sort({ createdAt: -1 });

// ✅ Ensure proper indexes exist
// db.admissions.createIndex({ collegeId: 1, status: 1, createdAt: -1 })

// ✅ Optimized query
const recentAdmissions = await Admission.find({
  collegeId: req.college._id,
  status: 'SUBMITTED',
  createdAt: { $gte: new Date(Date.now() - 7 * 24 * 60 * 60 * 1000) }
}).sort({ createdAt: -1 })
 .limit(100);  // Always limit results
```

### Bottleneck 3: Memory Leaks in Long-Running Operations

**Problem**: Accumulating data in memory during processing

```javascript
// ❌ Memory leak
const processAllStudents = async () => {
  const allStudents = await Student.find({}); // Loads ALL students into memory!
  
  const results = [];
  for (const student of allStudents) {
    results.push(await processStudent(student)); // Memory keeps growing
  }
  
  return results;
};

// ✅ Streaming/batch processing
const processAllStudents = async () => {
  const batchSize = 100;
  let skip = 0;
  const results = [];
  
  while (true) {
    const batch = await Student.find({})
      .limit(batchSize)
      .skip(skip);
    
    if (batch.length === 0) break;
    
    for (const student of batch) {
      const result = await processStudent(student);
      results.push(result);
      
      // Process in smaller chunks to manage memory
      if (results.length >= batchSize) {
        await saveBatch(results);
        results.length = 0; // Clear array
      }
    }
    
    skip += batchSize;
  }
  
  // Save remaining results
  if (results.length > 0) {
    await saveBatch(results);
  }
};
```

### Bottleneck 4: Blocking Operations

**Problem**: Synchronous operations blocking the event loop

```javascript
// ❌ Blocking operation
const generateReport = async (collegeId) => {
  const students = await Student.find({ collegeId });
  
  // This blocks the entire event loop!
  const report = heavyComputationalFunction(students);
  
  return report;
};

// ✅ Non-blocking operation
const generateReport = async (collegeId) => {
  const students = await Student.find({ collegeId });
  
  // Use worker threads for heavy computation
  const report = await runInWorkerThread(heavyComputationalFunction, students);
  
  return report;
};
```

---

## SECURITY VULNERABILITIES

### Vulnerability 1: Injection Attacks

**Problem**: Malicious input causing database/command injection

```javascript
// ❌ Vulnerable to injection
const searchStudents = async (searchTerm) => {
  // This could be exploited with malicious input
  const students = await Student.find({
    $text: { $search: searchTerm }  // Direct injection!
  });
};

// ✅ Sanitized input
const searchStudents = async (searchTerm) => {
  // Sanitize input to prevent injection
  const sanitizedTerm = searchTerm.replace(/[<>;"']/g, '');
  
  // Use proper validation
  if (sanitizedTerm.length > 100) {
    throw new Error('Search term too long');
  }
  
  const students = await Student.find({
    collegeId: req.college._id,
    $text: { $search: sanitizedTerm }
  });
};
```

### Vulnerability 2: Authentication Bypass

**Problem**: Skipping authentication checks

```javascript
// ❌ Missing authentication
app.get('/api/admin/stats', async (req, res) => {
  // No authentication check!
  const stats = await getAdminStats();
  res.json(stats);
});

// ✅ Proper authentication
app.get('/api/admin/stats', authenticate, authorizeRole('ADMIN'), async (req, res) => {
  const stats = await getAdminStats(req.college._id);
  res.json(stats);
});
```

### Vulnerability 3: Insufficient Rate Limiting

**Problem**: No protection against abuse

```javascript
// ❌ No rate limiting
app.post('/api/admissions/apply', async (req, res) => {
  // Anyone can spam this endpoint!
  const result = await submitApplication(req.body);
  res.json(result);
});

// ✅ Rate limiting
const rateLimiter = require('express-rate-limit');

const admissionLimiter = rateLimiter({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 5, // Limit each IP to 5 requests per windowMs
  message: 'Too many application submissions, please try again later'
});

app.post('/api/admissions/apply', admissionLimiter, async (req, res) => {
  const result = await submitApplication(req.body);
  res.json(result);
});
```

### Vulnerability 4: Information Disclosure

**Problem**: Exposing sensitive information in error messages

```javascript
// ❌ Revealing internal details
app.get('/api/students/:id', async (req, res) => {
  try {
    const student = await Student.findById(req.params.id);
    res.json(student);
  } catch (error) {
    res.status(500).json({ error: error.message }); // Reveals internal details!
  }
});

// ✅ Safe error handling
app.get('/api/students/:id', async (req, res) => {
  try {
    const student = await Student.findOne({
      _id: req.params.id,
      collegeId: req.college._id
    });
    
    if (!student) {
      return res.status(404).json({ error: 'Student not found' });
    }
    
    res.json(student);
  } catch (error) {
    // Log full error internally, return generic message to user
    console.error('Student fetch error:', error);
    res.status(500).json({ error: 'Internal server error' });
  }
});
```

---

## DATABASE TROUBLES

### Trouble 1: Connection Pool Exhaustion

**Problem**: Too many database connections causing timeouts

```javascript
// ❌ Poor connection management
const connectToDatabase = () => {
  // Creating new connections repeatedly
  return mongoose.createConnection(process.env.MONGODB_URI);
};

// ✅ Proper connection management
const connectToDatabase = async () => {
  if (mongoose.connection.readyState === 0) {
    await mongoose.connect(process.env.MONGODB_URI, {
      maxPoolSize: 10, // Maintain up to 10 socket connections
      serverSelectionTimeoutMS: 5000, // Keep trying to send operations for 5 seconds
      socketTimeoutMS: 45000, // Close sockets after 45 seconds of inactivity
    });
  }
  return mongoose.connection;
};
```

### Trouble 2: Transaction Issues

**Problem**: Partial updates due to failed operations

```javascript
// ❌ Not using transactions
const updateStudentAndAdmission = async (studentId, admissionId, updates) => {
  await Student.findByIdAndUpdate(studentId, updates.student);
  await Admission.findByIdAndUpdate(admissionId, updates.admission);
  // If second update fails, first one is still committed!
};

// ✅ Using transactions
const updateStudentAndAdmission = async (studentId, admissionId, updates) => {
  const session = await mongoose.startSession();
  
  try {
    await session.withTransaction(async () => {
      await Student.findByIdAndUpdate(studentId, updates.student, { session });
      await Admission.findByIdAndUpdate(admissionId, updates.admission, { session });
    });
  } finally {
    await session.endSession();
  }
};
```

### Trouble 3: Schema Evolution Issues

**Problem**: Changing schemas breaking existing data

```javascript
// ❌ Breaking changes
// Old schema
const studentSchema = new mongoose.Schema({
  name: String,
  email: String
});

// New schema (breaking change!)
const studentSchema = new mongoose.Schema({
  firstName: String,  // Renamed field
  lastName: String,   // New field
  email: String
});

// ✅ Safe schema evolution
const studentSchema = new mongoose.Schema({
  firstName: String,
  lastName: String,
  email: String,
  // Keep old field temporarily for migration
  name: { 
    type: String, 
    select: false  // Hide from queries by default
  }
});

// Add migration script
const migrateStudentNames = async () => {
  const students = await Student.find({ firstName: null });
  
  for (const student of students) {
    if (student.name) {
      const [firstName, lastName] = student.name.split(' ');
      await Student.findByIdAndUpdate(student._id, {
        firstName: firstName,
        lastName: lastName || firstName
      });
    }
  }
};
```

---

## API DEVELOPMENT TIPS

### Tip 1: Consistent Error Handling

```javascript
// Create a centralized error handler
class APIError extends Error {
  constructor(message, statusCode, errorCode) {
    super(message);
    this.statusCode = statusCode;
    this.errorCode = errorCode;
    this.isOperational = true;
    
    Error.captureStackTrace(this, this.constructor);
  }
}

// Use it consistently
const getStudent = async (req, res, next) => {
  try {
    const student = await Student.findOne({
      _id: req.params.id,
      collegeId: req.college._id
    });
    
    if (!student) {
      throw new APIError('Student not found', 404, 'STUDENT_NOT_FOUND');
    }
    
    res.status(200).json({ data: student });
  } catch (error) {
    next(error); // Pass to error handling middleware
  }
};

// Error handling middleware
const errorHandler = (err, req, res, next) => {
  let { statusCode, message, errorCode } = err;
  
  if (process.env.NODE_ENV === 'production') {
    // Don't leak internal details in production
    message = 'Internal server error';
    errorCode = 'INTERNAL_ERROR';
  }
  
  res.status(statusCode).json({
    success: false,
    message,
    errorCode,
    ...(process.env.NODE_ENV === 'development' && { stack: err.stack })
  });
};
```

### Tip 2: Proper Pagination

```javascript
// Robust pagination implementation
const paginateResults = async (Model, query, options = {}) => {
  const {
    page = 1,
    limit = 20,
    sort = '-createdAt',
    select = ''
  } = options;
  
  const skip = (page - 1) * limit;
  
  // Get results
  const results = await Model.find(query)
    .select(select)
    .sort(sort)
    .skip(skip)
    .limit(parseInt(limit));
  
  // Get total count
  const total = await Model.countDocuments(query);
  
  return {
    data: results,
    pagination: {
      page: parseInt(page),
      limit: parseInt(limit),
      total,
      pages: Math.ceil(total / limit),
      hasNext: page < Math.ceil(total / limit),
      hasPrev: page > 1
    }
  };
};

// Usage in controller
app.get('/api/students', async (req, res) => {
  const { page, limit, status } = req.query;
  
  const query = { collegeId: req.college._id };
  if (status) query.status = status;
  
  const result = await paginateResults(Student, query, {
    page,
    limit,
    sort: '-createdAt',
    select: 'name email phone status'
  });
  
  res.json(result);
});
```

### Tip 3: Input Validation

```javascript
// Use a validation library like Joi
const Joi = require('joi');

const studentValidationSchema = Joi.object({
  name: Joi.string().min(2).max(100).required(),
  email: Joi.string().email().required(),
  phone: Joi.string().pattern(/^(\+91)?[6-9]\d{9}$/).required(), // Indian phone format
  casteCategory: Joi.string().valid('GEN', 'SC', 'ST', 'OBC', 'EWS').required(),
  class12Score: Joi.number().min(0).max(100).required()
});

const validateStudentInput = (req, res, next) => {
  const { error, value } = studentValidationSchema.validate(req.body);
  
  if (error) {
    return res.status(400).json({
      success: false,
      message: 'Validation error',
      errors: error.details.map(detail => detail.message)
    });
  }
  
  req.validatedBody = value;
  next();
};

// Use in route
app.post('/api/students', validateStudentInput, async (req, res) => {
  const student = await Student.create({
    ...req.validatedBody,
    collegeId: req.college._id
  });
  
  res.status(201).json({ success: true, data: student });
});
```

---

## DEBUGGING STRATEGIES

### Strategy 1: Structured Logging

```javascript
// Use a proper logging library
const winston = require('winston');

const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json()
  ),
  transports: [
    new winston.transports.File({ filename: 'error.log', level: 'error' }),
    new winston.transports.File({ filename: 'combined.log' }),
    new winston.transports.Console({
      format: winston.format.simple()
    })
  ]
});

// Log with context
const processAdmission = async (req, res) => {
  const requestId = req.headers['x-request-id'] || generateId();
  
  logger.info('Admission processing started', {
    requestId,
    collegeId: req.college._id,
    userId: req.user._id,
    admissionData: req.body
  });
  
  try {
    const result = await admissionService.process(req.body, req.college._id);
    
    logger.info('Admission processed successfully', {
      requestId,
      collegeId: req.college._id,
      result
    });
    
    res.json({ success: true, data: result });
  } catch (error) {
    logger.error('Admission processing failed', {
      requestId,
      collegeId: req.college._id,
      error: error.message,
      stack: error.stack
    });
    
    res.status(500).json({ success: false, message: 'Processing failed' });
  }
};
```

### Strategy 2: Debugging Multi-Tenant Issues

```javascript
// Add tenant context to all logs
const tenantAwareLogger = (req, res, next) => {
  req.logContext = {
    tenantId: req.college?.code,
    userId: req.user?._id,
    requestId: req.headers['x-request-id'] || generateId(),
    url: req.url,
    method: req.method
  };
  
  req.logger = {
    info: (message, data) => logger.info(message, { ...req.logContext, ...data }),
    error: (message, data) => logger.error(message, { ...req.logContext, ...data }),
    warn: (message, data) => logger.warn(message, { ...req.logContext, ...data })
  };
  
  next();
};

// Usage
app.use(tenantAwareLogger);

app.get('/api/students/:id', async (req, res) => {
  req.logger.info('Fetching student', { studentId: req.params.id });
  
  const student = await Student.findOne({
    _id: req.params.id,
    collegeId: req.college._id
  });
  
  if (!student) {
    req.logger.warn('Student not found', { studentId: req.params.id });
    return res.status(404).json({ error: 'Student not found' });
  }
  
  req.logger.info('Student fetched successfully');
  res.json({ data: student });
});
```

### Strategy 3: Performance Monitoring

```javascript
// Monitor API performance
const performanceMonitor = async (req, res, next) => {
  const start = process.hrtime.bigint();
  
  // Override response methods to capture timing
  const originalSend = res.send;
  res.send = function(data) {
    const end = process.hrtime.bigint();
    const duration = Number(end - start) / 1000000; // Convert to milliseconds
    
    req.logger.info('API request completed', {
      duration,
      statusCode: res.statusCode,
      dataSize: typeof data === 'string' ? data.length : JSON.stringify(data).length
    });
    
    // Alert on slow requests
    if (duration > 2000) { // More than 2 seconds
      req.logger.warn('Slow API request detected', { duration });
    }
    
    return originalSend.call(this, data);
  };
  
  next();
};

app.use(performanceMonitor);
```

---

## TESTING CHALLENGES

### Challenge 1: Multi-Tenant Test Isolation

```javascript
// Test helper for creating isolated tenant data
const testHelpers = {
  createTestTenant: async () => {
    const college = await College.create({
      code: `TEST_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`,
      name: 'Test College',
      state: 'MAHARASHTRA',
      principalEmail: `test${Date.now()}@example.com`,
      isActive: true
    });
    return college;
  },

  createTestDataForTenant: async (collegeId) => {
    const user = await User.create({
      collegeId,
      email: `test${Date.now()}@example.com`,
      password: await bcrypt.hash('TestPass123!', 10),
      role: 'STAFF',
      isActive: true
    });

    const student = await Student.create({
      collegeId,
      userId: user._id,
      name: 'Test Student',
      email: `student${Date.now()}@example.com`,
      phone: '+919876543210'
    });

    return { user, student };
  },

  cleanupTenantData: async (collegeId) => {
    await Student.deleteMany({ collegeId });
    await User.deleteMany({ collegeId });
    await College.findByIdAndDelete(collegeId);
  }
};

// Integration test example
describe('Multi-Tenant Data Isolation', () => {
  let college1, college2;
  let user1, user2;
  let student1, student2;

  beforeEach(async () => {
    college1 = await testHelpers.createTestTenant();
    college2 = await testHelpers.createTestTenant();
    
    const data1 = await testHelpers.createTestDataForTenant(college1._id);
    const data2 = await testHelpers.createTestDataForTenant(college2._id);
    
    user1 = data1.user;
    user2 = data2.user;
    student1 = data1.student;
    student2 = data2.student;
  });

  afterEach(async () => {
    await testHelpers.cleanupTenantData(college1._id);
    await testHelpers.cleanupTenantData(college2._id);
  });

  test('should not allow cross-tenant data access', async () => {
    const token = jwt.sign(
      { userId: user1._id, collegeId: college1._id },
      process.env.JWT_SECRET
    );

    const response = await request(app)
      .get(`/api/students/${student2._id}`)  // Trying to access college2's student
      .set('Authorization', `Bearer ${token}`)
      .set('X-College-Code', college1.code);

    expect(response.status).toBe(404); // Should not find the student
  });
});
```

### Challenge 2: Mocking External Services

```javascript
// Mock external services for testing
jest.mock('../services/paymentService', () => ({
  processPayment: jest.fn(),
  verifyPayment: jest.fn(),
  refundPayment: jest.fn()
}));

// Test with mocked dependencies
describe('Payment Processing', () => {
  const mockPaymentService = require('../services/paymentService');

  beforeEach(() => {
    jest.clearAllMocks();
  });

  test('should process payment successfully', async () => {
    mockPaymentService.processPayment.mockResolvedValue({
      success: true,
      transactionId: 'txn_123',
      amount: 61800
    });

    const response = await request(app)
      .post('/api/payments/process')
      .send({ amount: 61800, method: 'upi' })
      .set('Authorization', `Bearer ${validToken}`);

    expect(mockPaymentService.processPayment).toHaveBeenCalledWith(
      expect.anything(), // payment data
      expect.any(String)  // collegeId
    );
    expect(response.status).toBe(200);
  });
});
```

---

## DEPLOYMENT ISSUES

### Issue 1: Environment Configuration Problems

**Problem**: Different behavior between environments

**Solution**: Use consistent environment management

```javascript
// config/environment.js
const environments = {
  development: {
    database: {
      uri: process.env.DEV_DB_URI || 'mongodb://localhost:27017/novaa_dev',
      options: { maxPoolSize: 5 }
    },
    logging: { level: 'debug' },
    features: { enableNewUI: true }
  },
  staging: {
    database: {
      uri: process.env.STAGING_DB_URI,
      options: { maxPoolSize: 10 }
    },
    logging: { level: 'info' },
    features: { enableNewUI: false }
  },
  production: {
    database: {
      uri: process.env.PROD_DB_URI,
      options: { maxPoolSize: 20 }
    },
    logging: { level: 'warn' },
    features: { enableNewUI: true }
  }
};

const currentEnv = process.env.NODE_ENV || 'development';
const config = environments[currentEnv];

// Validate required environment variables
const requiredVars = [
  'JWT_SECRET',
  'MONGODB_URI',
  'RAZORPAY_KEY_ID',
  'RAZORPAY_SECRET'
];

for (const varName of requiredVars) {
  if (!process.env[varName]) {
    throw new Error(`Missing required environment variable: ${varName}`);
  }
}

module.exports = config;
```

### Issue 2: Database Migration Failures

**Problem**: Schema changes breaking production

**Solution**: Safe migration strategies

```javascript
// migrations/001-add-student-phone-index.js
const mongoose = require('mongoose');

module.exports = {
  async up() {
    console.log('Creating student phone index...');
    
    // Create index in background to avoid blocking
    await mongoose.connection.collection('students').createIndex(
      { phone: 1 }, 
      { background: true }
    );
    
    console.log('Student phone index created successfully');
  },

  async down() {
    console.log('Dropping student phone index...');
    
    await mongoose.connection.collection('students').dropIndex({ phone: 1 });
    
    console.log('Student phone index dropped successfully');
  }
};

// Migration runner
const runMigrations = async () => {
  const migrations = [
    require('./001-add-student-phone-index'),
    require('./002-add-admission-status-index')
  ];

  for (const migration of migrations) {
    try {
      await migration.up();
      console.log(`Migration completed: ${migration.constructor.name}`);
    } catch (error) {
      console.error(`Migration failed: ${migration.constructor.name}`, error);
      throw error;
    }
  }
};
```

### Issue 3: Deployment Rollback Strategy

**Problem**: Need to quickly revert bad deployments

**Solution**: Implement rollback procedures

```bash
#!/bin/bash
# rollback.sh

set -e

echo "Starting rollback procedure..."

# Get previous deployment
PREVIOUS_VERSION=$(cat /opt/novaa/current_version.bak)

if [ -z "$PREVIOUS_VERSION" ]; then
    echo "No previous version found, cannot rollback"
    exit 1
fi

echo "Rolling back to version: $PREVIOUS_VERSION"

# Stop current service
sudo systemctl stop novaa-api

# Restore previous version
sudo rm -rf /opt/novaa/current
sudo cp -r "/opt/novaa/versions/$PREVIOUS_VERSION" /opt/novaa/current

# Restart service
sudo systemctl start novaa-api

# Verify service is running
sleep 10

if sudo systemctl is-active --quiet novaa-api; then
    echo "Rollback successful"
    echo "$PREVIOUS_VERSION" > /opt/novaa/current_version
else
    echo "Rollback failed - service not running"
    sudo systemctl start novaa-api
    exit 1
fi
```

---

## QUICK FIXES & WORKAROUNDS

### Quick Fix 1: Immediate Performance Improvement

```javascript
// Add caching to slow endpoints
const cache = new Map();
const CACHE_TTL = 5 * 60 * 1000; // 5 minutes

const cachedHandler = (handler, ttl = CACHE_TTL) => {
  return async (req, res, next) => {
    const cacheKey = `${req.originalUrl}_${req.college._id}`;
    const cached = cache.get(cacheKey);
    
    if (cached && Date.now() - cached.timestamp < ttl) {
      return res.json(cached.data);
    }
    
    try {
      const result = await handler(req, res, next);
      
      cache.set(cacheKey, {
        data: result,
        timestamp: Date.now()
      });
      
      return result;
    } catch (error) {
      next(error);
    }
  };
};

// Usage
app.get('/api/stats', cachedHandler(async (req, res) => {
  const stats = await getStats(req.college._id);
  res.json(stats);
}));
```

### Quick Fix 2: Immediate Security Enhancement

```javascript
// Add security headers middleware
const securityHeaders = (req, res, next) => {
  // Prevent MIME type sniffing
  res.setHeader('X-Content-Type-Options', 'nosniff');
  
  // Prevent clickjacking
  res.setHeader('X-Frame-Options', 'DENY');
  
  // Enable XSS protection
  res.setHeader('X-XSS-Protection', '1; mode=block');
  
  // Strict transport security
  if (process.env.NODE_ENV === 'production') {
    res.setHeader('Strict-Transport-Security', 'max-age=31536000; includeSubDomains');
  }
  
  next();
};

app.use(securityHeaders);
```

### Quick Fix 3: Memory Leak Detection

```javascript
// Add memory monitoring
setInterval(() => {
  const used = process.memoryUsage();
  const heapUsed = Math.round(used.heapUsed / 1024 / 1024 * 100) / 100;
  const heapTotal = Math.round(used.heapTotal / 1024 / 1024 * 100) / 100;
  
  console.log(`Memory Usage: ${heapUsed} MB / ${heapTotal} MB`);
  
  if (heapUsed > 500) { // Alert if using more than 500MB
    console.warn('High memory usage detected:', { heapUsed, heapTotal });
  }
}, 30000); // Check every 30 seconds
```

---

## BEST PRACTICES SUMMARY

### Golden Rules for SaaS Development

1. **Always verify tenant context** - Every request should include and verify tenant identity
2. **Never trust user input** - Always validate and sanitize all inputs
3. **Use proper error handling** - Don't expose internal details to users
4. **Implement comprehensive logging** - Log everything with tenant context
5. **Design for scale from day one** - Consider performance and scalability early
6. **Test multi-tenancy rigorously** - Ensure data isolation is bulletproof
7. **Monitor continuously** - Set up alerts for performance and security issues
8. **Plan for failures** - Have rollback and recovery procedures ready

### Quick Reference Checklist

**Before deploying any feature:**
- [ ] Tenant isolation verified
- [ ] Security reviewed
- [ ] Performance tested
- [ ] Error handling implemented
- [ ] Logging added
- [ ] Tests passing
- [ ] Documentation updated

**When debugging:**
- [ ] Check tenant context
- [ ] Review recent changes
- [ ] Look at logs with context
- [ ] Verify database queries
- [ ] Check external dependencies
- [ ] Test with different tenants

**When optimizing:**
- [ ] Profile the actual bottleneck
- [ ] Add proper indexes
- [ ] Implement caching where appropriate
- [ ] Consider pagination for large datasets
- [ ] Monitor the results

---

## TROUBLESHOOTING RESOURCES

### Common Error Messages & Solutions

| Error Message | Likely Cause | Solution |
|---------------|--------------|----------|
| "Document not found" | Missing tenant filter | Add collegeId to query |
| "Permission denied" | Authentication/authorization issue | Check token validity and roles |
| "Connection timeout" | Database overload | Check connection pool and indexes |
| "Duplicate key error" | Unique constraint violation | Check for proper uniqueness handling |
| "Maximum call stack" | Infinite recursion | Review recursive functions |
| "Out of memory" | Memory leak | Check for accumulated data |

### Useful Commands

```bash
# Check database connections
db.serverStatus().connections

# Check current operations
db.currentOp()

# Monitor performance
db.getCollection('students').stats()

# Check indexes
db.students.getIndexes()
```

---

## CONCLUSION

This troubleshooting guide covers the most common issues you'll encounter when developing SaaS applications like NOVAA. Remember that SaaS development requires extra attention to security, performance, and data isolation. 

When facing new challenges:
1. Check if it's a multi-tenancy issue first
2. Verify tenant context is properly maintained
3. Look for performance implications as scale increases
4. Ensure security measures are in place
5. Test thoroughly with multiple tenants

The key to success in SaaS development is building defensive, scalable, and maintainable code from the start. Use this guide as a reference when you encounter common issues, and don't hesitate to reach out to senior developers when facing complex problems.

---

**Document Owner**: Technical Lead  
**Last Updated**: February 5, 2026  
**Next Review**: Monthly with development team