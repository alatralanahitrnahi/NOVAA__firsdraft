# Master Admin Module Implementation

## MVP1 Admin Features

### Core Features
- College registration and management
- Staff account management
- System-wide settings
- Multi-tenancy oversight
- Audit trail viewing

### Database Schema (Colleges)
```javascript
{
  _id: ObjectId,
  code: String (unique), // "STX_MUMBAI_001"
  name: String,
  state: String (enum: "MAHARASHTRA"), // MVP1 only
  principalEmail: String,
  principalPhone: String,
  gstNumber: String,
  isActive: Boolean (default: true),
  createdAt: Date,
  updatedAt: Date
}
```

### Admin User Schema
```javascript
{
  _id: ObjectId,
  collegeId: ObjectId (ref: colleges),
  email: String (lowercase),
  password: String (bcrypt hashed),
  role: String (enum: ["ADMIN", "STAFF", "STUDENT"]),
  isActive: Boolean (default: true),
  lastLogin: Date,
  createdAt: Date
}
```

### College Registration
```javascript
const registerCollege = async (req, res) => {
  try {
    const { name, principalEmail, principalPhone, gstNumber } = req.body;

    // Generate unique college code
    const code = generateCollegeCode(name);

    // Validate uniqueness
    const existingCollege = await College.findOne({ 
      $or: [{ code }, { principalEmail }] 
    });

    if (existingCollege) {
      return res.status(400).json({ 
        error: 'College code or email already exists' 
      });
    }

    // Create college
    const college = await College.create({
      code,
      name,
      state: 'MAHARASHTRA', // MVP1: Maharashtra only
      principalEmail,
      principalPhone,
      gstNumber,
      isActive: true
    });

    // Create admin user
    const hashedPassword = await bcrypt.hash('TempPassword123!', 10);
    const adminUser = await User.create({
      collegeId: college._id,
      email: principalEmail,
      password: hashedPassword,
      role: 'ADMIN',
      isActive: true
    });

    // Send welcome email with temporary password
    await sendWelcomeEmail(principalEmail, 'TempPassword123!');

    res.status(201).json({
      message: 'College registered successfully',
      college: {
        id: college._id,
        code: college.code,
        name: college.name
      }
    });

  } catch (error) {
    res.status(500).json({ error: error.message });
  }
};
```

### Staff Management
```javascript
const createStaffAccount = async (req, res) => {
  try {
    const { email, name, role } = req.body;
    const collegeId = req.college._id;

    // Validate role
    if (!['STAFF', 'ADMIN'].includes(role)) {
      return res.status(400).json({ error: 'Invalid role' });
    }

    // Check if user already exists in college
    const existingUser = await User.findOne({ 
      email: email.toLowerCase(), 
      collegeId 
    });

    if (existingUser) {
      return res.status(400).json({ error: 'User already exists in this college' });
    }

    // Generate temporary password
    const tempPassword = generateTempPassword();
    const hashedPassword = await bcrypt.hash(tempPassword, 10);

    // Create staff user
    const staffUser = await User.create({
      collegeId,
      email: email.toLowerCase(),
      password: hashedPassword,
      role,
      isActive: true
    });

    // Send email with temporary password
    await sendStaffInviteEmail(email, tempPassword);

    res.status(201).json({
      message: 'Staff account created successfully',
      user: {
        id: staffUser._id,
        email: staffUser.email,
        role: staffUser.role
      }
    });

  } catch (error) {
    res.status(500).json({ error: error.message });
  }
};
```

### Multi-Tenancy Oversight
```javascript
const getCollegeStats = async (req, res) => {
  try {
    const collegeId = req.college._id;

    // Get stats for the college
    const stats = await Promise.all([
      User.countDocuments({ collegeId }),
      Student.countDocuments({ collegeId }),
      Admission.countDocuments({ collegeId }),
      Transaction.countDocuments({ collegeId, status: 'SUCCESS' }),
      Attendance.countDocuments({ collegeId, date: { $gte: new Date(Date.now() - 30*24*60*60*1000) } }) // Last 30 days
    ]);

    const [userCount, studentCount, admissionCount, paymentCount, attendanceCount] = stats;

    res.json({
      collegeId,
      stats: {
        totalUsers: userCount,
        totalStudents: studentCount,
        totalAdmissions: admissionCount,
        successfulPayments: paymentCount,
        recentAttendance: attendanceCount
      }
    });

  } catch (error) {
    res.status(500).json({ error: error.message });
  }
};
```

### Audit Trail
```javascript
// Audit log schema
const auditLogSchema = {
  _id: ObjectId,
  collegeId: ObjectId (ref: colleges),
  userId: ObjectId (ref: users),
  action: String (enum: ["DOCUMENT_VERIFIED", "ADMISSION_REJECTED", "PAYMENT_PROCESSED", "ATTENDANCE_MARKED"]),
  resourceType: String (enum: ["ADMISSION", "TRANSACTION", "ATTENDANCE"]),
  resourceId: ObjectId,
  oldValues: Object,
  newValues: Object,
  ipAddress: String,
  userAgent: String,
  createdAt: Date
};

const getAuditTrail = async (req, res) => {
  try {
    const collegeId = req.college._id;
    const { startDate, endDate, action, userId } = req.query;

    let query = { collegeId };

    if (startDate) query.createdAt = { $gte: new Date(startDate) };
    if (endDate) {
      query.createdAt = query.createdAt || {};
      query.createdAt.$lte = new Date(endDate);
    }
    if (action) query.action = action;
    if (userId) query.userId = userId;

    const auditLogs = await AuditLog.find(query)
      .populate('userId', 'email')
      .sort({ createdAt: -1 })
      .limit(100); // Limit for performance

    res.json({ auditLogs });

  } catch (error) {
    res.status(500).json({ error: error.message });
  }
};
```

### MVP1 Limitations & Future (MVP2+)
- **MVP1**: Maharashtra colleges only
- **MVP2**: Multi-state support
- **MVP1**: Basic admin features only
- **MVP2**: Advanced admin tools and analytics
- **MVP1**: Manual college registration
- **MVP3**: Self-service college onboarding