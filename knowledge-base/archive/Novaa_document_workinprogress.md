
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

Module 5: ATTENDANCE (Attendance Tracking) ✅ Completed (Manual – MVP Scope)
Purpose: Track student attendance
Implemented Features:
Attendance eligibility after admission approval


Manual attendance marking by Teacher


Daily attendance records stored with:


studentId


collegeId


date


status (Present / Absent)


MVP Limitation:
No QR-based or automated attendance


Status: Implemented as manual attendance for MVP





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


Payment failure recovery is manual



2️ Reporting & Analytics Limitations
Reports are read-only (no exports to Excel/PDF)


No historical trend analysis (month-over-month, year-over-year)


Dashboards are backend-ready but frontend visualizations are limited


No cross-college consolidated reports for Super Admin yet



3️ Attendance Limitations
Attendance is manual only


No QR-based or biometric attendance


No automated absence alerts to students or parents


No timetable-based auto attendance locking





4️ Notifications Limitations
Notifications are in-app only


No email, SMS, or WhatsApp delivery


No scheduled or recurring notifications


No read/unread acknowledgment tracking



5️  Automation & Workflow Limitations
Admission approvals require manual admin action


No automated escalation workflows


No background jobs (cron-based processing)


No real-time push updates (WebSockets)


All limitations are well-understood and addressable in the next development phase.




8. Conclusion
Stage-1 (MVP) of NOVAA – Smart College Management System has been successfully implemented and stabilized with all core operational modules in place. The system now supports end-to-end college operations including authentication, admissions, fee management, attendance tracking, reporting, and internal notifications, all within a secure multi-tenant architecture.
The current MVP delivers:
A strong backend foundation with clearly separated modules


Secure, role-based access control


Online payment capability via Stripe (test mode)


Actionable administrative reports


Structured internal communication through notifications


The documented limitations at this stage are intentional design choices, made to prioritize system stability, faster delivery, and validation of core workflows. These limitations do not block usability but instead define a clear roadmap for future enhancements.



