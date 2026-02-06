# MVP1 Development Environment Setup

## MVP1 Scope Only
Setup for test-mode development. Production deployment comes in MVP2.

## Prerequisites for MVP1
- Node.js 18+ (required)
- MongoDB 5.0+ (local or Atlas)
- Git
- VS Code (recommended)
- Test Razorpay account (not production)

## MVP1 Backend Setup
```bash
# Navigate to backend directory
cd backend

# Install dependencies
npm install

# Create environment file for MVP1 test mode
cp .env.example .env

# Edit .env for MVP1 with TEST keys:
# MONGODB_URI=mongodb://localhost:27017/novaa_mvp1_test
# RAZORPAY_KEY_ID=rzp_test_xxxxxxxxxxxxxxxx  # MVP1: Test key only
# RAZORPAY_KEY_SECRET=xxxxxxxxxxxxxxxxxxxxx  # MVP1: Test secret only
# RAZORPAY_WEBHOOK_SECRET=test_webhook_secret  # MVP1: Test webhook
# JWT_SECRET=mvp1_test_secret_123
# NODE_ENV=development
# PORT=5000

# Start MVP1 test server
npm run dev # Starts on http://localhost:5000
```

## MVP1 Frontend Setup
```bash
# Navigate to frontend directory
cd ../frontend

# Install dependencies
npm install

# Create environment file for MVP1
cp .env.example .env

# Edit .env for MVP1:
# REACT_APP_API_URL=http://localhost:5000
# REACT_APP_RAZORPAY_KEY_ID=rzp_test_xxxxxxxxxxxxxxxx  # MVP1: Test key

# Start MVP1 frontend
npm start # Starts on http://localhost:3000
```

## MVP1 Testing Setup
```bash
# Run MVP1 tests
npm test  # Runs unit and integration tests

# MVP1 Test Coverage Target: 80%+
npm run test:coverage
```

## MVP1 Critical Configuration
- **MVP1**: Use TEST Razorpay keys only (no production keys)
- **MVP1**: Local MongoDB for development (production DB in MVP2)
- **MVP1**: JWT 2-hour expiry (configurable in MVP2)
- **MVP1**: Rate limiting: 5 attempts/minute (configurable in MVP2)

## MVP1 Verification Steps
1. Backend server starts without errors
2. Frontend connects to backend API
3. Authentication works with test credentials
4. Test payment flow works (no real money)
5. All modules connect with collegeId context