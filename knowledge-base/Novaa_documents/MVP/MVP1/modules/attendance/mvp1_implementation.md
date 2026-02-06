# MVP1 Attendance System (Manual Only)

## MVP1 Scope: Manual Attendance
**CRITICAL**: This is MANUAL attendance only. QR-based attendance comes in MVP3.

## MVP1 Attendance Flow
```
STAFF FLOW (MVP1 - Manual):
1. Staff logs in with college context
2. Selects course and date
3. Manually marks students as PRESENT/ABSENT
4. System saves attendance record
5. No QR scanning in MVP1
```

## MVP1 Database Schema
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

## MVP1 Implementation Code
```javascript
// MVP1: Manual Attendance Marking
const markManualAttendance = async (req, res) => {
  try {
    const { studentIds, courseId, date, status } = req.body;
    const collegeId = req.college._id;
    const markedBy = req.user._id;

    // MVP1: Validate inputs
    if (!Array.isArray(studentIds) || !courseId || !date) {
      return res.status(400).json({ error: "Invalid input" });
    }

    // MVP1: Create attendance records for each student
    const attendanceRecords = studentIds.map(studentId => ({
      collegeId,
      studentId,
      courseId,
      date: new Date(date),
      status, // PRESENT/ABSENT/LEAVE
      markedBy,
      createdAt: new Date()
    }));

    // MVP1: Bulk insert attendance records
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

## MVP1 Critical Indexes
- `(collegeId, studentId, date)` unique - prevents duplicate marking
- `(collegeId, date)` - for daily reports
- `(collegeId, courseId)` - for course reports

## MVP1 Limitations & Future (MVP3)
- **MVP1**: Manual attendance only (no QR scanning)
- **MVP3**: QR code generation and scanning
- **MVP1**: No real-time attendance tracking
- **MVP3**: Real-time dashboard with QR scanning
- **MVP1**: No automated at-risk alerts
- **MVP3**: Automated alerts for low attendance