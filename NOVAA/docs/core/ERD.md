# Entity Relationship Diagram

## MVP1 Database Schema

### Core Entities and Relationships

```
┌─────────────────────────┐      ┌─────────────────────────┐
│        COLLEGES         │      │          USERS          │
│─────────────────────────│      │─────────────────────────│
│ _id              (PK)   │◄─────┤ collegeId        (FK)   │
│ code             (UK)   │      │ _id              (PK)   │
│ name                  │      │ email            (UK)   │
│ state                 │      │ password              │
│ principalEmail        │      │ role                  │
│ principalPhone        │      │ isActive              │
│ gstNumber             │      │ lastLogin             │
│ isActive              │      │ createdAt             │
│ createdAt             │      │                         │
│ updatedAt             │      └─────────────────────────┘
└─────────────────────────┘                   │
                      │                       │
                      │                       │
                      │                       ▼
                      │        ┌─────────────────────────┐
                      │        │        STUDENTS         │
                      │        │─────────────────────────│
                      │        │ _id              (PK)   │
                      │        │ collegeId        (FK)   │
                      │        │ userId           (FK)   │
                      │        │ name                  │
                      │        │ email                 │
                      │        │ phone                 │
                      │        │ aadhaarNumber         │
                      │        │ casteCategory         │
                      │        │ class12Score          │
                      │        │ admissionYear         │
                      │        │ createdAt             │
                      │        └─────────────────────────┘
                      │                       │
                      │                       │
                      │                       ▼
                      │        ┌─────────────────────────┐
                      │        │       ADMISSIONS        │
                      │        │─────────────────────────│
                      │        │ _id              (PK)   │
                      │        │ collegeId        (FK)   │
                      │        │ studentId        (FK)   │
                      │        │ courseId              │
                      │        │ status                │
                      │        │ documents             │
                      │        │ notes                 │
                      │        │ createdAt             │
                      │        │ updatedAt             │
                      │        └─────────────────────────┘
                      │                       │
                      │                       │
                      │                       ▼
                      │        ┌─────────────────────────┐
                      │        │     FEE_STRUCTURES      │
                      │        │─────────────────────────│
                      │        │ _id              (PK)   │
                      │        │ collegeId        (FK)   │
                      │        │ courseId              │
                      │        │ tuitionFee            │
                      │        │ labFee                │
                      │        │ sportsFee             │
                      │        │ effectiveDate         │
                      │        │ isActive              │
                      │        │ createdAt             │
                      │        └─────────────────────────┘
                      │                       │
                      │                       │
                      │                       ▼
                      │        ┌─────────────────────────┐
                      │        │     TRANSACTIONS        │
                      │        │─────────────────────────│
                      │        │ _id              (PK)   │
                      │        │ collegeId        (FK)   │
                      │        │ studentId        (FK)   │
                      │        │ admissionId      (FK)   │
                      │        │ amount                │
                      │        │ razorpayOrderId       │
                      │        │ razorpayPaymentId     │
                      │        │ status                │
                      │        │ idempotencyKey        │
                      │        │ receiptUrl            │
                      │        │ createdAt             │
                      │        │ updatedAt             │
                      │        └─────────────────────────┘
                      │                       │
                      │                       │
                      │                       ▼
                      │        ┌─────────────────────────┐
                      │        │      ATTENDANCE         │
                      │        │─────────────────────────│
                      │        │ _id              (PK)   │
                      │        │ collegeId        (FK)   │
                      │        │ studentId        (FK)   │
                      │        │ courseId              │
                      │        │ date                  │
                      │        │ status                │
                      │        │ markedBy         (FK)   │
                      │        │ createdAt             │
                      │        └─────────────────────────┘
                      │                       │
                      │                       │
                      │                       ▼
                      │        ┌─────────────────────────┐
                      │        │      AUDIT_LOGS         │
                      │        │─────────────────────────│
                      │        │ _id              (PK)   │
                      │        │ collegeId        (FK)   │
                      │        │ userId           (FK)   │
                      │        │ action                │
                      │        │ resourceType            │
                      │        │ resourceId            │
                      │        │ oldValues             │
                      │        │ newValues             │
                      │        │ ipAddress             │
                      │        │ userAgent             │
                      │        │ createdAt             │
                      │        └─────────────────────────┘
                      │
                      ▼
        ┌─────────────────────────┐
        │      DOCUMENTS          │
        │   (Stored in S3)        │
        │─────────────────────────│
        │ admissionId      (FK)   │
        │ fileName              │
        │ fileType              │
        │ s3Path                │
        │ uploadedAt            │
        │ uploadedBy       (FK)   │
        └─────────────────────────┘
```

### Key Relationships
- **COLLEGES** (1) → (Many) **USERS**: Each college has many users
- **COLLEGES** (1) → (Many) **STUDENTS**: Each college has many students
- **USERS** (1) → (Many) **STUDENTS**: Each user can be linked to one student
- **COLLEGES** (1) → (Many) **ADMISSIONS**: Each college has many admissions
- **STUDENTS** (1) → (Many) **ADMISSIONS**: Each student can have multiple admissions
- **COLLEGES** (1) → (Many) **FEE_STRUCTURES**: Each college has many fee structures
- **COLLEGES** (1) → (Many) **TRANSACTIONS**: Each college has many transactions
- **STUDENTS** (1) → (Many) **TRANSACTIONS**: Each student can have multiple transactions
- **ADMISSIONS** (1) → (Many) **TRANSACTIONS**: Each admission can have multiple transactions
- **COLLEGES** (1) → (Many) **ATTENDANCE**: Each college has many attendance records
- **STUDENTS** (1) → (Many) **ATTENDANCE**: Each student has many attendance records
- **USERS** (1) → (Many) **ATTENDANCE**: Each staff user can mark many attendances
- **COLLEGES** (1) → (Many) **AUDIT_LOGS**: Each college has many audit logs
- **USERS** (1) → (Many) **AUDIT_LOGS**: Each user can generate many audit logs

### Critical Indexes
- **COLLEGES**: `code` (unique)
- **USERS**: `(collegeId, email)` (unique)
- **STUDENTS**: `(collegeId, userId)` (unique)
- **ADMISSIONS**: `(collegeId, status)`, `(collegeId, studentId)`
- **TRANSACTIONS**: `(collegeId, razorpayOrderId)`, `idempotencyKey` (unique)
- **ATTENDANCE**: `(collegeId, studentId, date)` (unique)
- **AUDIT_LOGS**: `(collegeId, userId, createdAt)`

### MVP1 Constraints
- Every entity except COLLEGES must have a `collegeId` field
- All queries must filter by `collegeId` for multi-tenancy
- Foreign key relationships enforced at application level
- No circular dependencies between entities