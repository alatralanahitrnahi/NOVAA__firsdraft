# Data Retention Policy

## MVP1 Data Retention Rules

### Data Categories & Retention Periods

| Data Type | Retention Period | Reason |
|-----------|------------------|--------|
| Student Records | 7 years | Academic regulation compliance |
| Fee Receipts | 10 years | Tax/GST compliance |
| Attendance Records | 5 years | Accreditation requirements |
| Audit Logs | 1 year | Security investigation |
| Temporary Files | 30 days | Cleanup old uploads |
| Test Data | 6 months | Development cleanup |

### Data Deletion Process
1. **Automatic**: System automatically deletes data past retention period
2. **Manual Request**: Users can request early deletion (subject to compliance)
3. **Bulk Deletion**: Scheduled cleanup jobs run monthly

### Compliance Requirements
- **DPDPA 2026**: Right to erasure for users
- **GST Compliance**: Must retain fee receipts for 10 years
- **Academic Records**: Must maintain student records per university guidelines

### MVP1 Limitations
- **MVP1**: Manual deletion process only
- **MVP2**: Automated retention and deletion
- **MVP3**: Advanced compliance tools