# MVP1 - WHAT EXISTS (DO NOT REBUILD)

**Purpose**: Prevent duplicate work and protect stable modules  
**Target Audience**: All Developers  
**Rule**: DO NOT MODIFY unless explicitly instructed

---

## 🛡️ PROTECTED MODULES (DO NOT TOUCH)

### 1. Authentication System ✅ COMPLETE

**What's Implemented**:
- User registration and login
- JWT token generation and validation
- Password hashing with bcrypt
- Token refresh mechanism
- Role-based access control (RBAC)
- Middleware: `authenticate`, `authorizeRole`

**APIs**:
```
POST /auth/register
POST /auth/login
POST /auth/refresh-token
GET  /auth/me
POST /auth/logout
```

**Roles Supported**:
- SUPER_ADMIN
- COLLEGE_ADMIN
- TEACHER
- STUDENT

**DO NOT**:
- ❌ Change authentication logic
- ❌ Modify JWT token structure
- ❌ Add new roles without approval
- ❌ Change password hashing algorithm
- ❌ Modify middleware behavior

**You CAN**:
- ✅ Use existing auth middleware in new routes
- ✅ Reference user roles in your code
- ✅ Add role checks for new features

---

### 2. College Management ✅ COMPLETE

**What's Implemented**:
- College registration
- College profile (name, code, address, logo)
- Department management
- Course management
- Academic year configuration
- Staff management

**APIs**:
```
POST /colleges/register
GET  /colleges/my-college
PUT  /colleges/edit/my-college
POST /departments
GET  /departments
POST /courses
GET  /courses
```

**Multi-Tenancy Enforcement**:
- Every college has unique `collegeId`
- Every college has unique `code` for student registration
- College context enforced via middleware

**DO NOT**:
- ❌ Change college schema
- ❌ Modify college registration flow
- ❌ Remove collegeId from queries
- ❌ Change department/course structure

**You CAN**:
- ✅ Query colleges for dropdowns
- ✅ Use collegeId in your modules
- ✅ Reference departments and courses

---

### 3. Student Management ✅ COMPLETE

**What's Implemented**:
- Student self-registration via college code
- Student profile management
- Admission approval workflow
- Student status tracking (PENDING/APPROVED/REJECTED)
- Student-course-department mapping

**APIs**:
```
POST /students/register/:collegeCode
GET  /students/my-profile
PUT  /students/update-my-profile
GET  /students/registered (Admin)
PUT  /students/:studentId/approve (Admin)
PUT  /students/:studentId/reject (Admin)
GET  /students/approved-students (Admin)
```

**Student Schema** (Key Fields):
```javascript
{
  collegeId: ObjectId,
  departmentId: ObjectId,
  courseId: ObjectId,
  fullName: String,
  email: String,
  password: String (hashed),
  mobileNumber: String,
  category: Enum (GEN/OBC/SC/ST/OTHER),
  status: Enum (PENDING/APPROVED/REJECTED),
  admissionYear: Number,
  currentSemester: Number
}
```

**DO NOT**:
- ❌ Change student registration flow
- ❌ Modify approval workflow
- ❌ Remove category field (used for fee calculation)
- ❌ Change status enum values

**You CAN**:
- ✅ Query approved students for attendance
- ✅ Query students for fee assignment
- ✅ Use student data in reports

---

### 4. Teacher Management ✅ COMPLETE

**What's Implemented**:
- Teacher profile creation (by Admin)
- Teacher-department mapping
- Teacher-user account linking
- Employee ID management

**APIs**:
```
POST /teachers (Admin)
GET  /teachers (Admin)
GET  /teachers/:id (Admin)
PUT  /teachers/:id (Admin)
DELETE /teachers/:id (Admin)
GET  /teachers/my-profile (Teacher)
```

**Teacher Schema** (Key Fields):
```javascript
{
  collegeId: ObjectId,
  userId: ObjectId,
  departmentId: ObjectId,
  name: String,
  email: String,
  employeeId: String,
  designation: String,
  status: Enum (ACTIVE/INACTIVE)
}
```

**DO NOT**:
- ❌ Change teacher creation flow
- ❌ Modify teacher-user linking
- ❌ Remove employeeId field

**You CAN**:
- ✅ Query teachers for timetable assignment
- ✅ Use teacher data in attendance
- ✅ Reference teachers in reports

---

### 5. Timetable System ✅ COMPLETE

**What's Implemented**:
- Timetable creation (HOD/Admin)
- Timetable slots (day, time, subject, teacher)
- Slot-based attendance foundation
- Department-course-semester mapping

**APIs**:
```
POST /timetables
GET  /timetables
POST /timetables/:id/slots
GET  /timetables/:id/slots
```

**Timetable Slot Schema**:
```javascript
{
  collegeId: ObjectId,
  timetableId: ObjectId,
  departmentId: ObjectId,
  courseId: ObjectId,
  subjectId: ObjectId,
  teacherId: ObjectId,
  day: String,
  startTime: String,
  endTime: String
}
```

**DO NOT**:
- ❌ Change timetable structure
- ❌ Modify slot schema
- ❌ Remove any ID fields (used for attendance)

**You CAN**:
- ✅ Query slots for attendance session creation
- ✅ Use timetable data in reports
- ✅ Display timetable in frontend

---

### 6. Attendance System ✅ COMPLETE (Manual)

**What's Implemented**:
- Session-based attendance (slot-driven)
- Teacher creates attendance session for a slot
- Manual student marking (PRESENT/ABSENT)
- Duplicate prevention (one session per slot per day)
- Attendance records per student per session

**APIs**:
```
POST /attendance/sessions (Teacher)
GET  /attendance/sessions (Teacher)
GET  /attendance/sessions/:sessionId (Teacher)
PUT  /attendance/sessions/:sessionId (Teacher)
PUT  /attendance/sessions/:sessionId/close (Teacher)
DELETE /attendance/sessions/:sessionId (Teacher)
GET  /attendance/sessions/:sessionId/students (Teacher)
POST /attendance/sessions/:sessionId/mark (Teacher)
PUT  /attendance/sessions/:sessionId/edit (Teacher)
```

**Attendance Flow**:
```
1. Teacher selects timetable slot
2. System creates attendance session
3. System auto-fetches approved students
4. Teacher marks PRESENT/ABSENT
5. Teacher closes session (no further edits)
```

**DO NOT**:
- ❌ Change session-based model
- ❌ Modify attendance record structure
- ❌ Remove slot_id reference
- ❌ Allow cross-teacher attendance marking

**You CAN**:
- ✅ Query attendance for reports
- ✅ Calculate attendance percentage
- ✅ Display attendance in frontend

---

### 7. Fee Structure System ✅ COMPLETE

**What's Implemented**:
- Course + category based fee structures
- Installment-based fee breakdown
- Auto-creation of StudentFee on admission approval
- Fee dashboard APIs

**APIs**:
```
POST /fee-structures (Admin)
GET  /fee-structures (Admin)
GET  /fee-structures/:id (Admin)
PUT  /fee-structures/:id (Admin)
DELETE /fee-structures/:id (Admin)
```

**Fee Structure Schema**:
```javascript
{
  collegeId: ObjectId,
  courseId: ObjectId,
  category: Enum (GEN/OBC/SC/ST),
  totalFee: Number,
  installments: [{
    name: String,
    amount: Number,
    dueDate: Date
  }]
}
```

**StudentFee Schema**:
```javascript
{
  studentId: ObjectId,
  collegeId: ObjectId,
  courseId: ObjectId,
  totalFee: Number,
  paidAmount: Number,
  installments: [{
    name: String,
    amount: Number,
    dueDate: Date,
    status: Enum (PENDING/PAID),
    paidAt: Date,
    stripePaymentId: String
  }]
}
```

**DO NOT**:
- ❌ Change fee structure schema
- ❌ Modify installment structure
- ❌ Remove category-based fee logic
- ❌ Change auto-creation trigger

**You CAN**:
- ✅ Query student fees for payment processing
- ✅ Update installment status after payment
- ✅ Display fee dashboard in frontend

---

### 8. Notification System ✅ COMPLETE (Internal)

**What's Implemented**:
- In-app notifications
- Role-based notification creation
- Admin → Teachers/Students
- Teacher → Students
- Read receipt tracking
- Notification count APIs

**APIs**:
```
POST /notifications/admin/create (Admin)
POST /notifications/teacher/create (Teacher)
GET  /notifications/admin/read (Admin)
GET  /notifications/teacher/read (Teacher)
GET  /notifications/student/read (Student)
GET  /notifications/count/admin (Admin)
GET  /notifications/count/teacher (Teacher)
GET  /notifications/count/student (Student)
GET  /notifications/unread/bell (All)
POST /notifications/:notificationId/read (All)
PUT  /notifications/edit-note/:id (Creator)
DELETE /notifications/delete-note/:id (Creator)
```

**DO NOT**:
- ❌ Change notification schema
- ❌ Modify role-based visibility logic
- ❌ Remove read receipt tracking

**You CAN**:
- ✅ Display notifications in frontend
- ✅ Add notification bell icon
- ✅ Trigger notifications from your modules

---

## 🔐 MIDDLEWARE (DO NOT MODIFY)

### Authentication Middleware
```javascript
authenticate(req, res, next)
```
- Validates JWT token
- Attaches `req.user` with user data
- Use on all protected routes

### Role Authorization Middleware
```javascript
authorizeRole(['COLLEGE_ADMIN', 'TEACHER'])
```
- Checks if user has required role
- Must come after `authenticate`

### College Context Middleware
```javascript
collegeMiddleware(req, res, next)
```
- Validates college context
- Enforces multi-tenancy
- Attaches `req.user.collegeId`

**DO NOT**:
- ❌ Modify middleware logic
- ❌ Skip middleware on protected routes
- ❌ Create custom auth middleware

**You MUST**:
- ✅ Use existing middleware on all new routes
- ✅ Follow middleware order: authenticate → authorizeRole → collegeMiddleware

---

## 📊 REPORTS BACKEND ✅ COMPLETE

**What's Implemented**:
- Admission summary report API
- Payment summary report API
- Student-wise payment status API
- Attendance summary report API
- Low attendance student report API

**APIs**:
```
GET /reports/admission-summary (Admin)
GET /reports/payment-summary (Admin)
GET /reports/student-payment-status (Admin)
GET /reports/attendance-summary (Admin)
GET /reports/low-attendance-students (Admin)
```

**DO NOT**:
- ❌ Change report calculation logic
- ❌ Modify report data structure
- ❌ Remove college-level filtering

**You CAN**:
- ✅ Consume these APIs in frontend
- ✅ Add charts/graphs in frontend
- ✅ Add export functionality (MVP2)

---

## 🚫 CRITICAL RULES

### Rule 1: Multi-Tenancy is Sacred
Every query MUST include `collegeId`. No exceptions.

**Wrong**:
```javascript
const students = await Student.find({ status: 'APPROVED' });
```

**Right**:
```javascript
const students = await Student.find({ 
  collegeId: req.user.collegeId,
  status: 'APPROVED' 
});
```

### Rule 2: Never Skip Middleware
All protected routes MUST use authentication middleware.

**Wrong**:
```javascript
router.get('/students', getStudents);
```

**Right**:
```javascript
router.get('/students', authenticate, authorizeRole(['COLLEGE_ADMIN']), getStudents);
```

### Rule 3: Never Modify Existing Schemas
Changing schemas breaks existing data. Add new fields only if critical.

### Rule 4: Never Remove Existing APIs
Frontend may be using them. Deprecate, don't delete.

### Rule 5: Never Change Enum Values
Code depends on exact enum values. Adding is OK, changing is NOT.

---

## ✅ WHAT YOU CAN DO

1. **Use existing APIs** in your frontend
2. **Query existing collections** for your features
3. **Reference existing schemas** in your code
4. **Add new routes** that use existing middleware
5. **Build on top** of existing modules
6. **Add new fields** to schemas (if critical and approved)

---

## 🆘 WHEN YOU NEED TO MODIFY

If you MUST modify a protected module:

1. **Document why** it's necessary
2. **Get approval** from tech lead
3. **Test thoroughly** to ensure no breakage
4. **Update this document** after changes

---

**Remember**: These modules are STABLE and WORKING. Your job is to complete MVP2 gaps, not rebuild MVP1.
