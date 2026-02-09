# SYSTEM RULES - DO NOT BREAK

**Purpose**: Critical rules that MUST be followed  
**Consequence**: Breaking these rules causes data leakage, security issues, or system crashes

---

## 🔴 RULE 1: Multi-Tenancy is MANDATORY

### The Rule
**Every database query MUST include `collegeId`**

### Why It Matters
Without `collegeId`, queries return data from ALL colleges, causing:
- Data leakage between colleges
- Privacy violations
- Compliance failures
- Loss of customer trust

### Wrong ❌
```javascript
// BAD: Returns students from ALL colleges
const students = await Student.find({ status: 'APPROVED' });
```

### Right ✅
```javascript
// GOOD: Returns students from current college only
const students = await Student.find({ 
  collegeId: req.user.collegeId,
  status: 'APPROVED' 
});
```

### How to Enforce
1. Always use `req.user.collegeId` from authenticated user
2. Never trust `collegeId` from frontend request body
3. Use college middleware on all routes
4. Test with multiple colleges to verify isolation

### Testing Multi-Tenancy
```javascript
// Create test data for two colleges
const college1 = await College.create({ name: 'College A', code: 'COLL_A' });
const college2 = await College.create({ name: 'College B', code: 'COLL_B' });

// Create students for each college
await Student.create({ collegeId: college1._id, fullName: 'Student A' });
await Student.create({ collegeId: college2._id, fullName: 'Student B' });

// Query as College A admin
const studentsA = await Student.find({ collegeId: college1._id });
// Should return ONLY Student A

// Query as College B admin
const studentsB = await Student.find({ collegeId: college2._id });
// Should return ONLY Student B
```

---

## 🔴 RULE 2: Never Skip Authentication Middleware

### The Rule
**All protected routes MUST use `authenticate` middleware**

### Why It Matters
Without authentication:
- Anyone can access sensitive data
- No way to identify the user
- No way to enforce college context
- Security breach

### Wrong ❌
```javascript
// BAD: No authentication
router.get('/students', getStudents);
```

### Right ✅
```javascript
// GOOD: Authentication required
router.get('/students', authenticate, authorizeRole(['COLLEGE_ADMIN']), getStudents);
```

### Middleware Order
```javascript
// ALWAYS follow this order:
router.post('/endpoint',
  authenticate,           // 1. Verify JWT token
  authorizeRole([...]),   // 2. Check user role
  collegeMiddleware,      // 3. Validate college context
  yourController          // 4. Execute business logic
);
```

### Public Routes (Exceptions)
Only these routes should be public:
- `POST /auth/register`
- `POST /auth/login`
- `POST /students/register/:collegeCode` (student self-registration)
- `GET /health` (health check)

Everything else MUST be protected.

---

## 🔴 RULE 3: Never Trust Frontend Data

### The Rule
**Always validate and sanitize user input on the backend**

### Why It Matters
Frontend can be manipulated by users:
- Bypass validations
- Inject malicious data
- Escalate privileges
- Access other colleges' data

### Wrong ❌
```javascript
// BAD: Trusting collegeId from request body
const { collegeId, studentId } = req.body;
const student = await Student.findOne({ collegeId, _id: studentId });
```

### Right ✅
```javascript
// GOOD: Using collegeId from authenticated user
const { studentId } = req.body;
const student = await Student.findOne({ 
  collegeId: req.user.collegeId,  // From JWT token
  _id: studentId 
});
```

### What to Validate
1. **Required fields**: Check if present
2. **Data types**: Ensure correct type
3. **Format**: Validate email, phone, etc.
4. **Range**: Check min/max values
5. **Enum values**: Verify against allowed values

### Example Validation
```javascript
const validateStudent = (data) => {
  const errors = [];
  
  if (!data.fullName || data.fullName.trim() === '') {
    errors.push('Full name is required');
  }
  
  if (!data.email || !/\S+@\S+\.\S+/.test(data.email)) {
    errors.push('Valid email is required');
  }
  
  if (!data.mobileNumber || !/^\d{10}$/.test(data.mobileNumber)) {
    errors.push('Valid 10-digit mobile number is required');
  }
  
  if (!['GEN', 'OBC', 'SC', 'ST', 'OTHER'].includes(data.category)) {
    errors.push('Invalid category');
  }
  
  return errors;
};
```

---

## 🔴 RULE 4: Never Modify Existing Database Schemas Without Approval

### The Rule
**Changing schemas breaks existing data and code**

### Why It Matters
- Existing data becomes invalid
- Queries fail
- Frontend breaks
- Data loss risk

### What You CAN Do ✅
- Add new optional fields
- Add new collections
- Create indexes

### What You CANNOT Do ❌
- Remove existing fields
- Rename fields
- Change field types
- Change enum values
- Remove collections

### Adding New Fields (Safe)
```javascript
// GOOD: Adding optional field
const studentSchema = new Schema({
  // ... existing fields ...
  parentContact: {  // NEW FIELD
    type: String,
    required: false  // OPTIONAL
  }
});
```

### Changing Existing Fields (Dangerous)
```javascript
// BAD: Changing field type
category: {
  type: String,  // Was: Enum
  required: true
}

// BAD: Renaming field
studentName: String  // Was: fullName

// BAD: Removing field
// email: String  // DELETED
```

### If You Must Change Schema
1. Discuss with team
2. Create migration script
3. Test on copy of production data
4. Backup database
5. Run migration
6. Verify data integrity

---

## 🔴 RULE 5: Never Remove Existing API Endpoints

### The Rule
**Frontend may be using endpoints you don't know about**

### Why It Matters
- Frontend breaks silently
- Users see errors
- Features stop working
- No way to track usage

### What to Do Instead
1. **Deprecate** the endpoint (add warning)
2. **Create new endpoint** with better design
3. **Update frontend** to use new endpoint
4. **Remove old endpoint** after frontend migration

### Deprecation Example
```javascript
// OLD ENDPOINT (deprecated)
router.get('/students/list', authenticate, (req, res) => {
  console.warn('DEPRECATED: /students/list - Use /students instead');
  // ... existing logic ...
});

// NEW ENDPOINT (recommended)
router.get('/students', authenticate, (req, res) => {
  // ... improved logic ...
});
```

---

## 🔴 RULE 6: Never Store Sensitive Data in Plain Text

### The Rule
**Always encrypt/hash sensitive data**

### Sensitive Data Includes
- Passwords
- Payment tokens
- API keys
- Webhook secrets
- Personal identification numbers

### Wrong ❌
```javascript
// BAD: Plain text password
const user = await User.create({
  email: 'user@example.com',
  password: 'password123'  // PLAIN TEXT
});
```

### Right ✅
```javascript
// GOOD: Hashed password
const bcrypt = require('bcrypt');

const hashedPassword = await bcrypt.hash('password123', 10);
const user = await User.create({
  email: 'user@example.com',
  password: hashedPassword  // HASHED
});
```

### What to Hash
- Passwords (bcrypt)
- Security tokens (bcrypt)

### What to Encrypt
- API keys (crypto)
- Webhook secrets (crypto)
- Payment tokens (crypto)

### Never Log Sensitive Data
```javascript
// BAD
console.log('User password:', user.password);
console.log('Payment token:', paymentToken);

// GOOD
console.log('User created:', user.email);
console.log('Payment processed for student:', studentId);
```

---

## 🔴 RULE 7: Never Bypass Role-Based Access Control

### The Rule
**Users can only access data their role permits**

### Role Hierarchy
```
SUPER_ADMIN (platform analytics only)
    ↓
COLLEGE_ADMIN (full college access)
    ↓
TEACHER (academic operations)
    ↓
STUDENT (self-service only)
```

### Wrong ❌
```javascript
// BAD: No role check
router.delete('/students/:id', authenticate, deleteStudent);
// Any authenticated user can delete students!
```

### Right ✅
```javascript
// GOOD: Role-based access
router.delete('/students/:id', 
  authenticate, 
  authorizeRole(['COLLEGE_ADMIN']), 
  deleteStudent
);
```

### Role Permissions Matrix

| Action | SUPER_ADMIN | COLLEGE_ADMIN | TEACHER | STUDENT |
|--------|-------------|---------------|---------|---------|
| View all colleges | ✅ | ❌ | ❌ | ❌ |
| Manage college | ❌ | ✅ | ❌ | ❌ |
| Create teachers | ❌ | ✅ | ❌ | ❌ |
| Approve students | ❌ | ✅ | ❌ | ❌ |
| Mark attendance | ❌ | ❌ | ✅ | ❌ |
| View own profile | ❌ | ✅ | ✅ | ✅ |
| Make payments | ❌ | ❌ | ❌ | ✅ |

### Testing RBAC
```javascript
// Test as different roles
const adminToken = 'jwt-token-for-admin';
const teacherToken = 'jwt-token-for-teacher';
const studentToken = 'jwt-token-for-student';

// Admin can delete
const res1 = await request(app)
  .delete('/students/123')
  .set('Authorization', `Bearer ${adminToken}`);
// Should return 200

// Teacher cannot delete
const res2 = await request(app)
  .delete('/students/123')
  .set('Authorization', `Bearer ${teacherToken}`);
// Should return 403 Forbidden

// Student cannot delete
const res3 = await request(app)
  .delete('/students/123')
  .set('Authorization', `Bearer ${studentToken}`);
// Should return 403 Forbidden
```

---

## 🔴 RULE 8: Never Expose Internal IDs to Frontend

### The Rule
**Use meaningful identifiers, not MongoDB ObjectIds**

### Why It Matters
- ObjectIds reveal database structure
- Hard to debug
- Not user-friendly
- Security risk

### Wrong ❌
```javascript
// BAD: Exposing ObjectId
res.json({
  studentId: '507f1f77bcf86cd799439011',  // MongoDB ObjectId
  name: 'John Doe'
});
```

### Right ✅
```javascript
// GOOD: Using meaningful identifier
res.json({
  studentId: student._id,  // Still ObjectId internally
  studentCode: 'STU2024001',  // User-friendly code
  name: 'John Doe'
});
```

### Generate User-Friendly Codes
```javascript
// Student code: STU + Year + Sequential number
const generateStudentCode = async (collegeId) => {
  const year = new Date().getFullYear();
  const count = await Student.countDocuments({ 
    collegeId, 
    admissionYear: year 
  });
  return `STU${year}${String(count + 1).padStart(4, '0')}`;
};

// Example: STU20240001, STU20240002, etc.
```

---

## 🔴 RULE 9: Never Make Database Queries in Loops

### The Rule
**Batch queries instead of looping**

### Why It Matters
- Slow performance
- Database overload
- Timeout errors
- Poor user experience

### Wrong ❌
```javascript
// BAD: Query in loop (N+1 problem)
const students = await Student.find({ collegeId });

for (const student of students) {
  const fees = await StudentFee.findOne({ studentId: student._id });
  student.fees = fees;
}
```

### Right ✅
```javascript
// GOOD: Batch query
const students = await Student.find({ collegeId });
const studentIds = students.map(s => s._id);

const fees = await StudentFee.find({ 
  studentId: { $in: studentIds } 
});

const feesMap = fees.reduce((acc, fee) => {
  acc[fee.studentId] = fee;
  return acc;
}, {});

students.forEach(student => {
  student.fees = feesMap[student._id];
});
```

### Use Aggregation for Complex Queries
```javascript
// GOOD: Single aggregation query
const studentsWithFees = await Student.aggregate([
  { $match: { collegeId: mongoose.Types.ObjectId(collegeId) } },
  {
    $lookup: {
      from: 'student_fees',
      localField: '_id',
      foreignField: 'studentId',
      as: 'fees'
    }
  },
  { $unwind: '$fees' }
]);
```

---

## 🔴 RULE 10: Never Commit Secrets to Git

### The Rule
**Never commit `.env` files or secrets**

### Why It Matters
- Secrets exposed publicly
- Security breach
- Unauthorized access
- Compliance violations

### What to Do
1. Add `.env` to `.gitignore`
2. Use `.env.example` for documentation
3. Store secrets in environment variables
4. Use secret management tools in production

### .gitignore
```
# Environment variables
.env
.env.local
.env.production

# Logs
logs/
*.log

# Dependencies
node_modules/

# Build
dist/
build/
```

### .env.example
```bash
# Database
MONGODB_URI=mongodb://localhost:27017/novaa

# JWT
JWT_SECRET=your-secret-key-here
JWT_EXPIRE=7d

# Stripe
STRIPE_SECRET_KEY=sk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...

# Frontend
FRONTEND_URL=http://localhost:3000
```

### If You Accidentally Commit Secrets
1. **Immediately** rotate the secrets (generate new ones)
2. Remove from Git history (use `git filter-branch` or BFG Repo-Cleaner)
3. Update production environment variables
4. Notify team

---

## 🔴 RULE 11: Never Deploy Without Testing

### The Rule
**Test locally before deploying to production**

### Testing Checklist
- [ ] All new features work
- [ ] Existing features still work (regression testing)
- [ ] Multi-tenancy enforced
- [ ] Authentication working
- [ ] Role-based access working
- [ ] Error handling working
- [ ] No console errors
- [ ] No console warnings
- [ ] Database queries optimized
- [ ] API responses correct

### Test with Multiple Colleges
```javascript
// Create test data for 2 colleges
const college1 = await createTestCollege('College A');
const college2 = await createTestCollege('College B');

// Test as College A admin
const adminA = await loginAs(college1.admin);
const studentsA = await getStudents(adminA.token);
// Should return ONLY College A students

// Test as College B admin
const adminB = await loginAs(college2.admin);
const studentsB = await getStudents(adminB.token);
// Should return ONLY College B students
```

### Test with Different Roles
```javascript
// Test as admin
const admin = await loginAs('admin');
const deleteRes1 = await deleteStudent(admin.token, studentId);
// Should succeed

// Test as teacher
const teacher = await loginAs('teacher');
const deleteRes2 = await deleteStudent(teacher.token, studentId);
// Should fail with 403

// Test as student
const student = await loginAs('student');
const deleteRes3 = await deleteStudent(student.token, studentId);
// Should fail with 403
```

---

## 🔴 RULE 12: Never Ignore Error Handling

### The Rule
**Every async operation needs error handling**

### Wrong ❌
```javascript
// BAD: No error handling
const students = await Student.find({ collegeId });
res.json({ data: students });
```

### Right ✅
```javascript
// GOOD: Proper error handling
try {
  const students = await Student.find({ collegeId });
  res.json({ success: true, data: students });
} catch (error) {
  console.error('Error fetching students:', error);
  res.status(500).json({ 
    success: false, 
    message: 'Failed to fetch students' 
  });
}
```

### Use Async Handler
```javascript
// BETTER: Using async handler wrapper
const asyncHandler = require('../utils/asyncHandler');

exports.getStudents = asyncHandler(async (req, res) => {
  const students = await Student.find({ 
    collegeId: req.user.collegeId 
  });
  res.json({ success: true, data: students });
});
```

---

## ✅ RULE COMPLIANCE CHECKLIST

Before submitting code for review:

- [ ] All queries include `collegeId`
- [ ] All protected routes use `authenticate` middleware
- [ ] All user input is validated
- [ ] No schema changes without approval
- [ ] No API endpoints removed
- [ ] No sensitive data in plain text
- [ ] Role-based access enforced
- [ ] No database queries in loops
- [ ] No secrets committed to Git
- [ ] All code tested locally
- [ ] Error handling implemented
- [ ] No console.log in production code

---

## 🆘 WHEN YOU BREAK A RULE

1. **Stop immediately**
2. **Assess the impact**
3. **Notify the team**
4. **Fix the issue**
5. **Test thoroughly**
6. **Document what happened**
7. **Learn from the mistake**

---

**Remember**: These rules exist to protect user data, maintain system security, and ensure reliable operation. Breaking them has serious consequences.
