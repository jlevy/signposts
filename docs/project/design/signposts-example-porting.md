# Signposts Example: Python-to-Rust Porting Playbook

**Date:** 2026-02-17

**Status:** Draft / Comprehensive Example

## Overview

This document maps the real rust-porting-playbook repository to the signposts format,
showing how a complex, multi-phase workflow with reference materials, guidelines,
templates, and case studies would be structured.

## Source Material

Based on: `/Users/levy/wrk/github/flowmark-rs/repos/rust-porting-playbook/`

**Structure:**
- **reference/** - Comprehensive playbook, mapping reference, best practices, checklists
- **guidelines/** - Compact rules for AI agent context (~2-3k tokens each)
- **case-studies/** - Real-world porting examples with decisions and metrics

**The Workflow:**
- 8 phases: Assess → Research → Plan → Setup → Port → Fix → Finalize → Sync
- Test coverage preparatory workflow (6 phases)
- Ongoing sync process (for maintaining ports after initial completion)

## Signpost Structure

### Areas Overview

1. **Start** - Entry point, determine your scenario
2. **Prepare** - Test coverage improvement (before porting)
3. **Core Area** - The 8-phase porting process
4. **Reference** - Comprehensive reference documents
5. **Guidelines** - Compact agent-injectable rules
6. **Templates** - Checklists and observation templates
7. **Case Studies** - Real-world examples
8. **Meta** - Improving the playbook itself

### Routing Patterns

**Conditional entry:**
- Starting fresh port → Prepare (if coverage < 80%) or Core Area/Assess (if coverage
  good)
- Syncing existing port → Core Area/Sync
- Need test coverage help → Prepare or Reference/Test Coverage
- Just need library mapping → Reference/Mapping Reference

**Within core area:**
- Assessment shows low coverage → Prepare/Measure Coverage
- Research reveals library gaps → loop back to Research or escalate to human
- Port reveals new library differences → Fix phase
- Sync discovers breaking changes → may revisit Research or Plan

### Progress Tracking Patterns

**Per-phase state:**
- Assess: completed with coverage metrics, dependency risk table
- Research: in_progress with N/M libraries evaluated
- Port: in_progress with test count (45/111 tests passing)
- Fix: in_progress with workaround count (12 workarounds, 3 accepted differences)

**Project-specific additions:**
- Extra library evaluations beyond original plan
- Additional workarounds discovered
- New differences found during sync

## Example `signposts.yml`

```yaml
format: "SP/0.1"
name: Python-to-Rust Porting Playbook
start: start

areas:
  - id: start
    title: Start
    description: Determine your porting scenario
    docs:
      - ref#getting-started
    routes:
      - to: prepare/measure-coverage
        when: Starting fresh port with unknown test coverage
      - to: core-area/assess
        when: Starting fresh port with good test coverage (>80%)
      - to: core-area/sync
        when: Syncing existing port with Python upstream changes
      - to: reference/mapping-reference
        when: Just need Python-to-Rust type mappings
      - to: reference/test-coverage
        when: Just need test coverage guidance

  - id: prepare
    title: Prepare - Test Coverage
    description: Achieve high test coverage before porting begins
    paths:
      - id: measure-coverage
        title: Measure Current Coverage
        description: Understand current coverage and identify gaps
        docs:
          - ./reference/python-to-rust-test-coverage-playbook.md
        steps:
          - ./reference/python-to-rust-test-coverage-playbook.md#phase-1-measure-current-coverage
          - ./reference/python-to-rust-test-coverage-playbook.md#11-run-pytest-coverage
          - ./reference/python-to-rust-test-coverage-playbook.md#12-assess-coverage-by-module
          - ./reference/python-to-rust-test-coverage-playbook.md#13-identify-coverage-gaps
        routes:
          - to: expand-unit-tests
            when: Core modules below 80% coverage
          - to: core-area/assess
            when: Coverage meets thresholds (80%+ core, 90%+ public API)

      - id: expand-unit-tests
        title: Expand Unit Test Coverage
        description: Write tests for edge cases and uncovered code
        docs:
          - ./reference/python-to-rust-test-coverage-playbook.md
        steps:
          - ./reference/python-to-rust-test-coverage-playbook.md#phase-2-expand-unit-test-coverage
          - ./reference/python-to-rust-test-coverage-playbook.md#21-edge-case-tests
          - ./reference/python-to-rust-test-coverage-playbook.md#22-boundary-condition-tests
          - ./reference/python-to-rust-test-coverage-playbook.md#23-feature-interaction-tests
        routes:
          - to: golden-tests
          - to: measure-coverage
            when: Re-measure after adding tests

      - id: golden-tests
        title: Create Golden Tests
        description: Snapshot tests for CLI behavior and end-to-end workflows
        docs:
          - ./reference/python-to-rust-test-coverage-playbook.md
        steps:
          - ./reference/python-to-rust-test-coverage-playbook.md#phase-3-golden-tests
          - ./reference/python-to-rust-test-coverage-playbook.md#31-cli-snapshot-tests
          - ./reference/python-to-rust-test-coverage-playbook.md#32-end-to-end-workflow-tests
        routes:
          - to: measure-coverage
            when: Verify coverage improvement

  - id: core-area
    title: Core Area
    description: The 8-phase porting process
    paths:
      - id: assess
        title: Phase 1 - Assess
        description: Understand the project and decide if it's ready to port
        docs:
          - ./reference/python-to-rust-playbook.md#phase-1-assess-the-original-project
        steps:
          - ./reference/python-to-rust-playbook.md#11-measure-the-codebase
          - ./reference/python-to-rust-playbook.md#12-inventory-dependencies
          - ./reference/python-to-rust-playbook.md#13-assess-test-coverage
          - ./reference/python-to-rust-playbook.md#14-identify-areas-to-clarify
          - ./reference/python-to-rust-playbook.md#15-decision-is-this-project-ready-to-port
        routes:
          - to: research
            when: Project ready (>80% coverage, dependencies identified)
          - to: prepare/measure-coverage
            when: Coverage below thresholds
          - to: research
            when: Proceed with caution (some gaps but acceptable risk)

      - id: research
        title: Phase 2 - Research
        description: Evaluate Rust library candidates with real inputs
        docs:
          - ./reference/python-to-rust-playbook.md#phase-2-research-and-library-evaluation
          - ./reference/python-to-rust-mapping-reference.md
        steps:
          - ./reference/python-to-rust-mapping-reference.md
          - ./reference/python-to-rust-playbook.md#21-evaluate-candidates-for-every-high-risk-dependency
          - ./reference/python-to-rust-playbook.md#22-evaluation-criteria
          - ./reference/python-to-rust-playbook.md#23-critical-lesson-spec-compliance-is-a-false-signal
          - ./reference/python-to-rust-playbook.md#24-write-a-best-practices-survey
        routes:
          - to: plan
            when: All libraries evaluated and decisions made
          - to: assess
            when: Critical dependency has no viable Rust equivalent (reconsider scope)

      - id: plan
        title: Phase 3 - Plan
        description: Create concrete architecture and porting plan
        docs:
          - ./reference/python-to-rust-playbook.md#phase-3-plan-the-port
        steps:
          - ./reference/python-to-rust-playbook.md#31-define-the-architecture
          - ./reference/python-to-rust-playbook.md#32-create-a-feature-parity-matrix
          - ./reference/python-to-rust-playbook.md#33-plan-the-module-porting-order
          - ./reference/python-to-rust-playbook.md#34-define-acceptance-criteria
          - ./reference/python-to-rust-playbook.md#35-budget-for-workarounds
        routes:
          - to: setup

      - id: setup
        title: Phase 4 - Set Up
        description: Configure Cargo, CI, fixtures, and Python submodule
        docs:
          - ./reference/python-to-rust-playbook.md#phase-4-set-up-the-rust-project
          - ./guidelines/rust-project-setup.md
        steps:
          - ./reference/python-to-rust-playbook.md#41-create-the-project
          - ./reference/python-to-rust-playbook.md#42-configure-cargotoml
          - ./reference/python-to-rust-playbook.md#43-include-the-python-source-as-a-submodule
          - ./reference/python-to-rust-playbook.md#44-set-up-test-fixtures
          - ./reference/python-to-rust-playbook.md#45-set-up-ci
          - ./reference/python-to-rust-playbook.md#46-track-version-correspondence
        routes:
          - to: port

      - id: port
        title: Phase 5 - Port
        description: Port code module by module, tests first
        docs:
          - ./reference/python-to-rust-playbook.md#phase-5-port-the-code
          - ./reference/python-to-rust-porting-guide.md
          - ./guidelines/python-to-rust-porting-rules.md
        steps:
          - ./reference/python-to-rust-playbook.md#51-port-tests-first
          - ./reference/python-to-rust-playbook.md#52-per-module-workflow
          - ./reference/python-to-rust-playbook.md#53-maintain-traceability
          - ./reference/python-to-rust-playbook.md#54-key-pitfalls-to-watch-for
          - ./reference/python-to-rust-porting-guide.md#porting-order-leaf-to-root
        routes:
          - to: fix
            when: All modules ported, tests passing, ready for cross-validation
          - to: research
            when: Discovered library gap during porting (need different library)

      - id: fix
        title: Phase 6 - Fix
        description: Resolve library differences and behavioral mismatches
        docs:
          - ./reference/python-to-rust-playbook.md#phase-6-handle-library-differences
          - ./reference/python-to-rust-porting-guide.md#cross-validation
        steps:
          - ./reference/python-to-rust-playbook.md#61-run-cross-validation-and-categorize-failures
          - ./reference/python-to-rust-playbook.md#62-workaround-strategy
          - ./reference/python-to-rust-playbook.md#63-track-workarounds-systematically
          - ./reference/python-to-rust-playbook.md#64-handling-bugs-in-the-original
          - ./reference/python-to-rust-playbook.md#65-decision-point-library-switch
        routes:
          - to: finalize
            when: All differences resolved or documented
          - to: research
            when: Decided to switch libraries (>3 unfixable diffs)
          - to: port
            when: Found porting bugs that require module rework

      - id: finalize
        title: Phase 7 - Finalize
        description: Production-ready with CI, docs, and CLI parity
        docs:
          - ./reference/python-to-rust-playbook.md#phase-7-finalize-and-validate
          - ./reference/rust-cli-best-practices.md
          - ./reference/rust-code-review-checklist.md
        steps:
          - ./reference/python-to-rust-playbook.md#71-cross-validation-gate
          - ./reference/python-to-rust-playbook.md#72-cli-parity
          - ./reference/python-to-rust-playbook.md#73-ci-verification
          - ./reference/python-to-rust-playbook.md#74-documentation
          - ./reference/python-to-rust-playbook.md#75-release-configuration
          - ./reference/rust-code-review-checklist.md
        routes:
          - to: sync
            when: Port complete and released

      - id: sync
        title: Phase 8 - Sync
        description: Keep Rust port in sync with Python upstream changes
        docs:
          - ./reference/python-to-rust-playbook.md#phase-8-ongoing-synchronization
          - ./reference/port-checklist-update-template.md
        steps:
          - ./reference/port-checklist-update-template.md#1-update-python-submodule
          - ./reference/port-checklist-update-template.md#2-regenerate-test-fixtures
          - ./reference/port-checklist-update-template.md#3-run-cross-validation
          - ./reference/port-checklist-update-template.md#4-categorize-changes
          - ./reference/port-checklist-update-template.md#5-port-changes-to-rust
          - ./reference/port-checklist-update-template.md#6-update-version-correspondence
        routes:
          - to: fix
            when: New library differences found after sync
          - to: research
            when: Python added dependency with no Rust equivalent

  - id: reference
    title: Reference
    description: Comprehensive reference documents
    paths:
      - id: playbook
        title: Complete Playbook
        description: The full 8-phase process (primary document)
        docs:
          - ./reference/python-to-rust-playbook.md

      - id: mapping-reference
        title: Mapping Reference
        description: Python-to-Rust type/pattern/construct mapping
        docs:
          - ./reference/python-to-rust-mapping-reference.md

      - id: porting-guide
        title: Porting Guide
        description: Detailed methodology with pitfalls and automation
        docs:
          - ./reference/python-to-rust-porting-guide.md

      - id: test-coverage
        title: Test Coverage Playbook
        description: Pre-port test coverage strategy and tooling
        docs:
          - ./reference/python-to-rust-test-coverage-playbook.md

      - id: cli-best-practices
        title: Rust CLI Best Practices
        description: Modern Rust CLI project setup and tooling
        docs:
          - ./reference/rust-cli-best-practices.md

      - id: code-review
        title: Code Review Checklist
        description: Rust code review checklist for ports
        docs:
          - ./reference/rust-code-review-checklist.md

  - id: guidelines
    title: Guidelines
    description: Compact rules for AI agent context injection
    paths:
      - id: porting-rules
        title: Porting Rules
        description: Python-to-Rust porting rules and common pitfalls
        docs:
          - ./guidelines/python-to-rust-porting-rules.md

      - id: cli-porting
        title: CLI Porting
        description: CLI-specific porting (argparse to clap, exit codes)
        docs:
          - ./guidelines/python-to-rust-cli-porting.md

      - id: rust-general
        title: General Rust Rules
        description: General Rust coding rules (Edition 2024+)
        docs:
          - ./guidelines/rust-general-rules.md

      - id: rust-cli-patterns
        title: Rust CLI Patterns
        description: Rust CLI application patterns (clap, error handling)
        docs:
          - ./guidelines/rust-cli-app-patterns.md

      - id: rust-project-setup
        title: Rust Project Setup
        description: Cargo.toml, CI/CD, lint config, release workflow
        docs:
          - ./guidelines/rust-project-setup.md

      - id: test-coverage-guidelines
        title: Test Coverage for Porting
        description: Compact test coverage guidelines for agent context
        docs:
          - ./guidelines/test-coverage-for-porting.md

  - id: templates
    title: Templates
    description: Checklists and observation templates
    paths:
      - id: initial-checklist
        title: Initial Port Checklist
        description: 10-phase checklist template (copy and fill in)
        docs:
          - ./reference/port-checklist-initial-template.md

      - id: sync-checklist
        title: Sync Checklist
        description: Ongoing sync checklist template
        docs:
          - ./reference/port-checklist-update-template.md

      - id: observations-template
        title: Observations Template
        description: Template for recording observations during a port
        docs:
          - ./reference/case-study-observations-template.md

      - id: triage-template
        title: Improvement Triage Template
        description: Template for triaging observations into playbook changes
        docs:
          - ./reference/case-study-improvement-triage-template.md

  - id: case-studies
    title: Case Studies
    description: Real-world porting examples
    paths:
      - id: flowmark
        title: Flowmark Case Study
        description: Python Markdown formatter → Rust (2k lines, 14 workarounds)
        docs:
          - ./case-studies/flowmark/flowmark-port-analysis.md
          - ./case-studies/flowmark/flowmark-port-library-choices.md
          - ./case-studies/flowmark/flowmark-port-decision-log.md
          - ./case-studies/flowmark/flowmark-port-migration-plan.md
          - ./case-studies/flowmark/flowmark-port-cross-validation.md

  - id: meta
    title: Meta
    description: Improving the playbook itself
    paths:
      - id: improving-playbook
        title: Improving the Playbook
        description: Process for integrating case study feedback
        docs:
          - ./reference/meta-improving-this-playbook.md

refs:
  - id: getting-started
    title: Getting Started with Your Port
    body: |
      **What brings you here?**

      - **Starting a fresh port** → Check test coverage first
        - Coverage <80% on core modules? → Go to Prepare/Measure Coverage
        - Coverage good? → Go to Core Area/Assess

      - **Syncing an existing port with Python changes** → Go to Core Area/Sync

      - **Need library mapping help** → Go to Reference/Mapping Reference

      - **Need test coverage help** → Go to Reference/Test Coverage or Prepare workflow

      **Key insight:** Phases 5-6 (porting + fixing library differences) consume ~70% of
      total effort. Thorough library evaluation in Phase 2 is the single highest-leverage
      activity.

      Use `signposts where` to see your current position.
```

## Progress Tracking Examples

### Example 1: Fresh Port in Research Phase

```yaml
signposts: repos/rust-porting-playbook/signposts.yml
started: 2026-02-10
current_path: research

paths:
  assess:
    status: completed
    started: 2026-02-10
    completed: 2026-02-10
    steps:
      ./reference/python-to-rust-playbook.md#11-measure-the-codebase: completed
      ./reference/python-to-rust-playbook.md#12-inventory-dependencies: completed
      ./reference/python-to-rust-playbook.md#13-assess-test-coverage: completed
      ./reference/python-to-rust-playbook.md#14-identify-areas-to-clarify: completed
      ./reference/python-to-rust-playbook.md#15-decision-is-this-project-ready-to-port: completed
    notes: |
      Project: flowmark (Python Markdown formatter)
      Size: ~2,000 lines app + ~1,500 lines tests
      Coverage: 87% on core modules
      Dependencies: 3 high-risk (markdown parsing critical)

  research:
    status: in_progress
    started: 2026-02-10
    steps:
      ./reference/python-to-rust-mapping-reference.md: completed
      ./reference/python-to-rust-playbook.md#21-evaluate-candidates-for-every-high-risk-dependency: in_progress
      ./reference/python-to-rust-playbook.md#22-evaluation-criteria: not_started
    extra_steps:
      - ./case-studies/flowmark/flowmark-port-library-choices.md#comrak-evaluation: completed
      - ./case-studies/flowmark/flowmark-port-library-choices.md#pulldown-cmark-evaluation: in_progress
    notes: |
      Evaluating markdown parsers: comrak vs pulldown-cmark
      comrak: 5 behavioral diffs found on test fixtures
      pulldown-cmark: evaluation in progress
```

### Example 2: Port Phase with Test Progress

```yaml
paths:
  port:
    status: in_progress
    started: 2026-02-12
    steps:
      ./reference/python-to-rust-playbook.md#51-port-tests-first: in_progress
      ./reference/python-to-rust-playbook.md#52-per-module-area: in_progress
    notes: |
      Test progress: 67/141 tests passing
      Modules complete: text_utils, config, error types
      Modules in progress: formatter (24/45 tests), cli (0/28 tests)
      Current: Porting formatter module
```

### Example 3: Fix Phase with Workaround Tracking

```yaml
paths:
  fix:
    status: in_progress
    started: 2026-02-14
    steps:
      ./reference/python-to-rust-playbook.md#61-run-cross-validation-and-categorize-failures: completed
      ./reference/python-to-rust-playbook.md#62-workaround-strategy: in_progress
      ./reference/python-to-rust-playbook.md#63-track-workarounds-systematically: in_progress
    notes: |
      Cross-validation: 18 differences found
      Categorized:
      - 3 porting bugs (fixed)
      - 12 library differences (9 workarounds implemented, 3 in progress)
      - 2 Python bugs (decided to fix in Rust)
      - 1 intentional improvement (accepted, documented)

      Workarounds implemented:
      - Underscore escaping fix (post-processing)
      - Heading anchor normalization (post-processing)
      - List marker spacing (post-processing)
      ...

      Remaining:
      - Table cell alignment (evaluating pre-processing)
      - Link reference case sensitivity (may accept)
      - Code fence info string (vendor patch candidate)
```

### Example 4: Sync Phase (Ongoing Maintenance)

```yaml
paths:
  sync:
    status: in_progress
    started: 2026-03-01
    steps:
      ./reference/port-checklist-update-template.md#1-update-python-submodule: completed
      ./reference/port-checklist-update-template.md#2-regenerate-test-fixtures: completed
      ./reference/port-checklist-update-template.md#3-run-cross-validation: completed
      ./reference/port-checklist-update-template.md#4-categorize-changes: in_progress
    notes: |
      Python version: 0.5.5 → 0.6.0
      Changes found: 4 new tests, 2 bug fixes, 1 new CLI flag
      Cross-validation: 2 new failures (investigating)
```

## Key Differences from Antipatterns Example

| Aspect | Antipatterns | Porting Playbook |
| --- | --- | --- |
| **Area type** | Catalog + procedural | Linear with cycles |
| **Completion metric** | Violations found/fixed | Tests passing, phases complete |
| **Iteration pattern** | Re-check after fixes | Sync phase (periodic) |
| **Documentation** | Procedural + catalog | Reference + compact guidelines |
| **Routes** | Priority, context, scope | Phase progression, conditional loops |
| **Progress notes** | Violation counts | Test counts, workaround counts, coverage % |
| **Extra steps** | Project-specific violations | Extra library evaluations, workarounds |

## Interesting Signposts Features This Example Highlights

### 1. Conditional Entry Points

The `start` workflow routes to different paths based on scenario:
- Fresh port with unknown coverage → Prepare
- Fresh port with good coverage → Core Area/Assess
- Ongoing sync → Core Area/Sync
- Just need reference → Reference workflows

### 2. Preparatory Workflows

The `prepare` workflow is a prerequisite that may or may not be needed:
- If coverage is already good, skip it
- If coverage is low, complete it before starting core workflow
- Can loop back from Assess if coverage gaps discovered

### 3. Guidelines as Agent-Injectable Context

The `guidelines` workflow contains compact (~2-3k token) documents designed to be loaded
into agent context:
```bash
# In practice (outside signposts):
tbd guidelines python-to-rust-porting-rules
tbd guidelines rust-project-setup

# Signposts equivalent:
signposts path porting-rules
signposts path rust-project-setup
```

These are **reference material**, not workflow steps, but they’re organized as paths
because an agent might want to “work through” understanding each guideline.

### 4. Progress Metrics in Notes

The `notes` field captures quantitative progress:
- “Test progress: 67/141 tests passing”
- “12 workarounds implemented, 3 in progress”
- “Coverage: 87% on core modules”

This suggests a potential extension: structured metrics in `progress.yml`.

### 5. Case Studies as Paths

Real-world examples are paths with docs, allowing navigation:
```yaml
- id: flowmark
  docs:
    - ./case-studies/flowmark/flowmark-port-analysis.md
    - ./case-studies/flowmark/flowmark-port-library-choices.md
    - ./case-studies/flowmark/flowmark-port-decision-log.md
```

When working on library evaluation, an agent could reference:
```bash
signposts path flowmark
# Shows all flowmark case study docs
```

### 6. Templates as Paths

Templates are paths without steps (pure reference):
```yaml
- id: initial-checklist
  title: Initial Port Checklist
  docs:
    - ./reference/port-checklist-initial-template.md
```

The agent can navigate to templates when needed:
```bash
signposts path initial-checklist
# Displays the template for copying
```

### 7. Cycles in the Graph

The porting areas has natural cycles:
- **Port → Research** (discovered library gap)
- **Fix → Research** (decided to switch libraries)
- **Fix → Port** (found porting bugs)
- **Sync → Fix** (new differences after upstream changes)
- **Assess → Prepare** (coverage below threshold)

Signposts allows cycles explicitly -- progress tracking handles revisiting paths.

## CLI Command Examples

### Starting a Port

```bash
# Start at the entry point
signposts start

# Output:
# Starting area: start
# Routes available:
#   → prepare/measure-coverage (when: Starting fresh port with unknown coverage)
#   → core-area/assess (when: Starting fresh port with good coverage)
#   → core-area/sync (when: Syncing existing port)
#   → reference/mapping-reference (when: Just need type mappings)

# Navigate to assess
signposts start assess

# Mark first step done
signposts done assess \
  "./reference/python-to-rust-playbook.md#11-measure-the-codebase" \
  --note "Project: flowmark, ~2k lines, 87% coverage"
```

### Working Through Research Phase

```bash
# View current path
signposts where

# Output:
# Current path: research — Evaluate Rust library candidates
#   (area: Core Area)
#
#   Steps:
#     [x] ./reference/python-to-rust-mapping-reference.md
#     [>] ./reference/python-to-rust-playbook.md#21-evaluate-candidates...
#     [ ] ./reference/python-to-rust-playbook.md#22-evaluation-criteria
#
#   Routes:
#     → plan (when: All libraries evaluated)
#     → assess (when: No viable Rust equivalent found)

# Load mapping reference for context
signposts step research ./reference/python-to-rust-mapping-reference.md

# Add extra step for library evaluation
signposts note research \
  "Evaluating comrak vs pulldown-cmark for Markdown parsing"
```

### Tracking Port Progress

```bash
# Update notes with test progress
signposts note port "Test progress: 67/141 tests passing"

# View overall progress
signposts status

# Output:
# Progress: 4/8 core workflow phases completed
#
#   Prepare - Test Coverage
#     (skipped - coverage already good)
#
#   Core Area
#     assess             [completed]    5/5 steps done
#     research           [completed]    4/4 steps done + 2 extra
#     plan               [completed]    5/5 steps done
#     setup              [completed]    6/6 steps done
#     port               [in_progress]  2/5 steps done
#                        Test progress: 67/141 tests passing
#     fix                [not started]
#     finalize           [not started]
#     sync               [not started]
```

### Navigating to Reference Material

```bash
# Need CLI porting guidance
signposts path cli-porting

# Output:
# Path: cli-porting — CLI-specific porting (argparse to clap)
#   (area: Guidelines)
#
#   Documentation:
#     ./guidelines/python-to-rust-cli-porting.md
#
# Use 'signposts step cli-porting <docspec>' to view content

signposts step cli-porting ./guidelines/python-to-rust-cli-porting.md

# Displays the guideline content
```

### Looking Up a Case Study

```bash
# How was library evaluation done for flowmark?
signposts path flowmark

# Output:
# Path: flowmark — Python Markdown formatter → Rust case study
#   (area: Case Studies)
#
#   Documentation:
#     ./case-studies/flowmark/flowmark-port-analysis.md
#     ./case-studies/flowmark/flowmark-port-library-choices.md
#     ./case-studies/flowmark/flowmark-port-decision-log.md
#     ./case-studies/flowmark/flowmark-port-migration-plan.md
#     ./case-studies/flowmark/flowmark-port-cross-validation.md

# View specific decision log
signposts step flowmark \
  ./case-studies/flowmark/flowmark-port-decision-log.md
```

## Potential Extensions Suggested by This Example

### 1. Quantitative Progress Metrics

```yaml
paths:
  port:
    metrics:
      tests_total: 141
      tests_passing: 67
      tests_failing: 74
      modules_complete: 3
      modules_in_progress: 2
```

Enables:
- `signposts stats` → aggregate metrics
- Progress bars in CLI output
- Completion percentage calculations

### 2. Phase-Specific Metadata

```yaml
paths:
  research:
    decisions:
      - library: comrak
        chosen: true
        reason: "Better AST access, active maintenance"
        alternatives: [pulldown-cmark, markdown-rs]
      - library: clap
        chosen: true
        reason: "Standard for Rust CLI"
```

Captures structured decisions rather than free-form notes.

### 3. Dependency Tracking Between Paths

```yaml
paths:
  port:
    requires:
      - research
      - plan
      - setup
```

Could warn if trying to start a path without completing prerequisites.

### 4. Time Estimates vs Actuals

```yaml
paths:
  research:
    estimated_time: "30-60 minutes"
    actual_time: "75 minutes"
    started: 2026-02-10T10:00:00Z
    completed: 2026-02-10T11:15:00Z
```

Helps calibrate future estimates.

### 5. Multi-Document Paths with Precedence

```yaml
paths:
  assess:
    docs:
      - ./reference/python-to-rust-playbook.md#phase-1-assess  # Primary
      - ./guidelines/test-coverage-for-porting.md              # Supporting
```

Could display primary doc by default, with “see also” for supporting docs.

## Conclusion

The porting playbook example demonstrates:

1. **Linear workflow with conditional branches** - different from the catalog structure
   of antipatterns
2. **Reference material organization** - guidelines, templates, case studies as
   navigable paths
3. **Preparatory workflows** - test coverage work before main workflow
4. **Ongoing workflows** - sync phase for long-term maintenance
5. **Cycles in the graph** - library switching, coverage improvement
6. **Quantitative progress** - test counts, workaround counts (captured in notes, could
   be structured)

The core signposts concepts (areas, paths, steps, routes, progress) map cleanly to this
real-world complex workflow.
The main open question is whether to add structured metrics beyond free-form notes.

Next steps:
- Validate whether metrics extensions are worth the complexity
- Consider whether signposts should integrate with project-specific tooling (e.g.,
  `cargo test --list` to get test counts)
- Explore how an agent would use signposts during actual porting work (pull docs at each
  step, track decisions, navigate case studies)
