# MVP1 Authentication API Patterns

## MVP1 Scope Only
JWT authentication with 2-hour expiry. Production webhook handling comes in MVP2.

## Core Authentication Flow
```javascript
// Middleware: Enforce college context (MVP1 requirement)
app.use(requireCollegeContext); // Attaches req.college from headers

// MVP1 Login Endpoint
app.post("/api/auth/login", loginLimiter, async (req, res) => {
  const { email, password, collegeCode } = req.body;
  
  // MVP1: Verify college context
  const college = await College.findOne({ code: collegeCode, isActive: true });
  if (!college) {
    return res.status(403).json({ error: "INVALID_COLLEGE" });
  }

  // MVP1: Authenticate user within college context
  const user = await User.findOne({ 
    email: email.toLowerCase(), 
    collegeId: college._id 
  });

  if (!user || !await bcrypt.compare(password, user.password)) {
    return res.status(401).json({ error: "INVALID_CREDENTIALS" });
  }

  // MVP1: Generate JWT token (2-hour expiry for security)
  const token = jwt.sign(
    { userId: user._id, collegeId: user.collegeId },
    process.env.JWT_SECRET,
    { expiresIn: "2h" }
  );

  res.json({
    token,
    user: { id: user._id, email: user.email, role: user.role },
    college: { id: college._id, code: college.code, name: college.name }
  });
});
```

## MVP1 Error Handling
```javascript
// MVP1 Standardized error responses
{
  "status": "error",
  "message": "User not found in college context",
  "errorCode": "USER_NOT_FOUND",
  "details": {
    "email": "user@college.edu",
    "collegeCode": "ST_XAVIER_MUMBAI"
  }
}
```

## MVP1 Security Requirements
- All endpoints must use `requireCollegeContext` middleware
- JWT tokens expire in 2 hours (MVP2: configurable)
- Rate limiting: 5 attempts per minute (MVP2: configurable)