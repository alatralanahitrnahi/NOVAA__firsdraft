# Attendance Module Implementation

## MVP1 Attendance System (Manual Only)

### Core Features
- Manual attendance marking (no QR scanning in MVP1)
- Student attendance tracking
- Course-wise attendance reports
- At-risk student identification (attendance < 75%)

### Database Schema
```javascript
{
  _id: ObjectId,
  collegeId: ObjectId (ref: colleges),
  studentId: ObjectId (ref: students),
  courseId: String,
  date: Date,
  status: String (enum: ["PRESENT", "ABSENT", "LEAVE"]),
  markedBy: ObjectId (ref: users), // Staff who marked
  createdAt: Date
}
```

### Critical Indexes
- `(collegeId, studentId, date)` unique - prevents duplicate marking
- `(collegeId, date)` - for daily reports
- `(collegeId, courseId)` - for course reports

### Manual Attendance Flow
```
STAFF ATTENDANCE MARKING (MVP1):
1. Staff logs in with college context
2. Selects course and date
3. Manually marks students as PRESENT/ABSENT
4. System saves attendance record
5. No QR scanning in MVP1
```

### Implementation Code
```javascript
const markManualAttendance = async (req, res) => {
  try {
    const { studentIds, courseId, date, status } = req.body;
    const collegeId = req.college._id;
    const markedBy = req.user._id;

    // Validate inputs
    if (!Array.isArray(studentIds) || !courseId || !date) {
      return res.status(400).json({ error: "Invalid input" });
    }

    // Create attendance records for each student
    const attendanceRecords = studentIds.map(studentId => ({
      collegeId,
      studentId,
      courseId,
      date: new Date(date),
      status, // PRESENT/ABSENT/LEAVE
      markedBy,
      createdAt: new Date()
    }));

    // Bulk insert attendance records
    const result = await Attendance.insertMany(attendanceRecords);

    res.json({
      message: `Attendance marked for ${result.length} students`,
      records: result,
      manualMode: true // MVP1 indicator
    });

  } catch (error) {
    res.status(500).json({ error: error.message });
  }
};
```

### At-Risk Student Detection
```javascript
const getAtRiskStudents = async (req, res) => {
  const collegeId = req.college._id;
  const courseId = req.query.courseId;

  const students = await Student.find({ collegeId });

  const atRiskList = [];

  for (const student of students) {
    const totalClasses = await Attendance.countDocuments({
      courseId,
      studentId: student._id,
      status: { $in: ['PRESENT', 'ABSENT', 'LEAVE'] }
    });

    const attendedClasses = await Attendance.countDocuments({
      courseId,
      studentId: student._id,
      status: 'PRESENT'
    });

    const percentage = totalClasses > 0 ? (attendedClasses / totalClasses) * 100 : 0;

    if (percentage < 75) {
      atRiskList.push({
        studentId: student._id,
        name: student.name,
        attendance: percentage,
        totalClasses,
        attendedClasses,
        neededClasses: Math.ceil((75 * totalClasses - attendedClasses * 100) / 25)
      });
    }
  }

  res.json({ atRiskStudents: atRiskList });
};
```

### MVP1 Limitations & Future (MVP3)
- **MVP1**: Manual attendance only (no QR scanning)
- **MVP3**: QR code generation and scanning
- **MVP1**: No real-time attendance tracking
- **MVP3**: Real-time dashboard with QR scanning
- **MVP1**: No automated at-risk alerts
- **MVP3**: Automated alerts for low attendance