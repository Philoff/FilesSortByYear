# FilesByYear - Project Dependencies Map

**Purpose:** Visual and textual mapping of module dependencies to understand code organization and identify refactoring opportunities.

**Date:** 2025-12-17

---

## 1. Dependency Graph (Visual)

```
┌────────────────────────────────────────────────────────────┐
│                    CLI Entry Point                         │
│                     (cli.py - 810L)                        │
│  Main subcommands: analyze, preview, classify              │
└──────────────────────────┬─────────────────────────────────┘
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
        ▼                  ▼                  ▼
┌──────────────────┐ ┌──────────────────┐ ┌──────────────────┐
│  date_detector   │ │ file_classifier  │ │  file_operations │
│   (330 lines)    │ │   (613 lines)    │ │   (311 lines)    │
│                  │ │                  │ │                  │
│ • detect_pattern │ │ • Job creation   │ │ • safe_copy_file │
│ • extract_date   │ │ • Plan building  │ │ • safe_move_file │
│ • Manual pattern │ │ • Orchestration  │ │ • Verification   │
└────────┬─────────┘ └────────┬─────────┘ └────────┬─────────┘
         │                    │                    │
         └────────────────────┼────────────────────┘
                              │
                    ┌─────────▼──────────┐
                    │     utils.py       │
                    │   (208 lines)      │
                    │                    │
                    │ • validate_date    │
                    │ • get_file_dates   │
                    │ • format_size      │
                    │ • format_duration  │
                    │ • calc_dest_path   │
                    └─────────┬──────────┘
                              │
                    ┌─────────▼──────────┐
                    │    models.py       │
                    │   (235 lines)      │
                    │                    │
                    │ • DatePattern      │
                    │ • FileEntry        │
                    │ • ClassificationJob│
                    │ • OperationLog     │
                    │ • ClassificationRep
                    └────────────────────┘

        ┌─────────────────────────────────┐
        │  Supporting Modules              │
        │                                  │
        │ • logger.py (190 lines)         │
        │ • rollback.py (223 lines)       │
        │ • __init__.py (9 lines)         │
        │ • __main__.py (11 lines)        │
        └─────────────────────────────────┘
```

---

## 2. Detailed Dependency Matrix

### By Module

#### cli.py (Entry Point)
```
Imports FROM:
  ├── date_detector: detect_pattern
  ├── file_classifier: create_classification_job, build_classification_plan,
  │                    execute_classification, generate_classification_report
  ├── logger: setup_logging
  ├── models: DatePattern, ClassificationJob, ClassificationReport
  └── utils: format_file_size, format_duration

Exports TO:
  └── main(): Called when running as module
```

**Dependency Count:** 11 imports
**Dependency Type:** HUB (connects all modules)

---

#### file_classifier.py (Orchestrator)
```
Imports FROM:
  ├── models: ClassificationJob, DatePattern, FileEntry, OperationLog, ClassificationReport
  ├── utils: get_file_dates, calculate_destination_path
  ├── date_detector: detect_pattern, extract_date_from_filename, create_manual_pattern
  ├── file_operations: check_destination_conflict, create_year_directory,
  │                     safe_copy_file, safe_move_file
  └── rollback: generate_rollback_script

Imports (internal):
  ├── datetime
  ├── pathlib
  └── typing
```

**Dependency Count:** 14 imports from FilesByYear + stdlib
**Dependency Type:** HUB (coordinates other modules)

---

#### date_detector.py (Pattern Detection)
```
Imports FROM:
  ├── models: DatePattern
  ├── utils: validate_date, convert_two_digit_year

Imports (internal):
  ├── pathlib
  ├── re
  ├── random
  └── typing
```

**Dependency Count:** 5 imports from FilesByYear + stdlib
**Dependency Type:** LEAF (used by file_classifier and cli)

---

#### file_operations.py (File I/O)
```
Imports FROM:
  ├── models: OperationLog
  ├── logger: setup_logging

Imports (internal):
  ├── pathlib
  ├── shutil
  ├── datetime
  └── typing
```

**Dependency Count:** 4 imports from FilesByYear + stdlib
**Dependency Type:** LEAF (used by file_classifier)

---

#### utils.py (Utilities)
```
Imports FROM:
  ├── (none - pure utilities)

Imports (internal):
  ├── datetime
  └── pathlib
```

**Dependency Count:** 0 imports from FilesByYear (pure utilities)
**Dependency Type:** LEAF (no internal dependencies)

---

#### models.py (Data Structures)
```
Imports FROM:
  ├── (none - pure data classes)

Imports (internal):
  ├── pathlib
  ├── datetime
  ├── typing
  ├── dataclasses
  └── uuid
```

**Dependency Count:** 0 imports from FilesByYear (pure data)
**Dependency Type:** FOUNDATION (no dependencies on other modules)

---

#### logger.py (Logging)
```
Imports FROM:
  ├── (none - logger setup)

Imports (internal):
  ├── logging
  ├── pathlib
  ├── datetime
  └── sys
```

**Dependency Count:** 0 imports from FilesByYear
**Dependency Type:** UTILITY (logging infrastructure)

---

#### rollback.py (Rollback Script Generation)
```
Imports FROM:
  ├── (none - standalone rollback generation)

Imports (internal):
  ├── pathlib
  ├── csv
  ├── datetime
  └── subprocess
```

**Dependency Count:** 0 imports from FilesByYear
**Dependency Type:** UTILITY (used by file_classifier for rollback)

---

## 3. Dependency Statistics

### Import Chains

```
DEEPEST CHAIN (4 levels):
cli.py
  → file_classifier.py
    → date_detector.py
      → utils.py

CIRCULAR DEPENDENCIES: NONE (acyclic dependency graph ✓)

UNUSED IMPORTS: NONE (all imports are used)

EXTERNAL DEPENDENCIES: 0 (only Python stdlib)
```

### Module Relationship Types

| Type | Count | Examples |
|------|-------|----------|
| 1-to-1 (linear) | 2 | logger→cli, rollback→file_classifier |
| 1-to-many (hub) | 2 | cli→multiple, file_classifier→multiple |
| Circular | 0 | None (good design!) |
| Foundation (no deps) | 3 | models, utils, logger |

---

## 4. Redundancy Points Identified

### Point 1: ClassificationJob Reconstruction
```
file_classifier.py:198-289

PATTERN: Reconstructing frozen ClassificationJob with 16 copied fields + 1-2 new fields

LOCATIONS:
  1. Line 198: Manual pattern detected
  2. Line 226: Auto-detection successful
  3. Line 248: Auto-detection failed → fallback to mtime
  4. Line 270: Exception during auto-detection → fallback to mtime

IMPACT: 75 lines of boilerplate across 4 locations
        HIGH risk of bugs when adding new ClassificationJob fields
        POOR maintainability (DRY violation)

SOLUTION: Create _update_classification_job() helper
```

---

### Point 2: Output Formatting
```
cli.py:29-285

PATTERN: 3 separate formatting functions with identical structure
         Header decoration, key-value pairs, footer

LOCATIONS:
  1. format_analyze_output_text(): Lines 29-71 (43 lines)
  2. format_preview_output_text(): Lines 104-175 (70+ lines)
  3. format_report_output_text(): Lines 234-285 (50+ lines)

DUPLICATION:
  - Header formatting (3 times): "=" * 60 header pattern
  - Key-value formatting (repeated): f"{key:<20} {value}"
  - Footer decoration (3 times): "=" * 60 footer pattern

IMPACT: 160+ lines of shared logic
        INCONSISTENT spacing/styling across outputs
        HARD to update styling (must change 3 places)

SOLUTION: Create OutputFormatter base class with utilities
```

---

### Point 3: Validation Logic Scattered
```
LOCATIONS:
  1. utils.py:11-37 - validate_date() function
  2. date_detector.py:217-219 - Implicit date validation in try/except
  3. models.py:42-47 - DatePattern validation in __post_init__

PATTERN: Each location validates dates independently
         No single source of truth for date validation rules

IMPACT: Bug in one validator might not be in others
        Adding new validation requires updating 3 places
        INCONSISTENT error messages

SOLUTION: Create validators.py module with centralized DateValidator
```

---

### Point 4: File Operations Inconsistency
```
file_operations.py: Inconsistent operation signatures

FUNCTIONS:
  1. safe_copy_file(source, dest, verify=True, job_id=None) -> OperationLog
  2. safe_move_file(source, dest, job_id=None) -> OperationLog
  3. check_destination_conflict(path) -> bool  ← Different return type!

USAGE IN file_classifier.py:476-488:
  if job.operation_mode == "copy":
    operation_log = file_operations.safe_copy_file(...)
  else:
    operation_log = file_operations.safe_move_file(...)

IMPACT: No polymorphic interface
        Hard to add new operation types (e.g., symlink, compress)
        Conditional logic scattered in execute_classification()

SOLUTION: Create operations.py with FileOperation base class + Strategy pattern
```

---

## 5. Current Module Responsibilities

### Layered Architecture

```
PRESENTATION LAYER:
  ├── cli.py - User interface (CLI parsing, output formatting)
  └── logger.py - Logging infrastructure

ORCHESTRATION LAYER:
  └── file_classifier.py - Workflow coordination

CORE LOGIC LAYER:
  ├── date_detector.py - Date pattern detection
  ├── file_operations.py - File I/O operations
  └── rollback.py - Rollback script generation

UTILITIES LAYER:
  ├── utils.py - Helper functions
  └── models.py - Data structures

DATA MODELS LAYER:
  └── models.py - Dataclasses (6 types)
```

**Assessment:** Good layering, but some cross-layer dependencies (cli → models directly)

---

## 6. Call Flow Analysis

### Command: `classify`

```
1. cli.py::cmd_classify()
   ├─ Parse arguments
   ├─ Validate directory
   ├─ Create classification job
   │  └─ file_classifier.py::create_classification_job()
   ├─ Build classification plan
   │  └─ file_classifier.py::build_classification_plan()
   │     ├─ date_detector.py::detect_pattern() [if auto]
   │     ├─ date_detector.py::extract_date_from_filename() [per file]
   │     ├─ utils.py::get_file_dates() [per file]
   │     ├─ utils.py::calculate_destination_path() [per file]
   │     └─ file_operations.py::check_destination_conflict() [per file]
   ├─ Execute classification
   │  └─ file_classifier.py::execute_classification()
   │     ├─ file_operations.py::create_year_directory() [per year]
   │     ├─ file_operations.py::safe_copy_file() [if copy mode]
   │     ├─ file_operations.py::safe_move_file() [if move mode]
   │     ├─ logger.py::log_operation() [per operation]
   │     └─ rollback.py::generate_rollback_script() [after completion]
   ├─ Generate report
   │  └─ file_classifier.py::generate_classification_report()
   └─ Format and display output
      └─ cli.py::format_*_output_*()

Total call depth: 5 levels
Total branches: 20+
```

---

## 7. Proposed New Dependencies (Post-Refactoring)

### New Modules to Add

```
validators.py (NEW)
  ├─ DateValidator
  ├─ PathValidator
  └─ PatternValidator

operations.py (NEW)
  ├─ FileOperation (abstract)
  ├─ CopyOperation
  ├─ MoveOperation
  └─ OperationFactory
```

### New Dependency Relationships

```
BEFORE:
  file_classifier.py → file_operations.safe_copy_file()
                    → file_operations.safe_move_file()

AFTER:
  file_classifier.py → operations.OperationFactory.get_operation()
                    ↓
                operations.CopyOperation / MoveOperation
                    ↓
                file_operations.safe_copy_file()
                file_operations.safe_move_file()
```

**Benefit:** Adds abstraction layer, enables extensibility

```
BEFORE:
  date_detector.py → validate_date() [in utils.py]
  models.py → date validation [inline]
  utils.py → validate_date()

AFTER:
  date_detector.py → DateValidator [in validators.py]
  models.py → DateValidator [in validators.py]
  utils.py → DateValidator [in validators.py]
```

**Benefit:** Single source of truth for date validation

---

## 8. Dependency Impact Assessment

### HIGH IMPACT MODULES
(Changes affect many other modules)

1. **models.py** (235 lines)
   - Used by: cli, file_classifier, date_detector, file_operations, rollback
   - Impact of change: VERY HIGH (cascades to 5 modules)
   - Risk level: HIGH
   - Recommendation: Stable, minimize changes

2. **file_classifier.py** (613 lines)
   - Used by: cli (directly)
   - Uses: date_detector, file_operations, utils, models, rollback
   - Impact of change: VERY HIGH (orchestrator)
   - Risk level: MEDIUM (good test coverage)
   - Recommendation: Refactor with care (but our refactoring is safe)

3. **utils.py** (208 lines)
   - Used by: file_classifier, date_detector
   - Uses: nothing internal
   - Impact of change: MEDIUM (utility functions)
   - Risk level: LOW (pure functions)
   - Recommendation: Good candidate for refactoring

### MEDIUM IMPACT MODULES

4. **date_detector.py** (330 lines)
   - Used by: file_classifier, cli
   - Uses: utils, models
   - Impact: MEDIUM
   - Risk: LOW
   - Recommendation: Refactoring candidate

5. **file_operations.py** (311 lines)
   - Used by: file_classifier
   - Uses: models, logger
   - Impact: MEDIUM
   - Risk: MEDIUM (core file I/O)
   - Recommendation: Refactor carefully with tests

### LOW IMPACT MODULES

6. **cli.py** (810 lines)
   - Used by: __main__ (entry point)
   - Uses: everything (as hub)
   - Impact: LOW (presentation layer)
   - Risk: LOW (output formatting change)
   - Recommendation: Good refactoring candidate

7. **logger.py** (190 lines)
   - Used by: file_operations, cli
   - Uses: nothing internal
   - Impact: LOW
   - Risk: LOW
   - Recommendation: Stable, no changes needed

8. **rollback.py** (223 lines)
   - Used by: file_classifier
   - Uses: nothing internal
   - Impact: LOW
   - Risk: LOW
   - Recommendation: Stable, no changes needed

---

## 9. Refactoring Dependency Changes

### Phase 1: Quick Wins

```
BEFORE:
  cli.py → [3 separate formatter functions]

AFTER:
  cli.py → OutputFormatter [new base class]
           ↓
         [3 refactored formatter functions using OutputFormatter]

IMPACT: cli.py imports change (internal)
        No external dependency changes
        RISK: LOW
```

```
BEFORE:
  file_classifier.py → ClassificationJob [4 reconstruction sites]

AFTER:
  file_classifier.py → _update_classification_job() [1 helper]
                       ↓
                     ClassificationJob [1 reconstruction point]

IMPACT: file_classifier.py internal refactoring
        No external dependency changes
        RISK: NONE (internal only)
```

### Phase 2: Architecture

```
BEFORE:
  date_detector.py → utils.validate_date()
  models.py → [inline validation]
  utils.py → validate_date()

AFTER:
  date_detector.py → validators.DateValidator [new module]
  models.py → validators.DateValidator
  utils.py → validators.DateValidator [delegation]

IMPACT: 3 modules now depend on validators.py (new)
        utils.validate_date() becomes wrapper
        RISK: LOW (behavior preserved)
```

```
BEFORE:
  file_classifier.py {
    if job.operation_mode == "copy":
      file_operations.safe_copy_file()
    else:
      file_operations.safe_move_file()
  }

AFTER:
  file_classifier.py {
    operation = operations.OperationFactory.get_operation()
    operation.execute()
  }

IMPACT: file_classifier.py depends on operations.py (new)
        operations.py is abstraction over file_operations.py
        RISK: MEDIUM (core logic refactoring, but well-tested)
```

---

## 10. Dependency Quality Metrics

### Acyclicity ✅
- **Circular dependencies:** 0
- **Status:** EXCELLENT (no cycles detected)

### Depth ✅
- **Maximum depth:** 4 levels
- **Average depth:** 2.5 levels
- **Status:** GOOD (not too deep)

### Fan-in/Fan-out
```
File               Fan-In    Fan-Out   Type
────────────────── ───────── ───────── ──────────
models.py          5         0         Foundation
utils.py           2         1         Utility
logger.py          2         1         Utility
date_detector.py   2         2         Core
file_operations.py 1         2         Core
file_classifier.py 1         5         Orchestrator
rollback.py        1         0         Utility
cli.py             1         6         Hub/Entry
────────────────── ───────── ───────── ──────────
TOTAL              15        17
```

**Analysis:** cli.py has highest fan-out (6), which is appropriate for a hub/entry point

### Coupling Score

| Aspect | Score | Status |
|--------|-------|--------|
| Afferent Coupling (fan-in avg) | 2.1 | GOOD |
| Efferent Coupling (fan-out avg) | 2.1 | GOOD |
| Abstractness (foundation modules) | HIGH | GOOD |
| Stability | HIGH | GOOD |
| Modularity | GOOD | ✓ |

---

## 11. Refactoring Impact on Dependencies

### Before Refactoring
```
Dependency Metrics:
  - Total imports: 45
  - Circular imports: 0
  - Import depth: 4
  - Modules with >3 dependencies: 3 (cli, file_classifier)
  - Pure utility modules: 3 (models, utils, logger)
```

### After Refactoring
```
Dependency Metrics:
  - Total imports: 50 (+5, for new modules)
  - Circular imports: 0 (maintained)
  - Import depth: 4 (maintained)
  - Modules with >3 dependencies: 3 (maintained)
  - Pure utility modules: 5 (+2 new: validators, operations)
```

**Impact:** Minimal impact to dependency structure. New modules are "leaf" utilities with no cyclic dependencies.

---

## 12. Dependency Graph (Refined)

### Current State
```
┌─ models.py (foundation) ─┐
│                           │
├─ utils.py ───────────────┤
│                           │
├─ logger.py ───────────────┤
│                           │
├─ date_detector.py ────────┤
│     ↓                      │
├─ file_operations.py ──────┤
│     ↓                      │
├─ rollback.py ─────────────┤
│     ↓                      │
├─ file_classifier.py ──────┤
│     ↓                      │
└─ cli.py (entry point)
```

### Post-Refactoring State
```
┌─ models.py (foundation) ──────────────┐
│                                        │
├─ utils.py ────────────────────────────┤
│                                        │
├─ validators.py (NEW) ──────────────────┤
│                                        │
├─ logger.py ────────────────────────────┤
│                                        │
├─ date_detector.py ─────────────────────┤
│     ↓                                   │
├─ file_operations.py ────────────────────┤
│     ↓                                   │
├─ operations.py (NEW) ───────────────────┤
│     ↓                                   │
├─ rollback.py ──────────────────────────┤
│     ↓                                   │
├─ file_classifier.py ────────────────────┤
│     ↓                                   │
└─ cli.py (entry point)
```

**Observation:** New modules maintain "leaf" dependency structure (no cycles created)

---

## Summary

### Dependency Health Assessment

| Aspect | Rating | Notes |
|--------|--------|-------|
| Acyclicity | ✅ EXCELLENT | No circular dependencies |
| Simplicity | ✅ GOOD | Clean separation of concerns |
| Maintainability | ⚠️ FAIR | Some boilerplate and duplication |
| Testability | ✅ GOOD | Modules are testable |
| Extensibility | ⚠️ FAIR | Hard to add new operation types |
| Documentation | ❌ POOR | No architecture docs |

### Refactoring Benefits

1. **Reduces logical coupling** between date validation implementations
2. **Adds operational abstraction** for future extensibility
3. **Eliminates boilerplate** in job reconstruction
4. **Improves consistency** in output formatting
5. **Maintains acyclic structure** (no circular dependencies introduced)

---

**Document Created:** 2025-12-17
**Last Updated:** 2025-12-17
**Version:** 1.0

