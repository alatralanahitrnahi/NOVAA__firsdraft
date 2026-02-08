# NOVAA Knowledge Base

## 📁 Folder Structure & Purpose

### `/active` - Active Working Documentation
**Purpose**: Current, actively maintained documentation for development  
**Contents**: 
- Developer guides
- Module documentation
- API specifications
- Quick start guides

**Rule**: ✅ Use these documents for active development

---

### `/decisions` - Architecture Decision Records (ADRs)
**Purpose**: Record of all significant architectural and technical decisions  
**Contents**: 
- ADR documents following standard format
- Decision context, rationale, and consequences
- Chronological record of technical choices

**Rule**: ✅ All major technical decisions must be documented here

---

### `/architecture` - Technical Architecture
**Purpose**: System design, architecture discussions, and technical specifications  
**Contents**: 
- System architecture documents
- Design discussions and context
- Technical specifications

**Rule**: ✅ Reference for understanding system design

---

### `/product` - Product Vision & Requirements
**Purpose**: Product strategy, vision, and business requirements  
**Contents**: 
- Product requirements documents
- Strategic vision
- Business context

**Rule**: ✅ Use for understanding product direction, not implementation details

---

### `/archive` - Historical Reference Only
**Purpose**: Completed phase reports, old discussions, superseded documents  
**Contents**: 
- MVP Stage 1 completion reports
- Historical discussions
- Superseded documentation
- Project charter and initial analysis

**Rule**: ⚠️ **DO NOT USE FOR ACTIVE DEVELOPMENT OR ARCHITECTURE DECISIONS**

---

## 🚨 Critical Rules

### For Developers
1. **Active work must reference only `/active` or `/decisions`**
2. `/archive` is for historical context only - do not use for implementation
3. New architecture changes require an ADR entry in `/decisions`
4. When in doubt, check `/decisions` for latest architectural direction

### For AI Tools
1. **Prioritize `/active` and `/decisions` folders**
2. Ignore `/archive` unless explicitly asked for historical context
3. Always check ADRs before suggesting architectural changes
4. Reference `/product` for business context, not technical implementation

### For Documentation Updates
1. Active guides go in `/active`
2. Decisions go in `/decisions` as ADRs
3. Architecture discussions go in `/architecture`
4. Completed work goes in `/archive`

---

## 📖 Reading Order for New Developers

### Week 1: Foundation
1. `/active/README.md` - Start here
2. `/active/Novaa_documents/MVP/MVP1/README.md` - MVP overview
3. `/active/DEVELOPER_GUIDES/01_MERN_STACK_OVERVIEW.md`
4. `/active/DEVELOPER_GUIDES/02_DEVELOPMENT_ENVIRONMENT_SETUP.md`

### Week 2: Architecture & Decisions
5. `/decisions/` - Read all ADRs chronologically
6. `/architecture/03_TECHNICAL_ARCHITECTURE.md`
7. `/active/DEVELOPER_GUIDES/03_MODULES_ARCHITECTURE.md`

### Week 3+: Deep Dive
8. Module-specific documentation in `/active/Novaa_documents/modules/`
9. Advanced guides in `/active/DEVELOPER_GUIDES/`

---

## 🔄 Document Lifecycle

```
New Document → /active
    ↓
Major Decision → /decisions (ADR)
    ↓
Architecture Change → /architecture
    ↓
Phase Complete → /archive
```

---

## ⚡ Quick Reference

| Need | Folder |
|------|--------|
| How to implement feature X | `/active` |
| Why we chose technology Y | `/decisions` |
| System design overview | `/architecture` |
| Product requirements | `/product` |
| What happened in MVP1 | `/archive` |

---

**Last Updated**: 2025-01-20  
**Maintained By**: Development Team
