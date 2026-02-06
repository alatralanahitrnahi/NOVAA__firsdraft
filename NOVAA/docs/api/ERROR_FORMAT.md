# API Error Format

## Standard Error Response Format

### General Format
```json
{
  "status": "error",
  "message": "Human-readable error message",
  "errorCode": "MACHINE_READABLE_ERROR_CODE",
  "details": {
    "field": "specific field causing error",
    "value": "value that caused error",
    "reason": "specific reason for error"
  }
}
```

### Common Error Codes
- `MISSING_COLLEGE_CONTEXT` - Missing x-college-code header
- `INVALID_COLLEGE` - Invalid or inactive college code
- `UNAUTHORIZED_ACCESS` - User doesn't have permission
- `VALIDATION_ERROR` - Input validation failed
- `DUPLICATE_ENTRY` - Attempting to create duplicate record
- `RESOURCE_NOT_FOUND` - Requested resource doesn't exist

### Success Format
```json
{
  "status": "success",
  "data": {
    // Response data
  },
  "message": "Optional success message"
}
```