# MVP1 Admissions Collection Schema

## MVP1 Scope Only
This schema is for MVP1 with test-mode limitations. Production features will be added in MVP2.

```javascript
{
  _id: ObjectId,
  collegeId: ObjectId (ref: colleges),
  studentId: ObjectId (ref: students),
  courseId: String,
  status: String (enum: ["DRAFT", "SUBMITTED", "PENDING_VERIFICATION", "VERIFIED", "REJECTED", "APPROVED"]),
  documents: [
    {
      url: String, // S3 path for test uploads
      category: String (enum: ["AADHAAR", "MARKSHEET", "CASTE_CERT"]),
      verificationStatus: String (enum: ["PENDING", "APPROVED", "REJECTED"]),
      rejectionReason: String,
      uploadedAt: Date
    }
  ],
  notes: String,
  createdAt: Date,
  updatedAt: Date
}
```

## Critical MVP1 Indexes
- `(collegeId, status)` - for admin dashboards
- `(collegeId, studentId)` - for student tracking

## MVP1 Limitations
- Document uploads use test S3 bucket
- No automated verification (manual only)
- No advanced document validation