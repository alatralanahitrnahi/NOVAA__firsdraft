
NOVAA – Smart College Management System
Stage 1 (MVP) Project Documentation

1. Project Overview
NOVAA is a Smart College Management System designed to manage end-to-end college operations in a centralized, secure, and scalable way. The system follows a multi-tenant architecture, where each college’s data is isolated using a collegeId.
This document is prepared to clearly explain what has been implemented and what is remaining in Stage 1 (MVP level) for management review.

2. MVP Stage 1 Objective
The objective of Stage 1 (MVP) is to:
Build core operational modules


Enable student onboarding and basic administration


Establish a strong backend foundation


Deliver a usable system without advanced automation



3. Core Modules – Implementation Status
Module 1: AUTH (Authentication & Access Control) ✅ Completed
Purpose: Secure system access and role management
Implemented Features:
User login


JWT token generation


Role-based access control (Admin / Staff / Student)


College context handling via collegeId


Status: Fully implemented and stable

Module 2: COLLEGES (Tenant Management) ✅ Completed
Purpose: Manage multiple colleges on the same platform
Implemented Features:
College registration


College-level settings configuration


Staff management


Strict data isolation between colleges


Status: Fully implemented

Module 3: ADMISSIONS (Student Applications) ✅ Completed
Purpose: Handle student admission lifecycle
Implemented Features:
Student application form submission


Admin-side approval


Application status tracking


Approval workflow (payment dependent)


Status: Implemented (final approval depends on payment module)

Module 4: PAYMENTS (Fee Management) ✅ Completed (Stripe – MVP)
Purpose: Manage student fee collection with secure, installment-based online payments.
Implemented Features (MVP):
Course + category-based fee structure management
Automatic StudentFee creation on admission approval
Installment-based fee calculation (semester / admission-wise)
Student fee dashboard APIs (total fee, paid, pending)
Stripe Checkout integration (test mode)
Server-side payment verification using Stripe APIs
Installment status updates (PENDING → PAID)
Admin payment reports (paid vs pending)
College-wise (multi-tenant) payment isolation
Stripe Payment Flow:
Student initiates payment for a pending installment
Backend creates Stripe Checkout Session
Student completes payment on Stripe-hosted page
Backend verifies payment status using Stripe API
Fee records and dashboards update automatically
Security & Validation:
No card data stored in backend
Server-side payment verification
Installments cannot be double-paid
Payments mapped to student & college
MVP Limitations:
Webhooks not implemented (confirmation via redirect)
Refunds not supported
Production credentials pending
Status: Fully implemented and demo-ready using Stripe test mode

Module 5: ATTENDANCE (Timetable Slot–Based Attendance) ✅ Completed (Finalized – MVP+)
Purpose
Provide a secure, slot-driven attendance system that mirrors real-world college operations, ensuring accuracy, role ownership, and strict multi-tenant data isolation.
Attendance is designed to be teacher-driven but system-controlled, preventing manual misuse while enabling scalable reporting.

Finalized Attendance Architecture (MVP)
Attendance is strictly derived from timetable slots, not manually selected courses or subjects.
Attendance is created based on:
Timetable Slot


Lecture Date


Lecture Number


Assigned Teacher


College (Tenant Context)


🧠 Core Principle
Attendance is always created for a Timetable Slot, not directly for a course or subject.
This guarantees:
Correct subject & course mapping


Teacher ownership enforcement


No cross-college or cross-teacher attendance



Data Model Alignment (Finalized)
Timetable (HOD Created)
Source of truth for:
college_id


department_id


course_id


semester


academicYear


TimetableSlot (HOD Created – Self-Contained)
Represents one lecture slot in the week.
 Stores:
college_id


timetable_id


department_id


course_id


subject_id


teacher_id


day


startTime


endTime


✅ Slot is fully self-contained and attendance-ready.
AttendanceSession (Teacher Created)
Represents attendance for one lecture on one date.
 Stores:
college_id


department_id


course_id


subject_id


teacher_id


slot_id


lectureDate


lectureNumber


totalStudents


status (OPEN / CLOSED)


AttendanceRecord
One record per student per session


Prevents duplicate attendance marking



Attendance Session Logic (Implemented)
Preconditions
Teacher is authenticated (JWT)


Timetable exists


Timetable Slot exists


Slot belongs to logged-in teacher


Slot belongs to same college (tenant)


Session Creation Rules
Only one attendance session per slot per day per lecture number


Duplicate prevention enforced using DB index:

 slot_id + lectureDate + lectureNumber


Students are auto-counted based on:


college_id


course_id


APPROVED admission status


Frontend is not trusted for:


course


department


student list


college context


Session Status Flow
OPEN → Attendance marking allowed


CLOSED → No edits permitted





Attendance APIs (Finalized)
Attendance Session APIs
POST /attendance/sessions


GET /attendance/sessions


GET /attendance/sessions/:sessionId


PUT /attendance/sessions/:sessionId


DELETE /attendance/sessions/:sessionId (Teacher only)


PUT /attendance/sessions/:sessionId/close


Attendance Marking APIs
GET /attendance/sessions/:sessionId/students


POST /attendance/sessions/:sessionId/mark


PUT /attendance/sessions/:sessionId/edit



Attendance Session Creation Flow (System-Controlled)
1️⃣ Teacher sends request with:
{
 "slot_id": "SLOT_OBJECT_ID",
 "lectureDate": "YYYY-MM-DD",
 "lectureNumber": 2
}
2️⃣ Backend resolves:
Teacher identity


College context


Slot ownership


3️⃣ Slot validation ensures:
Correct teacher


Correct college


Correct course & department mapping


4️⃣ Duplicate session check enforced
5️⃣ Student count auto-derived from approved admissions
6️⃣ Attendance session created with derived data only

Security & Validation Guarantees
✔ Teacher cannot mark attendance for another teacher
 ✔ Attendance cannot be created without approved students
 ✔ No duplicate attendance per slot/day/lecture
 ✔ No cross-college data access
 ✔ No manual student selection
 ✔ No frontend-controlled tenant data
Validation errors such as:
“Invalid timetable slot for this teacher”


“No approved students found for this course”


are expected and confirm correct system behavior.

Migration & Stability Measures
One-time migration performed to update older timetable slots with:


college_id


department_id


course_id


Schema and controller logic fully aligned


Backward compatibility ensured






Module 6: REPORTS (Analytics & Dashboards) ✅ Completed (Backend – MVP Scope)
Purpose: Provide read-only analytics and insights from existing system data
Implemented Features:
Dedicated Reports module with clean architecture:


reports.routes.js


reports.controller.js


reports.service.js



Read-only reporting APIs (no data mutation)


Dynamic aggregation using existing collections


Mandatory filtering by collegeId


JWT authentication and role-based access control


Restricted access to:


COLLEGE_ADMIN


SUPER_ADMIN


Scalable design for future roles (Teacher, Student)

4. Reports Module – Implemented Reports (MVP)
🟢 Admission Summary Report
Used by: College Admin
 Data Source: Admissions
Output:
Total applications


Approved students


Pending students


Rejected students

🟢 Payment Summary Report
Used by: College Admin

 Data Source: Student Fees / Payments
Output:
Total expected fee


Total collected fee


Total pending fee



🟢 Student-wise Payment Status Report
Used by: College Admin
Output:
Student payment status


Installment-level paid / pending information



🟢 Attendance Summary Report
Used by: College Admin
Output:
Overall attendance percentage


Total attendance records

🟢 Low Attendance Student Report
Used by: College Admin
Output:
Student-wise attendance percentage


Students below configurable threshold (default: 75%)







NOTIFICATIONS (Internal Communication System)      Completed
Purpose
Enable structured, role-based communication within the college by delivering academic, administrative, and system notifications to the appropriate users with strict access control.
This module ensures clear communication while maintaining ownership, security, and college-level data isolation.

Implemented Features (MVP Scope)
College-wide announcements by College Admin


Faculty-to-student notifications by Teachers


Role-wise notification visibility


Dashboard notification badge (bell icon) support






Role-Based Access & Visibility
College Admin


Can create notifications for teachers and students


Can update/delete only their own notifications


Cannot modify teacher-created notifications



Teacher


Can create notifications for students only


Can update/delete only their own notifications


Student


Read-only access


Can view admin and teacher notifications


No create/update/delete permissions



Security & Data Isolation
Strict role validation using middleware


College-level isolation enforced using college_id


No cross-role modification allowed


Notifications are protected from unauthorized access or overwrites



Notification Dashboard Support
Role-wise notification segregation:


Admin Dashboard → Admin & Teacher notifications


Teacher Dashboard → Own & Admin notifications


Student Dashboard → Admin & Teacher notifications


Notification count APIs implemented




Status
✅ Fully Implemented (MVP)
Notification CRUD completed


Role-based visibility enforced


Dashboard notification counts supported


5. MVP Stage 1 Summary Table
Module
Status
Authentication
✅ Completed
Colleges
✅ Completed
Admissions
✅ Completed
Payments
✅ Completed (Backend)
Attendance
✅ Completed (Manual)
Reports

Notification
✅ Completed (Backend)

✅ Completed (Backend)


6. Current MVP stage-1 Limitations & stage    2-Enhancement
       1 Payments & Financial Limitations
Stripe payments are implemented in test/demo mode


No Stripe webhooks (payment confirmation relies on frontend redirect)


Refunds and partial refunds are not supported


No automated payment reminders or dunning




2️ Reporting & Analytics Limitations
Reports are read-only (no exports to Excel/PDF)


No historical trend analysis (month-over-month, year-over-year)


Dashboards are backend-ready but frontend visualizations are limited



3️ Attendance Limitations
Attendance is manual only


No QR-based or biometric attendance


No automated absence alerts to students 


No timetable-based auto attendance locking





4️ Notifications Limitations
Notifications are in-app only


No email, SMS, or WhatsApp delivery


No scheduled or recurring notifications




5️  Automation & Workflow Limitations
Admission approvals require manual admin action


No automated escalation workflows


No background jobs (cron-based processing)


No real-time push updates (WebSockets)


All limitations are well-understood and addressable in the next development phase.


Student Collection (students)
Purpose 
Stores complete student lifecycle data from registration to approval, linked to college, department, and course.

Field
Type
Required
Description
college_id
ObjectId → College
✅
Tenant isolation
department_id
ObjectId → Department
✅
Academic department
course_id
ObjectId → Course
✅
Enrolled course
fullName
String
✅
Student name
email
String
✅
Login & communication
password
String
✅
Hashed password
mobileNumber
String
✅
Primary contact
gender
Enum
✅
Male / Female / Other
dateOfBirth
Date
✅
DOB
addressLine
String
✅
Address
city
String
✅
City
state
String
✅
State
pincode
String
✅
Postal code
admissionYear
Number
✅
Admission batch
currentSemester
Number
✅
Current semester
category
Enum
✅
GEN / OBC / SC / ST / OTHER
nationality
String
❌
Default: Indian
bloodGroup
String
❌
Medical info
alternateMobile
String
❌
Backup contact
status
Enum
✅
PENDING / APPROVED / REJECTED
registeredVia
Enum
✅
SELF
approvedBy
ObjectId → User
❌
Admin approver
approvedAt
Date
❌
Approval time
rejectionReason
String
❌
Reason for rejection
createdAt / updatedAt
Date
Auto
Timestamps

Schema FieldsIndexes
Unique: (college_id + email)
 → Prevents duplicate student registration per college



📦 Teacher Collection (teachers)
Purpose
Stores faculty information linked to system users and departments.



Schema Fields
Field
Type
Required
Description
college_id
ObjectId → College
✅
Tenant isolation
user_id
ObjectId → User
✅
Login identity
department_id
ObjectId → Department
✅
Assigned department
name
String
✅
Teacher name
email
String
✅
Official email
employeeId
String
✅
College employee ID
designation
String
✅
Assistant Prof / HOD etc
qualification
String
✅
Academic qualification
experienceYears
Number
✅
Teaching experience
status
Enum
✅
ACTIVE / INACTIVE
createdBy
ObjectId → User
✅
Admin who created
createdAt / updatedAt
Date
Auto
Timestamps



Indexes
Unique: (college_id + employeeId)


Unique: user_id



2️⃣ Collection List (So Far)
Collection Name
Purpose
students
Student lifecycle & academic mapping
teachers
Faculty records
users
Authentication & roles (referenced)
colleges
Tenant root entity
departments
Academic departments
courses
Academic courses


3️⃣ Relationships (Actual, Implemented)
Student Relationships
College → Students (1 : Many)


Department → Students (1 : Many)


Course → Students (1 : Many)


User → Student Approval (1 : Many via approvedBy)


Teacher Relationships
College → Teachers (1 : Many)


Department → Teachers (1 : Many)


User → Teacher (1 : 1)


User → Teacher.createdBy (Admin ownership)


🧠 Design Note
 All student & teacher records are strictly college-scoped, enforcing multi-tenant isolation at query level.

4️⃣ Roles & Permissions (Table Format – Reality)
Student Module Permissions
Role
Register
View Own Profile
Update Own Profile
Approve / Reject
View Students
Delete
Public
✅
❌
❌
❌
❌
❌
STUDENT
❌
✅
✅
❌
❌
❌
COLLEGE_ADMIN
❌
❌
✅
✅
✅
✅
TEACHER
❌
❌
❌
❌
❌
❌


Teacher Module Permissions
Role
Create
Read
Update
Delete
View Own Profile
COLLEGE_ADMIN
✅
✅
✅
✅
❌
TEACHER
❌
❌
❌
❌
✅
STUDENT
❌
❌
❌
❌
❌


5️⃣ API Reality – Implemented APIs ONLY
🎓 Student APIs
Public
POST /students/register/:collegeCode
 → Student self-registration


Student (Authenticated)
GET /students/my-profile


PUT /students/update-my-profile


College Admin
GET /students/registered


GET /students/registered/:id


GET /students/approved-students


GET /students/approved-stud/:id


PUT /students/:studentId/approve


PUT /students/:studentId/reject


PUT /students/:id


DELETE /students/:id



👨‍🏫 Teacher APIs
Teacher
GET /teachers/my-profile


College Admin
POST /teachers


GET /teachers


GET /teachers/:id


PUT /teachers/:id


DELETE /teachers/:id



6️⃣ Key Architectural Guarantees (From Code)
✔ Strict college-level isolation (collegeMiddleware)
 ✔ Role-based access enforced (role.middleware)
 ✔ Teacher & student identity validated via middleware
 ✔ No cross-role privilege escalation
 ✔ No frontend-controlled tenant data

 College Collection (colleges)
Purpose
Acts as the root tenant entity in the system. Every academic, financial, and operational record is scoped under a college.
Schema Fields
Field
Type
Required
Description
name
String
✅
College name
code
String
✅
Unique college code (used for registration)
email
String
✅
Official contact email
contactNumber
String
✅
College contact
address
String
✅
Physical address
establishedYear
Number
✅
Year of establishment
logo
String
❌
Logo URL / file path
isActive
Boolean
❌
College active status
registrationUrl
String
✅
Public student registration URL
registrationQr
String
✅
QR code for registration
createdAt
Date
Auto
Creation timestamp

Indexes
Unique: code



📦 AttendanceSession Collection (attendance_sessions)
Purpose
Represents attendance for one lecture slot on one date, created and controlled by the system.
Schema Fields
Field
Type
Required
Description
college_id
ObjectId → College
✅
Tenant isolation
department_id
ObjectId → Department
✅
Academic department
course_id
ObjectId → Course
✅
Course
subject_id
ObjectId → Subject
✅
Subject
teacher_id
ObjectId → Teacher
✅
Assigned teacher
timetable_id
ObjectId → Timetable
✅
Parent timetable
slot_id
ObjectId → TimetableSlot
✅
Lecture slot
lectureDate
Date
✅
Lecture date
totalStudents
Number
✅
Auto-counted
status
Enum
✅
OPEN / CLOSED
createdAt / updatedAt
Date
Auto
Timestamps

Indexes
Unique: (slot_id + lectureDate)
 → Prevents duplicate attendance per lecture per day



📦 AttendanceRecord Collection (attendance_records)
Purpose
Stores student-wise attendance marking for a session.
Schema Fields
Field
Type
Required
Description
college_id
ObjectId → College
✅
Tenant isolation
session_id
ObjectId → AttendanceSession
✅
Parent session
student_id
ObjectId → Student
✅
Student
status
Enum
✅
PRESENT / ABSENT
markedBy
ObjectId → Teacher
✅
Teacher
createdAt / updatedAt
Date
Auto
Timestamps

Indexes
Unique: (session_id + student_id)
 → One record per student per session



📦 Attendance Collection (attendances) ⚠️ Legacy
Purpose
Earlier subject-based attendance model (pre session-based design).
Schema Fields
Field
 Type
Required
Description
studentId
ObjectId → Student
✅
Student
subjectId
ObjectId → Subject
✅
Subject
teacherId
ObjectId → Teacher
✅
Teacher
date
String
✅
YYYY-MM-DD
status
Enum
✅
Present / Absent

Indexes
Unique: (studentId + subjectId + date)


🧠 Note:
 This model exists in codebase but has been superseded by the AttendanceSession + AttendanceRecord architecture.

2️⃣ Collection List (So Far)
Collection
Purpose
colleges
Tenant root
students
Student lifecycle
teachers
Faculty records
attendance_sessions
Slot-based attendance
attendance_records
Student attendance
attendances
Legacy attendance
users
Authentication
departments
Academic units
courses
Academic programs
subjects
Subjects







3️⃣ Relationships (Actual & Enforced)
College Relationships
College → Students (1 : Many)


College → Teachers (1 : Many)


College → AttendanceSessions (1 : Many)


College → AttendanceRecords (1 : Many)


Attendance Relationships
TimetableSlot → AttendanceSession (1 : Many over time)


AttendanceSession → AttendanceRecord (1 : Many)


Student → AttendanceRecord (1 : Many)


Teacher → AttendanceSession (1 : Many)


🛡️ All attendance data is strictly scoped by college_id.

4️⃣ Roles & Permissions (Reality Matrix)
College Module
Role
View College
Update College
COLLEGE_ADMIN
✅
✅
TEACHER
❌
❌
STUDENT
❌
❌
SUPER_ADMIN
❌
❌



Attendance Module
Role
Create 
Session
Mark
Attendance
Edit
Close Session
View
TEACHER
✅
✅
✅
✅
✅
COLLEGE_ADMIN
❌
❌
❌
❌
✅
STUDENT
❌
❌
❌
❌
❌


5️⃣ API Reality – Implemented APIs Only

🏫 College APIs
Method
Endpoint
Role
GET
/college/my-college
COLLEGE_ADMIN
PUT
/college/edit/my-college
COLLEGE_ADMIN


	

📚 Attendance APIs (Session-Based)
Attendance Session
POST /attendance/sessions → Teacher


GET /attendance/sessions → Teacher / Admin


GET /attendance/sessions/:sessionId → Teacher


PUT /attendance/sessions/:sessionId → Teacher


PUT /attendance/sessions/:sessionId/close → Teacher


DELETE /attendance/sessions/:sessionId → Teacher


Attendance Records
GET /attendance/sessions/:sessionId/students → Teacher


POST /attendance/sessions/:sessionId/mark → Teacher


PUT /attendance/sessions/:sessionId/edit → Teacher



6️⃣ Architectural Guarantees (Confirmed)
✔ Slot-based attendance (ERP correct model)
 ✔ One session per lecture per day
 ✔ One attendance per student per session
 ✔ No frontend trust for tenant or course data
 ✔ Strict role & college isolation
 ✔ Legacy attendance retained but not used



 FeeStructure Collection (fee_structures)
Purpose
Defines course + category–based fee configuration at the college level.
 Acts as the source of truth for student fee generation upon admission approval.
Schema Fields
Field
Type
Required
Description
college_id
ObjectId → College
✅
Tenant isolation
course_id
ObjectId → Course
✅
Course
category
Enum
✅
GEN / OBC / SC / ST
totalFee
Number
✅
Total payable fee
installments
Array
✅
Fee breakup
installments.name
String
✅
Installment name
installments.amount
Number
✅
Amount
installments.dueDate
Date
✅
Due date
createdAt / updatedAt
Date
Auto
Timestamps

Indexing & Constraints
Fee structures are college-scoped


One structure per (college_id + course_id + category) enforced at application logic level



📦 StudentFee Collection (student_fees)
Purpose
Stores student-specific fee ledger, auto-created after admission approval using FeeStructure.
Schema Fields
Field
Type
Required
Description
student_id
ObjectId → Student
✅
Student
college_id
ObjectId → College
✅
Tenant isolation
course_id
ObjectId → Course
✅
Course
totalFee
Number
✅
Total payable fee
paidAmount
Number
❌
Default: 0
installments
Array
✅
Installment tracking
installments.name
String
❌
Installment name
installments.amount
Number
❌
Installment amount
installments.dueDate
Date
✅
Due date
installments.status
Enum
❌
PENDING / PAID
installments.razorpayPaymentId
String
❌
Payment reference
installments.paidAt
Date
❌
Payment timestamp
installments.reminderSent
Boolean
❌
Reminder flag

Design Notes
Installments are derived from FeeStructure


No manual installment creation by frontend


Prevents double payment by status tracking








2️⃣ Collection List (Payments Module)
Collection
Purpose
fee_structures
Course-wise fee definitions
student_fees
Student-level fee ledger
students
Fee ownership
colleges
Tenant root
courses
Academic linkage


3️⃣ Relationships (Actual & Enforced)
Fee Structure Relationships
College → FeeStructure (1 : Many)


Course → FeeStructure (1 : Many)


Student Fee Relationships
Student → StudentFee (1 : 1)


College → StudentFee (1 : Many)


Course → StudentFee (1 : Many)


🛡️ All payment data is strictly scoped using college_id.

4️⃣ Roles & Permissions (Reality Matrix)
Fee Structure Management
Role
Create
Read
Update
Delete
COLLEGE_ADMIN
✅
✅
✅
✅
STUDENT
❌
❌
❌
❌
TEACHER
❌
❌
❌
❌


Student Payments
Role
View Fee Dashboard
Create Payment
Confirm Payment
STUDENT
✅
✅
✅
COLLEGE_ADMIN
❌
❌
❌
TEACHER
❌
❌
❌


5️⃣ API Reality – Implemented APIs ONLY

💰 Fee Structure APIs (College Admin)
Method
Endpoint
Description
POST
/fee-structures
Create fee structure
GET
/fee-structures
Get all fee structures
GET
/fee-structures/:feeStructureId
Get fee structure by ID
PUT
/fee-structures/:feeStructureId
Update fee structure
DELETE
/fee-structures/:feeStructureId
Delete fee structure


💳 Student Fee APIs
Student (Authenticated)
Method
Endpoint
Description
GET
/payments/my-fee-dashboard
Student fee dashboard
POST
/payments/create-order
Create payment order
POST
/payments/mock-success
Mock payment (dev only)


💳 Stripe Payment APIs (Student)
Method
Endpoint
Description
POST
/stripe/create-checkout-session
Create Stripe checkout
POST
/stripe/confirm-payment
Confirm payment



6️⃣ Payment Flow 
 1️⃣ Admission approved
 2️⃣ StudentFee auto-created from FeeStructure
 3️⃣ Student views dashboard
 4️⃣ Student initiates payment
 5️⃣ Stripe Checkout session created
 6️⃣ Payment confirmed server-side
 7️⃣ Installment marked PAID





7️⃣ Security & Design Guarantees
✔ No card data stored
 ✔ Server-side payment confirmation
 ✔ Installment-based tracking
 ✔ No duplicate payments
 ✔ College-level isolation
 ✔ Mock payments isolated for dev


Notification Collection (notifications)
Purpose
Stores college-wide announcements & alerts.
Schema
Field
Type
Description
college_id
ObjectId → College
Tenant isolation
createdByRole
Enum
COLLEGE_ADMIN / TEACHER
createdBy
ObjectId
Creator
title
String
Notification title
message
String
Body
type
Enum
GENERAL, EXAM, FEE, etc
target
Enum
ALL / STUDENTS
actionUrl
String
Optional redirect
expiresAt
Date
Optional
isActive
Boolean
Soft control
isRead
Boolean
Legacy/global
timestamps
Auto
createdAt, updatedAt


📦 NotificationRead Collection (notification_reads)
Purpose
Tracks per-user read status (true read receipts).
Schema
Field
Type
Description
notification_id
ObjectId → Notification
Reference
user_id
ObjectId
User
role
Enum
ADMIN / TEACHER / STUDENT
readAt
Date
Read timestamp

Constraints
Unique (notification_id + user_id) index


Prevents duplicate read entries



🔔 Notification APIs (Reality)
Create Notifications
Role
Method
Endpoint
COLLEGE_ADMIN
POST
/notifications/admin/create
TEACHER
POST
/notifications/teacher/create







Read Notifications
Role
Method
Endpoint
COLLEGE_ADMIN
GET
/notifications/admin/read
TEACHER
GET
/notifications/teacher/read
STUDENT
GET
/notifications/student/read


Notification Count APIs
Role
   Endpoint
COLLEGE_ADMIN
/notifications/count/admin
TEACHER
/notifications/count/teacher
STUDENT
/notifications/count/student


Other Actions
Action
Method
Endpoint
Bell unread
GET
/notifications/unread/bell
Mark as read
POST
/notifications/:notificationId/read
Edit
PUT
/notifications/edit-note/:id
Delete
DELETE
/notifications/delete-note/:id





  8 . Conclusion
This system successfully implements a secure, role-based, multi-tenant college management platform designed to handle student admissions, faculty management, attendance, fees, notifications, and reporting in a scalable and maintainable way.
The architecture follows clear separation of concerns, with authentication, authorization, business logic, and data access handled independently. Role-Based Access Control (RBAC) is strictly enforced through middleware layers, ensuring that every request is validated for identity, role eligibility, and college scope before accessing any resource.
Each role in the system has a well-defined responsibility:
SUPER_ADMIN operates at the platform level, focusing on analytics and governance across all colleges.


COLLEGE_ADMIN manages all academic and administrative operations within their own college.


TEACHER handles academic execution such as attendance, timetable management (HOD-specific), and student communication.


STUDENT interacts only with approved, self-related data such as profile, fees, payments, and notifications.


The database design is normalized, relationship-driven, and indexed for integrity, ensuring:
No duplicate or inconsistent records


Strong linkage between colleges, users, students, teachers, and academic entities


Safe multi-college isolation across all collections


All APIs are protected using JWT-based authentication, role validation, and college-level authorization, preventing unauthorized access and data leakage. Approval workflows, attendance sessions, fee installments, and notification tracking are handled in a structured and auditable manner.
Overall, this system is:
Secure – strict role enforcement and tenant isolation


Scalable – supports multiple colleges and future roles


Maintainable – modular code, clean schemas, reusable middleware


Production-ready – real-world workflows, reports, and validations


This documentation represents a complete, enterprise-grade backend foundation for a smart college management system and can be confidently extended with additional modules such as exams, results, parents, or mobile applications.




