# DEVELOPMENT NAVIGATION MAP

**Purpose**: Visual guide for navigating NOVAA documentation  
**Audience**: Junior developers, interns, new team members  
**Last Updated**: 2025-01-20

---

## 🎯 WHERE TO START

```
YOU ARE HERE
    ↓
/knowledge-base/active/README.md ← START HERE FIRST
    ↓
Read this navigation map
    ↓
Follow the path below based on your role
```

---

## 🗺️ DOCUMENTATION STRUCTURE (VISUAL)

```
/knowledge-base/
│
├── active/ ✅ USE THIS FOR DEVELOPMENT
│   ├── README.md ⭐ PRIMARY ENTRY POINT
│   ├── CURRENT_DEV_SOURCE_OF_TRUTH.md ⭐ WHAT TO USE/AVOID
│   ├── DEVELOPMENT_NAVIGATION_MAP.md ⭐ YOU ARE HERE
│   │
│   ├── DEVELOPER_GUIDES/ (22 guides)
│   │   ├── 01_MERN_STACK_OVERVIEW.md
│   │   ├── 02_DEVELOPMENT_ENVIRONMENT_SETUP.md
│   │   ├── 03_MODULES_ARCHITECTURE.md
│   │   ├── 04_MODULE_INTERCONNECTIONS.md
│   │   ├── 05_DATABASE_DEVELOPER_GUIDE.md
│   │   ├── 06_API_DEVELOPMENT_GUIDE.md
│   │   ├── 07_FRONTEND_DEVELOPMENT_GUIDE.md
│   │   ├── 08_CODE_STANDARDS_CONVENTIONS.md
│   │   ├── 09_AUTHENTICATION_SECURITY_GUIDE.md
│   │   ├── 10_PAYMENT_PROCESSING_GUIDE.md
│   │   ├── 11_TESTING_DEVELOPER_GUIDE.md
│   │   ├── 12_DEBUGGING_TROUBLESHOOTING.md
│   │   └── ... (more guides)
│   │
│   └── Novaa_documents/
│       ├── README.md
│       └── MVP/
│           ├── modules/ (Current module docs)
│           │   ├── auth/
│           │   ├── admissions/
│           │   ├── colleges/
│           │   ├── attendance/
│           │   ├── payments/
│           │   ├── reports/
│           │   └── notifications/
│           ├── MVP1/ (Weeks 1-4 specs)
│           ├── MVP2/ (Weeks 5-6 specs)
│           └── MVP3/ (Weeks 7-10 specs)
│
├── decisions/ ⚠️ CHECK BEFORE ARCHITECTURE CHANGES
│   ├── ADR-001-repository-restructure.md (DRAFT)
│   ├── ADR-002-api-first-direction.md (DRAFT)
│   └── ADR-003-ai-processing-architecture.md (DRAFT)
│
├── architecture/ 📚 REFERENCE ONLY
│   ├── 03_TECHNICAL_ARCHITECTURE.md
│   └── 95_ARCHITECTURE_DISCUSSIONS_CONTEXT.json (historical)
│
├── product/ 💼 BUSINESS CONTEXT ONLY
│   ├── 02_PRODUCT_REQUIREMENTS_DOCUMENT.md
│   └── 90_STRATEGIC_VISION_DO_NOT_USE_FOR_DEV.md
│
└── archive/ 🗄️ HISTORICAL - DO NOT USE FOR DEV
    ├── 00_SOURCE_OF_TRUTH_MVP_STAGE1.md
    ├── 01_PROJECT_CHARTER.md
    └── ... (completion reports)
```

---

## 🚀 NAVIGATION PATHS BY ROLE

### Path 1: Backend Developer

```
Day 1-2: Foundation
├─ /active/README.md (15 min)
├─ /active/DEVELOPER_GUIDES/01_MERN_STACK_OVERVIEW.md (30 min)
├─ /active/DEVELOPER_GUIDES/02_DEVELOPMENT_ENVIRONMENT_SETUP.md (2 hours)
└─ Setup complete ✓

Day 3: Architecture
├─ /active/DEVELOPER_GUIDES/03_MODULES_ARCHITECTURE.md (1 hour)
├─ /active/DEVELOPER_GUIDES/04_MODULE_INTERCONNECTIONS.md (1 hour)
└─ Understand system ✓

Day 4-5: Backend Deep Dive
├─ /active/DEVELOPER_GUIDES/05_DATABASE_DEVELOPER_GUIDE.md (2 hours)
├─ /active/DEVELOPER_GUIDES/06_API_DEVELOPMENT_GUIDE.md (2 hours)
├─ /active/DEVELOPER_GUIDES/09_AUTHENTICATION_SECURITY_GUIDE.md (1 hour)
└─ Ready to code ✓

Ongoing: Module Work
├─ /active/Novaa_documents/MVP/modules/{your-module}/
├─ /active/Novaa_documents/MVP/MVP1/modules/{your-module}/
└─ Reference as needed
```

### Path 2: Frontend Developer

```
Day 1-2: Foundation
├─ /active/README.md (15 min)
├─ /active/DEVELOPER_GUIDES/01_MERN_STACK_OVERVIEW.md (30 min)
├─ /active/DEVELOPER_GUIDES/02_DEVELOPMENT_ENVIRONMENT_SETUP.md (2 hours)
└─ Setup complete ✓

Day 3: Architecture
├─ /active/DEVELOPER_GUIDES/03_MODULES_ARCHITECTURE.md (1 hour)
├─ /active/DEVELOPER_GUIDES/04_MODULE_INTERCONNECTIONS.md (1 hour)
└─ Understand system ✓

Day 4-5: Frontend Deep Dive
├─ /active/DEVELOPER_GUIDES/07_FRONTEND_DEVELOPMENT_GUIDE.md (2 hours)
├─ /active/DEVELOPER_GUIDES/06_API_DEVELOPMENT_GUIDE.md (1 hour - API integration)
└─ Ready to code ✓

Ongoing: Module Work
├─ /active/Novaa_documents/MVP/modules/{your-module}/
└─ Reference as needed
```

### Path 3: Full-Stack Developer

```
Day 1-2: Foundation
├─ /active/README.md (15 min)
├─ /active/DEVELOPER_GUIDES/01_MERN_STACK_OVERVIEW.md (30 min)
├─ /active/DEVELOPER_GUIDES/02_DEVELOPMENT_ENVIRONMENT_SETUP.md (2 hours)
└─ Setup complete ✓

Day 3-4: Architecture
├─ /active/DEVELOPER_GUIDES/03_MODULES_ARCHITECTURE.md (1 hour)
├─ /active/DEVELOPER_GUIDES/04_MODULE_INTERCONNECTIONS.md (1 hour)
├─ /active/DEVELOPER_GUIDES/05_DATABASE_DEVELOPER_GUIDE.md (2 hours)
└─ Understand system ✓

Day 5-7: Full Stack
├─ /active/DEVELOPER_GUIDES/06_API_DEVELOPMENT_GUIDE.md (2 hours)
├─ /active/DEVELOPER_GUIDES/07_FRONTEND_DEVELOPMENT_GUIDE.md (2 hours)
├─ /active/DEVELOPER_GUIDES/09_AUTHENTICATION_SECURITY_GUIDE.md (1 hour)
└─ Ready to code ✓

Ongoing: Module Work
├─ /active/Novaa_documents/MVP/modules/{your-module}/
└─ Reference as needed
```

---

## 📚 WHAT EACH DOCUMENT DEFINES

### Architecture
**Where**: `/active/DEVELOPER_GUIDES/03_MODULES_ARCHITECTURE.md`  
**Defines**:
- System structure
- Module responsibilities
- Component organization
- Technology stack decisions

**When to use**: Understanding how the system is organized

---

### API Specifications
**Where**: `/active/DEVELOPER_GUIDES/06_API_DEVELOPMENT_GUIDE.md`  
**Defines**:
- Endpoint patterns
- Request/response formats
- Error handling
- Authentication flow
- Status codes

**When to use**: Building or consuming APIs

---

### Module Logic
**Where**: `/active/Novaa_documents/MVP/modules/{module-name}/`  
**Defines**:
- Business rules
- Data models
- Workflows
- Edge cases
- Module-specific APIs

**When to use**: Implementing specific module features

---

### Database Design
**Where**: `/active/DEVELOPER_GUIDES/05_DATABASE_DEVELOPER_GUIDE.md`  
**Defines**:
- Collection schemas
- Indexes
- Relationships
- Query patterns
- Multi-tenancy enforcement

**When to use**: Writing database queries or designing schemas

---

### Security & Authentication
**Where**: `/active/DEVELOPER_GUIDES/09_AUTHENTICATION_SECURITY_GUIDE.md`  
**Defines**:
- JWT implementation
- RBAC patterns
- Password hashing
- Security middleware
- Common vulnerabilities

**When to use**: Implementing auth or securing endpoints

---

### Code Standards
**Where**: `/active/DEVELOPER_GUIDES/08_CODE_STANDARDS_CONVENTIONS.md`  
**Defines**:
- Naming conventions
- File structure
- Code formatting
- Best practices
- Review checklist

**When to use**: Writing code or reviewing PRs

---

## 🔄 HOW TO MOVE BETWEEN DOCUMENTS

### Scenario 1: "I need to build a new API endpoint"

```
Start: /active/DEVELOPER_GUIDES/06_API_DEVELOPMENT_GUIDE.md
  ↓ (understand endpoint pattern)
Check: /active/DEVELOPER_GUIDES/05_DATABASE_DEVELOPER_GUIDE.md
  ↓ (write query)
Check: /active/Novaa_documents/MVP/modules/{module}/
  ↓ (business logic)
Check: /active/DEVELOPER_GUIDES/09_AUTHENTICATION_SECURITY_GUIDE.md
  ↓ (add auth middleware)
Check: /active/DEVELOPER_GUIDES/08_CODE_STANDARDS_CONVENTIONS.md
  ↓ (format code)
Done: Submit PR
```

### Scenario 2: "I need to understand how modules interact"

```
Start: /active/DEVELOPER_GUIDES/04_MODULE_INTERCONNECTIONS.md
  ↓ (see interaction patterns)
Check: /active/DEVELOPER_GUIDES/03_MODULES_ARCHITECTURE.md
  ↓ (understand module structure)
Check: /active/Novaa_documents/MVP/modules/{module}/
  ↓ (specific module details)
Done: Understand flow
```

### Scenario 3: "I need to implement a frontend component"

```
Start: /active/DEVELOPER_GUIDES/07_FRONTEND_DEVELOPMENT_GUIDE.md
  ↓ (component patterns)
Check: /active/Novaa_documents/MVP/modules/{module}/
  ↓ (UI requirements)
Check: /active/DEVELOPER_GUIDES/06_API_DEVELOPMENT_GUIDE.md
  ↓ (API integration)
Check: /active/DEVELOPER_GUIDES/08_CODE_STANDARDS_CONVENTIONS.md
  ↓ (naming conventions)
Done: Build component
```

### Scenario 4: "I want to make an architectural change"

```
Start: /decisions/ (check existing ADRs)
  ↓ (verify no conflicts)
Check: /active/DEVELOPER_GUIDES/03_MODULES_ARCHITECTURE.md
  ↓ (understand current architecture)
Create: New ADR in /decisions/
  ↓ (document proposal)
Discuss: With tech lead
  ↓ (get approval)
Update: ADR status to "ACCEPTED"
  ↓ (only then implement)
Done: Make change
```

---

## 🎯 QUICK REFERENCE TABLE

| I need to... | Go to... |
|--------------|----------|
| Set up my environment | `/active/DEVELOPER_GUIDES/02_*` |
| Understand the system | `/active/DEVELOPER_GUIDES/03_*` + `04_*` |
| Write a database query | `/active/DEVELOPER_GUIDES/05_*` |
| Build an API | `/active/DEVELOPER_GUIDES/06_*` |
| Build a UI component | `/active/DEVELOPER_GUIDES/07_*` |
| Follow code standards | `/active/DEVELOPER_GUIDES/08_*` |
| Implement authentication | `/active/DEVELOPER_GUIDES/09_*` |
| Integrate payments | `/active/DEVELOPER_GUIDES/10_*` |
| Write tests | `/active/DEVELOPER_GUIDES/11_*` |
| Debug an issue | `/active/DEVELOPER_GUIDES/12_*` |
| Understand module logic | `/active/Novaa_documents/MVP/modules/{module}/` |
| Check MVP phase specs | `/active/Novaa_documents/MVP/MVP{1,2,3}/` |
| Propose architecture change | `/decisions/` (create ADR) |
| Understand business context | `/product/` (reference only) |

---

## ⚠️ COMMON MISTAKES TO AVOID

### ❌ Mistake 1: Reading from /archive
**Wrong**: "Let me check `/archive/00_SOURCE_OF_TRUTH_MVP_STAGE1.md`"  
**Right**: "Let me check `/active/Novaa_documents/MVP/modules/`"

### ❌ Mistake 2: Implementing Draft ADRs
**Wrong**: "ADR-002 says API-first, so I'll refactor everything"  
**Right**: "ADR-002 is DRAFT. I'll wait for approval before implementing"

### ❌ Mistake 3: Skipping Module Dependencies
**Wrong**: "I'll build payments first since it's my module"  
**Right**: "I'll wait for auth → users → colleges → admissions to be ready"

### ❌ Mistake 4: Using Product Docs for Technical Decisions
**Wrong**: "The strategic vision says we'll use microservices"  
**Right**: "The active architecture guide defines our current structure"

### ❌ Mistake 5: Not Checking Multi-Tenancy
**Wrong**: "This query works, ship it"  
**Right**: "Does this query include collegeId? Let me verify"

---

## 🆘 WHEN YOU'RE LOST

**Step 1**: Go back to `/active/README.md`  
**Step 2**: Check this navigation map  
**Step 3**: Ask yourself: "What am I trying to do?"  
**Step 4**: Use the "I need to..." table above  
**Step 5**: Still stuck? Ask tech lead

---

## 📞 HELP RESOURCES

| Question Type | Resource |
|---------------|----------|
| "Where do I start?" | `/active/README.md` |
| "What should I use?" | `/active/CURRENT_DEV_SOURCE_OF_TRUTH.md` |
| "How do I navigate?" | This file |
| "How do I build X?" | `/active/DEVELOPER_GUIDES/` |
| "What's the module logic?" | `/active/Novaa_documents/MVP/modules/` |
| "Can I make this change?" | `/decisions/` (check ADRs) |
| "Still confused?" | Ask tech lead |

---

## ✅ NAVIGATION CHECKLIST

Before starting development:
- [ ] Read `/active/README.md`
- [ ] Read `/active/CURRENT_DEV_SOURCE_OF_TRUTH.md`
- [ ] Read this navigation map
- [ ] Identify your role path (backend/frontend/full-stack)
- [ ] Follow the day-by-day guide for your role
- [ ] Bookmark key documents for your work
- [ ] Understand what NOT to use (`/archive`, `/product`, Draft ADRs)

---

**Remember**: When in doubt, start from `/active/README.md` and follow the paths in this map.

**This map is your compass. Use it every time you're unsure where to go next.**
