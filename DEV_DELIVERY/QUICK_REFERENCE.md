# QUICK REFERENCE GUIDE

**Purpose**: Fast lookup for common questions  
**Format**: Question → Answer → Document Reference

---

## 🚀 GETTING STARTED

### "Where do I start?"
**Answer**: Read `README.md` first, then follow your role path.

**Backend Developer Path**:
```
README.md (15 min)
  ↓
00_PROJECT_REALITY.md (15 min)
  ↓
01_MVP1_WHAT_EXISTS_DO_NOT_REBUILD.md (20 min)
  ↓
04_SYSTEM_RULES_DO_NOT_BREAK.md (15 min)
  ↓
03_DEVELOPER_TASK_BREAKDOWN.md → Backend Tasks
```

**Frontend Developer Path**:
```
README.md (15 min)
  ↓
00_PROJECT_REALITY.md (15 min)
  ↓
05_FRONTEND_PRIORITY_FOR_SALES.md (20 min)
  ↓
03_DEVELOPER_TASK_BREAKDOWN.md → Frontend Tasks
```

---

## 🎯 PRIORITIES

### "What should I work on first?"
**Answer**: Priority 1 tasks (blocking features)

**Priority 1 (MUST COMPLETE)**:
1. Stripe webhooks → Backend Developer
2. Reports dashboard → Frontend Developer
3. Production deployment → Full-Stack Developer

**See**: `02_MVP2_EXECUTION_SCOPE.md` → Priority Breakdown

---

### "Can I skip Priority 2 tasks?"
**Answer**: Only if running out of time. Document why.

**Priority 2 (IMPORTANT)**:
- Security hardening
- Error handling
- UX polish
- Monitoring

**See**: `02_MVP2_EXECUTION_SCOPE.md` → Priority 2

---

## 🔍 FINDING CODE

### "Where is the authentication code?"
**Answer**: Don't modify it. It's complete and protected.

**Location**: Backend `/auth` module  
**Status**: ✅ Complete, do not modify  
**See**: `01_MVP1_WHAT_EXISTS_DO_NOT_REBUILD.md` → Authentication System

---

### "Where is the payment code?"
**Answer**: Partially complete. Add webhooks only.

**Location**: Backend `/payments` module  
**Status**: ⚠️ Needs webhooks  
**Task**: Add webhook endpoint  
**See**: `03_DEVELOPER_TASK_BREAKDOWN.md` → Backend Task 1

---

### "Where are the report APIs?"
**Answer**: Backend complete. Build frontend only.

**Location**: Backend `/reports` module  
**Status**: ✅ Backend complete, ❌ Frontend missing  
**Task**: Build dashboard with charts  
**See**: `03_DEVELOPER_TASK_BREAKDOWN.md` → Frontend Task 1

---

## 🔐 SECURITY

### "How do I enforce multi-tenancy?"
**Answer**: Always include `collegeId` in queries.

**Wrong**:
```javascript
Student.find({ status: 'APPROVED' })
```

**Right**:
```javascript
Student.find({ 
  collegeId: req.user.collegeId,
  status: 'APPROVED' 
})
```

**See**: `04_SYSTEM_RULES_DO_NOT_BREAK.md` → Rule 1

---

### "How do I protect a route?"
**Answer**: Use authentication middleware.

**Pattern**:
```javascript
router.get('/endpoint',
  authenticate,
  authorizeRole(['COLLEGE_ADMIN']),
  controller
);
```

**See**: `04_SYSTEM_RULES_DO_NOT_BREAK.md` → Rule 2

---

### "Can I trust data from frontend?"
**Answer**: NO. Always validate on backend.

**Rule**: Never trust `collegeId` from request body.  
**Use**: `req.user.collegeId` from JWT token.

**See**: `04_SYSTEM_RULES_DO_NOT_BREAK.md` → Rule 3

---

## 📊 REPORTS

### "What reports do I need to build?"
**Answer**: 5 reports with charts.

**Reports**:
1. Admission Summary (pie chart)
2. Payment Summary (bar chart)
3. Attendance Summary (line chart)
4. Student Payment Table (filterable)
5. Low Attendance Table (filterable)

**See**: `05_FRONTEND_PRIORITY_FOR_SALES.md` → Priority 1

---

### "Which chart library should I use?"
**Answer**: Chart.js (recommended) or Recharts.

**Installation**:
```bash
npm install chart.js react-chartjs-2
```

**See**: `05_FRONTEND_PRIORITY_FOR_SALES.md` → Chart Libraries

---

### "Do I need to build export functionality?"
**Answer**: Only if ahead of schedule (Priority 3).

**Priority**: 🟢 Nice to have  
**Format**: PDF and Excel  
**See**: `02_MVP2_EXECUTION_SCOPE.md` → Priority 3

---

## 💳 PAYMENTS

### "How do I implement Stripe webhooks?"
**Answer**: Follow step-by-step guide.

**Steps**:
1. Create webhook endpoint
2. Verify signature
3. Handle events
4. Update installment status

**See**: `03_DEVELOPER_TASK_BREAKDOWN.md` → Backend Task 1

---

### "What events should I handle?"
**Answer**: 3 main events.

**Events**:
1. `checkout.session.completed` → Update installment to PAID
2. `payment_intent.succeeded` → Log success
3. `payment_intent.payment_failed` → Log failure

**See**: `03_DEVELOPER_TASK_BREAKDOWN.md` → Backend Task 1.2

---

### "Where do I get webhook secret?"
**Answer**: Stripe Dashboard → Developers → Webhooks.

**Steps**:
1. Add endpoint URL
2. Select events
3. Copy webhook secret
4. Add to `.env`

**See**: `03_DEVELOPER_TASK_BREAKDOWN.md` → Backend Task 1.5

---

## 🚀 DEPLOYMENT

### "How do I deploy to production?"
**Answer**: Follow deployment checklist.

**Steps**:
1. Setup environment variables
2. Create production database
3. Deploy backend to server
4. Deploy frontend
5. Configure NGINX
6. Setup SSL certificate

**See**: `03_DEVELOPER_TASK_BREAKDOWN.md` → Full-Stack Task 1

---

### "What environment variables do I need?"
**Answer**: 8 critical variables.

**Required**:
```bash
NODE_ENV=production
MONGODB_URI=...
JWT_SECRET=...
STRIPE_SECRET_KEY=sk_live_...
STRIPE_WEBHOOK_SECRET=whsec_...
FRONTEND_URL=https://...
```

**See**: `03_DEVELOPER_TASK_BREAKDOWN.md` → Full-Stack Task 1.1

---

### "How do I setup SSL?"
**Answer**: Use Certbot (Let's Encrypt).

**Command**:
```bash
sudo certbot --nginx -d yourdomain.com
```

**See**: `03_DEVELOPER_TASK_BREAKDOWN.md` → Full-Stack Task 1.5

---

## 🎨 FRONTEND

### "What loading states do I need?"
**Answer**: On all API calls and form submissions.

**Pattern**:
```jsx
{loading ? <LoadingSpinner /> : <Content />}
```

**See**: `05_FRONTEND_PRIORITY_FOR_SALES.md` → Priority 2.1

---

### "How do I show success/error messages?"
**Answer**: Use toast notifications.

**Library**: react-toastify  
**Usage**:
```javascript
toast.success('Success!');
toast.error('Error!');
```

**See**: `05_FRONTEND_PRIORITY_FOR_SALES.md` → Priority 2.2

---

### "What validations do I need?"
**Answer**: Required fields, email format, phone format.

**Validate**:
- Name: Required
- Email: Required + valid format
- Mobile: Required + 10 digits
- Password: Required + min 8 characters

**See**: `05_FRONTEND_PRIORITY_FOR_SALES.md` → Priority 2.3

---

### "What colors should I use?"
**Answer**: Follow design system.

**Colors**:
- Primary: `#3498db`
- Success: `#4CAF50`
- Warning: `#FFC107`
- Danger: `#F44336`

**See**: `05_FRONTEND_PRIORITY_FOR_SALES.md` → Design System

---

## 🧪 TESTING

### "How do I test multi-tenancy?"
**Answer**: Create 2 test colleges and verify isolation.

**Test**:
1. Create College A and College B
2. Add students to each
3. Login as College A admin
4. Verify only College A students visible

**See**: `04_SYSTEM_RULES_DO_NOT_BREAK.md` → Rule 1

---

### "How do I test role-based access?"
**Answer**: Test as each role.

**Test**:
1. Login as Admin → Can delete students
2. Login as Teacher → Cannot delete students (403)
3. Login as Student → Cannot delete students (403)

**See**: `04_SYSTEM_RULES_DO_NOT_BREAK.md` → Rule 7

---

### "What should I test before deploying?"
**Answer**: Use deployment checklist.

**Test**:
- All features work
- Multi-tenancy enforced
- Authentication working
- No console errors
- Database queries optimized

**See**: `04_SYSTEM_RULES_DO_NOT_BREAK.md` → Rule 11

---

## ❌ COMMON MISTAKES

### "Can I modify the student schema?"
**Answer**: NO. Get approval first.

**Rule**: Never modify existing schemas without approval.  
**Why**: Breaks existing data and code.

**See**: `04_SYSTEM_RULES_DO_NOT_BREAK.md` → Rule 4

---

### "Can I remove an old API endpoint?"
**Answer**: NO. Deprecate instead.

**Rule**: Never remove existing endpoints.  
**Why**: Frontend may be using them.

**See**: `04_SYSTEM_RULES_DO_NOT_BREAK.md` → Rule 5

---

### "Can I skip authentication on this route?"
**Answer**: NO. All protected routes need auth.

**Exception**: Only public routes (login, register, health check).

**See**: `04_SYSTEM_RULES_DO_NOT_BREAK.md` → Rule 2

---

### "Can I log user passwords for debugging?"
**Answer**: NO. Never log sensitive data.

**Never Log**:
- Passwords
- Payment tokens
- API keys
- Personal data

**See**: `04_SYSTEM_RULES_DO_NOT_BREAK.md` → Rule 6

---

## 🆘 WHEN STUCK

### "I'm stuck on a task for 30+ minutes"
**Answer**: Ask for help.

**Process**:
1. Check documentation
2. Google the error
3. Ask team in Slack/Teams
4. Pair program if needed

**See**: `README.md` → When You're Stuck

---

### "I found a bug in existing code"
**Answer**: Document and notify team.

**Process**:
1. Document the bug
2. Assess severity
3. If critical, fix immediately
4. If minor, add to backlog

**See**: `README.md` → Support

---

### "I want to add a new feature"
**Answer**: Check if it's in MVP2 scope.

**Process**:
1. Check `02_MVP2_EXECUTION_SCOPE.md`
2. If not in scope, skip it
3. If in scope, proceed
4. If unsure, ask team

**See**: `02_MVP2_EXECUTION_SCOPE.md` → Out of Scope

---

## 📅 TIMELINE

### "How long should each task take?"
**Answer**: See task breakdown.

**Backend Tasks**:
- Stripe webhooks: 2-3 days
- Security hardening: 1 day
- Error handling: 1 day
- Monitoring: 1 day

**Frontend Tasks**:
- Reports dashboard: 3-4 days
- UX polish: 2 days

**See**: `03_DEVELOPER_TASK_BREAKDOWN.md`

---

### "What if I'm behind schedule?"
**Answer**: Drop Priority 3 tasks.

**Priority**:
1. Complete Priority 1 (blocking)
2. Complete Priority 2 (important)
3. Skip Priority 3 (nice to have)

**See**: `02_MVP2_EXECUTION_SCOPE.md` → Priority Breakdown

---

## ✅ COMPLETION

### "How do I know MVP2 is complete?"
**Answer**: Check completion checklist.

**Must Have**:
- ✅ Stripe webhooks working
- ✅ Reports dashboard complete
- ✅ Production deployed
- ✅ HTTPS working
- ✅ Sales demo successful

**See**: `README.md` → Final Checklist

---

### "What happens after MVP2?"
**Answer**: Bug fixes, then MVP3 planning.

**Next Steps**:
1. Fix production bugs
2. Gather feedback
3. Plan MVP3 (QR attendance, etc.)
4. Optimize performance

**See**: `COMPLETION_SUMMARY.md` → Post-MVP2 Roadmap

---

## 📞 CONTACTS

### "Who do I ask about...?"

| Question Type | Contact |
|---------------|---------|
| Technical issues | Tech Lead |
| Task priorities | Project Manager |
| Business requirements | Product Owner |
| Deployment issues | DevOps Lead |
| Urgent bugs | Tech Lead (immediately) |

---

## 🔗 DOCUMENT MAP

```
README.md
├── Quick start guide
├── Role-based paths
└── Success criteria

00_PROJECT_REALITY.md
├── Current system status
├── MVP1 complete/incomplete
└── MVP2 goals

01_MVP1_WHAT_EXISTS_DO_NOT_REBUILD.md
├── Protected modules
├── Existing APIs
└── What not to modify

02_MVP2_EXECUTION_SCOPE.md
├── Priority 1 (blocking)
├── Priority 2 (important)
├── Priority 3 (nice to have)
└── 2-week sprint plan

03_DEVELOPER_TASK_BREAKDOWN.md
├── Backend tasks (step-by-step)
├── Frontend tasks (step-by-step)
└── Full-stack tasks (step-by-step)

04_SYSTEM_RULES_DO_NOT_BREAK.md
├── Multi-tenancy rules
├── Security rules
└── Common mistakes

05_FRONTEND_PRIORITY_FOR_SALES.md
├── Reports dashboard design
├── UX polish guidelines
└── Design system

COMPLETION_SUMMARY.md
└── Executive summary
```

---

**Remember**: This is a quick reference. For detailed information, see the full documents.
