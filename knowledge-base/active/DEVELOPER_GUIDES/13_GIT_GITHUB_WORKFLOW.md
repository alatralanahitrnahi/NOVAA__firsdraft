# Git & GitHub Workflow Guide

**Last Updated**: January 2026  
**Status**: Complete  
**Audience**: All Developers, Team Leads, Organization Admins

---

## 📋 Table of Contents

1. [Git Fundamentals](#git-fundamentals)
2. [GitHub Workflow](#github-workflow)
3. [Commit Message Conventions](#commit-message-conventions)
4. [Pull Request Process](#pull-request-process)
5. [Issues Management](#issues-management)
6. [Organization Structure](#organization-structure)
7. [Developer Responsibilities](#developer-responsibilities)
8. [Organization Responsibilities](#organization-responsibilities)
9. [Multi-Developer Collaboration](#multi-developer-collaboration)
10. [Common Workflows](#common-workflows)

---

## Git Fundamentals

### What is Git?

Git is a version control system that tracks changes to code. It allows multiple developers to work on the same project simultaneously without overwriting each other's work.

### When to Use Git

| Scenario | Use Git |
|----------|---------|
| Making code changes | ✅ ALWAYS |
| Adding new features | ✅ ALWAYS |
| Fixing bugs | ✅ ALWAYS |
| Updating documentation | ✅ ALWAYS |
| Making quick edits | ✅ YES - even small changes |
| Experimenting with code | ✅ Create a branch |
| Work in progress | ✅ Commit frequently |

### Basic Git Commands

#### 1. **Clone Repository** (First Time Setup)
```bash
git clone https://github.com/NOVAA-Organization/NOVAA__development.git
cd NOVAA__development
```

#### 2. **Check Status**
```bash
git status
```

**Output Example**:
```
On branch main
Your branch is up to date with 'origin/main'.

Changes not staged for commit:
  (use "git add <file>..." to stage changes)
        modified:   src/components/Dashboard.jsx

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        src/utils/newHelper.js

nothing added to commit but untracked files present (working directory clean)
```

#### 3. **Create a Branch** (Always do this before coding)
```bash
# Create and switch to a new branch
git checkout -b feature/student-dashboard

# OR (newer syntax)
git switch -c feature/student-dashboard
```

**Branch Naming Convention**:
```
feature/description       # New feature
bugfix/description        # Bug fix
hotfix/description        # Urgent production fix
docs/description          # Documentation
refactor/description      # Code refactoring
test/description          # Testing
```

**Examples**:
```bash
git checkout -b feature/payment-gateway
git checkout -b bugfix/login-error
git checkout -b docs/api-documentation
git checkout -b refactor/authentication-service
```

#### 4. **Stage Changes**
```bash
# Stage specific file
git add src/components/Dashboard.jsx

# Stage all changes
git add .

# Stage multiple files
git add src/ docs/
```

#### 5. **Commit Changes**
```bash
git commit -m "feat: add student dashboard with search filter"
```

#### 6. **Push to GitHub**
```bash
# First time pushing a new branch
git push -u origin feature/student-dashboard

# Subsequent pushes
git push
```

#### 7. **Pull Latest Changes**
```bash
# Before starting work
git pull origin main

# Or to sync your branch
git pull
```

#### 8. **Switch Branches**
```bash
git checkout main
git checkout feature/student-dashboard

# OR (newer syntax)
git switch main
git switch feature/student-dashboard
```

#### 9. **Delete Branch**
```bash
# Delete local branch
git branch -d feature/student-dashboard

# Delete remote branch
git push origin --delete feature/student-dashboard
```

---

## GitHub Workflow

### Step-by-Step Process

#### **Phase 1: Preparation**

```
1. Go to GitHub
   ↓
2. Create/Assign Issue (what needs to be done)
   ↓
3. Clone repo locally
   ↓
4. Create feature branch from main
   ↓
5. Make code changes
```

#### **Phase 2: Submission**

```
6. Commit changes with proper messages
   ↓
7. Push branch to GitHub
   ↓
8. Create Pull Request (request to merge)
   ↓
9. Request code review
```

#### **Phase 3: Review & Merge**

```
10. Address review comments
    ↓
11. Make additional commits
    ↓
12. Get approval
    ↓
13. Merge to main
    ↓
14. Delete branch
    ↓
15. Deploy to staging/production
```

---

## Commit Message Conventions

### Format

```
<type>(<scope>): <subject>

<body>

<footer>
```

### Types

| Type | Usage | Example |
|------|-------|---------|
| **feat** | New feature | `feat: add student dashboard` |
| **fix** | Bug fix | `fix: resolve login timeout issue` |
| **docs** | Documentation | `docs: update API documentation` |
| **style** | Code style (spacing, formatting) | `style: format payment service` |
| **refactor** | Code refactoring | `refactor: simplify authentication logic` |
| **perf** | Performance improvement | `perf: optimize database queries` |
| **test** | Add/update tests | `test: add unit tests for payment` |
| **chore** | Maintenance tasks | `chore: update dependencies` |
| **ci** | CI/CD configuration | `ci: add GitHub Actions workflow` |

### Subject Line Rules

✅ **DO**:
- Start with lowercase
- Use imperative mood ("add" not "added" or "adds")
- Keep under 50 characters
- Be specific and clear
- Reference issue number if applicable

❌ **DON'T**:
- Use period at end
- Use ALL CAPS
- Use vague messages like "update" or "fix bug"
- Exceed 50 characters

### Examples

**GOOD** ✅
```bash
git commit -m "feat: implement payment verification webhook"
git commit -m "fix: resolve duplicate transaction issue (#456)"
git commit -m "docs: add payment processing guide"
git commit -m "refactor: extract authentication logic to utility"
git commit -m "test: add unit tests for GST calculator"
git commit -m "perf: add database indexes for fee queries"
```

**BAD** ❌
```bash
git commit -m "Updates" 
git commit -m "Fixed bug"
git commit -m "CHANGES TO PAYMENT SYSTEM"
git commit -m "updated stuff in the authentication module and also fixed a small issue"
git commit -m "More improvements."
```

### Extended Format (For Complex Changes)

```bash
git commit -m "feat: implement multi-tenancy isolation

- Add collegeId to all database queries
- Implement middleware for tenant validation
- Update API endpoints to filter by organization

Fixes #123
Closes #456"
```

---

## Pull Request Process

### What is a Pull Request (PR)?

A PR is a request to merge your code changes into the main branch. It's a formal review process.

### Creating a Pull Request

#### **Step 1: Push Your Branch**
```bash
git push -u origin feature/student-dashboard
```

#### **Step 2: Go to GitHub**
Visit: `https://github.com/NOVAA-Organization/NOVAA__development`

#### **Step 3: Create PR**
- Click "Pull Requests" tab
- Click "New Pull Request"
- Base: `main`
- Compare: `feature/student-dashboard`
- Click "Create Pull Request"

#### **Step 4: Fill PR Template**

**PR Title** (same as your feature):
```
feat: add student dashboard with search filter
```

**PR Description**:
```markdown
## Description
Implemented a new student dashboard that displays:
- List of all students
- Real-time search filter
- Pagination support
- Export to CSV functionality

## Type of Change
- [x] New feature
- [ ] Bug fix
- [ ] Breaking change

## Testing Done
- [x] Unit tests added (95% coverage)
- [x] Integration tests passed
- [x] Manual testing on staging

## Checklist
- [x] Code follows style guidelines
- [x] Self-review completed
- [x] Comments added for complex logic
- [x] Documentation updated
- [x] No new warnings generated
- [x] Tests added/updated
- [x] All tests passing locally

## Screenshots (if applicable)
[Add screenshots here]

## Related Issues
Closes #123
Related to #456
```

### PR Title Format

```
<type>(<scope>): <description>
```

**Examples**:
```
feat: add student dashboard
fix: resolve payment timeout error
docs: update authentication guide
refactor: simplify service layer
```

### PR Review Checklist (For Reviewers)

When someone requests your code review, check:

```markdown
## Code Quality
- [ ] Code follows conventions (README guide)
- [ ] No console.log or debugging statements
- [ ] No hardcoded values
- [ ] Error handling is proper
- [ ] No duplicate code

## Functionality
- [ ] Changes work as described
- [ ] Edge cases handled
- [ ] No breaking changes
- [ ] Tests added/updated

## Performance
- [ ] No performance regressions
- [ ] Database queries optimized
- [ ] No memory leaks

## Security (CRITICAL for NOVAA)
- [ ] Multi-tenancy checks present (collegeId filters)
- [ ] Authentication/authorization enforced
- [ ] No SQL injection vulnerabilities
- [ ] No data exposure issues
- [ ] Input validation present

## Documentation
- [ ] Code comments present for complex logic
- [ ] README updated if needed
- [ ] API documentation updated
```

### Addressing Review Comments

```
1. See review comment
   ↓
2. Make requested changes
   ↓
3. Commit with message referencing comment
   git commit -m "refactor: update based on PR review (#PR_NUMBER)"
   ↓
4. Push changes
   git push
   ↓
5. Mark comment as resolved
   ↓
6. Request re-review (click "Re-request review")
```

### Merging a PR

**Merge Types**:

1. **Squash and Merge** (Recommended for features)
   - Combines all commits into one
   - Cleaner history
   - Use when: Single feature with multiple commit attempts

2. **Create a Merge Commit** (For releases)
   - Keeps commit history intact
   - Shows who merged and when
   - Use when: Important milestones

3. **Rebase and Merge** (For hotfixes)
   - Linear history
   - No merge commit
   - Use when: Quick bug fixes

**Steps to Merge**:
```
1. PR is approved ✅
   ↓
2. All tests pass ✅
   ↓
3. No conflicts ✅
   ↓
4. Click "Squash and merge"
   ↓
5. Confirm merge
   ↓
6. Delete branch (click "Delete branch" button)
```

---

## Issues Management

### What is a GitHub Issue?

An Issue is a way to track work, bugs, feature requests, and discussions. It's the source of truth for what needs to be done.

### Creating an Issue

**Go to**: `https://github.com/NOVAA-Organization/NOVAA__development/issues`

**Click**: "New Issue"

#### **Issue Template: Bug Report**

```markdown
## Description
Brief description of the bug

## Steps to Reproduce
1. Go to [page/feature]
2. Click [button]
3. See error

## Expected Behavior
What should happen

## Actual Behavior
What actually happens

## Screenshots
[Add screenshots]

## Environment
- Browser: Chrome 120
- OS: Windows 11
- Backend: v1.2.3

## Additional Context
Any other relevant info
```

#### **Issue Template: Feature Request**

```markdown
## Feature Description
What should be built?

## Problem It Solves
Why is this needed?

## Proposed Solution
How should it work?

## Acceptance Criteria
- [ ] Criterion 1
- [ ] Criterion 2
- [ ] Criterion 3

## Related Issues
#123, #456
```

#### **Issue Template: Documentation**

```markdown
## Documentation Needed
What needs documentation?

## Scope
What should be covered?

## Reference Links
Links to relevant code, discussions, etc.

## Acceptance Criteria
- [ ] Complete
- [ ] Reviewed
- [ ] Published
```

### Issue Workflow

```
1. Create Issue
   (Title, description, labels)
   ↓
2. Assign to Developer
   (Who will work on it)
   ↓
3. Developer creates branch
   (Branch name matches issue)
   ↓
4. Developer works on issue
   (Commits reference issue: "fixes #123")
   ↓
5. Create Pull Request
   (PR links to issue)
   ↓
6. Code Review & Approval
   ↓
7. Merge PR
   (Issue auto-closes)
   ↓
8. Deploy to production
```

### Issue Labels

**Type**:
- `bug` - Something is broken
- `feature` - New functionality
- `enhancement` - Improvement to existing feature
- `documentation` - Docs need update

**Priority**:
- `critical` - Blocks deployment/production
- `high` - Should do soon
- `medium` - Can wait a sprint
- `low` - Nice to have

**Status**:
- `in-progress` - Someone is working on it
- `review` - Waiting for code review
- `blocked` - Waiting on something else
- `ready` - Ready to start

**Scope**:
- `backend` - Server/API changes
- `frontend` - UI/UX changes
- `database` - Database changes
- `devops` - Infrastructure/deployment

### Linking Issue to PR

**In PR description**:
```markdown
Closes #123
Fixes #456
Related to #789
```

**In commit message**:
```bash
git commit -m "feat: implement payment webhook fixes #123"
```

---

## Organization Structure

### Repository Setup

```
GitHub Organization: NOVAA-Organization
  ↓
Repository: NOVAA__development (Production code)
  ├── Main branch (deployed code)
  ├── Develop branch (staging)
  ├── Feature branches (developers work here)
  └── Release branches (for version releases)

Repository: NOVAA__documentation (Docs repo)
  └── All guides and documentation

Repository: NOVAA__infrastructure (DevOps)
  └── Deployment configs, Docker, CI/CD
```

### Access Levels

| Role | Permissions |
|------|-------------|
| **Developer** | Clone, create branches, create PRs |
| **Team Lead** | Above + approve PRs, merge code |
| **Organization Admin** | All + manage users, settings, deployments |

---

## Developer Responsibilities

### On GitHub

#### **Before Starting Work**

✅ **DO**:
- [ ] Check existing issues (don't duplicate work)
- [ ] Create new issue or get assigned to existing one
- [ ] Read issue description and acceptance criteria
- [ ] Comment "I'm working on this" on the issue
- [ ] Create feature branch from the issue

#### **During Development**

✅ **DO**:
- [ ] Work in your feature branch (never in main)
- [ ] Commit frequently with clear messages
- [ ] Reference issue number in commits: `fixes #123`
- [ ] Push to GitHub regularly (backup)
- [ ] Keep your branch up to date with main
- [ ] Write unit tests for your code
- [ ] Test locally before pushing

❌ **DON'T**:
- [ ] Commit directly to main
- [ ] Push untested code
- [ ] Use vague commit messages
- [ ] Create overly large PRs (>500 lines)
- [ ] Ignore code review comments
- [ ] Push sensitive information (API keys, passwords)

#### **Submitting Code**

✅ **DO**:
- [ ] Create comprehensive PR description
- [ ] Add screenshots/demos if relevant
- [ ] Run tests locally and verify passing
- [ ] Self-review your own code first
- [ ] Request review from team lead
- [ ] Be responsive to review comments

**PR Size Guidelines**:
- Small PR: 50-200 lines (Fast review)
- Medium PR: 200-500 lines (Normal)
- Large PR: >500 lines (Hard to review - split it)

#### **Code Review Process**

When you receive feedback:

```
1. Read comment carefully
2. Understand the concern
3. Discuss in comment if unclear
4. Make necessary changes
5. Commit with reference to PR
6. Request re-review
```

**Review Comment Response Examples**:

```markdown
Good point! I've updated the code to handle this edge case.
See commit abc123.
```

```markdown
I'm not sure I understand the concern. Can you clarify? 
Should I validate the input at the API level or service level?
```

---

## Organization Responsibilities

### DevOps/Admin Tasks

#### **Repository Management**

✅ **Organization Does**:
- [ ] Create and configure repositories
- [ ] Set up branch protection rules
- [ ] Configure required status checks
- [ ] Set up CI/CD pipelines
- [ ] Manage user access and permissions
- [ ] Configure merge strategies
- [ ] Set up automated testing

**Branch Protection Rules**:
```
Main branch:
  ✅ Require pull request reviews (minimum 1)
  ✅ Require status checks to pass (tests, linting)
  ✅ Require branches to be up to date
  ✅ Restrict who can push (admins only)
  ✅ Dismiss stale reviews when new commits pushed
```

#### **Code Review Approval**

✅ **Organization Does**:
- [ ] Designates code reviewers (usually team leads)
- [ ] Ensures reviewers understand code standards
- [ ] Approves and merges PRs
- [ ] Tracks review SLA (response time)
- [ ] Monitors code quality metrics

#### **Deployment Pipeline**

✅ **Organization Does**:
- [ ] Merge to main triggers staging deployment
- [ ] Organization approves production deployment
- [ ] Tracks deployment history and rollbacks
- [ ] Monitors for issues post-deployment

**Deployment Flow**:
```
Developer PR → Review → Merge to main → 
  ↓
Auto-deploy to staging (npm test, build, deploy)
  ↓
Testing team QAs staging
  ↓
Organization approves prod deployment
  ↓
Manual/auto deploy to production
  ↓
Monitor for issues
```

#### **Release Management**

✅ **Organization Does**:
- [ ] Create release branches
- [ ] Tag releases (v1.0.0, v1.1.0, etc.)
- [ ] Maintain changelog
- [ ] Manage version numbering (semantic versioning)

**Release Tagging**:
```bash
# Create a release tag
git tag -a v1.2.0 -m "Release version 1.2.0"

# Push tag to GitHub
git push origin v1.2.0
```

**Semantic Versioning Format**:
```
v1.2.3
 ↓ ↓ ↓
 │ │ └── Patch (bug fixes: v1.2.3 → v1.2.4)
 │ └──── Minor (new features: v1.2.0 → v1.3.0)
 └────── Major (breaking changes: v1.0.0 → v2.0.0)
```

#### **Issue Triage**

✅ **Organization Does**:
- [ ] Review new issues
- [ ] Add appropriate labels
- [ ] Set priorities
- [ ] Assign to developers
- [ ] Close duplicates
- [ ] Update issue status

---

## Multi-Developer Collaboration

### Simultaneous Development

**Scenario**: Multiple developers working on different features

```
Developer A                Developer B
    ↓                           ↓
Create branch:            Create branch:
feature/dashboard         feature/payments
    ↓                           ↓
Work independently        Work independently
    ↓                           ↓
Push to GitHub            Push to GitHub
    ↓                           ↓
Create PR                 Create PR
    ↓                           ↓
Code Review               Code Review
    ↓                           ↓
Merge to main             Merge to main
    ↓                           ↓
Production Deploy (both features together)
```

### Handling Conflicts

**Conflict**: Two developers modify the same file

**Example**:
```
Developer A changes line 10-20 in auth.js
Developer B changes line 15-25 in auth.js
↓
When merging: CONFLICT - Git can't decide which changes to keep
```

**Resolution**:

```bash
# 1. Pull latest changes
git pull origin main

# 2. Git will show conflicts
# 3. Open conflicted file
# 4. Find conflict markers:

<<<<<<< HEAD (your changes)
  your code here
=======
  their changes here
>>>>>>> branch-name

# 5. Manually resolve (keep what's needed)
# 6. Remove conflict markers
# 7. Stage and commit

git add auth.js
git commit -m "refactor: resolve merge conflict in auth service"
git push
```

### Keeping Branch Updated

**Scenario**: Main branch has new changes, your feature branch is behind

```bash
# Option 1: Merge main into your branch (recommended for active branches)
git pull origin main
# Resolve any conflicts if they occur

# Option 2: Rebase your branch on main (cleaner history)
git rebase origin/main
# This replays your commits on top of main
```

---

## Common Workflows

### Workflow 1: Simple Feature Development

```bash
# 1. Update main
git checkout main
git pull origin main

# 2. Create feature branch
git checkout -b feature/user-profile

# 3. Make changes
# Edit files...

# 4. Stage and commit
git add .
git commit -m "feat: add user profile page with avatar upload"

# 5. Push to GitHub
git push -u origin feature/user-profile

# 6. Create PR on GitHub (web interface)
# - Fill PR description
# - Request review

# 7. Address any review comments
# git add .
# git commit -m "refactor: update based on PR review"
# git push

# 8. Merge PR (via GitHub web interface)
```

### Workflow 2: Bug Fix on Production

```bash
# 1. Create issue: "Login button not working"
# 2. Create hotfix branch from main
git checkout main
git pull origin main
git checkout -b bugfix/login-button

# 3. Fix the bug
# Edit files...

# 4. Test locally
npm test

# 5. Commit with issue reference
git add .
git commit -m "fix: resolve login button click handler issue #234"

# 6. Push and create PR
git push -u origin bugfix/login-button

# 7. Fast-track review (hotfixes get priority)
# 8. Merge PR
# 9. Deploy to production immediately
```

### Workflow 3: Collaborative Feature with Multiple PRs

```bash
# Developer A (Core API)
git checkout -b feature/payment-api
# ... implement payment API endpoints ...
# PR #101 → Review → Merge

# Developer B (UI Component)
git checkout -b feature/payment-ui
# ... needs Developer A's API changes ...
git pull origin main  # Gets Developer A's merged code
# ... implement payment UI using API ...
# PR #102 → Review → Merge
```

### Workflow 4: Update Your Branch with Latest Main

```bash
# You're working on feature branch
git checkout feature/dashboard

# Main branch got new changes you need
git pull origin main

# OR if you want clean history
git fetch origin
git rebase origin/main

# If conflicts, resolve them
# Then continue rebase
git rebase --continue

# Force push (only on your feature branch!)
git push --force-with-lease
```

---

## Best Practices

### Commit Frequency

✅ **GOOD PATTERN**:
```
Morning: Commit 1 (setup)
         Commit 2 (implementation)
         Commit 3 (tests)
         Commit 4 (documentation)

Afternoon: Commit 5 (bug fix)
           Commit 6 (review changes)
           Commit 7 (final tests)

Reason: Easier to track changes, easier to revert if needed
```

### Branch Naming

```
✅ DO:
  feature/user-authentication
  bugfix/payment-calculation
  docs/api-endpoints
  refactor/service-layer
  test/unit-tests

❌ DON'T:
  feature
  fix-bug
  random-changes
  wip-stuff
  myfeature
```

### PR Size

| Size | Ideal Review Time | Example |
|------|------------------|---------|
| Tiny (<50 lines) | 5 min | Simple bug fix |
| Small (50-200 lines) | 15 min | New utility |
| Medium (200-500 lines) | 30 min | Feature with tests |
| Large (>500 lines) | >1 hour | Complex feature |

**Recommendation**: Aim for small/medium sized PRs

### Code Review Comments

✅ **GOOD FEEDBACK**:
```markdown
I think this could be more efficient using Array.map instead of a for loop.
See: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map

Example:
const names = users.map(user => user.name);
```

❌ **UNHELPFUL FEEDBACK**:
```markdown
This is wrong.
Change it.
```

### Frequency of Communication

- **Daily**: Commit code, push to GitHub
- **Daily**: Check for PR reviews, respond to comments
- **Before PR**: Communicate with team about approach
- **In PR**: Explain complex logic in comments
- **Post-merge**: Update team if issues found

---

## Emergency: Production Hotfix

**Scenario**: Production is broken, need immediate fix

```bash
# 1. Create urgent issue
# 2. Create hotfix branch from main
git checkout main
git pull origin main
git checkout -b hotfix/critical-payment-error

# 3. Fix the issue
# 4. Test thoroughly
npm test

# 5. Commit
git commit -m "fix: resolve critical payment processing error #999"

# 6. Fast-track PR (notify team lead)
git push -u origin hotfix/critical-payment-error
# Create PR, mention URGENT

# 7. Quick review (15 min target)
# 8. Merge immediately
# 9. Deploy to production immediately

# 10. Post-mortem discussion:
#     - Why did this happen?
#     - How to prevent next time?
#     - Add tests to catch this in future
```

---

## Checklist Before Pushing

**Before every `git push`, check**:

```
[ ] I'm on the correct branch (not main)
[ ] Code builds successfully (npm run build)
[ ] All tests pass (npm test)
[ ] No console.log statements
[ ] No hardcoded values
[ ] No sensitive data (keys, passwords)
[ ] Code follows conventions
[ ] Commit message is clear
[ ] Commit message references issue
[ ] No merge conflicts
[ ] Latest changes from main pulled
```

---

## Troubleshooting

### Problem: "Please commit your changes before switching branches"

**Cause**: You have unsaved changes

**Solution**:
```bash
# Option 1: Commit changes
git add .
git commit -m "work in progress"

# Option 2: Stash changes (save for later)
git stash
git checkout main
# Later, restore:
git stash pop
```

### Problem: "Commit with this message already exists"

**Cause**: Likely rebasing old code

**Solution**:
```bash
# Use --force-with-lease (safer than --force)
git push --force-with-lease origin feature-branch
```

### Problem: "Merge conflict"

**Solution**:
```bash
# 1. Open conflicted files
# 2. Find conflict markers (<<<, ===, >>>)
# 3. Manually resolve
# 4. Remove markers
# 5. Stage and commit

git add .
git commit -m "refactor: resolve merge conflicts"
git push
```

### Problem: "Nothing to commit, working tree clean"

**Cause**: No changes were made

**Solution**: Verify you actually made changes and saved files

---

## Summary: Who Does What

### Developer Does:

1. ✅ Creates feature branch
2. ✅ Writes code and tests
3. ✅ Commits with clear messages
4. ✅ Pushes to GitHub
5. ✅ Creates Pull Request
6. ✅ Requests code review
7. ✅ Addresses review feedback
8. ✅ Participates in code review (reviewing others' code)

### Team Lead Does:

1. ✅ Approves Pull Requests
2. ✅ Merges code to main
3. ✅ Triages issues
4. ✅ Assigns issues to developers
5. ✅ Manages sprint board
6. ✅ Ensures code quality standards

### Organization/Admin Does:

1. ✅ Configures repositories
2. ✅ Sets up CI/CD pipelines
3. ✅ Manages user access
4. ✅ Deploys to staging/production
5. ✅ Manages releases and versioning
6. ✅ Monitors production
7. ✅ Sets organization policies

---

## Quick Reference Card

```bash
# Daily Commands
git status                    # Check what changed
git add .                     # Stage all changes
git commit -m "message"       # Commit with message
git push                      # Push to GitHub
git pull origin main          # Get latest from main

# Branch Commands
git checkout -b feature/name  # Create new branch
git checkout main             # Switch to main
git branch -d feature/name    # Delete branch

# Before Creating PR
npm test                      # Run tests
npm run build                 # Build project
git log -1                    # Check last commit

# After PR Review
git add .                     # Stage changes
git commit -m "refactor: ..."  # Commit new changes
git push                      # Push again

# Emergency Hotfix
git checkout main
git pull origin main
git checkout -b hotfix/issue-name
# Fix code
git commit -m "fix: ..."
git push -u origin hotfix/issue-name
# Create PR with URGENT label
```

---

## Next Steps

1. **Setup**: Clone the repository locally
2. **First PR**: Create your first feature branch and PR
3. **Collaboration**: Review a team member's PR
4. **Emergency**: Practice hotfix workflow
5. **Mastery**: Mentor new team members on Git

---

**Questions?** Ask in #dev-help Slack channel or reach out to your Team Lead.
