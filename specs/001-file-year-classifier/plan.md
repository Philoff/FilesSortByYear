# Implementation Plan: File Year Classifier (FilesByYear)

**Branch**: `FilesSortByYear` | **Date**: 2025-12-15 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `/specs/FilesSortByYear/spec.md`

**Note**: This template is filled in by the `/speckit.plan` command. See `.specify/templates/commands/plan.md` for the execution workflow.

## Summary

FilesByYear is a cross-platform CLI tool for organizing files into year-based subdirectories. The system intelligently detects date patterns in filenames through statistical analysis, supports filesystem date extraction as fallback, and prioritizes user data safety through preview mode and copy-by-default operations. The tool will be implemented in Python 3.9+ with minimal dependencies, using a modular architecture for pattern detection, file classification, and safe file operations.

## Technical Context

**Language/Version**: Python 3.9+
**Primary Dependencies**: Python standard library (os, pathlib, re, datetime, shutil, argparse, json, logging)
**Storage**: Filesystem only - no database required. Log files for operation audit trail
**Testing**: pytest for unit and integration tests, pytest-cov for coverage reporting
**Target Platform**: Cross-platform CLI - Windows 10+, Linux (any modern distro), macOS 10.15+
**Project Type**: Single CLI application with modular library structure
**Performance Goals**:
- Analyze 1000 files in <5 seconds
- Classify 1000 files in <30 seconds (excluding I/O time)
- Handle up to 10,000 files without memory issues (<100MB RAM usage)

**Constraints**:
- Must use only Python standard library for core functionality (minimal external dependencies)
- Must work identically on Windows and Linux
- Must handle Unicode filenames correctly
- Must never lose or corrupt user files (safety-first design)

**Scale/Scope**:
- Target: Individual users organizing personal file collections (100s to 10,000s of files)
- CLI-only interface (no GUI)
- 5 main modules: date detection, pattern matching, file classification, file operations, CLI interface

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

### I. Sécurité des Données (NON-NÉGOCIABLE)
- ✅ **Pass**: Default mode is copy (not move) - aligns with constitution
- ✅ **Pass**: Preview/dry-run mode required before execution - FR-009
- ✅ **Pass**: User confirmation required for file operations - FR-020
- ✅ **Pass**: Comprehensive logging of all operations - FR-016
- ✅ **Pass**: Path validation and conflict detection - FR-014

### II. Intelligence de Détection
- ✅ **Pass**: Statistical sampling for pattern detection - FR-001, FR-004
- ✅ **Pass**: Support for multiple date formats (6 and 8 digits) - FR-002
- ✅ **Pass**: Separator detection (-, _, ., none) - FR-003
- ✅ **Pass**: Confidence scoring for detected patterns - FR-004
- ✅ **Pass**: Interactive confirmation of detected pattern - User Story 1

### III. Flexibilité des Sources
- ✅ **Pass**: Filename date extraction (priority) - FR-001, FR-002
- ✅ **Pass**: Filesystem creation date option - FR-006
- ✅ **Pass**: Filesystem modification date option - FR-007
- ✅ **Pass**: Manual format specification - FR-005
- ✅ **Pass**: Fallback strategy when detection fails - User Story 1, scenario 4

### IV. Interface CLI Simple
- ✅ **Pass**: Simple command structure `filesbyyear <folder>` - User Story 5
- ✅ **Pass**: Interactive mode with guided questions - implied by User Story 1
- ✅ **Pass**: Preview mode (`--preview`) - FR-009
- ✅ **Pass**: Clear structured output reports - FR-015, SC-009

### V. Robustesse et Fiabilité
- ✅ **Pass**: Handling files without detectable dates - Edge cases
- ✅ **Pass**: 2-digit vs 4-digit year support - FR-013
- ✅ **Pass**: Date validation (impossible dates rejected) - FR-012
- ✅ **Pass**: Permission handling (read-only, hidden files) - FR-019, Edge cases
- ✅ **Pass**: Unicode filename support - FR-018

### Workflow de Développement
- ✅ **Pass**: Test-first approach will be followed (TDD)
- ✅ **Pass**: pytest for all test types (unit, integration) - Technical Context
- ✅ **Pass**: Documentation required (README, docstrings) - to be created

### Structure Modulaire (from Constitution)
- ✅ **Pass**: Planned modules align with constitution:
  - `date_detector.py` - pattern detection and analysis
  - `file_classifier.py` - classification logic
  - `cli.py` - user interface
  - `file_operations.py` - safe file operations
  - `logger.py` - audit logging

**Gate Status**: ✅ **PASSED** - All constitutional requirements are addressed in the specification. No violations require justification.

---

## Constitution Check - Post-Design Re-Evaluation

*Re-evaluated after Phase 1 design (data model, contracts, quickstart)*

### I. Sécurité des Données (NON-NÉGOCIABLE)
- ✅ **Pass**: Data model includes preview mode (ClassificationJob.preview_mode)
- ✅ **Pass**: OperationLog provides complete audit trail
- ✅ **Pass**: safe_copy_file() and safe_move_file() include verification steps
- ✅ **Pass**: check_destination_conflict() detects collisions before operations
- ✅ **Pass**: CLI contract includes confirmation prompts and --yes flag

### II. Intelligence de Détection
- ✅ **Pass**: DatePattern model includes confidence scoring
- ✅ **Pass**: detect_pattern() supports sample_size and min_confidence parameters
- ✅ **Pass**: 6-digit and 8-digit format support confirmed in research
- ✅ **Pass**: Separator detection included in DatePattern model
- ✅ **Pass**: CLI analyze command provides interactive pattern confirmation

### III. Flexibilité des Sources
- ✅ **Pass**: ClassificationJob.date_source supports "auto", "filename", "ctime", "mtime"
- ✅ **Pass**: Manual pattern override via ClassificationJob.manual_pattern
- ✅ **Pass**: get_file_dates() utility extracts filesystem timestamps
- ✅ **Pass**: Priority handling confirmed in build_classification_plan() logic

### IV. Interface CLI Simple
- ✅ **Pass**: Three clear commands: analyze, preview, classify
- ✅ **Pass**: Progress reporting specified in CLI contract
- ✅ **Pass**: Structured output formats (text and JSON) defined
- ✅ **Pass**: Help system via argparse --help

### V. Robustesse et Fiabilité
- ✅ **Pass**: FileEntry.classification_status tracks states including "failed" and "skipped"
- ✅ **Pass**: convert_two_digit_year() implements century logic (00-49 → 2000s, 50-99 → 1900s)
- ✅ **Pass**: validate_date() handles leap years and month boundaries
- ✅ **Pass**: Error handling philosophy: "continue on error" with comprehensive reporting
- ✅ **Pass**: Unicode support via pathlib throughout

### Modular Structure
- ✅ **Pass**: Confirmed in project structure - all 5 modules present with clear responsibilities
- ✅ **Pass**: Module dependency graph shows clean separation (utils and logger are leaf modules)
- ✅ **Pass**: Python API contract defines clear interfaces between modules

### Testing Requirements
- ✅ **Pass**: pytest configured for unit and integration tests
- ✅ **Pass**: Quickstart guide emphasizes TDD approach (tests first)
- ✅ **Pass**: Test patterns documented with examples
- ✅ **Pass**: Integration test examples provided for cross-platform testing

**Final Gate Status**: ✅ **PASSED** - Design maintains all constitutional requirements. Implementation can proceed.

## Project Structure

### Documentation (this feature)

```text
specs/[###-feature]/
├── plan.md              # This file (/speckit.plan command output)
├── research.md          # Phase 0 output (/speckit.plan command)
├── data-model.md        # Phase 1 output (/speckit.plan command)
├── quickstart.md        # Phase 1 output (/speckit.plan command)
├── contracts/           # Phase 1 output (/speckit.plan command)
└── tasks.md             # Phase 2 output (/speckit.tasks command - NOT created by /speckit.plan)
```

### Source Code (repository root)

```text
filesbyyear/                          # Root project directory
├── src/
│   └── filesbyyear/
│       ├── __init__.py
│       ├── __main__.py          # Entry point for python -m filesbyyear
│       ├── cli.py               # CLI interface and argument parsing
│       ├── date_detector.py     # Pattern detection and confidence scoring
│       ├── file_classifier.py   # File classification logic
│       ├── file_operations.py   # Safe file copy/move operations
│       ├── logger.py            # Audit logging functionality
│       ├── models.py            # Data models (FileEntry, DatePattern, etc.)
│       └── utils.py             # Utility functions (date validation, etc.)
│
├── tests/
│   ├── unit/
│   │   ├── test_date_detector.py
│   │   ├── test_file_classifier.py
│   │   ├── test_file_operations.py
│   │   ├── test_models.py
│   │   └── test_utils.py
│   ├── integration/
│   │   ├── test_full_workflow.py
│   │   ├── test_cross_platform.py
│   │   └── test_edge_cases.py
│   └── fixtures/
│       └── sample_files/        # Test files with various naming patterns
│
├── docs/
│   ├── README.md
│   ├── EXAMPLES.md
│   └── API.md                   # Internal API documentation
│
├── setup.py                     # Package installation
├── requirements.txt             # Python dependencies (minimal)
├── requirements-dev.txt         # Development dependencies (pytest, etc.)
├── .gitignore
└── LICENSE
```

**Structure Decision**: Single project structure selected as this is a standalone CLI application. The modular design with separate files for each concern (detection, classification, operations) aligns with the constitution's requirement for clear separation of responsibilities. The `src/filesbyyear/` layout follows Python best practices for packaging and enables both direct execution (`python -m filesbyyear`) and installation as a package (`pip install filesbyyear`).

## Complexity Tracking

> **Fill ONLY if Constitution Check has violations that must be justified**

**No violations** - All constitution requirements are met. No complexity justification needed.
