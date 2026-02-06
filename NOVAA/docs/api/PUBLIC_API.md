# Public API Documentation

## Authentication
All API endpoints require:
- JWT token in `Authorization: Bearer <token>` header
- College context in `x-college-code` header

## Base URL
`https://api.novaa.in` (Production)
`http://localhost:5000` (Development)

## Core Endpoints

### Authentication
- `POST /api/auth/login` - User login
- `POST /api/auth/logout` - User logout
- `GET /api/auth/profile` - Get user profile

### Admissions
- `POST /api/admissions/apply` - Submit application
- `GET /api/admissions/:id` - Get application details
- `PUT /api/admissions/:id/verify` - Admin verify documents
- `GET /api/admissions` - List applications (with filters)

### Payments (MVP1: Test Mode)
- `POST /api/payments/orders` - Create test payment order
- `POST /api/payments/verify` - Verify test payment
- `GET /api/payments/transactions/:id` - Get transaction details

### Attendance (MVP1: Manual)
- `POST /api/attendance/mark` - Mark attendance manually
- `GET /api/attendance/student/:studentId` - Get student attendance
- `GET /api/attendance/class/:courseId` - Get class attendance

### Reports
- `GET /api/reports/admission-funnel` - Admission analytics
- `GET /api/reports/fee-collection` - Fee collection dashboard
- `GET /api/reports/attendance-trends` - Attendance analytics