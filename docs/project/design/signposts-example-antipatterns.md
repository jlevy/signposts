# Signposts Example: Code Antipattern Detection Workflow

**Date:** 2026-02-17

**Status:** Draft / Exploratory

## Overview

This document explores how signposts could map a **code antipattern detection and
cleanup workflow**, using the ai-trade-arena antipatterns document as a reference case.

This differs from the porting playbook example in several key ways:
- **Non-linear workflow**: You don’t go through antipatterns in strict order
- **Detection + fixing**: Each antipattern has a detect → triage → fix → verify cycle
- **Progress tracking complexity**: Need to track both “which antipatterns checked” and
  “how many violations remaining per antipattern”
- **Context-sensitive routing**: What you check next depends on what you found, priority
  level, and scope

## Structure Comparison

### Porting Playbook (Linear Workflow)

```
Assess → Research → Plan → Setup → Port → Fix → Finalize → Sync
         ↑_______________|  (loop back if library gap found)
```

### Antipattern Detection (Catalog + Procedural)

```
Choose scope → Run detection → Triage results → Apply fixes → Verify
                     ↓              ↓              ↓
              [20 antipatterns]  [false positives]  [run tests]
                   ↑_________________|
              (re-check after fixes)
```

## Proposed Signpost Structure

### Areas

#### 1. Start Area

Determine cleanup scope and priority:
- **Pre-commit check** (5 min, high-priority only)
- **Pre-release audit** (30 min, high + medium priority)
- **Full cleanup** (hours, all antipatterns systematically)
- **Targeted fix** (specific antipattern or file)

#### 2. Detection Playbook Area

The meta-workflow for systematic detection:
- **Run automated checks** (step through grep commands)
- **Triage results** (identify false positives vs real violations)
- **Apply fixes** (import utilities, replace patterns)
- **Verify changes** (typecheck, tests)
- **Document new antipatterns** (if found during work)

#### 3. High-Priority Antipatterns Area (P0)

Critical issues that almost always need fixing:
- **Inline error extraction**
- **Fire-and-forget promises**
- **Missing Node.js imports** (dxterm-cli)
- **Silent catch blocks** (dxterm-cli)
- **Error message swallowing** (dxterm-cli)

#### 4. Medium-Priority Antipatterns Area (P1)

Review in context, fix if clearly wrong:
- **Inline badge colors**
- **Ad-hoc error categorization**
- **Hardcoded ANSI codes**
- **Console errors without events**
- **Non-atomic file writes**

#### 5. Low-Priority Antipatterns Area (P2)

Spot check, cleanup during refactoring:
- **Magic numbers**
- **LLM provider config duplication**
- **Trivial test bloat**

#### 6. Systematic Migration Antipatterns Area

Large-scale cleanups requiring coordination:
- **Uncontrolled current time** (~150 files)
- **Ad hoc date/time usage** (~40 files)
- **`as any` casts** (38 files, many legitimate)
- **Inconsistent Node.js imports**

#### 7. Reference Area

Individual antipattern documentation:
- Each antipattern as a path with:
  - Description + examples (BAD/GOOD)
  - Detection command
  - Fix procedure
  - Files typically affected
  - Legitimate exceptions

### Example Path: Inline Error Extraction

```yaml
- id: inline-error-extraction
  title: Inline Error Extraction
  description: Detect and fix duplicated error message extraction logic
  docs:
    - ./reference/antipatterns/inline-error-extraction.md
  steps:
    - ./reference/antipatterns/inline-error-extraction.md#detection
    - ./reference/antipatterns/inline-error-extraction.md#triage
    - ./reference/antipatterns/inline-error-extraction.md#apply-fix
    - ./reference/antipatterns/inline-error-extraction.md#verify
  routes:
    - to: ad-hoc-error-categorization
      when: Found error.message.includes() patterns during fix
    - to: verify-changes
      when: All fixes applied
```

### Example Path: Detection Playbook

```yaml
- id: detection-playbook
  title: Systematic Detection Playbook
  description: Step-by-step process for detecting and fixing antipatterns
  docs:
    - ./reference/antipattern-detection-playbook.md
  steps:
    - ./reference/antipattern-detection-playbook.md#step-1-run-automated-checks
    - ./reference/antipattern-detection-playbook.md#step-2-triage-results
    - ./reference/antipattern-detection-playbook.md#step-3-apply-fixes
    - ./reference/antipattern-detection-playbook.md#step-4-verify-changes
    - ./reference/antipattern-detection-playbook.md#step-5-document-new-antipatterns
  routes:
    - to: high-priority/inline-error-extraction
      when: Found error extraction violations
    - to: high-priority/fire-and-forget
      when: Found unawaited promises
```

## Progress Tracking Challenges

### Per-Antipattern State

Unlike the porting playbook where paths are “complete” or “in progress”, antipattern
cleanup has more nuance:

```yaml
paths:
  inline-error-extraction:
    status: in_progress
    started: 2026-02-17
    steps:
      ./reference/antipatterns/inline-error-extraction.md#detection: completed
      ./reference/antipatterns/inline-error-extraction.md#triage: completed
      ./reference/antipatterns/inline-error-extraction.md#apply-fix: in_progress
      ./reference/antipatterns/inline-error-extraction.md#verify: not_started
    notes: |
      Found 18 violations across 12 files.
      Fixed 8 violations in 4 files (tools/, exec/).
      Remaining: 10 violations in migration files (low priority).
    extra_steps:
      - ./src/tools/errorHandler.ts: completed
      - ./src/exec/actionRunner.ts: completed
```

### Project-Specific Violation Tracking

The `notes` field captures counts, but could we model violation tracking more
explicitly?

**Possible extension to progress.yml:**
```yaml
paths:
  inline-error-extraction:
    status: in_progress
    violations:
      total: 18
      fixed: 8
      remaining: 10
      files_checked: 12
      files_fixed: 4
    notes: "Migration files remaining (low priority)"
```

This would allow:
- `signposts status` to show violation counts across all antipatterns
- `signposts stats` to aggregate totals (e.g., “42 violations remaining across 8
  antipatterns”)

### Audit Log / Run History

The antipatterns doc mentions “we might want to have a log of the runs we’ve been
through on a codebase to find all the antipatterns.”

**Possible extension: `audit_log` in progress.yml**
```yaml
signposts: repos/code-quality/antipatterns-signposts.yml
started: 2026-02-10
current_path: inline-error-extraction

audit_log:
  - date: 2026-02-10
    type: pre-commit
    paths_checked:
      - inline-error-extraction
      - fire-and-forget
    violations_found: 3
    violations_fixed: 3
    notes: "Quick pre-commit check before release"

  - date: 2026-02-15
    type: full-audit
    paths_checked:
      - inline-error-extraction
      - inline-badge-colors
      - ad-hoc-error-categorization
      - magic-numbers
    violations_found: 42
    violations_fixed: 28
    notes: "Full cleanup pass before v2.0 release"
```

This enables:
- **Historical tracking**: “When did we last check X?”
- **Progress over time**: “We fixed 28/42 violations last week”
- **Audit trail**: “What was found in the pre-release audit?”

## Routing Patterns

### Priority-Based Routing

```yaml
- id: start
  title: Choose Cleanup Scope
  description: Determine priority level and scope
  routes:
    - to: high-priority/inline-error-extraction
      when: Pre-commit check (5 min)
    - to: detection-playbook/run-automated-checks
      when: Pre-release audit (30 min)
    - to: reference/all-antipatterns
      when: Full cleanup (hours)
```

### Context-Sensitive Routing

After fixing one antipattern, route to related ones:
```yaml
- id: inline-error-extraction
  routes:
    - to: ad-hoc-error-categorization
      when: Found error.message.includes() patterns
    - to: insufficient-error-context
      when: Working in Node.js action files
    - to: console-errors-without-events
      when: Found console.error in workflow files
```

### Scope-Based Routing

```yaml
- id: choose-scope
  routes:
    - to: web-antipatterns/inline-badge-colors
      when: Working on web/ codebase
    - to: cli-antipatterns/missing-nodejs-imports
      when: Working on dxterm-cli/ codebase
    - to: all-antipatterns
      when: Full codebase cleanup
```

## Document Structure

Unlike the porting playbook where docs are **reference materials** (markdown guides),
antipattern docs have a dual nature:
1. **Procedural** (how to detect, triage, fix)
2. **Catalog** (what is the antipattern, why it matters, examples)

### Option 1: Split Documents

```
reference/antipatterns/
  inline-error-extraction.md          # Catalog (what/why/examples)
  inline-error-extraction-procedure.md # How to detect/fix
```

### Option 2: Unified Document with Locspecs

```
reference/antipatterns/inline-error-extraction.md
  #description        → What is this antipattern?
  #examples           → BAD/GOOD code samples
  #why-it-matters     → Rationale
  #detection          → Grep command to find violations
  #triage             → How to identify false positives
  #apply-fix          → Step-by-step fix procedure
  #verify             → How to verify changes
  #exceptions         → Known legitimate uses
```

Signposts steps would use locspecs:
```yaml
steps:
  - ./reference/antipatterns/inline-error-extraction.md#detection
  - ./reference/antipatterns/inline-error-extraction.md#triage
  - ./reference/antipatterns/inline-error-extraction.md#apply-fix
  - ./reference/antipatterns/inline-error-extraction.md#verify
```

**Recommendation**: Option 2 (unified with locspecs) keeps related content together and
leverages signposts’ locspec navigation.

## Differences from Porting Playbook

| Aspect | Porting Playbook | Antipattern Detection |
| --- | --- | --- |
| **Workflow shape** | Linear with cycles | Catalog + meta-procedure |
| **Path completion** | Binary (done/not done) | Graduated (N violations remaining) |
| **Routing logic** | Phase progression | Priority, context, scope |
| **Progress metrics** | Paths complete | Violations found/fixed |
| **Documentation** | Reference guides | Procedural + catalog |
| **Revisiting** | Sync phase (periodic) | Re-check after fixes (iterative) |
| **History tracking** | Not needed | Audit log useful |

## Open Questions

### 1. Should antipatterns be paths or workflows?

**Option A**: Each antipattern is a **path** within a workflow (grouped by priority).
```
High-Priority Area
  → inline-error-extraction (path)
  → fire-and-forget (path)
```

**Option B**: Each antipattern is a **workflow** with detection/fix/verify paths.
```
Inline Error Extraction (workflow)
  → detect (path)
  → triage (path)
  → fix (path)
  → verify (path)
```

**Recommendation**: Option A. Antipatterns are conceptual units, not multi-stage
processes. The detect→triage→fix→verify cycle is captured in **steps** within each path.

### 2. How to model “check all high-priority antipatterns”?

Could use a **composite path** that references other paths:
```yaml
- id: pre-commit-check
  title: Pre-Commit Quick Check
  description: Run high-priority antipattern checks (5 min)
  docs:
    - ./playbooks/pre-commit-antipattern-check.md
  steps:
    - ref#run-inline-error-extraction-check
    - ref#run-fire-and-forget-check
    - ref#run-missing-imports-check
  routes:
    - to: high-priority/inline-error-extraction
      when: Found violations in error extraction
    - to: high-priority/fire-and-forget
      when: Found unawaited promises
```

### 3. How to track violations per antipattern?

**Extension to progress.yml schema**:
```yaml
paths:
  <path-id>:
    violations:
      total: number
      fixed: number
      remaining: number
      files_checked: number
      files_fixed: number
```

This would enable:
- `signposts stats` → violation counts
- `signposts status` → show remaining violations per path

### 4. Should there be a “legitimate exceptions” registry?

Could track false positives discovered during triage:
```yaml
paths:
  inline-error-extraction:
    exceptions:
      - file: "src/execWithRetryRateLimit.ts"
        line: 145
        reason: "Test environment detection, not error categorization"
      - file: "src/agentExecutionActions.ts"
        line: 350
        reason: "Sync setTimeout callback, void is intentional"
```

This would:
- Document why certain matches are skipped
- Avoid re-triaging the same false positives
- Provide context for future reviewers

## Potential Signposts Extensions

Based on this use case, signposts might benefit from:

### 1. Violation Tracking Schema

Extend `progress.yml` to support antipattern-specific metrics:
```yaml
paths:
  <path-id>:
    violations:
      total: number
      fixed: number
      remaining: number
```

### 2. Audit Log

Track historical cleanup runs:
```yaml
audit_log:
  - date: ISO date
    type: pre-commit | pre-release | full-audit | targeted
    paths_checked: [path-id, ...]
    violations_found: number
    violations_fixed: number
    notes: string
```

### 3. Exception Registry

Track known false positives:
```yaml
paths:
  <path-id>:
    exceptions:
      - file: path
        line: number (optional)
        pattern: string (optional)
        reason: string
```

### 4. Composite Paths

Allow paths to reference other paths for “run all X” workflows:
```yaml
- id: pre-commit-check
  type: composite  # New field
  references:
    - high-priority/inline-error-extraction
    - high-priority/fire-and-forget
  docs:
    - ./playbooks/pre-commit-check.md
```

## Draft `signposts.yml` for Antipatterns

```yaml
format: "SP/0.1"
name: Code Antipattern Detection & Cleanup
start: choose-scope

areas:
  - id: choose-scope
    title: Choose Cleanup Scope
    description: Determine what kind of cleanup you're doing
    docs:
      - ref#cleanup-scope-guide
    routes:
      - to: high-priority/inline-error-extraction
        when: Pre-commit check (5 min, high-priority only)
      - to: detection-playbook/run-checks
        when: Pre-release audit (30 min, high + medium priority)
      - to: reference/antipattern-catalog
        when: Browse all antipatterns

  - id: detection-playbook
    title: Detection Playbook
    description: Systematic detection and fixing procedure
    paths:
      - id: run-checks
        title: Run Automated Checks
        description: Execute grep commands for antipattern detection
        docs:
          - ./playbooks/detection-playbook.md
        steps:
          - ./playbooks/detection-playbook.md#run-high-priority-checks
          - ./playbooks/detection-playbook.md#run-medium-priority-checks
          - ./playbooks/detection-playbook.md#run-low-priority-checks
        routes:
          - to: triage-results

      - id: triage-results
        title: Triage Results
        description: Identify false positives vs real violations
        docs:
          - ./playbooks/detection-playbook.md#triage
        steps:
          - ./playbooks/detection-playbook.md#check-legitimate-exceptions
          - ./playbooks/detection-playbook.md#classify-fixable-now-vs-later
        routes:
          - to: apply-fixes
            when: Found fixable violations
          - to: document-exceptions
            when: Found new false positive patterns

      - id: apply-fixes
        title: Apply Fixes
        description: Fix violations and verify changes
        docs:
          - ./playbooks/detection-playbook.md#apply-fixes
        steps:
          - ./playbooks/detection-playbook.md#import-utilities
          - ./playbooks/detection-playbook.md#replace-patterns
          - ./playbooks/detection-playbook.md#run-typecheck
          - ./playbooks/detection-playbook.md#run-tests
        routes:
          - to: run-checks
            when: Re-check after fixes to verify

  - id: high-priority
    title: High-Priority Antipatterns (P0)
    description: Critical issues that almost always need fixing
    paths:
      - id: inline-error-extraction
        title: Inline Error Extraction
        description: Duplicated error message extraction logic
        docs:
          - ./antipatterns/inline-error-extraction.md
        steps:
          - ./antipatterns/inline-error-extraction.md#detection
          - ./antipatterns/inline-error-extraction.md#triage
          - ./antipatterns/inline-error-extraction.md#apply-fix
          - ./antipatterns/inline-error-extraction.md#verify
        routes:
          - to: ad-hoc-error-categorization
            when: Found error.message.includes() patterns

      - id: fire-and-forget
        title: Fire-and-Forget Promises
        description: Unawaited promises in Convex actions
        docs:
          - ./antipatterns/fire-and-forget-promises.md
        steps:
          - ./antipatterns/fire-and-forget-promises.md#detection
          - ./antipatterns/fire-and-forget-promises.md#triage
          - ./antipatterns/fire-and-forget-promises.md#apply-fix
          - ./antipatterns/fire-and-forget-promises.md#verify

      - id: missing-nodejs-imports
        title: Missing Node.js Imports
        description: Using Node.js functions without importing them (dxterm-cli)
        docs:
          - ./antipatterns/missing-nodejs-imports.md
        steps:
          - ./antipatterns/missing-nodejs-imports.md#detection
          - ./antipatterns/missing-nodejs-imports.md#add-imports
          - ./antipatterns/missing-nodejs-imports.md#verify

  - id: medium-priority
    title: Medium-Priority Antipatterns (P1)
    description: Review in context, fix if clearly wrong
    paths:
      - id: inline-badge-colors
        title: Inline Badge Colors
        description: Hardcoded Tailwind color classes for status badges
        docs:
          - ./antipatterns/inline-badge-colors.md
        steps:
          - ./antipatterns/inline-badge-colors.md#detection
          - ./antipatterns/inline-badge-colors.md#triage
          - ./antipatterns/inline-badge-colors.md#apply-fix

      - id: ad-hoc-error-categorization
        title: Ad-hoc Error Categorization
        description: Classifying errors inline instead of using centralized system
        docs:
          - ./antipatterns/ad-hoc-error-categorization.md
        steps:
          - ./antipatterns/ad-hoc-error-categorization.md#detection
          - ./antipatterns/ad-hoc-error-categorization.md#use-categorize-error

  - id: systematic-migrations
    title: Systematic Migration Antipatterns
    description: Large-scale cleanups requiring coordination
    paths:
      - id: uncontrolled-current-time
        title: Uncontrolled Current Time Access
        description: Using Date.now() or new Date() directly (~150 files)
        docs:
          - ./antipatterns/uncontrolled-current-time.md
        steps:
          - ./antipatterns/uncontrolled-current-time.md#detection
          - ./antipatterns/uncontrolled-current-time.md#migration-plan
          - ./antipatterns/uncontrolled-current-time.md#add-eslint-rule
        routes:
          - to: ad-hoc-date-time
            when: Found inline date formatting during migration

      - id: ad-hoc-date-time
        title: Ad Hoc Date/Time Usage
        description: Inline .toISOString().split('T')[0] instead of utilities
        docs:
          - ./antipatterns/ad-hoc-date-time.md
        steps:
          - ./antipatterns/ad-hoc-date-time.md#detection
          - ./antipatterns/ad-hoc-date-time.md#use-time-utils

  - id: reference
    title: Reference
    description: Antipattern catalog and guides
    paths:
      - id: antipattern-catalog
        title: All Antipatterns
        description: Complete catalog of antipatterns with examples
        docs:
          - ./reference/antipattern-catalog.md

      - id: legitimate-exceptions
        title: Legitimate Exceptions
        description: Known false positives and when patterns are acceptable
        docs:
          - ./reference/legitimate-exceptions.md

      - id: fixes-history
        title: Fixes History
        description: Historical record of antipattern cleanups
        docs:
          - ./reference/fixes-history.md

refs:
  - id: cleanup-scope-guide
    title: Choosing Your Cleanup Scope
    body: |
      **Pre-commit check (5 min)**
      - Run high-priority antipatterns only
      - Focus on code you just changed
      - Use quick single-command check

      **Pre-release audit (30 min)**
      - Run high + medium priority antipatterns
      - Review results in context
      - Fix clear violations, document ambiguous cases

      **Full cleanup (hours)**
      - Systematic detection playbook
      - Work through all antipatterns
      - Update backlog and history

      Use `signposts where` to see your current position.
```

## Implementation Notes

### CLI Commands

```bash
# Start a cleanup session
signposts start choose-scope

# Navigate to a specific antipattern
signposts path inline-error-extraction

# Mark detection step done, record violations found
signposts done inline-error-extraction \
  "./antipatterns/inline-error-extraction.md#detection" \
  --violations-found 18

# View progress with violation counts
signposts status
# Output:
# Progress: 3/20 antipatterns checked, 42 violations remaining
#
#   High-Priority Antipatterns
#     inline-error-extraction  [in_progress]  8/18 violations fixed
#     fire-and-forget          [completed]    3/3 violations fixed
#     missing-nodejs-imports   [not started]

# Log an audit run
signposts audit --type pre-release \
  --paths inline-error-extraction,fire-and-forget \
  --violations-found 21 --violations-fixed 11 \
  --notes "Pre-v2.0 release cleanup"

# Add an exception (false positive)
signposts exception inline-error-extraction \
  --file src/execWithRetryRateLimit.ts --line 145 \
  --reason "Test environment detection, not error categorization"
```

## Conclusion

This exploration reveals that signposts can model antipattern detection workflows, but
may benefit from extensions:

1. **Violation tracking**: Count found/fixed violations per path
2. **Audit log**: Historical record of cleanup runs
3. **Exception registry**: Document false positives
4. **Composite paths**: “Run all X” meta-paths

The core signpost concepts (areas, paths, steps, routes, progress) apply well, but the
**metrics** and **iteration patterns** differ from linear workflows like porting.

Next steps:
- Validate whether these extensions are worth the complexity
- Consider whether signposts should stay simple (structure + navigation) and leave
  metrics to other tools
- Explore other use cases (onboarding, security assessments, migration checklists) to
  see if similar patterns emerge
