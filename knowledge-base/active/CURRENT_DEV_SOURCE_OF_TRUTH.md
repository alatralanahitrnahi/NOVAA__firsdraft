# CURRENT DEV SOURCE OF TRUTH

**Last Updated**: 2025-01-20  
**Status**: Active Development Guide  
**Phase**: Post-MVP Stage 1 / Pre-MVP Stage 2

---

## 🎯 PRIMARY DEVELOPMENT ENTRY POINT

**ALL DEVELOPERS MUST START HERE:**

📍 **`/knowledge-base/active/README.md`**

This is your single source of truth for:
- What documentation to use
- Reading order for onboarding
- Module navigation
- Current development status

**DO NOT start development without reading this file first.**

---

## ⚠️ CRITICAL: What Developers MUST Use

### ✅ ACTIVE DEVELOPMENT SOURCES

**Use ONLY these folders for implementation:**

1. **`/knowledge-base/active/DEVELOPER_GUIDES/`**
   - All 22 developer guides
   - Code standards, testing, debugging
   - Security, payment processing
   - Git workflow, onboarding

2. **`/knowledge-base/active/Novaa_documents/`**
   - All MVP module documentation
   - MVP1, MVP2, MVP3 specifications
   - Module edge cases and implementation details

3. **`/knowledge-base/decisions/`**
   - Architecture Decision Records (ADRs)
   - Check before making architectural changes
   - All ADRs currently in "Draft" status

4. **`/NOVAA/docs/`** (Application folder)
   - API documentation
   - Compliance guides
   - Core architecture (ERD, Security, Tenancy)
   - Module-specific docs

---

## 🚫 DO NOT USE FOR DEVELOPMENT

### ❌ NON-DEVELOPMENT SOURCES

**These folders are NOT for implementation:**

1. **`/knowledge-base/archive/`** ⛔
   - Historical reference ONLY
   - Completed phase reports
   - Old discussions and superseded docs
   - MVP Stage 1 completion summaries
   - **DO NOT implement from these files**

2. **`/knowledge-base/product/`** ⛔
   - Product vision and strategy
   - Business requirements (not technical specs)
   - Strategic vision documents
   - **Use for context, NOT implementation**

3. **`/knowledge-base/architecture/`** ⚠️
   - `95_ARCHITECTURE_DISCUSSIONS_CONTEXT.json` is historical
   - Use `03_TECHNICAL_ARCHITECTURE.md` for reference only
   - Active architecture decisions are in `/decisions/` ADRs

### 🚫 DO NOT USE DIRECTLY

**NEVER use files with these prefixes for implementation:**
- ❌ `90_*` - Strategic/vision documents (non-technical)
- ❌ `95_*` - Historical discussions (outdated)
- ❌ `00_*` in `/archive/` - Superseded documentation

**If you see these prefixes, they are reference material only.**

---

## 📋 MVP EXECUTION ORDER

### ⚡ CRITICAL: Build Sequence

**MVP Phases MUST be executed in order:**

```
MVP1 (Weeks 1-4) → MVP2 (Weeks 5-6) → MVP3 (Weeks 7-10)
```

**DO NOT skip to MVP2 or MVP3 before MVP1 is complete.**

### 🔗 Module Dependency Chain

**Build modules in this exact order:**

```
1. auth (Authentication & Authorization)
   ↓
2. users (User Management)
   ↓
3. colleges (College Setup)
   ↓
4. admissions (Student Admissions)
   ↓
5. attendance (Attendance Tracking)
   ↓
6. payments (Fee Management)
   ↓
7. reports (Analytics & Reports)
   ↓
8. notifications (Communication)
```

**Why this order?**
- Each module depends on the previous ones
- Auth must exist before users can be created
- Colleges must exist before admissions
- Students must be admitted before attendance
- Fees depend on admissions data
- Reports aggregate all module data
- Notifications communicate across all modules

---

## 📋 MVP PHASE 1 ACTIVE SCOPE

### Current Implementation Status

**7 Core Modules (All in `/active/Novaa_documents/MVP/`):**

1. **Authentication** (`auth/`)
   - JWT-based authentication
   - Role-based access control (RBAC)
   - Multi-tenant user management

2. **College Management** (`colleges/`)
   - College profile management
   - Department and course setup
   - Academic year configuration

3. **Admissions** (`admissions/`)
   - Student application workflow
   - Document verification
   - Approval process

4. **Attendance** (`attendance/`)
   - Session-based attendance
   - Manual marking (MVP1)
   - QR-based planned for MVP3

5. **Payments** (`payments/`)
   - Stripe integration (test mode)
   - Fee structure management
   - Installment tracking

6. **Notifications** (`notifications/`)
   - Internal communication system
   - Role-based notifications
   - Read receipt tracking

7. **Reports** (`reports/`)
   - Backend APIs complete
   - Frontend visualization pending
   - Export functionality planned

---

## 🎯 DEVELOPMENT WORKFLOW

### Before Starting Any Task:

1. **Check Active Documentation**
   - Read relevant guide in `/active/DEVELOPER_GUIDES/`
   - Review module docs in `/active/Novaa_documents/`

2. **Check ADRs**
   - Review `/knowledge-base/decisions/` for architectural decisions
   - Verify no conflicting decisions exist

3. **Check Application Docs**
   - Review `/NOVAA/docs/` for API and compliance requirements

4. **DO NOT Reference**
   - Anything in `/archive/`
   - Strategic vision in `/product/` (unless for business context)
   - Historical discussions in `/architecture/95_*`

---

## 🔐 CRITICAL RULES

### Multi-Tenancy (MANDATORY)
- **Every database query MUST include `collegeId`**
- No exceptions for cross-college queries
- Validate college context in middleware
- Test tenant isolation for all features

### Security (MANDATORY)
- JWT authentication for all protected endpoints
- RBAC enforcement at middleware level
- Input validation and sanitization
- No PII in logs or error messages

### Code Standards (MANDATORY)
- Follow ESLint configuration
- Use Prettier for formatting
- Write tests for new features
- Document complex logic

---

## 📂 QUICK REFERENCE

| Need | Location |
|------|----------|
| How to implement feature | `/active/DEVELOPER_GUIDES/` |
| Module specifications | `/active/Novaa_documents/MVP/` |
| Architecture decisions | `/decisions/` (ADRs) |
| API documentation | `/NOVAA/docs/api/` |
| Security guidelines | `/active/DEVELOPER_GUIDES/09_*` |
| Testing guide | `/active/DEVELOPER_GUIDES/11_*` |
| Code standards | `/active/DEVELOPER_GUIDES/08_*` |

---

## ⚡ ONBOARDING PATH

### Week 1: Foundation
1. `/active/Novaa_documents/README.md`
2. `/active/DEVELOPER_GUIDES/01_MERN_STACK_OVERVIEW.md`
3. `/active/DEVELOPER_GUIDES/02_DEVELOPMENT_ENVIRONMENT_SETUP.md`
4. `/active/DEVELOPER_GUIDES/03_MODULES_ARCHITECTURE.md`

### Week 2: Implementation
5. `/active/Novaa_documents/MVP/MVP1/README.md`
6. Module-specific docs in `/active/Novaa_documents/MVP/modules/`
7. `/active/DEVELOPER_GUIDES/06_API_DEVELOPMENT_GUIDE.md`
8. `/active/DEVELOPER_GUIDES/07_FRONTEND_DEVELOPMENT_GUIDE.md`

### Week 3: Advanced
9. All ADRs in `/decisions/`
10. `/active/DEVELOPER_GUIDES/09_AUTHENTICATION_SECURITY_GUIDE.md`
11. `/active/DEVELOPER_GUIDES/10_PAYMENT_PROCESSING_GUIDE.md`

---

## 👶 FOR JUNIOR DEVELOPERS

### Critical Rules for New Team Members

**1. Never Assume Architecture Beyond Active Folder**
- ❌ Do NOT read `/archive/` and assume it's current
- ❌ Do NOT use `/product/` files for technical decisions
- ❌ Do NOT implement from files with `90_` or `95_` prefixes
- ✅ ONLY use `/active/` and approved `/decisions/` ADRs

**2. Never Follow ADRs Unless Status Changes from Draft**
- All ADRs currently show: **"STATUS: DRAFT – NOT APPROVED"**
- Draft ADRs are proposals, NOT approved architecture
- Wait for ADR status to change to "ACCEPTED" before implementing
- Check with tech lead if unsure about ADR status

**3. Always Start from Primary Entry Point**
- Read `/knowledge-base/active/README.md` first
- Follow the onboarding path exactly
- Don't skip ahead to advanced topics
- Ask questions if documentation is unclear

**4. Follow Module Dependency Order**
- Build auth before users
- Build colleges before admissions
- Don't start a module if its dependencies aren't complete

**5. Multi-Tenancy is Non-Negotiable**
- Every query MUST include `collegeId`
- Test with multiple colleges
- Never skip tenant validation

---

## 🚨 RED FLAGS

**Stop and ask if you find yourself:**
- Reading files from `/archive/` for implementation
- Using strategic vision docs for technical decisions
- Implementing features not in `/active/` documentation
- Skipping multi-tenancy validation
- Not checking ADRs before architectural changes
- Following a Draft ADR without approval
- Building modules out of dependency order

---

## 📞 WHEN IN DOUBT

1. Check `/active/` first
2. Review relevant ADR in `/decisions/`
3. Consult tech lead
4. Do NOT assume from `/archive/` or `/product/`

---

**Remember**: 
- `/active/` = Current implementation source
- `/decisions/` = Architectural direction
- `/archive/` = Historical context only
- `/product/` = Business context only

**This document is the authoritative guide for what to use during active development.**
