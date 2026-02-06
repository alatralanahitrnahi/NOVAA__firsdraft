# NOVAA SaaS Development Quick Reference Guide

**For**: New Developers Starting with NOVAA SaaS Platform  
**Version**: 1.0  
**Date**: February 5, 2026  

---

## 📋 TABLE OF CONTENTS

1. [Getting Started](#getting-started)
2. [Essential Concepts](#essential-concepts)
3. [Code Structure](#code-structure)
4. [Multi-Tenancy Best Practices](#multi-tenancy-best-practices)
5. [Security Guidelines](#security-guidelines)
6. [Performance Tips](#performance-tips)
7. [Common Patterns](#common-patterns)
8. [Testing Guidelines](#testing-guidelines)
9. [Deployment Checklist](#deployment-checklist)
10. [Troubleshooting](#troubleshooting)
11. [Quick References](#quick-references)

---

## GETTING STARTED

### Prerequisites
- Node.js 18+ installed
- MongoDB 5.0+ running
- Git configured
- Basic understanding of Express.js and Mongoose
- Familiarity with REST APIs

### Initial Setup
```bash
# Clone the repository
git clone https://github.com/your-org/novaa-backend.git
cd novaa-backend

# Install dependencies
npm install

# Copy environment file
cp .env.example .env

# Edit .env with your configuration
# MONGODB_URI=mongodb://localhost:27017/novaa_dev
# JWT_SECRET=your-secret-key-here

# Start development server
npm run dev
```

### Development Commands
```bash
npm run dev          # Start development server with hot reload
npm test            # Run all tests
npm run test:watch  # Run tests in watch mode
npm run lint        # Check code for linting issues
npm run format      # Format all code
npm run build       # Build for production
```

---

## ESSENTIAL CONCEPTS

### Multi-Tenancy
**Definition**: Single application instance serving multiple tenants (colleges) with data isolation.

**Key Principles**:
- Every database query must include `collegeId` filter
- Users can only access their own college's data
- Configuration and features can be tenant-specific
- Billing and usage tracking per tenant

### NOVAA Architecture
```
Frontend (React) ←→ Backend (Node.js/Express) ←→ Database (MongoDB)
     ↑                      ↑                        ↑
  User Browser        Multi-Tenant API        Tenant-Isolated Data
```

### Core Entities
- **College**: Top-level tenant entity
- **User**: Belongs to one college
- **Student**: Belongs to one college
- **Admission**: Belongs to one college and one student
- **Transaction**: Belongs to one college and one student

---

## CODE STRUCTURE

### Directory Structure
```
backend/
├── src/
│   ├── models/          # Database schemas
│   ├── controllers/     # Request handlers
│   ├── services/        # Business logic
│   ├── middleware/      # Request processing
│   ├── routes/          # API route definitions
│   ├── utils/          # Helper functions
│   ├── config/         # Configuration files
│   └── app.js          # Main application file
├── __tests__/          # Test files
├── scripts/            # Utility scripts
├── .env.example        # Environment variables template
├── package.json        # Dependencies and scripts
└── README.md          # Project documentation
```

### File Naming Convention
- **Models**: `Student.js`, `Admission.js` (PascalCase, singular)
- **Controllers**: `studentController.js`, `admissionController.js` (camelCase)
- **Services**: `studentService.js`, `admissionService.js` (camelCase)
- **Middleware**: `authMiddleware.js`, `tenantMiddleware.js` (camelCase)
- **Routes**: `studentRoutes.js`, `admissionRoutes.js` (camelCase)

---

## MULTI-TENANCY BEST PRACTICES

### ✅ DO: Always Include Tenant Filter
```javascript
// CORRECT - Always filter by collegeId
const students = await Student.find({
  collegeId: req.college._id,  // ✅ Tenant filter
  status: 'ACTIVE'
});
```

### ❌ DON'T: Skip Tenant Filter
```javascript
// INCORRECT - Missing tenant filter
const students = await Student.find({
  status: 'ACTIVE'  // ❌ Could return other colleges' data
});
```

### ✅ DO: Use Tenant-Aware Middleware
```javascript
// Middleware that enforces tenant context
const enforceTenantContext = async (req, res, next) => {
  if (!req.college || !req.college._id) {
    return res.status(400).json({ error: 'Missing tenant context' });
  }
  next();
};

// Apply to all routes that need tenant isolation
app.use('/api', enforceTenantContext);
```

### ✅ DO: Validate Tenant Ownership
```javascript
// Validate that requested resource belongs to current tenant
const getStudent = async (req, res) => {
  const student = await Student.findOne({
    _id: req.params.id,
    collegeId: req.college._id  // Verify ownership
  });
  
  if (!student) {
    return res.status(404).json({ error: 'Resource not found' });
  }
  
  res.json({ data: student });
};
```

### Database Indexing for Multi-Tenancy
```javascript
// Always create compound indexes with collegeId first
const studentSchema = new mongoose.Schema({
  collegeId: { type: mongoose.Schema.Types.ObjectId, ref: 'College', index: true },
  name: String,
  email: String
});

// Index: [collegeId, createdAt] - tenant isolation + sorting
studentSchema.index({ collegeId: 1, createdAt: -1 });
studentSchema.index({ collegeId: 1, status: 1 }); // tenant + filter
```

---

## SECURITY GUIDELINES

### Authentication Flow
```javascript
// 1. Login endpoint
app.post('/api/auth/login', async (req, res) => {
  const { email, password, collegeCode } = req.body;
  
  // Find college by code
  const college = await College.findOne({ code: collegeCode.toUpperCase() });
  if (!college) {
    return res.status(400).json({ error: 'Invalid college' });
  }
  
  // Find and verify user
  const user = await User.findOne({ email: email.toLowerCase() });
  if (!user || !await bcrypt.compare(password, user.password)) {
    return res.status(401).json({ error: 'Invalid credentials' });
  }
  
  // Verify user belongs to correct college
  if (user.collegeId.toString() !== college._id.toString()) {
    return res.status(401).json({ error: 'Invalid credentials' });
  }
  
  // Generate token with tenant context
  const token = jwt.sign({
    userId: user._id,
    collegeId: user.collegeId,
    role: user.role
  }, process.env.JWT_SECRET, { expiresIn: '2h' });
  
  res.json({ token, user: { id: user._id, email: user.email, role: user.role } });
});

// 2. Authentication middleware
const authenticate = async (req, res, next) => {
  const token = req.headers.authorization?.replace('Bearer ', '');
  if (!token) {
    return res.status(401).json({ error: 'No token provided' });
  }
  
  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    const user = await User.findById(decoded.userId).populate('collegeId');
    
    if (!user || !user.isActive || user.collegeId._id.toString() !== decoded.collegeId) {
      return res.status(401).json({ error: 'Invalid token' });
    }
    
    req.user = user;
    req.college = user.collegeId;
    next();
  } catch (error) {
    return res.status(401).json({ error: 'Invalid token' });
  }
};
```

### Input Validation
```javascript
// Always validate and sanitize input
const validateStudentInput = (req, res, next) => {
  const { name, email, phone, casteCategory } = req.body;
  
  // Validate required fields
  if (!name || !email || !phone || !casteCategory) {
    return res.status(400).json({ error: 'Missing required fields' });
  }
  
  // Validate data types and formats
  if (typeof name !== 'string' || name.length < 2) {
    return res.status(400).json({ error: 'Invalid name' });
  }
  
  if (!validator.isEmail(email)) {
    return res.status(400).json({ error: 'Invalid email format' });
  }
  
  if (!/^(\+91)?[6-9]\d{9}$/.test(phone)) {
    return res.status(400).json({ error: 'Invalid phone format' });
  }
  
  const validCategories = ['GEN', 'SC', 'ST', 'OBC', 'EWS'];
  if (!validCategories.includes(casteCategory.toUpperCase())) {
    return res.status(400).json({ error: 'Invalid caste category' });
  }
  
  next();
};
```

### Rate Limiting
```javascript
// Protect against abuse
const rateLimit = require('express-rate-limit');

const apiLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // Limit each IP to 100 requests per windowMs
  message: 'Too many requests from this IP'
});

const sensitiveLimiter = rateLimit({
  windowMs: 60 * 1000, // 1 minute
  max: 5, // Very strict for sensitive endpoints
  message: 'Too many attempts, please try again later'
});

app.use('/api/', apiLimiter);
app.post('/api/login', sensitiveLimiter);
```

---

## PERFORMANCE TIPS

### Database Optimization
```javascript
// 1. Use proper indexing
studentSchema.index({ collegeId: 1, status: 1, createdAt: -1 });

// 2. Use projection to limit returned fields
const students = await Student.find({ collegeId: req.college._id })
  .select('_id name email phone status')  // Only return needed fields
  .limit(50);

// 3. Use populate efficiently
const admissions = await Admission.find({ collegeId: req.college._id })
  .populate('studentId', 'name email');  // Only populate needed fields

// 4. Use aggregation for complex queries
const stats = await Admission.aggregate([
  { $match: { collegeId: req.college._id } },
  { $group: { _id: '$status', count: { $sum: 1 } } }
]);
```

### Caching Strategies
```javascript
// Simple in-memory caching
const cache = new Map();
const CACHE_TTL = 5 * 60 * 1000; // 5 minutes

const getCachedData = async (key, fetcher) => {
  const cached = cache.get(key);
  if (cached && Date.now() - cached.timestamp < CACHE_TTL) {
    return cached.data;
  }
  
  const data = await fetcher();
  cache.set(key, { data, timestamp: Date.now() });
  return data;
};

// Usage
const getStudentStats = async (collegeId) => {
  const cacheKey = `stats_${collegeId}`;
  return await getCachedData(cacheKey, () => 
    Student.countDocuments({ collegeId })
  );
};
```

### Pagination
```javascript
// Implement proper pagination
const getPaginatedResults = async (Model, query, options = {}) => {
  const {
    page = 1,
    limit = 20,
    sort = '-createdAt',
    select = ''
  } = options;
  
  const skip = (page - 1) * limit;
  
  const results = await Model.find(query)
    .select(select)
    .sort(sort)
    .skip(skip)
    .limit(parseInt(limit));
  
  const total = await Model.countDocuments(query);
  
  return {
    data: results,
    pagination: {
      page: parseInt(page),
      limit: parseInt(limit),
      total,
      pages: Math.ceil(total / limit)
    }
  };
};
```

---

## COMMON PATTERNS

### Controller Pattern
```javascript
// controllers/studentController.js
const Student = require('../models/Student');

const studentController = {
  // GET /api/students
  async getAll(req, res) {
    try {
      const { page = 1, limit = 20, status } = req.query;
      
      const query = { collegeId: req.college._id };
      if (status) query.status = status;
      
      const results = await getPaginatedResults(Student, query, {
        page,
        limit,
        select: '_id name email phone status createdAt'
      });
      
      res.json(results);
    } catch (error) {
      res.status(500).json({ error: 'Internal server error' });
    }
  },
  
  // GET /api/students/:id
  async getById(req, res) {
    try {
      const student = await Student.findOne({
        _id: req.params.id,
        collegeId: req.college._id
      });
      
      if (!student) {
        return res.status(404).json({ error: 'Student not found' });
      }
      
      res.json({ data: student });
    } catch (error) {
      res.status(500).json({ error: 'Internal server error' });
    }
  },
  
  // POST /api/students
  async create(req, res) {
    try {
      const student = await Student.create({
        ...req.body,
        collegeId: req.college._id
      });
      
      res.status(201).json({ data: student });
    } catch (error) {
      if (error.code === 11000) {
        return res.status(409).json({ error: 'Student already exists' });
      }
      res.status(400).json({ error: error.message });
    }
  }
};

module.exports = studentController;
```

### Service Pattern
```javascript
// services/studentService.js
class StudentService {
  constructor(tenantId) {
    this.tenantId = tenantId;
  }
  
  async createStudent(studentData) {
    return await Student.create({
      ...studentData,
      collegeId: this.tenantId
    });
  }
  
  async getStudentById(studentId) {
    return await Student.findOne({
      _id: studentId,
      collegeId: this.tenantId
    });
  }
  
  async updateStudent(studentId, updateData) {
    return await Student.findOneAndUpdate(
      { _id: studentId, collegeId: this.tenantId },
      { ...updateData, updatedAt: new Date() },
      { new: true }
    );
  }
  
  async deleteStudent(studentId) {
    return await Student.findOneAndDelete({
      _id: studentId,
      collegeId: this.tenantId
    });
  }
}

module.exports = StudentService;
```

### Repository Pattern
```javascript
// repositories/studentRepository.js
class StudentRepository {
  constructor(tenantId) {
    this.tenantId = tenantId;
  }
  
  async findById(id) {
    return await Student.findById(id);
  }
  
  async findByEmail(email) {
    return await Student.findOne({
      email: email.toLowerCase(),
      collegeId: this.tenantId
    });
  }
  
  async create(data) {
    return await Student.create({
      ...data,
      collegeId: this.tenantId
    });
  }
  
  async update(id, data) {
    return await Student.findByIdAndUpdate(
      { _id: id, collegeId: this.tenantId },
      { ...data, updatedAt: new Date() },
      { new: true }
    );
  }
}

module.exports = StudentRepository;
```

---

## TESTING GUIDELINES

### Test Structure
```
__tests__/
├── unit/
│   ├── models/
│   ├── services/
│   └── utils/
├── integration/
│   ├── auth.test.js
│   ├── students.test.js
│   └── admissions.test.js
├── fixtures/
│   └── testData.js
└── setup/
    └── testHelpers.js
```

### Testing Best Practices
```javascript
// __tests__/integration/students.test.js
const request = require('supertest');
const app = require('../../src/app');
const { testHelpers } = require('../setup/testHelpers');

describe('Student API', () => {
  let college, user, token;

  beforeEach(async () => {
    college = await testHelpers.createTestCollege();
    user = await testHelpers.createTestUser(college._id);
    token = await testHelpers.getAuthToken(college._id, user._id);
  });

  describe('GET /api/students', () => {
    test('should return students for current tenant only', async () => {
      // Create student for current tenant
      const student1 = await testHelpers.createTestStudent(college._id);
      
      // Create student for different tenant (should not appear)
      const otherCollege = await testHelpers.createTestCollege();
      await testHelpers.createTestStudent(otherCollege._id);
      
      const response = await request(app)
        .get('/api/students')
        .set('Authorization', `Bearer ${token}`)
        .set('X-College-Code', college.code)
        .expect(200);
      
      expect(response.body.data).toHaveLength(1);
      expect(response.body.data[0]._id).toBe(student1._id.toString());
    });
  });

  describe('POST /api/students', () => {
    test('should create student successfully', async () => {
      const newStudent = {
        name: 'John Doe',
        email: 'john@example.com',
        phone: '+919876543210',
        casteCategory: 'GEN',
        class12Score: 85
      };

      const response = await request(app)
        .post('/api/students')
        .set('Authorization', `Bearer ${token}`)
        .set('X-College-Code', college.code)
        .send(newStudent)
        .expect(201);
      
      expect(response.body.data.name).toBe(newStudent.name);
      expect(response.body.data.collegeId).toBe(college._id.toString());
    });
  });
});
```

### Test Helpers
```javascript
// __tests__/setup/testHelpers.js
const mongoose = require('mongoose');
const bcrypt = require('bcryptjs');
const jwt = require('jsonwebtoken');

const testHelpers = {
  createTestCollege: async (overrides = {}) => {
    const College = require('../../src/models/College');
    
    const college = await College.create({
      code: `TEST_${Date.now()}_${Math.random().toString(36).substr(2, 5)}`,
      name: 'Test College',
      state: 'MAHARASHTRA',
      principalEmail: `test${Date.now()}@example.com`,
      isActive: true,
      ...overrides
    });
    
    return college;
  },

  createTestUser: async (collegeId, overrides = {}) => {
    const User = require('../../src/models/User');
    
    const user = await User.create({
      collegeId,
      email: `user${Date.now()}@test.com`,
      password: await bcrypt.hash('TestPass123!', 10),
      role: 'STAFF',
      isActive: true,
      ...overrides
    });
    
    return user;
  },

  getAuthToken: async (collegeId, userId) => {
    return jwt.sign(
      { userId, collegeId },
      process.env.JWT_SECRET || 'test-secret'
    );
  }
};

module.exports = { testHelpers };
```

---

## DEPLOYMENT CHECKLIST

### Pre-Deployment
- [ ] Run all tests: `npm test`
- [ ] Check code coverage: `npm run test:coverage`
- [ ] Lint code: `npm run lint`
- [ ] Format code: `npm run format`
- [ ] Update documentation if needed
- [ ] Review database migration scripts
- [ ] Verify environment variables
- [ ] Test with staging data

### During Deployment
- [ ] Backup production database
- [ ] Deploy to staging first
- [ ] Run smoke tests on staging
- [ ] Deploy to production
- [ ] Monitor application logs
- [ ] Verify health checks pass

### Post-Deployment
- [ ] Verify application is running
- [ ] Check that all endpoints work
- [ ] Monitor error logs
- [ ] Verify performance metrics
- [ ] Test with real user data
- [ ] Update deployment documentation

### Rollback Plan
```bash
# If deployment fails, rollback commands:
# 1. Revert to previous version
git checkout <previous-commit-hash>

# 2. Deploy previous version
npm run deploy:prod

# 3. Verify application is working
curl -f http://your-app.com/health
```

---

## TROUBLESHOOTING

### Common Issues & Solutions

**Issue**: "Document not found" for existing records
**Solution**: Check if `collegeId` filter is missing in query
```javascript
// Wrong
const student = await Student.findById(req.params.id);

// Correct
const student = await Student.findOne({
  _id: req.params.id,
  collegeId: req.college._id
});
```

**Issue**: "Permission denied" for valid requests
**Solution**: Verify authentication middleware is properly set up
```javascript
// Ensure authentication is applied before other middleware
app.use('/api', authenticate, require('./routes'));
```

**Issue**: Slow API responses
**Solution**: Check for missing indexes and N+1 queries
```javascript
// Add indexes for commonly queried fields
db.students.createIndex({ collegeId: 1, status: 1 })

// Use populate instead of multiple queries
const admissions = await Admission.find({ collegeId })
  .populate('studentId'); // Single query instead of N queries
```

**Issue**: Memory leaks
**Solution**: Check for unclosed database connections and accumulated data
```javascript
// Always close connections when done
await mongoose.connection.close();

// Don't accumulate large amounts of data in memory
// Use streaming or pagination for large datasets
```

### Debugging Commands
```bash
# Check running processes
ps aux | grep node

# Check application logs
tail -f logs/combined-$(date +%Y-%m-%d).log

# Check MongoDB connections
mongo --eval "db.serverStatus().connections"

# Monitor memory usage
top -p $(pgrep -f "node server.js")

# Check disk space
df -h

# Check network connections
netstat -tulpn | grep :5000
```

### Environment Variables Checklist
```bash
# Essential environment variables
NODE_ENV=production
PORT=5000
MONGODB_URI=mongodb://localhost:27017/novaa
JWT_SECRET=your-super-secret-jwt-key
RAZORPAY_KEY_ID=your-razorpay-key
RAZORPAY_SECRET=your-razorpay-secret
AWS_ACCESS_KEY_ID=your-aws-key
AWS_SECRET_ACCESS_KEY=your-aws-secret
REDIS_URL=redis://localhost:6379
LOG_LEVEL=info
```

---

## QUICK REFERENCES

### HTTP Status Codes
- `200 OK` - Request successful
- `201 Created` - Resource created successfully
- `204 No Content` - Request successful, no content to return
- `400 Bad Request` - Client error (invalid data)
- `401 Unauthorized` - Authentication required/failed
- `403 Forbidden` - Authenticated but not authorized
- `404 Not Found` - Resource doesn't exist
- `409 Conflict` - Data conflict (duplicate email, etc.)
- `429 Too Many Requests` - Rate limit exceeded
- `500 Server Error` - Server error

### Common Validation Rules
```javascript
// Email validation
const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;

// Indian phone validation
const indianPhoneRegex = /^(\+91)?[6-9]\d{9}$/;

// Name validation (letters, spaces, hyphens, apostrophes)
const nameRegex = /^[a-zA-Z\s'-]+$/;

// Valid caste categories
const validCategories = ['GEN', 'SC', 'ST', 'OBC', 'EWS'];
```

### Database Query Patterns
```javascript
// Find by ID with tenant filter
Model.findOne({ _id: id, collegeId: tenantId })

// Find multiple with tenant filter
Model.find({ collegeId: tenantId, status: 'ACTIVE' })

// Update with tenant filter
Model.findOneAndUpdate(
  { _id: id, collegeId: tenantId },
  { $set: { field: value } },
  { new: true }
)

// Delete with tenant filter
Model.findOneAndDelete({ _id: id, collegeId: tenantId })
```

### Error Handling Pattern
```javascript
const handleError = (error, res, operation = 'operation') => {
  if (error.name === 'ValidationError') {
    return res.status(400).json({ 
      error: 'VALIDATION_ERROR', 
      message: 'Invalid input data' 
    });
  }
  
  if (error.code === 11000) {
    return res.status(409).json({ 
      error: 'DUPLICATE_ENTRY', 
      message: 'Resource already exists' 
    });
  }
  
  console.error(`${operation} error:`, error);
  res.status(500).json({ 
    error: 'INTERNAL_ERROR', 
    message: 'Internal server error' 
  });
};
```

### Testing Assertions
```javascript
// Common Jest assertions
expect(response.status).toBe(200);
expect(response.body.data).toHaveLength(5);
expect(response.body.data[0]).toHaveProperty('id');
expect(response.body.data[0].name).toBe('Expected Name');
expect(response.body.pagination).toMatchObject({
  page: 1,
  limit: 20,
  total: 100
});

// Async operations
await expect(promise).rejects.toThrow('Expected error message');

// Mock functions
expect(mockFunction).toHaveBeenCalledWith(expectedArg);
expect(mockFunction).toHaveBeenCalledTimes(1);
```

### Git Best Practices
```bash
# Good commit messages
git commit -m "feat: add student search functionality"
git commit -m "fix: resolve tenant isolation issue in student API"
git commit -m "docs: update student API documentation"
git commit -m "refactor: extract validation logic to separate module"

# Branch naming
feature/add-student-search
bugfix/tenant-isolation-issue
hotfix/critical-security-fix
docs/update-api-documentation

# Sync with main regularly
git checkout main
git pull origin main
git checkout feature/my-feature
git rebase main
```

---

## DEVELOPMENT WORKFLOW

### Daily Routine
1. **Start**: Pull latest changes from main
   ```bash
   git checkout main
   git pull origin main
   ```

2. **Create feature branch**:
   ```bash
   git checkout -b feature/my-new-feature
   ```

3. **Develop**: Write code following patterns above

4. **Test**: Run tests locally
   ```bash
   npm test
   npm run lint
   ```

5. **Commit**: Make small, focused commits
   ```bash
   git add .
   git commit -m "feat: implement student validation"
   ```

6. **Sync**: Keep feature branch updated
   ```bash
   git checkout main
   git pull origin main
   git checkout feature/my-new-feature
   git rebase main
   ```

7. **Push**: Push to GitHub
   ```bash
   git push -u origin feature/my-new-feature
   ```

8. **PR**: Create pull request following template

### Code Review Process
1. **Self-review**: Check your own code before submitting
2. **Peer review**: Get at least one team member to review
3. **Address feedback**: Make requested changes
4. **Final review**: Get final approval
5. **Merge**: Merge to main (delete branch after merge)

---

## EMERGENCY PROCEDURES

### Production Issues
1. **Assess**: Determine severity and impact
2. **Notify**: Inform team and stakeholders
3. **Mitigate**: Apply quick fixes if possible
4. **Investigate**: Find root cause
5. **Fix**: Implement permanent solution
6. **Verify**: Confirm fix works
7. **Document**: Record incident and resolution

### Database Emergency
```javascript
// If you accidentally delete data, restore from backup:
// 1. Stop the application
// 2. Restore from latest backup
// 3. Verify data integrity
// 4. Restart application
// 5. Monitor for issues
```

### Security Incident
1. **Isolate**: Disconnect affected systems
2. **Assess**: Determine scope of breach
3. **Notify**: Inform security team and legal
4. **Investigate**: Find source and method
5. **Remediate**: Fix vulnerability
6. **Communicate**: Inform affected parties
7. **Review**: Update security procedures

---

## RESOURCES

### Documentation
- [NOVAA API Documentation](link-to-api-docs)
- [Database Schema Documentation](link-to-schema)
- [Deployment Procedures](link-to-deployment)
- [Security Guidelines](link-to-security)

### Support Channels
- **Slack**: #dev-team channel
- **Email**: dev-support@novaa.com
- **Emergency**: Call ops team (on-call rotation)

### Learning Resources
- [MongoDB Documentation](https://docs.mongodb.com)
- [Express.js Guide](https://expressjs.com)
- [Node.js Best Practices](https://github.com/goldbergyoni/nodebestpractices)
- [SaaS Architecture Patterns](link-to-architecture-guide)

---

## CONTACT INFORMATION

**Technical Lead**: [Name] - [email]  
**DevOps Engineer**: [Name] - [email]  
**Security Contact**: [Name] - [email]  
**Emergency On-Call**: [phone number]

---

**Document Owner**: Technical Lead  
**Last Updated**: February 5, 2026  
**Next Review**: Monthly with development team