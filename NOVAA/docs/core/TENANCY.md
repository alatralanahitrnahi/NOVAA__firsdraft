# Multi-Tenancy Architecture

## MVP1 Data Isolation Strategy

### Approach: Shared Database, Tenant-Specific Schema

```
MongoDB Collections Structure:
┌──────────────────────────────────────┐
│ colleges                              │
├──────────────────────────────────────┤
│ { _id, code, name, state, isActive } │
└──────────────────────────────────────┘
         │
         ├─ users (has collegeId)
         ├─ students (has collegeId)
         ├─ admissions (has collegeId)
         ├─ feeStructures (has collegeId)
         ├─ transactions (has collegeId)
         ├─ attendance (has collegeId)
         └─ auditLogs (has collegeId)

CRITICAL: Every collection EXCEPT colleges has collegeId field
```

### Critical Safeguards

#### Middleware Enforcement
```javascript
const enforceMultiTenancy = async (req, res, next) => {
  const collegeCode = req.headers["x-college-code"];

  if (!collegeCode) {
    return res.status(400).json({
      error: "MISSING_COLLEGE_CONTEXT"
    });
  }

  const college = await College.findOne({
    code: collegeCode,
    isActive: true
  });

  if (!college) {
    return res.status(403).json({
      error: "INVALID_COLLEGE"
    });
  }

  req.college = college;
  next();
};
```

#### Query Pattern
```javascript
// ✅ CORRECT - Always include collegeId
const user = await User.findOne({
  collegeId: req.college._id,
  email: userEmail
});

// ❌ FORBIDDEN - Missing collegeId (security vulnerability)
const user = await User.findOne({
  email: userEmail
});
```

### MVP1 Limitations
- **MVP1**: Single database for all colleges
- **MVP2**: Database sharding by collegeCode
- **MVP3**: Per-college service replicas