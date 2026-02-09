# DEVELOPER TASK BREAKDOWN

**Purpose**: Step-by-step tasks for each developer  
**Format**: Actionable, testable, junior-friendly

---

## 👨‍💻 BACKEND DEVELOPER TASKS

### Task 1: Stripe Webhook Implementation (Priority 1)

**Estimated Time**: 2-3 days

#### Step 1.1: Create Webhook Endpoint
```javascript
// File: routes/webhook.routes.js

const express = require('express');
const router = express.Router();
const { handleStripeWebhook } = require('../controllers/webhook.controller');

// IMPORTANT: Use raw body parser for Stripe signature verification
router.post('/stripe', 
  express.raw({ type: 'application/json' }), 
  handleStripeWebhook
);

module.exports = router;
```

**Test**: `POST /webhooks/stripe` returns 200

---

#### Step 1.2: Implement Webhook Handler
```javascript
// File: controllers/webhook.controller.js

const stripe = require('stripe')(process.env.STRIPE_SECRET_KEY);
const StudentFee = require('../models/StudentFee');

exports.handleStripeWebhook = async (req, res) => {
  const sig = req.headers['stripe-signature'];
  const webhookSecret = process.env.STRIPE_WEBHOOK_SECRET;
  
  let event;
  
  try {
    // Verify webhook signature
    event = stripe.webhooks.constructEvent(req.body, sig, webhookSecret);
  } catch (err) {
    console.error('Webhook signature verification failed:', err.message);
    return res.status(400).send(`Webhook Error: ${err.message}`);
  }
  
  // Handle the event
  switch (event.type) {
    case 'checkout.session.completed':
      await handleCheckoutCompleted(event.data.object);
      break;
    case 'payment_intent.succeeded':
      await handlePaymentSucceeded(event.data.object);
      break;
    case 'payment_intent.payment_failed':
      await handlePaymentFailed(event.data.object);
      break;
    default:
      console.log(`Unhandled event type: ${event.type}`);
  }
  
  res.json({ received: true });
};

async function handleCheckoutCompleted(session) {
  // Extract metadata
  const { studentId, installmentIndex } = session.metadata;
  
  // Update StudentFee
  const studentFee = await StudentFee.findOne({ studentId });
  
  if (!studentFee) {
    console.error('StudentFee not found for studentId:', studentId);
    return;
  }
  
  // Update installment status
  studentFee.installments[installmentIndex].status = 'PAID';
  studentFee.installments[installmentIndex].paidAt = new Date();
  studentFee.installments[installmentIndex].stripePaymentId = session.payment_intent;
  
  // Update paid amount
  studentFee.paidAmount += studentFee.installments[installmentIndex].amount;
  
  await studentFee.save();
  
  console.log('Payment confirmed for student:', studentId);
}

async function handlePaymentSucceeded(paymentIntent) {
  console.log('Payment succeeded:', paymentIntent.id);
  // Additional logging or notifications
}

async function handlePaymentFailed(paymentIntent) {
  console.error('Payment failed:', paymentIntent.id);
  // Log failed payment, notify admin
}
```

**Test**: 
1. Trigger test webhook from Stripe dashboard
2. Verify installment status updates to PAID
3. Verify paidAmount increases

---

#### Step 1.3: Update Stripe Checkout Session Creation
```javascript
// File: controllers/payment.controller.js

// Add metadata to checkout session
const session = await stripe.checkout.sessions.create({
  payment_method_types: ['card'],
  line_items: [{
    price_data: {
      currency: 'inr',
      product_data: {
        name: `${installment.name} - ${student.fullName}`,
      },
      unit_amount: installment.amount * 100, // Convert to paise
    },
    quantity: 1,
  }],
  mode: 'payment',
  success_url: `${process.env.FRONTEND_URL}/payment/success?session_id={CHECKOUT_SESSION_ID}`,
  cancel_url: `${process.env.FRONTEND_URL}/payment/cancel`,
  metadata: {
    studentId: student._id.toString(),
    installmentIndex: installmentIndex.toString(),
    collegeId: student.collegeId.toString()
  }
});
```

**Test**: 
1. Create payment session
2. Complete payment
3. Verify metadata passed to webhook

---

#### Step 1.4: Environment Variables
```bash
# Add to .env
STRIPE_WEBHOOK_SECRET=whsec_...
```

**Test**: Webhook signature verification passes

---

#### Step 1.5: Register Webhook in Stripe Dashboard
1. Go to Stripe Dashboard → Developers → Webhooks
2. Add endpoint: `https://your-domain.com/webhooks/stripe`
3. Select events: `checkout.session.completed`, `payment_intent.succeeded`, `payment_intent.payment_failed`
4. Copy webhook secret to `.env`

**Test**: Webhook receives events from Stripe

---

### Task 2: Security Hardening (Priority 2)

**Estimated Time**: 1 day

#### Step 2.1: Add Rate Limiting
```javascript
// File: middleware/rateLimiter.js

const rateLimit = require('express-rate-limit');

const apiLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // limit each IP to 100 requests per windowMs
  message: 'Too many requests from this IP, please try again later.'
});

const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 5, // limit each IP to 5 login attempts per windowMs
  message: 'Too many login attempts, please try again later.'
});

module.exports = { apiLimiter, authLimiter };
```

```javascript
// File: server.js

const { apiLimiter, authLimiter } = require('./middleware/rateLimiter');

app.use('/api/', apiLimiter);
app.use('/auth/login', authLimiter);
```

**Test**: Exceed rate limit and verify 429 error

---

#### Step 2.2: Configure CORS
```javascript
// File: server.js

const cors = require('cors');

const corsOptions = {
  origin: process.env.FRONTEND_URL || 'http://localhost:3000',
  credentials: true,
  optionsSuccessStatus: 200
};

app.use(cors(corsOptions));
```

**Test**: Frontend can make requests, other origins blocked

---

#### Step 2.3: Add Security Headers
```javascript
// File: server.js

const helmet = require('helmet');

app.use(helmet());
```

**Test**: Check response headers include security headers

---

#### Step 2.4: Input Sanitization
```javascript
// File: server.js

const mongoSanitize = require('express-mongo-sanitize');

app.use(mongoSanitize());
```

**Test**: Try injecting `$ne` in request, verify it's sanitized

---

### Task 3: Enhanced Error Handling (Priority 2)

**Estimated Time**: 1 day

#### Step 3.1: Create Error Classes
```javascript
// File: utils/AppError.js

class AppError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.statusCode = statusCode;
    this.isOperational = true;
    Error.captureStackTrace(this, this.constructor);
  }
}

module.exports = AppError;
```

**Test**: Throw AppError and verify it's caught

---

#### Step 3.2: Global Error Handler
```javascript
// File: middleware/errorHandler.js

const errorHandler = (err, req, res, next) => {
  let error = { ...err };
  error.message = err.message;
  
  // Log error
  console.error(err);
  
  // Mongoose bad ObjectId
  if (err.name === 'CastError') {
    const message = 'Resource not found';
    error = new AppError(message, 404);
  }
  
  // Mongoose duplicate key
  if (err.code === 11000) {
    const message = 'Duplicate field value entered';
    error = new AppError(message, 400);
  }
  
  // Mongoose validation error
  if (err.name === 'ValidationError') {
    const message = Object.values(err.errors).map(val => val.message);
    error = new AppError(message, 400);
  }
  
  res.status(error.statusCode || 500).json({
    success: false,
    message: error.message || 'Server Error',
    ...(process.env.NODE_ENV === 'development' && { stack: err.stack })
  });
};

module.exports = errorHandler;
```

```javascript
// File: server.js

const errorHandler = require('./middleware/errorHandler');

// Must be last middleware
app.use(errorHandler);
```

**Test**: Trigger various errors and verify proper responses

---

#### Step 3.3: Async Handler Wrapper
```javascript
// File: utils/asyncHandler.js

const asyncHandler = (fn) => (req, res, next) => {
  Promise.resolve(fn(req, res, next)).catch(next);
};

module.exports = asyncHandler;
```

```javascript
// Usage in controllers
const asyncHandler = require('../utils/asyncHandler');

exports.getStudents = asyncHandler(async (req, res, next) => {
  const students = await Student.find({ collegeId: req.user.collegeId });
  res.json({ success: true, data: students });
});
```

**Test**: Throw error in async function and verify it's caught

---

### Task 4: Monitoring and Logging (Priority 2)

**Estimated Time**: 1 day

#### Step 4.1: Request Logging
```javascript
// File: server.js

const morgan = require('morgan');

if (process.env.NODE_ENV === 'development') {
  app.use(morgan('dev'));
} else {
  app.use(morgan('combined'));
}
```

**Test**: Make requests and verify logs appear

---

#### Step 4.2: Error Logging
```javascript
// File: utils/logger.js

const winston = require('winston');

const logger = winston.createLogger({
  level: 'info',
  format: winston.format.json(),
  transports: [
    new winston.transports.File({ filename: 'logs/error.log', level: 'error' }),
    new winston.transports.File({ filename: 'logs/combined.log' })
  ]
});

if (process.env.NODE_ENV !== 'production') {
  logger.add(new winston.transports.Console({
    format: winston.format.simple()
  }));
}

module.exports = logger;
```

**Test**: Trigger error and verify it's logged to file

---

#### Step 4.3: Health Check Endpoint
```javascript
// File: routes/health.routes.js

const express = require('express');
const router = express.Router();

router.get('/', (req, res) => {
  res.status(200).json({
    status: 'OK',
    timestamp: new Date().toISOString(),
    uptime: process.uptime()
  });
});

module.exports = router;
```

**Test**: `GET /health` returns 200 with status

---

## 🎨 FRONTEND DEVELOPER TASKS

### Task 1: Reports Dashboard (Priority 1)

**Estimated Time**: 3-4 days

#### Step 1.1: Create Dashboard Layout
```jsx
// File: pages/Dashboard.jsx

import React, { useState, useEffect } from 'react';
import AdmissionReport from '../components/reports/AdmissionReport';
import PaymentReport from '../components/reports/PaymentReport';
import AttendanceReport from '../components/reports/AttendanceReport';
import StudentPaymentTable from '../components/reports/StudentPaymentTable';
import LowAttendanceTable from '../components/reports/LowAttendanceTable';

const Dashboard = () => {
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  return (
    <div className="dashboard-container">
      <h1>Reports Dashboard</h1>
      
      <div className="reports-grid">
        <AdmissionReport />
        <PaymentReport />
        <AttendanceReport />
      </div>
      
      <div className="tables-section">
        <StudentPaymentTable />
        <LowAttendanceTable />
      </div>
    </div>
  );
};

export default Dashboard;
```

**Test**: Dashboard page renders without errors

---

#### Step 1.2: Admission Report Component
```jsx
// File: components/reports/AdmissionReport.jsx

import React, { useState, useEffect } from 'react';
import { Pie } from 'react-chartjs-2';
import axios from 'axios';

const AdmissionReport = () => {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    fetchData();
  }, []);
  
  const fetchData = async () => {
    try {
      setLoading(true);
      const response = await axios.get('/api/reports/admission-summary');
      setData(response.data.data);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  };
  
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  if (!data) return null;
  
  const chartData = {
    labels: ['Approved', 'Pending', 'Rejected'],
    datasets: [{
      data: [data.approved, data.pending, data.rejected],
      backgroundColor: ['#4CAF50', '#FFC107', '#F44336']
    }]
  };
  
  return (
    <div className="report-card">
      <h3>Admission Summary</h3>
      <div className="stats">
        <div className="stat">
          <span className="label">Total Applications</span>
          <span className="value">{data.total}</span>
        </div>
        <div className="stat">
          <span className="label">Approved</span>
          <span className="value">{data.approved}</span>
        </div>
        <div className="stat">
          <span className="label">Pending</span>
          <span className="value">{data.pending}</span>
        </div>
        <div className="stat">
          <span className="label">Rejected</span>
          <span className="value">{data.rejected}</span>
        </div>
      </div>
      <div className="chart">
        <Pie data={chartData} />
      </div>
    </div>
  );
};

export default AdmissionReport;
```

**Test**: 
1. Component renders
2. API call succeeds
3. Chart displays correctly
4. Loading state shows
5. Error state shows if API fails

---

#### Step 1.3: Payment Report Component
```jsx
// File: components/reports/PaymentReport.jsx

import React, { useState, useEffect } from 'react';
import { Bar } from 'react-chartjs-2';
import axios from 'axios';

const PaymentReport = () => {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    fetchData();
  }, []);
  
  const fetchData = async () => {
    try {
      setLoading(true);
      const response = await axios.get('/api/reports/payment-summary');
      setData(response.data.data);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  };
  
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  if (!data) return null;
  
  const chartData = {
    labels: ['Collected', 'Pending'],
    datasets: [{
      label: 'Amount (₹)',
      data: [data.totalCollected, data.totalPending],
      backgroundColor: ['#4CAF50', '#FFC107']
    }]
  };
  
  const collectionRate = ((data.totalCollected / data.totalExpected) * 100).toFixed(2);
  
  return (
    <div className="report-card">
      <h3>Payment Summary</h3>
      <div className="stats">
        <div className="stat">
          <span className="label">Total Expected</span>
          <span className="value">₹{data.totalExpected.toLocaleString()}</span>
        </div>
        <div className="stat">
          <span className="label">Collected</span>
          <span className="value">₹{data.totalCollected.toLocaleString()}</span>
        </div>
        <div className="stat">
          <span className="label">Pending</span>
          <span className="value">₹{data.totalPending.toLocaleString()}</span>
        </div>
        <div className="stat">
          <span className="label">Collection Rate</span>
          <span className="value">{collectionRate}%</span>
        </div>
      </div>
      <div className="chart">
        <Bar data={chartData} />
      </div>
    </div>
  );
};

export default PaymentReport;
```

**Test**: Similar to AdmissionReport

---

#### Step 1.4: Student Payment Table
```jsx
// File: components/reports/StudentPaymentTable.jsx

import React, { useState, useEffect } from 'react';
import axios from 'axios';

const StudentPaymentTable = () => {
  const [data, setData] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  const [filter, setFilter] = useState('ALL');
  
  useEffect(() => {
    fetchData();
  }, []);
  
  const fetchData = async () => {
    try {
      setLoading(true);
      const response = await axios.get('/api/reports/student-payment-status');
      setData(response.data.data);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  };
  
  const filteredData = filter === 'ALL' 
    ? data 
    : data.filter(student => student.paymentStatus === filter);
  
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  
  return (
    <div className="table-card">
      <h3>Student Payment Status</h3>
      
      <div className="filters">
        <button onClick={() => setFilter('ALL')}>All</button>
        <button onClick={() => setFilter('PAID')}>Paid</button>
        <button onClick={() => setFilter('PARTIAL')}>Partial</button>
        <button onClick={() => setFilter('PENDING')}>Pending</button>
      </div>
      
      <table>
        <thead>
          <tr>
            <th>Student Name</th>
            <th>Course</th>
            <th>Total Fee</th>
            <th>Paid</th>
            <th>Pending</th>
            <th>Status</th>
          </tr>
        </thead>
        <tbody>
          {filteredData.map(student => (
            <tr key={student._id}>
              <td>{student.fullName}</td>
              <td>{student.course}</td>
              <td>₹{student.totalFee.toLocaleString()}</td>
              <td>₹{student.paidAmount.toLocaleString()}</td>
              <td>₹{(student.totalFee - student.paidAmount).toLocaleString()}</td>
              <td>
                <span className={`status ${student.paymentStatus.toLowerCase()}`}>
                  {student.paymentStatus}
                </span>
              </td>
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
};

export default StudentPaymentTable;
```

**Test**: 
1. Table renders with data
2. Filters work correctly
3. Data displays properly

---

### Task 2: UX Polish (Priority 2)

**Estimated Time**: 2 days

#### Step 2.1: Add Loading States
```jsx
// File: components/common/LoadingSpinner.jsx

const LoadingSpinner = () => (
  <div className="spinner-container">
    <div className="spinner"></div>
    <p>Loading...</p>
  </div>
);

export default LoadingSpinner;
```

```css
/* File: styles/spinner.css */

.spinner {
  border: 4px solid #f3f3f3;
  border-top: 4px solid #3498db;
  border-radius: 50%;
  width: 40px;
  height: 40px;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}
```

**Test**: Loading spinner displays during API calls

---

#### Step 2.2: Add Toast Notifications
```jsx
// File: utils/toast.js

import { toast } from 'react-toastify';

export const showSuccess = (message) => {
  toast.success(message);
};

export const showError = (message) => {
  toast.error(message);
};

export const showInfo = (message) => {
  toast.info(message);
};
```

```jsx
// Usage in components
import { showSuccess, showError } from '../utils/toast';

const handleSubmit = async () => {
  try {
    await axios.post('/api/students', data);
    showSuccess('Student created successfully!');
  } catch (err) {
    showError('Failed to create student');
  }
};
```

**Test**: Toast notifications appear on success/error

---

#### Step 2.3: Form Validations
```jsx
// File: components/forms/StudentForm.jsx

const [errors, setErrors] = useState({});

const validate = () => {
  const newErrors = {};
  
  if (!formData.fullName) {
    newErrors.fullName = 'Name is required';
  }
  
  if (!formData.email) {
    newErrors.email = 'Email is required';
  } else if (!/\S+@\S+\.\S+/.test(formData.email)) {
    newErrors.email = 'Email is invalid';
  }
  
  if (!formData.mobileNumber) {
    newErrors.mobileNumber = 'Mobile number is required';
  } else if (!/^\d{10}$/.test(formData.mobileNumber)) {
    newErrors.mobileNumber = 'Mobile number must be 10 digits';
  }
  
  setErrors(newErrors);
  return Object.keys(newErrors).length === 0;
};

const handleSubmit = async (e) => {
  e.preventDefault();
  
  if (!validate()) {
    return;
  }
  
  // Submit form
};
```

**Test**: Form shows validation errors before submission

---

## 🚀 FULL-STACK DEVELOPER TASKS

### Task 1: Production Deployment (Priority 1)

**Estimated Time**: 2-3 days

#### Step 1.1: Environment Configuration
```bash
# File: .env.production

NODE_ENV=production
PORT=5000
MONGODB_URI=mongodb+srv://username:password@cluster.mongodb.net/novaa
JWT_SECRET=your-super-secret-jwt-key-change-this
JWT_EXPIRE=7d
STRIPE_SECRET_KEY=sk_live_...
STRIPE_WEBHOOK_SECRET=whsec_...
FRONTEND_URL=https://novaa.example.com
CORS_ORIGIN=https://novaa.example.com
```

**Test**: Environment variables load correctly

---

#### Step 1.2: Database Setup
1. Create MongoDB Atlas account
2. Create new cluster
3. Create database user
4. Whitelist server IP
5. Get connection string
6. Update `.env.production`

**Test**: Backend connects to production database

---

#### Step 1.3: Server Deployment (Example: DigitalOcean)
```bash
# SSH into server
ssh root@your-server-ip

# Install Node.js
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs

# Install PM2
sudo npm install -g pm2

# Clone repository
git clone https://github.com/your-repo/novaa.git
cd novaa

# Install dependencies
npm install

# Create .env file
nano .env

# Start application
pm2 start server.js --name novaa-backend

# Setup PM2 to start on boot
pm2 startup
pm2 save
```

**Test**: Backend accessible via server IP

---

#### Step 1.4: NGINX Configuration
```nginx
# File: /etc/nginx/sites-available/novaa

server {
    listen 80;
    server_name api.novaa.example.com;
    
    location / {
        proxy_pass http://localhost:5000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

```bash
# Enable site
sudo ln -s /etc/nginx/sites-available/novaa /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

**Test**: Backend accessible via domain

---

#### Step 1.5: SSL Certificate
```bash
# Install Certbot
sudo apt-get install certbot python3-certbot-nginx

# Get certificate
sudo certbot --nginx -d api.novaa.example.com

# Auto-renewal
sudo certbot renew --dry-run
```

**Test**: HTTPS working

---

## ✅ TASK COMPLETION CHECKLIST

### Backend Developer
- [ ] Stripe webhooks implemented
- [ ] Webhook signature verification working
- [ ] Payment confirmation automated
- [ ] Rate limiting configured
- [ ] CORS configured
- [ ] Security headers added
- [ ] Error handling improved
- [ ] Logging setup complete
- [ ] Health check endpoint added

### Frontend Developer
- [ ] Reports dashboard created
- [ ] All 5 reports displaying
- [ ] Charts rendering correctly
- [ ] Loading states added
- [ ] Form validations working
- [ ] Toast notifications added
- [ ] Responsive design complete
- [ ] Error handling improved

### Full-Stack Developer
- [ ] Production environment configured
- [ ] Database setup complete
- [ ] Backend deployed
- [ ] Frontend deployed
- [ ] HTTPS working
- [ ] Domain configured
- [ ] Stripe production mode active
- [ ] End-to-end testing complete

---

**Remember**: Test each step before moving to the next. Don't accumulate untested code.
