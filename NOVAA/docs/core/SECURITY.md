# Security Implementation

## MVP1 Security Safeguards

### Authentication & Authorization

#### JWT Token Implementation
```javascript
// Token generation
const token = jwt.sign(
  { userId: user._id, collegeId: user.collegeId },
  process.env.JWT_SECRET,
  { expiresIn: "2h" } // 2-hour expiry for security
);
```

#### Password Security
```javascript
// Password hashing with bcrypt (salt rounds = 10)
const hashedPassword = await bcrypt.hash(password, 10);

// Password validation
const validatePassword = (password) => {
  if (password.length < 8) {
    throw new Error("Password must be 8+ characters");
  }
  if (!/[A-Z]/.test(password)) {
    throw new Error("Password must include uppercase letter");
  }
  if (!/[0-9]/.test(password)) {
    throw new Error("Password must include number");
  }
};
```

### Rate Limiting
```javascript
const loginLimiter = rateLimit({
  windowMs: 60000, // 1 minute
  max: 5, // 5 attempts per minute
  message: "Too many login attempts. Try again in 1 minute.",
  keyGenerator: (req) => req.body.email
});
```

### Data Isolation
```javascript
// Critical: Every query must include collegeId
const originalFind = mongoose.Query.prototype.find;
mongoose.Query.prototype.find = function(query = {}) {
  if (!query.collegeId && !query._id) {
    throw new Error(
      `SECURITY_VIOLATION: Query missing collegeId filter. ` +
      `Model: ${this.model.modelName}. ` +
      `Query: ${JSON.stringify(query)}`
    );
  }
  return originalFind.call(this, { ...query, collegeId: req.college._id });
};
```

### Payment Security
- **Idempotency Keys**: Prevent duplicate charges
- **Webhook Verification**: Verify Razorpay signatures
- **Test Mode Only**: No real money in MVP1

### MVP1 Security Checklist
- [ ] All database queries include collegeId filter
- [ ] Passwords hashed with bcrypt (salt=10)
- [ ] JWT tokens expire in 2 hours
- [ ] Rate limiting on auth endpoints
- [ ] Razorpay webhook signatures verified
- [ ] No sensitive data in logs
- [ ] File upload validation (no executable files)