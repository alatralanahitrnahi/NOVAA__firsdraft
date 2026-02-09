# MVP2 EXECUTION SCOPE

**Timeline**: 2 Weeks  
**Goal**: Sales-Ready System  
**Team**: 3 Junior Developers (Backend, Frontend, Full-Stack)

---

## 🎯 MVP2 OBJECTIVES

1. **Production Deployment** - System accessible externally
2. **Reliable Payments** - Webhook-based payment confirmation
3. **Visual Dashboards** - Reports with charts and graphs
4. **Security Hardening** - Production-grade security
5. **Sales Demo Ready** - Polished UX for demonstrations

---

## 📋 PRIORITY BREAKDOWN

### 🔴 PRIORITY 1 (BLOCKING - Must Complete)

These features BLOCK sales demos. Complete first.

#### 1.1 Stripe Webhook Implementation
**Why Critical**: Payment confirmation currently unreliable (depends on frontend redirect)

**What to Build**:
- Webhook endpoint: `POST /webhooks/stripe`
- Signature verification using Stripe webhook secret
- Handle `checkout.session.completed` event
- Auto-update installment status to PAID
- Handle `payment_intent.succeeded` event
- Handle `payment_intent.payment_failed` event
- Error logging for failed webhooks

**Technical Requirements**:
```javascript
// Webhook endpoint must use raw body parser
app.post('/webhooks/stripe', 
  express.raw({type: 'application/json'}), 
  handleStripeWebhook
);

// Verify signature
const sig = req.headers['stripe-signature'];
const event = stripe.webhooks.constructEvent(req.body, sig, webhookSecret);

// Handle events
switch (event.type) {
  case 'checkout.session.completed':
    // Update StudentFee installment status
    break;
  case 'payment_intent.succeeded':
    // Log successful payment
    break;
  case 'payment_intent.payment_failed':
    // Log failed payment, notify admin
    break;
}
```

**Success Criteria**:
- ✅ Webhook receives events from Stripe
- ✅ Signature verification passes
- ✅ Installment status updates automatically
- ✅ Payment confirmation works without frontend redirect
- ✅ Failed payments logged properly

**Estimated Time**: 2-3 days  
**Assigned To**: Backend Developer

---

#### 1.2 Reports Frontend Dashboard
**Why Critical**: Admins need visual insights for decision-making

**What to Build**:
- Dashboard page with 5 report cards
- Charts using Chart.js or Recharts
- Responsive layout
- Loading states
- Error handling

**Reports to Visualize**:

1. **Admission Summary**
   - Total applications (number)
   - Approved students (number + percentage)
   - Pending applications (number + percentage)
   - Rejected applications (number + percentage)
   - Pie chart showing distribution

2. **Payment Summary**
   - Total expected fee (₹)
   - Total collected fee (₹)
   - Total pending fee (₹)
   - Collection rate (percentage)
   - Bar chart showing collected vs pending

3. **Student Payment Status**
   - Table with columns: Student Name, Course, Total Fee, Paid, Pending, Status
   - Filter by course
   - Filter by payment status
   - Search by student name

4. **Attendance Summary**
   - Overall attendance percentage
   - Total sessions conducted
   - Average attendance per session
   - Line chart showing attendance trend

5. **Low Attendance Students**
   - Table with columns: Student Name, Course, Attendance %, Status
   - Highlight students below 75%
   - Filter by course
   - Sort by attendance percentage

**Technical Requirements**:
```javascript
// Use existing backend APIs
GET /reports/admission-summary
GET /reports/payment-summary
GET /reports/student-payment-status
GET /reports/attendance-summary
GET /reports/low-attendance-students

// Frontend components
<DashboardLayout>
  <ReportCard title="Admissions" data={admissionData} />
  <ReportCard title="Payments" data={paymentData} />
  <ReportCard title="Attendance" data={attendanceData} />
  <StudentPaymentTable data={studentPayments} />
  <LowAttendanceTable data={lowAttendance} />
</DashboardLayout>
```

**Success Criteria**:
- ✅ All 5 reports display correctly
- ✅ Charts render properly
- ✅ Data updates on page load
- ✅ Loading states show during API calls
- ✅ Error messages display if API fails
- ✅ Responsive on desktop and tablet

**Estimated Time**: 3-4 days  
**Assigned To**: Frontend Developer

---

#### 1.3 Production Deployment Preparation
**Why Critical**: System must be accessible for sales demos

**What to Build**:
- Environment configuration
- Production database setup
- Server deployment scripts
- SSL certificate setup
- Domain configuration

**Tasks**:
1. Create `.env.production` file
2. Setup MongoDB Atlas (or production DB)
3. Configure Stripe production keys
4. Setup deployment server (AWS/DigitalOcean/Heroku)
5. Configure NGINX (if applicable)
6. Setup SSL certificate (Let's Encrypt)
7. Configure domain DNS
8. Deploy backend
9. Deploy frontend
10. Test production deployment

**Environment Variables Needed**:
```bash
NODE_ENV=production
PORT=5000
MONGODB_URI=mongodb+srv://...
JWT_SECRET=...
JWT_EXPIRE=7d
STRIPE_SECRET_KEY=sk_live_...
STRIPE_WEBHOOK_SECRET=whsec_...
FRONTEND_URL=https://novaa.example.com
CORS_ORIGIN=https://novaa.example.com
```

**Success Criteria**:
- ✅ Backend accessible via HTTPS
- ✅ Frontend accessible via HTTPS
- ✅ Database connected
- ✅ Stripe production mode working
- ✅ CORS configured correctly
- ✅ SSL certificate valid

**Estimated Time**: 2-3 days  
**Assigned To**: Full-Stack Developer / DevOps

---

### 🟡 PRIORITY 2 (IMPORTANT - Complete if Time Permits)

These features improve system quality but don't block demos.

#### 2.1 Security Hardening

**What to Add**:
1. **Rate Limiting**
   ```javascript
   const rateLimit = require('express-rate-limit');
   
   const limiter = rateLimit({
     windowMs: 15 * 60 * 1000, // 15 minutes
     max: 100 // limit each IP to 100 requests per windowMs
   });
   
   app.use('/api/', limiter);
   ```

2. **CORS Configuration**
   ```javascript
   const cors = require('cors');
   
   app.use(cors({
     origin: process.env.FRONTEND_URL,
     credentials: true
   }));
   ```

3. **Security Headers (Helmet.js)**
   ```javascript
   const helmet = require('helmet');
   app.use(helmet());
   ```

4. **Input Sanitization**
   ```javascript
   const mongoSanitize = require('express-mongo-sanitize');
   app.use(mongoSanitize());
   ```

**Success Criteria**:
- ✅ Rate limiting active on all routes
- ✅ CORS configured for production domain
- ✅ Security headers added
- ✅ Input sanitization working

**Estimated Time**: 1 day  
**Assigned To**: Backend Developer

---

#### 2.2 Enhanced Error Handling

**What to Add**:
1. **Global Error Handler**
   ```javascript
   app.use((err, req, res, next) => {
     console.error(err.stack);
     res.status(err.statusCode || 500).json({
       success: false,
       message: err.message || 'Server Error',
       ...(process.env.NODE_ENV === 'development' && { stack: err.stack })
     });
   });
   ```

2. **Custom Error Classes**
   ```javascript
   class AppError extends Error {
     constructor(message, statusCode) {
       super(message);
       this.statusCode = statusCode;
       this.isOperational = true;
     }
   }
   ```

3. **Async Error Wrapper**
   ```javascript
   const asyncHandler = (fn) => (req, res, next) => {
     Promise.resolve(fn(req, res, next)).catch(next);
   };
   ```

**Success Criteria**:
- ✅ All errors caught and handled
- ✅ User-friendly error messages
- ✅ No server crashes on errors
- ✅ Errors logged properly

**Estimated Time**: 1 day  
**Assigned To**: Backend Developer

---

#### 2.3 Frontend UX Polish

**What to Improve**:
1. **Loading States**
   - Add spinners during API calls
   - Skeleton loaders for tables
   - Disable buttons during submission

2. **Form Validations**
   - Client-side validation before API calls
   - Show validation errors inline
   - Prevent duplicate submissions

3. **Success/Error Messages**
   - Toast notifications for actions
   - Success messages after form submission
   - Error messages if API fails

4. **Responsive Design**
   - Mobile-friendly layouts
   - Tablet optimization
   - Desktop optimization

**Success Criteria**:
- ✅ All forms have loading states
- ✅ All forms have validations
- ✅ Toast notifications working
- ✅ Responsive on all screen sizes

**Estimated Time**: 2 days  
**Assigned To**: Frontend Developer

---

#### 2.4 Monitoring and Logging

**What to Add**:
1. **Request Logging (Morgan)**
   ```javascript
   const morgan = require('morgan');
   app.use(morgan('combined'));
   ```

2. **Error Logging (Winston)**
   ```javascript
   const winston = require('winston');
   
   const logger = winston.createLogger({
     level: 'info',
     format: winston.format.json(),
     transports: [
       new winston.transports.File({ filename: 'error.log', level: 'error' }),
       new winston.transports.File({ filename: 'combined.log' })
     ]
   });
   ```

3. **Health Check Endpoint**
   ```javascript
   app.get('/health', (req, res) => {
     res.status(200).json({ 
       status: 'OK', 
       timestamp: new Date().toISOString() 
     });
   });
   ```

**Success Criteria**:
- ✅ All requests logged
- ✅ Errors logged to file
- ✅ Health check endpoint working

**Estimated Time**: 1 day  
**Assigned To**: Backend Developer

---

### 🟢 PRIORITY 3 (NICE TO HAVE - Only if Ahead of Schedule)

These features enhance user experience but are not critical for MVP2.

#### 3.1 Export Reports (PDF/Excel)

**What to Build**:
- Export button on each report
- Generate PDF using `pdfkit` or `puppeteer`
- Generate Excel using `exceljs`
- Download file to user's device

**Success Criteria**:
- ✅ PDF export working
- ✅ Excel export working
- ✅ Downloaded files formatted correctly

**Estimated Time**: 2 days  
**Assigned To**: Full-Stack Developer

---

#### 3.2 Email Notifications

**What to Build**:
- Email service integration (SendGrid/Nodemailer)
- Payment confirmation emails
- Admission approval emails
- Low attendance alerts

**Success Criteria**:
- ✅ Emails sent after payment
- ✅ Emails sent after admission approval
- ✅ Email templates formatted properly

**Estimated Time**: 2 days  
**Assigned To**: Backend Developer

---

#### 3.3 Advanced Report Filters

**What to Build**:
- Date range picker
- Course filter dropdown
- Department filter dropdown
- Status filter dropdown
- Search functionality

**Success Criteria**:
- ✅ Filters update report data
- ✅ Multiple filters work together
- ✅ Clear filters button working

**Estimated Time**: 1-2 days  
**Assigned To**: Frontend Developer

---

## 📅 2-WEEK SPRINT PLAN

### Week 1: Core Features

**Days 1-2**:
- Backend: Start Stripe webhooks
- Frontend: Start reports dashboard
- Full-Stack: Setup production environment

**Days 3-4**:
- Backend: Complete Stripe webhooks, start security hardening
- Frontend: Complete reports dashboard
- Full-Stack: Deploy to staging server

**Day 5**:
- Backend: Complete security hardening
- Frontend: Start UX polish
- Full-Stack: Test staging deployment

---

### Week 2: Polish & Deploy

**Days 6-7**:
- Backend: Enhanced error handling, monitoring setup
- Frontend: Complete UX polish
- Full-Stack: Production deployment

**Days 8-9**:
- All: Testing and bug fixes
- All: Documentation updates

**Day 10**:
- All: Final testing
- All: Sales demo preparation
- All: MVP2 completion review

---

## ✅ MVP2 COMPLETION CHECKLIST

### Must Have (Blocking)
- [ ] Stripe webhooks implemented and tested
- [ ] Reports dashboard with all 5 reports
- [ ] Charts and graphs displaying correctly
- [ ] Production deployment complete
- [ ] HTTPS working
- [ ] Stripe production mode active

### Should Have (Important)
- [ ] Rate limiting configured
- [ ] CORS configured
- [ ] Security headers added
- [ ] Error handling improved
- [ ] Loading states on all forms
- [ ] Form validations working
- [ ] Toast notifications added
- [ ] Monitoring and logging setup

### Nice to Have (Optional)
- [ ] Export reports to PDF/Excel
- [ ] Email notifications
- [ ] Advanced report filters

---

## 🚫 OUT OF SCOPE FOR MVP2

These features are NOT part of MVP2:

❌ QR-based attendance (MVP3)  
❌ Biometric attendance (Future)  
❌ SMS notifications (MVP3)  
❌ WhatsApp notifications (Future)  
❌ Mobile app (V2.0)  
❌ AI features (Future)  
❌ Automated workflows (MVP3)  
❌ Real-time updates (Future)  
❌ Parent portal (Future)  
❌ Exam management (Future)  
❌ Result management (Future)  

---

## 📞 DAILY STANDUP FORMAT

**What did you complete yesterday?**  
**What will you work on today?**  
**Any blockers?**

Keep standups under 15 minutes. Focus on progress and blockers.

---

## 🆘 ESCALATION PROCESS

**Blocker Identified** → Inform team immediately  
**Can't resolve in 2 hours** → Escalate to tech lead  
**Critical bug found** → Stop and fix immediately  
**Scope creep detected** → Discuss with team before proceeding  

---

**Remember**: MVP2 is about COMPLETING critical gaps, not adding new features. Stay focused on the priorities.
