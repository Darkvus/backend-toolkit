---
argument-hint: <github-issue-number>
description: Implement a GitHub issue created by create-issues-from-plan in the current microservice repository
---

# SDD Level 4 GitHub Issue Implementation

## Input

Issue number: `$ARGUMENTS`

This command implements a GitHub issue that was created by `create-issues-from-plan.md`. It reads the issue details, extracts the implementation plan, and executes it following DDD principles.

---

## Phase 1: Setup

### 1.1 Fetch Latest Changes

```bash
git fetch origin
git status
```

### 1.2 Identify Current Repository

```bash
# Get current repo name
basename $(git rev-parse --show-toplevel)

# Get remote URL
git remote get-url origin
```

### 1.3 Get Issue Details

```bash
# Fetch issue content
gh issue view $ARGUMENTS --json title,body,labels,state,comments

# Fetch with comments
gh issue view $ARGUMENTS --comments
```

---

## Phase 2: Analysis

### 2.1 Parse Issue Content

From the issue body (created by create-issues-from-plan), extract:

**Context Section:**
- Issue key (e.g., SMP-2494)
- Overall feature description
- This microservice's role

**Reference Documents:**
- Technical Specification path
- Implementation Plan path
- Jira Issue URL

**Implementation Summary:**
- Domain Changes (new/modified entities)
- Files to Create (table)
- Files to Modify (table)
- API Endpoints (table)
- Database Migrations
- Events (publish/consume)

**Integration Points:**
- Dependencies on other microservices
- What this provides to other microservices

**Definition of Done:**
- Checklist items

**Testing Checklist:**
- Test scenarios to implement

### 2.2 Read Reference Documents (if accessible)

If the reference documents are accessible from this repository:

```bash
# Check if implementation plan exists locally
ls -la .claude/doc/{issue_key}/backend_implementation_plan.md
```

If not accessible, rely on the issue body content which should be self-contained.

### 2.3 Analyze Current Codebase

Explore the current repository structure:

```bash
# Project structure
tree -L 3 -I '__pycache__|*.pyc|.git|node_modules|.venv|venv'

# Find existing models
find . -name "models.py" -o -name "entities.py" | head -20

# Find existing API endpoints
find . -name "views.py" -o -name "viewsets.py" -o -name "routers.py" | head -20

# Find existing tests
find . -name "test_*.py" -o -name "*_test.py" | head -20
```

### 2.4 Create Implementation Checklist

Based on the issue content, create a detailed TODO list:

```markdown
## Implementation Checklist for Issue #{issue_number}

### Domain Layer
- [ ] Create/modify entity: {entity_name}
- [ ] Create/modify value object: {vo_name}
- [ ] Add domain service: {service_name}

### Application Layer
- [ ] Create use case: {use_case_name}
- [ ] Create/modify DTO: {dto_name}
- [ ] Add application service: {service_name}

### Infrastructure Layer
- [ ] Create/modify model: {model_name}
- [ ] Create repository implementation: {repo_name}
- [ ] Create database migration

### API Layer
- [ ] Create/modify endpoint: {method} {path}
- [ ] Create/modify serializer: {serializer_name}
- [ ] Update URL routing

### Events
- [ ] Implement event publisher: {event_name}
- [ ] Implement event consumer: {event_name}

### Testing
- [ ] Unit tests for domain entities
- [ ] Unit tests for use cases
- [ ] Integration tests for API endpoints
- [ ] Integration tests for events

### Documentation
- [ ] Update API documentation
- [ ] Update README if needed
```

---

## Phase 3: Branch Setup

### 3.1 Create Feature Branch

Extract the issue key from the issue body and create a branch:

```bash
# Branch naming convention: feature/{issue_key}
git checkout -b feature/{issue_key}
```

Or if branch already exists:

```bash
git checkout feature/{issue_key}
git pull origin feature/{issue_key}
```

---

## Phase 4: Implementation

### 4.1 Implementation Order

Follow this order to respect layer dependencies:

1. **Domain Layer First**
   - Entities and aggregates
   - Value objects
   - Domain services
   - Domain events

2. **Infrastructure Layer Second**
   - Database models
   - Repository implementations
   - External service adapters
   - Database migrations

3. **Application Layer Third**
   - Use cases
   - Application services
   - DTOs

4. **API Layer Last**
   - Serializers
   - ViewSets/Routers
   - URL configuration

5. **Events**
   - Publishers
   - Consumers

### 4.2 Test-Driven Development

For each component:

1. **Write tests first**
   ```bash
   # Run tests to see them fail
   pytest tests/unit/domain/test_{entity}.py -v
   ```

2. **Implement the component**

3. **Run tests to verify**
   ```bash
   pytest tests/unit/domain/test_{entity}.py -v
   ```

4. **Run full test suite**
   ```bash
   pytest --tb=short
   ```

### 4.3 Database Migrations

After creating/modifying models:

```bash
# For Django
python manage.py makemigrations
python manage.py migrate

# For Alembic (FastAPI)
alembic revision --autogenerate -m "{issue_key}: {description}"
alembic upgrade head
```

### 4.4 Continuous Validation

After each significant change:

```bash
# Run linter
ruff check . --fix
# or
flake8 .

# Run type checker
mypy .

# Run tests
pytest --tb=short

# Run full build
make build
# or
docker-compose build
```

---

## Phase 5: Commit & Push

### 5.1 Stage Changes

```bash
git status
git add .
```

### 5.2 Commit with Conventional Format

```bash
git commit -m "$(cat <<'EOF'
✨ feat({issue_key}): {brief description}

- Implemented {component 1}
- Added {component 2}
- Created tests for {component 3}

Refs: #{issue_number}

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"
```

### 5.3 Push Branch

```bash
git push -u origin feature/{issue_key}
```

---

## Phase 6: Create Pull Request

### 6.1 Create PR

```bash
gh pr create \
  --title "✨ feat({issue_key}): {brief description}" \
  --body "$(cat <<'EOF'
## Related Issue

Closes #{issue_number}

## Summary

{Summary of changes implemented}

## Changes

### Domain Layer
- {change 1}
- {change 2}

### Infrastructure Layer
- {change 1}
- {change 2}

### API Layer
- {change 1}
- {change 2}

### Tests
- {test 1}
- {test 2}

## Testing

- [ ] Unit tests pass
- [ ] Integration tests pass
- [ ] Manual testing completed

## Checklist

- [ ] Code follows project conventions
- [ ] Tests added/updated
- [ ] Documentation updated
- [ ] No breaking changes (or documented)

---

🤖 Generated with [Claude Code](https://claude.com/claude-code)
EOF
)"
```

### 6.2 Capture PR URL

```bash
gh pr view --json url -q .url
```

---

## Phase 7: CI/CD Verification

### 7.1 Monitor Pipeline Status

```bash
# Check PR status
gh pr view {pr_number} --json statusCheckRollup,state,mergeable,url
```

### 7.2 If Checks Fail

1. **Read the failure logs**
   ```bash
   gh pr checks {pr_number}
   ```

2. **Fix the issues**

3. **Commit and push fixes**
   ```bash
   git add .
   git commit -m "🐛 fix({issue_key}): fix CI failures"
   git push
   ```

4. **Repeat until all checks pass**

---

## Phase 8: Update Issue

### 8.1 Add Implementation Comment

```bash
gh issue comment $ARGUMENTS --body "$(cat <<'EOF'
## ✅ Implementation Complete

### PR
{pr_url}

### Changes Implemented

**Domain Layer:**
- {change 1}
- {change 2}

**Infrastructure Layer:**
- {change 1}
- {change 2}

**API Layer:**
- {change 1}
- {change 2}

**Tests Added:**
- {test 1}
- {test 2}

### Definition of Done Status

- [x] Implementation complete following the plan
- [x] Unit tests added
- [x] Integration tests added
- [x] API documentation updated
- [x] Database migrations created
- [x] CI/CD passes
- [ ] Code review approved (pending)
- [ ] Manual testing complete (pending)

---

🤖 Generated with [Claude Code](https://claude.com/claude-code)
EOF
)"
```

---

## Phase 9: Results Report

Present final status:

```markdown
## Implementation Results for Issue #{issue_number}

### Summary of Requirements Implemented

- ✅ {requirement 1}
- ✅ {requirement 2}
- ✅ {requirement 3}

### Requirements Pending

- ⏳ {pending item} (reason)

### Files Created

| File | Purpose |
|------|---------|
| {path} | {purpose} |

### Files Modified

| File | Changes |
|------|---------|
| {path} | {changes} |

### Tests Implemented and Status

```
pytest results:
tests/unit/domain/test_entity.py ............ PASSED
tests/unit/application/test_use_case.py ..... PASSED
tests/integration/test_api.py ............... PASSED

Total: X passed, 0 failed
```

### Build Status

```
✅ Linting: Passed
✅ Type checking: Passed
✅ Unit tests: Passed
✅ Integration tests: Passed
✅ Build: Passed
```

### Overall Status: [Needs More Work / All Completed]

### PR: {github_pr_url}

### Next Steps

{If Needs More Work: list what remains}
{If All Completed: "Ready for code review and merge"}
```

---

## Rules

### General Rules
- **DO** follow Test-Driven Development (write tests first)
- **DO** follow the layer implementation order (Domain → Infrastructure → Application → API)
- **DO** run tests continuously during implementation
- **DO** commit with conventional commit format
- **DO** update the issue with implementation progress
- **DO** wait for CI/CD to pass before marking complete
- **ALWAYS** use `gh` CLI for GitHub operations
- **NEVER** implement manual tests (only automated tests)
- **NEVER** push directly to main/master branch
- **NEVER** skip tests

### DDD Layer Rules
- **Domain Layer**: No framework dependencies, pure business logic
- **Infrastructure Layer**: Database models, external integrations
- **Application Layer**: Use cases, orchestration, DTOs
- **API Layer**: HTTP concerns only, thin adapters

### Commit Conventions

| Type | Emoji | Usage |
|------|-------|-------|
| feat | ✨ | New feature |
| fix | 🐛 | Bug fix |
| refactor | 🔨 | Code refactoring |
| test | ✅ | Adding tests |
| docs | 📝 | Documentation |
| chore | 🔧 | Maintenance |

### CI/CD Loop

```
Push → Check Status → If Failed → Fix → Push → Repeat
                    → If Passed → Done
```

Continue the loop until all checks are green.

---

## Error Handling

### If Issue Not Found
```
Error: Issue #{number} not found in this repository.
Please verify:
1. You're in the correct repository
2. The issue number is correct
3. You have access to this repository
```

### If Branch Already Exists
```
Branch feature/{issue_key} already exists.
Options:
A) Continue working on existing branch
B) Delete and recreate branch
C) Create branch with different name
```

### If Tests Fail
```
Tests failed. Before proceeding:
1. Review the test output
2. Fix the failing tests
3. Run tests again
4. Only continue when all tests pass
```