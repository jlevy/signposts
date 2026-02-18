# Comprehensive Review: Signposts Design

**Reviewer:** Claude Sonnet 4.5 **Date:** 2026-02-17 **Document:**
[signposts-design.md](signposts-design.md)

## Executive Summary

The signposts design is well-conceived with clear goals, strong prior art analysis, and
thoughtful design decisions.
The core concepts (areas, paths, steps, routes) form a coherent model for navigable
knowledge graphs.

**Strengths:**
- Clear separation between graph structure (signposts.yml) and progress tracking
  (progress.yml)
- Flexible routing with descriptive conditions rather than enforced gates
- Well-defined schema with good constraint validation
- Comprehensive example showing real-world usage
- Excellent documentation of design decisions and rationale

**Areas for enhancement:**
- Need more diverse examples beyond porting areass
- Missing details on format evolution and migration
- Collaboration scenarios (multiple users/agents) need clarification
- Error handling and edge cases could be more explicit
- CLI examples focus on output; need more invocation examples

## Detailed Review

### 1. Overview and Goals (Strong ✓)

**What works well:**
- Clear articulation of the problem (implicit structure in prose)
- Strong design philosophy: “YAML is the map; docs are the territory”
- Goals are specific and achievable

**Suggestions:**
1. Add a “Quick Example” section right after Overview showing a 10-line minimal
   signposts.yml
2. Include a visual diagram of the concept hierarchy (area → path → step)
3. Add concrete metrics for success (e.g., “reduce time to find relevant docs by 50%”)

**Missing example: Minimal signposts.yml**

```yaml
# Simplest possible signposts.yml
format: "SP/0.1"
name: Quick Start Guide
start: getting-started

areas:
  - id: getting-started
    title: Getting Started
    paths:
      - id: setup
        title: Initial Setup
        docs:
          - ./docs/setup.md
        steps:
          - ./docs/setup.md#install
          - ./docs/setup.md#configure
        routes:
          - to: first-project

      - id: first-project
        title: Your First Project
        docs:
          - ./docs/tutorial.md
```

### 2. Schema and Format (Very Strong ✓)

**What works well:**
- Docspec/locspec design is elegant and unambiguous
- Constraints are well-documented
- Field tables are comprehensive

**Missing details:**

#### 2.1 Format Evolution

How does SP/0.1 → SP/0.2 work?
Add section:

```markdown
### Format Versioning

signposts uses semantic versioning for the format string:

- **SP/0.1** — Current version. Breaking changes will increment the minor version.
- **SP/0.2** — Future version (example). Tools check compatibility and error if format > supported version.
- **Forward compatibility:** Tools should warn but not fail on unknown fields (allows gradual rollout).
- **Backward compatibility:** Deprecated fields are supported for 2+ versions before removal.

Migration tools:
- `signposts migrate` — Upgrade signposts.yml to current format version
- `signposts validate --strict` — Check for deprecated fields
```

#### 2.2 More Examples of Docspecs in Practice

Add a dedicated section with edge cases:

````markdown
### Docspec Examples: Edge Cases

**Spaces in filenames (must use ./ prefix):**
```yaml
# Valid
- ./docs/Getting Started.md

# Invalid (no recognized prefix)
- docs/Getting Started.md  # ❌ Parse error
````

**Refs with complex content:**
```yaml
refs:
  - id: dependency-checklist
    title: External Dependencies Checklist
    body: |
      Check each dependency:
      - [ ] License compatible (MIT, Apache-2.0, BSD)
      - [ ] Actively maintained (commit in last 6 months)
      - [ ] Security audit if critical path
      - [ ] Size/performance acceptable

      See ./docs/dependency-guidelines.md for full criteria.
```

**Remote sources with branches containing slashes:**
```yaml
# Branch: feature/new-design
- github:jlevy/repo@feature/new-design//docs/spec.md

# Tag
- github:jlevy/repo@v1.2.3//docs/api.md

# Commit hash
- github:jlevy/repo@a1b2c3d//docs/architecture.md
```

**URLs with anchors:**
```yaml
# External docs
- https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html
- https://docs.rs/clap/latest/clap/
```
````

### 3. Progress Tracking (Good, needs clarification)

**What works well:**
- Status values are clear (not_started, in_progress, completed, skipped)
- Extra steps allow project-specific additions

**Missing details:**

#### 3.1 Collaboration Scenarios

Add section:

```markdown
### Collaboration and Merge Conflicts

**Single agent/human:** Straightforward. progress.yml tracks your position.

**Multiple agents in sequence (handoff):**
- Agent A completes steps 1-3, commits progress.yml
- Agent B pulls, sees current_path and completed steps
- Agent B continues from step 4
- Works well ✓

**Multiple agents/humans in parallel:**
- Both working on different paths → merge cleanly ✓
- Both working on same path → merge conflict possible

**Merge conflict resolution:**
```yaml
# If both agents mark different steps complete, combine them:
paths:
  research:
    steps:
      ./docs/step1.md: completed  # Agent A
      ./docs/step2.md: completed  # Agent B
      # Both valid, merge and keep both
````

**Sync detection:** When loading progress.yml, check:
1. Does the `signposts` path still point to a valid file?
2. Do all path IDs in progress.yml still exist in signposts.yml?
3. Do all step docspecs in progress.yml still exist in the target paths?

If not, warn:
```
⚠️  Progress file references paths no longer in signposts.yml:
    - old-path (status: completed)

    Run 'signposts sync-progress' to update or archive these.
```
````

#### 3.2 Signposts Evolution Scenarios

What happens when signposts.yml changes?

```markdown
### Signposts Evolution

**Path renamed:**
```yaml
# signposts.yml changes: assess → assessment
# progress.yml still has: assess

# Tool behavior:
⚠️  Path 'assess' not found in signposts.yml
    Possible rename detected. Update progress.yml:
      assess → assessment
````

**Steps reordered:**
```yaml
# signposts.yml changes step order
# progress.yml tracks completion by docspec (stable)
# ✓ No issue — step identity is docspec, not position
```

**Steps added/removed:**
```yaml
# signposts.yml adds new step to existing path
# progress.yml doesn't have it
# ✓ Defaults to not_started (merge semantics already handle this)

# signposts.yml removes a step
# progress.yml has it as completed
# ✓ Preserved in extra_steps (project may have done it)
```

**Recommended approach:**
```bash
# After pulling signposts.yml updates:
signposts validate-progress
# Shows: new steps, removed steps, renamed paths, orphaned progress
```
````

### 4. CLI Commands (Good, needs more examples)

**What works well:**
- Output examples are clear and well-formatted
- Commands cover all workflow steps

**Missing: Actual command invocations with arguments**

Add section:

```markdown
### CLI Command Examples

#### Starting a new flow

```bash
# Initialize progress tracking
$ signposts init
Created progress.yml
Set current_path to 'assess' (first path in 'getting-started' workflow)

# See where you are
$ signposts where
Current path: assess — Understand scope, dependencies, and test coverage
  (area: Getting Started)

  Steps:
    [ ] ./reference/playbook.md#dependency-risk-table
    [ ] ./reference/test-coverage.md
    [ ] ref#go-no-go-criteria
````

#### Working through a path

```bash
# View documentation for current step
$ signposts step assess "./reference/playbook.md#dependency-risk-table"
[prints the dependency-risk-table section]

# Mark step complete
$ signposts done assess "./reference/playbook.md#dependency-risk-table"
✓ Step completed: ./reference/playbook.md#dependency-risk-table
  Progress: 1/3 steps complete in 'assess'

# View current position
$ signposts next
Next step: ./reference/test-coverage.md
  (path: assess, area: Getting Started)

  View: signposts step assess "./reference/test-coverage.md"
  Done: signposts done assess "./reference/test-coverage.md"
```

#### Navigating between paths

```bash
# Complete current path, move to next
$ signposts done assess "./reference/playbook.md#go-no-go-criteria"
✓ All steps in 'assess' completed!

  Routes available:
    -> research (default)
    -> improve-coverage (when: Test coverage below 80% on core modules)

  Continue? signposts start research

# Jump to a different path
$ signposts start improve-coverage
Set current_path to 'improve-coverage'
Status: not_started → in_progress

# Skip a path entirely
$ signposts skip improve-coverage
Status: in_progress → skipped

# Add a note explaining why
$ signposts note improve-coverage "Coverage already at 85%, skipping"
```

#### Advanced usage

```bash
# View full graph structure
$ signposts

# Filter to specific workflow
$ signposts --area=core-area

# JSON output for agent consumption
$ signposts status --json
{
  "current_path": "research",
  "total_paths": 9,
  "completed": 1,
  "in_progress": 1,
  "not_started": 7,
  "workflows": [...]
}

# Check against a different signposts file
$ signposts --file=../other-repo/signposts.yml
```

#### Guide alias (simpler interface)

```bash
# List all paths
$ signposts guide
[flat listing of all paths with descriptions]

# View docs for a path
$ signposts guide research
[prints all docs for the research path]

# Difference from signposts command:
# - guide: static view, no progress tracking
# - signposts: dynamic, tracks where you are
```
````

### 5. Non-Porting Examples (Critical Gap)

The document heavily focuses on Python→Rust porting. Add diverse examples:

```markdown
### Example: Employee Onboarding

```yaml
format: "SP/0.1"
name: Engineering Onboarding
start: welcome

areas:
  - id: welcome
    title: Welcome
    description: Choose your onboarding track
    docs:
      - ref#welcome-message
    routes:
      - to: backend-engineer/setup
        when: Backend engineering role
      - to: frontend-engineer/setup
        when: Frontend engineering role
      - to: devops-engineer/setup
        when: DevOps/SRE role

  - id: backend-engineer
    title: Backend Engineer Track
    paths:
      - id: setup
        title: Development Setup
        docs:
          - ./onboarding/backend-setup.md
        steps:
          - ./onboarding/backend-setup.md#accounts
          - ./onboarding/backend-setup.md#tools
          - ./onboarding/backend-setup.md#repos
        routes:
          - to: first-task

      - id: first-task
        title: Your First Pull Request
        docs:
          - ./onboarding/first-pr-guide.md
        steps:
          - ./onboarding/first-pr-guide.md#pick-issue
          - ./onboarding/first-pr-guide.md#implement
          - ./onboarding/first-pr-guide.md#review
        routes:
          - to: team-meetings

  - id: company-wide
    title: Company-Wide (All Roles)
    paths:
      - id: team-meetings
        title: Meet Your Team
        docs:
          - ./onboarding/meetings.md

      - id: benefits
        title: Benefits and Policies
        docs:
          - ./onboarding/benefits.md
          - https://intranet.company.com/hr/benefits

refs:
  - id: welcome-message
    title: Welcome to Engineering!
    body: |
      Welcome! We're excited to have you join the team.

      Your onboarding will be customized based on your role.
      We'll guide you through setup, your first tasks, and company processes.

      Expected timeline: 2-4 weeks to full productivity.
````

### Example: Security Assessment Workflow

```yaml
format: "SP/0.1"
name: Web Application Security Assessment
start: scoping

areas:
  - id: scoping
    title: Scoping
    paths:
      - id: initial-recon
        title: Initial Reconnaissance
        docs:
          - ./security/recon-checklist.md
        steps:
          - ./security/recon-checklist.md#asset-discovery
          - ./security/recon-checklist.md#tech-stack
          - ./security/recon-checklist.md#attack-surface
        routes:
          - to: assessment/authentication

  - id: assessment
    title: Security Assessment
    paths:
      - id: authentication
        title: Authentication & Authorization
        docs:
          - ./security/auth-testing.md
          - https://owasp.org/www-project-web-security-testing-guide/
        steps:
          - ./security/auth-testing.md#password-policy
          - ./security/auth-testing.md#session-management
          - ./security/auth-testing.md#privilege-escalation
        routes:
          - to: injection

      - id: injection
        title: Injection Vulnerabilities
        docs:
          - ./security/injection-testing.md
        steps:
          - ./security/injection-testing.md#sql-injection
          - ./security/injection-testing.md#xss
          - ./security/injection-testing.md#command-injection
        routes:
          - to: reporting/findings

  - id: reporting
    title: Reporting
    paths:
      - id: findings
        title: Document Findings
        docs:
          - ./security/report-template.md

      - id: remediation
        title: Remediation Verification
        docs:
          - ./security/retest-checklist.md
```

### Example: API Migration Workflow

```yaml
format: "SP/0.1"
name: REST API to GraphQL Migration
start: planning

areas:
  - id: planning
    title: Planning
    paths:
      - id: inventory
        title: API Inventory
        description: Catalog all REST endpoints
        docs:
          - ./migration/api-inventory.md
        steps:
          - ./migration/api-inventory.md#list-endpoints
          - ./migration/api-inventory.md#document-usage
          - ./migration/api-inventory.md#identify-clients
        routes:
          - to: schema-design

      - id: schema-design
        title: GraphQL Schema Design
        docs:
          - ./migration/schema-design.md
        steps:
          - ./migration/schema-design.md#types
          - ./migration/schema-design.md#queries
          - ./migration/schema-design.md#mutations
        routes:
          - to: implementation/resolvers

  - id: implementation
    title: Implementation
    paths:
      - id: resolvers
        title: Build Resolvers
        docs:
          - ./migration/resolver-guide.md
        routes:
          - to: testing

      - id: testing
        title: Integration Testing
        docs:
          - ./migration/testing-guide.md
        routes:
          - to: rollout/gradual

  - id: rollout
    title: Rollout
    paths:
      - id: gradual
        title: Gradual Migration
        description: Run REST and GraphQL in parallel
        docs:
          - ./migration/gradual-rollout.md
        steps:
          - ./migration/gradual-rollout.md#enable-graphql
          - ./migration/gradual-rollout.md#client-migration
          - ./migration/gradual-rollout.md#monitor-metrics
        routes:
          - to: deprecation
            when: All clients migrated to GraphQL
          - to: gradual
            when: Issues found, need to rollback and iterate

      - id: deprecation
        title: Deprecate REST API
        docs:
          - ./migration/deprecation-plan.md
```
````

### 6. Error Handling and Validation (Missing)

Add section:

```markdown
### Error Handling

#### Parse Errors

**Invalid format version:**
````
$ signposts Error: Unsupported format version ‘SP/1.0’ in signposts.yml Supported:
SP/0.1

Upgrade signposts CLI or migrate: signposts migrate --from SP/1.0
````

**Invalid docspec prefix:**
```yaml
# signposts.yml
paths:
  - id: setup
    docs:
      - docs/setup.md  # ❌ No recognized prefix

# Error:
Error: Invalid docspec 'docs/setup.md' in path 'setup'
  Docspecs must start with: ./ (file), ref# (ref), github: (remote), or https:// (URL)

  Did you mean: ./docs/setup.md
````

**Duplicate path IDs:**
```yaml
areas:
  - id: getting-started
    paths:
      - id: setup
        ...
  - id: advanced
    paths:
      - id: setup  # ❌ Duplicate

# Error:
Error: Duplicate path ID 'setup'
  First defined: area 'getting-started'
  Duplicate in: area 'advanced'

  Path IDs must be globally unique.
```

**Invalid route target:**
```yaml
routes:
  - to: nonexistent-path  # ❌ Path doesn't exist

# Error:
Error: Invalid route in path 'research'
  Target path 'nonexistent-path' not found

  Available paths: assess, improve-coverage, plan, setup, ...
```

#### Runtime Warnings

**Missing file (warning, not error):**
```
$ signposts path setup
⚠️  File not found: ./docs/setup.md
   Referenced in: path 'setup', docs[0]

   This is expected if docs are in a separate repository.
   Path display will continue without content preview.
```

**Progress file mismatch:**
```
$ signposts where
⚠️  Progress file references signposts.yml at old location:
    Expected: repos/playbook/signposts.yml
    Found in progress.yml: ../old-path/signposts.yml

    Update progress.yml or run: signposts fix-progress
```

**Orphaned progress:**
```
$ signposts status
⚠️  Progress tracked for paths no longer in signposts.yml:
    - old-experimental-path (completed)
    - removed-step (in_progress)

    These will be preserved in progress.yml but won't affect counts.
```
````

### 7. Advanced Features (Expand)

#### 7.1 Workflow-Level Routes (Underspecified)

The design mentions workflow-level routes but doesn't show them in action. Add example:

```markdown
### Area-Level Routes: Decision Trees

Areas can act as decision points with routes:

```yaml
areas:
  - id: start
    title: Start Here
    description: Determine your path
    docs:
      - ref#getting-started-guide
    routes:
      - to: fresh-install/prerequisites
        when: New installation
      - to: upgrade/compatibility-check
        when: Upgrading from v1.x
      - to: troubleshooting/diagnostics
        when: Existing installation has issues

  - id: fresh-install
    title: Fresh Installation
    paths:
      - id: prerequisites
        ...

  - id: upgrade
    title: Upgrade Path
    paths:
      - id: compatibility-check
        ...

  - id: troubleshooting
    title: Troubleshooting
    paths:
      - id: diagnostics
        ...
````

When an agent encounters an area with routes:
```bash
$ signposts
Currently at: start

Routes available:
  1. fresh-install/prerequisites (when: New installation)
  2. upgrade/compatibility-check (when: Upgrading from v1.x)
  3. troubleshooting/diagnostics (when: Existing installation has issues)

# Agent can ask user or infer from context, then:
$ signposts start fresh-install/prerequisites
```
````

#### 7.2 Extra Steps (Underspecified)

Expand the `extra_steps` feature:

```markdown
### Extra Steps: Project-Specific Additions

During work, you may discover steps not in the original signposts.yml:

```bash
# While working on 'research' path, you need to evaluate a library not mentioned
$ signposts add-step research "./notes/tokio-vs-async-std-comparison.md"
Added extra step to path 'research'

# This updates progress.yml:
```yaml
paths:
  research:
    status: in_progress
    steps:
      ./reference/mapping-reference.md: completed
      ./reference/playbook.md#library-evaluation: in_progress
    extra_steps:
      - ./notes/tokio-vs-async-std-comparison.md: in_progress
````

Extra steps:
- Tracked in progress.yml only (not in signposts.yml)
- Preserved across signposts.yml updates
- Shown in `signposts status` with “(extra)” indicator
- Useful for project-specific research, spikes, or discovered work

When to use:
- ✓ Project-specific investigations
- ✓ One-off analyses not applicable to all users of this signpost
- ✗ Steps that should be in signposts.yml for everyone (contribute back!)
```
```

### 8. JSON Output (Missing)

The design mentions `--json` but doesn’t show what it looks like:

````markdown
### JSON Output Format

All commands support `--json` for agent consumption:

```bash
$ signposts status --json
{
  "format": "SP/0.1",
  "name": "Python to Rust Porting Playbook",
  "progress_file": "progress.yml",
  "current_path": "research",
  "current_area": "core-area",
  "started": "2026-02-17",
  "summary": {
    "total_paths": 9,
    "completed": 1,
    "in_progress": 1,
    "not_started": 7,
    "skipped": 0
  },
  "workflows": [
    {
      "id": "getting-started",
      "title": "Getting Started",
      "paths": [
        {
          "id": "assess",
          "title": "Assess",
          "status": "completed",
          "steps_total": 3,
          "steps_completed": 3,
          "started": "2026-02-17",
          "completed": "2026-02-17"
        },
        {
          "id": "improve-coverage",
          "title": "Improve Test Coverage",
          "status": "not_started"
        }
      ]
    },
    {
      "id": "core-area",
      "title": "Core Area",
      "paths": [
        {
          "id": "research",
          "title": "Research",
          "status": "in_progress",
          "steps_total": 3,
          "steps_completed": 1,
          "extra_steps": 1,
          "started": "2026-02-17"
        }
      ]
    }
  ]
}
````

```bash
$ signposts where --json
{
  "current_path": {
    "id": "research",
    "title": "Research",
    "description": "Evaluate Rust library candidates with real inputs",
    "area": "core-area",
    "status": "in_progress"
  },
  "steps": [
    {
      "docspec": "./reference/mapping-reference.md",
      "title": "Python-to-Rust mapping reference",
      "status": "completed"
    },
    {
      "docspec": "./reference/playbook.md#library-evaluation",
      "title": "Library evaluation",
      "status": "in_progress",
      "current": true
    },
    {
      "docspec": "./reference/playbook.md#library-decisions",
      "title": "Library decisions",
      "status": "not_started"
    }
  ],
  "routes": [
    {
      "to": "plan",
      "area": "core-area"
    },
    {
      "to": "assess",
      "area": "getting-started",
      "when": "Risk too high, reconsider scope"
    }
  ]
}
```
````

### 9. Implementation Notes (Add)

The implementation plan is good but add practical notes:

```markdown
### Implementation Notes

#### Locspec Extraction

**Decision: Print whole file with section marker**

When a step references `file.md#section-name`, extracting just that section is fragile:
- Heading text might change
- Section boundaries aren't always clear (when does a section end?)
- Nested headings complicate extraction

**Implementation:**
```bash
$ signposts step research "./playbook.md#library-evaluation"

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
File: ./reference/python-to-rust-playbook.md
Section: #library-evaluation
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

[full file content]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
👉 Focus on section: Library Evaluation (line 245)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
````

Future: Could add `--section-only` flag that attempts extraction but warns about
fragility.

#### Progress File Idempotency

**Design goal:** Running the same command twice has no effect.

```bash
# First time
$ signposts done assess "./playbook.md#step1"
✓ Step completed

# Second time (idempotent)
$ signposts done assess "./playbook.md#step1"
✓ Step already completed (no change)
```

This allows agents to be defensive without side effects.

#### Concurrency and File Locking

**Issue:** Multiple agents modifying progress.yml simultaneously

**Solution:** Advisory file locking
```python
import fcntl

with open('progress.yml', 'r+') as f:
    fcntl.flock(f, fcntl.LOCK_EX)  # Exclusive lock
    progress = yaml.safe_load(f)
    # ... modify progress ...
    f.seek(0)
    f.truncate()
    yaml.dump(progress, f)
    fcntl.flock(f, fcntl.LOCK_UN)  # Release
```

If lock can’t be acquired:
```
⚠️  progress.yml is locked (another process active)
   Waiting up to 5 seconds...

   If stuck, remove lock file: rm progress.yml.lock
```
````

### 10. Best Practices Guide (Missing)

Add section on authoring good signposts:

```markdown
### Best Practices for Authoring Signposts

#### Area Organization

**✓ Group by concern, not chronology**
```yaml
# Good
areas:
  - id: getting-started   # Prerequisites and setup
  - id: core-area     # Main development cycle
  - id: reference         # Look-up material
  - id: troubleshooting   # Problem-solving

# Avoid
areas:
  - id: week-1           # Time-based grouping is too rigid
  - id: week-2
````

#### Path Granularity

**Paths should be:**
- Completable in one session (2-4 hours)
- Focused on a single outcome
- 3-7 steps (not too granular, not too coarse)

```yaml
# Good: Focused path
- id: setup-ci
  title: Set Up CI/CD
  steps:
    - ./docs/ci.md#github-actions
    - ./docs/ci.md#test-workflow
    - ./docs/ci.md#deploy-workflow

# Too coarse: Multiple concerns
- id: setup-everything
  title: Complete Project Setup
  steps:
    - ./docs/tools.md
    - ./docs/ci.md
    - ./docs/deploy.md
    - ./docs/monitoring.md  # Too much for one path

# Too granular: Tiny steps
- id: install-tool-a
  steps:
    - ./docs/tool-a.md
- id: install-tool-b
  steps:
    - ./docs/tool-b.md
# Should be combined into "install-dev-tools"
```

#### Routes: When to Use

**Use routes for:**
- ✓ Conditional branching (different paths for different scenarios)
- ✓ Error recovery (problem found → go back to fix)
- ✓ Cycles (research → implement → discover gap → research)

**Don’t use routes for:**
- ✗ Sequential workflow (just list paths in order)
- ✗ Enforcing prerequisites (routes are guidance, not gates)

```yaml
# Good: Conditional routing
routes:
  - to: quick-start
    when: Small project (< 1000 LOC)
  - to: full-setup
    when: Large project or monorepo

# Unnecessary: Just use path order
routes:
  - to: step-2  # If this is always next, just list step-2 as the next path
```

#### Steps vs Docs

**`docs`** — Background reading, reference material **`steps`** — Checklist of tasks to
complete

```yaml
# Good: Docs for context, steps for action
- id: security-review
  docs:
    - ./security/owasp-top-10.md      # Background
    - ./security/checklist-guide.md   # How to use checklist
  steps:
    - ./security/checklist.md#authentication
    - ./security/checklist.md#authorization
    - ./security/checklist.md#data-validation

# Avoid: Duplicating same content in both
- id: setup
  docs:
    - ./setup.md
  steps:
    - ./setup.md  # Redundant if docs already cover it
```

#### Refs: When to Use

Use `refs` for:
- ✓ Decision criteria (short checklists embedded in signpost structure)
- ✓ Transitional guidance between paths
- ✓ Content specific to this signpost’s navigation (not general docs)

Keep in separate files:
- ✗ Long-form documentation
- ✗ Content that should be accessible outside the signpost
- ✗ Content that might be referenced elsewhere

```yaml
# Good use of ref
refs:
  - id: readiness-check
    title: Ready to Proceed?
    body: |
      Before starting implementation, confirm:
      - [ ] Design reviewed by team
      - [ ] Dependencies approved
      - [ ] Test plan drafted

# Bad: Should be a file
refs:
  - id: entire-coding-standards
    title: Coding Standards
    body: |
      [5 pages of coding standards...]
      # Put this in ./docs/coding-standards.md instead
```
```

## Summary of Recommendations

### Critical Additions
1. ✅ **More diverse examples** (onboarding, security, API migration) — see Section 5
2. ✅ **Format evolution and migration** — see Section 2.1
3. ✅ **Collaboration and conflict handling** — see Section 3.1
4. ✅ **Error handling examples** — see Section 6
5. ✅ **JSON output format** — see Section 8

### Important Enhancements
6. ✅ **CLI invocation examples** — see Section 4
7. ✅ **Area-level routes in action** — see Section 7.1
8. ✅ **Extra steps expansion** — see Section 7.2
9. ✅ **Implementation notes** — see Section 9
10. ✅ **Authoring best practices** — see Section 10

### Nice to Have
11. **Visual diagrams** — Graph structure, state transitions
12. **Migration guides** — From other formats (Runme, Jupyter Book)
13. **Agent prompt templates** — "How an agent should use signposts"
14. **Performance considerations** — Large signpost files (100+ paths)
15. **Testing examples** — Golden tests for CLI output

## Open Questions (Additional)

Beyond the original open questions:

1. **Internationalization:** Should docspecs support language variants? E.g., `./docs/setup.md?lang=es`
2. **Conditional steps:** Should steps have `when` fields like routes?
3. **Time estimates:** Should paths/steps have optional `duration` hints?
4. **Dependencies between steps:** Should steps be able to declare prerequisites within a path?
5. **Archived paths:** How to mark paths as deprecated/obsolete without removing them?
6. **Templating:** Should signposts support variables? E.g., `./docs/${PROJECT_TYPE}/setup.md`
7. **Includes:** Should signposts.yml support including fragments from other files?

## Conclusion

The signposts design is strong and ready for implementation with the additions outlined above. The core concepts are sound, the schema is well-thought-out, and the use cases are clear.

**Recommended next steps:**
1. Integrate feedback from this review (Sections 2-10)
2. Prototype Phase 1 with minimal example
3. Test with non-porting areas (onboarding or security)
4. Iterate based on real usage

The design will benefit most from:
- **Concrete examples** across diverse domains
- **Edge case documentation** (errors, conflicts, evolution)
- **Practical implementation notes** (locking, idempotency, performance)
```
