# Admissions Module Implementation

## MVP1 Admissions Workflow

### Core Features
- Student application form submission
- Document upload (Aadhaar, Marksheet, Caste Certificate)
- Admin verification workflow
- Application status tracking
- Reservation category handling (Maharashtra only)

### Database Schema
```javascript
{
  _id: ObjectId,
  collegeId: ObjectId (ref: colleges),
  studentId: ObjectId (ref: students),
  courseId: String,
  status: String (enum: ["DRAFT", "SUBMITTED", "PENDING_VERIFICATION", "VERIFIED", "REJECTED", "APPROVED"]),
  documents: [
    {
      url: String, // S3 path
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

### Critical Indexes
- `(collegeId, status)` - for admin dashboards
- `(collegeId, studentId)` - for student tracking

### Application Flow
```
STUDENT APPLICATION FLOW:
1. Student fills digital application form
2. Uploads required documents to S3
3. Submits application → status: SUBMITTED
4. Admin reviews documents → status: PENDING_VERIFICATION
5. Admin approves/rejects → status: VERIFIED/REJECTED
6. Student pays fees → status: APPROVED
```

### Admin Verification Workflow
```javascript
const verifyDocument = async (req, res) => {
  const { applicationId, documentId, action, rejectionReason } = req.body;
  const collegeId = req.college._id;

  // Verify application belongs to college
  const application = await Admission.findOne({
    _id: applicationId,
    collegeId
  });

  if (!application) {
    return res.status(404).json({ error: 'Application not found' });
  }

  // Update document verification status
  const update = {
    $set: {
      [`documents.$[elem].verificationStatus`]: action === 'approve' ? 'APPROVED' : 'REJECTED',
      [`documents.$[elem].rejectionReason`]: rejectionReason || null,
      updatedAt: new Date()
    }
  };

  const options = {
    arrayFilters: [{ 'elem._id': documentId }]
  };

  await Admission.updateOne({ _id: applicationId }, update, options);

  res.json({ message: 'Document verified' });
};
```

### MVP1 Limitations
- **MVP1**: Manual document verification only
- **MVP2**: Automated verification tools
- **MVP1**: Maharashtra reservation rules only
- **MVP2**: Multi-state rules support