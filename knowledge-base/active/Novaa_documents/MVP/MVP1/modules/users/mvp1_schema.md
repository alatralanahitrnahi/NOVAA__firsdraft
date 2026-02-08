# MVP1 Users Collection Schema

## MVP1 Scope Only
This schema is for MVP1 with test-mode limitations. Production features will be added in MVP2.

```javascript
{
  _id: ObjectId,
  collegeId: ObjectId (ref: colleges), // MANDATORY for multi-tenancy
  email: String (lowercase),
  password: String (bcrypt hashed with salt=10),
  role: String (enum: ["ADMIN", "STAFF", "STUDENT"]),
  isActive: Boolean (default: true),
  lastLogin: Date,
  createdAt: Date
}
```

## Critical MVP1 Indexes
- `(collegeId, email)` unique - prevents cross-college access
- **MVP1 Note**: All queries must include collegeId filter for data isolation

## MVP1 Security Requirements
- Passwords MUST be bcrypt hashed (salt=10)
- All queries MUST include collegeId filter
- **MVP1 Limitation**: Only test-mode authentication (no production tokens yet)