# NOVAA Repository Rules

## 🎯 Purpose
This document establishes critical rules for working with the NOVAA repository to ensure consistency, prevent confusion, and maintain code quality during scaling.

---

## 📋 Documentation Rules

### Rule 1: Active Work References
**Active development must reference only `/knowledge-base/active` or `/knowledge-base/decisions`**

❌ **DO NOT**:
- Use documents from `/knowledge-base/archive` for implementation
- Reference outdated architecture discussions
- Follow superseded patterns from historical documents

✅ **DO**:
- Check `/knowledge-base/active` for current implementation guides
- Review `/knowledge-base/decisions` ADRs for architectural direction
- Update documentation when patterns change

---

### Rule 2: Archive is Historical Reference Only
**Files inside `/knowledge-base/archive` must NOT be used for active development or architecture decisions**

The archive contains:
- Completed phase reports (MVP1 completion, etc.)
- Historical discussions and context
- Superseded documentation
- Old strategic vision documents

**Use archive only when**:
- Understanding historical context
- Reviewing past decisions
- Onboarding context for "why we changed X"

---

### Rule 3: Architecture Decision Records (ADRs)
**New architecture changes require an ADR entry in `/knowledge-base/decisions`**

Before making significant technical decisions:
1. Create new ADR document: `ADR-XXX-decision-name.md`
2. Follow ADR template (Title, Status, Context, Decision, Consequences, Date)
3. Discuss with team
4. Update status to "Accepted" or "Rejected"
5. Implement only after ADR acceptance

**Significant decisions include**:
- Technology stack changes
- Database schema major changes
- API design patterns
- Security architecture
- Third-party service integrations

---

### Rule 4: AI Tools Priority
**AI tools must prioritize `/knowledge-base/active` and `/knowledge-base/decisions` folders**

When using AI assistants (GitHub Copilot, Amazon Q, ChatGPT):
- Configure to prioritize active documentation
- Explicitly exclude `/knowledge-base/archive` from context
- Always verify suggestions against current ADRs
- Report if AI suggests outdated patterns

---

## 💻 Code Rules

### Rule 5: Branch Strategy
- `main` - Active development branch (protected)
- `archive/pre-step2-baseline` - Historical baseline (read-only)
- Feature branches: `feature/module-name`
- Hotfix branches: `hotfix/issue-description`

### Rule 6: Code Standards
- Follow ESLint configuration
- Use Prettier for formatting
- Write tests for new features
- Document complex logic
- Review `/knowledge-base/active/DEVELOPER_GUIDES/08_CODE_STANDARDS_CONVENTIONS.md`

### Rule 7: Multi-Tenancy Enforcement
**Every database query MUST include `collegeId`**
- No exceptions for cross-college queries
- Validate college context in middleware
- Test tenant isolation for all features

---

## 🔐 Security Rules

### Rule 8: Authentication & Authorization
- JWT tokens for all authenticated endpoints
- Role-based access control (RBAC) enforced
- No sensitive data in logs
- Follow `/knowledge-base/active/DEVELOPER_GUIDES/09_AUTHENTICATION_SECURITY_GUIDE.md`

### Rule 9: Data Protection
- Comply with DPDPA 2026
- Hash passwords with bcrypt
- Sanitize all user inputs
- No PII in error messages

---

## 🚀 Deployment Rules

### Rule 10: Environment Separation
- Development: Local/Codespace
- Staging: Pre-production testing
- Production: Live system

**Never**:
- Test with production data locally
- Deploy directly to production
- Share production credentials

---

## 📝 Commit Rules

### Rule 11: Commit Messages
Format: `type(scope): description`

Types:
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation
- `refactor`: Code refactoring
- `test`: Test additions
- `chore`: Maintenance

Example: `feat(admissions): add document verification API`

### Rule 12: Pull Request Requirements
- Descriptive title and description
- Link to related issue/ADR
- Tests passing
- Code review approval
- No merge conflicts

---

## 🧪 Testing Rules

### Rule 13: Test Coverage
- Unit tests for business logic
- Integration tests for APIs
- E2E tests for critical flows
- Minimum 70% coverage for new code

### Rule 14: Test Data
- Use factories/fixtures
- No hardcoded production data
- Clean up test data after runs
- Isolate test database

---

## 📞 Communication Rules

### Rule 15: Documentation Updates
When changing code:
1. Update relevant documentation in `/knowledge-base/active`
2. Create ADR if architectural change
3. Update API documentation
4. Notify team of breaking changes

### Rule 16: Issue Tracking
- Create issue before starting work
- Link commits to issues
- Update issue status regularly
- Close with PR reference

---

## ⚠️ Breaking These Rules

**Consequences**:
- PR rejection
- Code review required
- Documentation update required
- Team discussion for repeated violations

**Exceptions**:
- Must be discussed with tech lead
- Documented in ADR
- Approved by team

---

## 🔄 Rule Updates

These rules are living documents:
- Propose changes via ADR
- Discuss in team meetings
- Update with team consensus
- Version control all changes

---

**Version**: 1.0  
**Last Updated**: 2025-01-20  
**Next Review**: 2025-02-20  
**Maintained By**: Development Team

---

## Quick Checklist

Before committing code:
- [ ] Follows code standards
- [ ] Tests written and passing
- [ ] Documentation updated
- [ ] ADR created if needed
- [ ] Multi-tenancy enforced
- [ ] Security reviewed
- [ ] References active docs only
