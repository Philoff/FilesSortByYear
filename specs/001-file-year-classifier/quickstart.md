# Quickstart Guide: FilesByYear Development

**Feature**: FilesSortByYear
**Date**: 2025-12-15
**Purpose**: Get developers started with FilesByYear implementation

## Prerequisites

- Python 3.9 or higher
- git
- pip (Python package manager)
- A code editor (VS Code, PyCharm, or similar)

## Project Setup

### 1. Clone and Setup Repository

```bash
# Navigate to project root
cd /path/to/filesbyyear

# Create virtual environment
python -m venv venv

# Activate virtual environment
# On Windows:
venv\Scripts\activate
# On Linux/Mac:
source venv/bin/activate

# Install development dependencies
pip install -r requirements-dev.txt
```

### 2. Verify Setup

```bash
# Run tests (should pass - we'll write them first!)
pytest

# Check Python version
python --version  # Should be 3.9+
```

---

## Development Workflow (TDD Approach)

FilesByYear follows Test-Driven Development as mandated by the constitution:
1. Write tests first
2. Watch them fail (Red)
3. Implement functionality
4. Watch tests pass (Green)
5. Refactor if needed

### Example TDD Cycle: Implementing Date Validation

**Step 1: Write the test** (`tests/unit/test_utils.py`):
```python
import pytest
from filesbyyear.utils import validate_date

@pytest.mark.parametrize("year,month,day,expected", [
    (2024, 2, 29, True),   # Leap year
    (2023, 2, 29, False),  # Not leap year
    (2024, 1, 31, True),   # Valid
    (2024, 13, 1, False),  # Invalid month
    (2024, 4, 31, False),  # April has only 30 days
])
def test_validate_date(year, month, day, expected):
    assert validate_date(year, month, day) == expected
```

**Step 2: Run test - it should FAIL**:
```bash
pytest tests/unit/test_utils.py::test_validate_date
# Error: ImportError: cannot import name 'validate_date'
```

**Step 3: Implement** (`src/filesbyyear/utils.py`):
```python
from datetime import datetime

def validate_date(year: int, month: int, day: int) -> bool:
    """Validate that a date is valid."""
    try:
        datetime(year, month, day)
        return True
    except ValueError:
        return False
```

**Step 4: Run test - it should PASS**:
```bash
pytest tests/unit/test_utils.py::test_validate_date
# ✓ All tests passed
```

**Step 5: Refactor if needed** (optional):
```python
# Code is simple and clear - no refactoring needed
```

---

## Project Structure Overview

```
filesbyyear/
├── src/filesbyyear/          # Source code
│   ├── __init__.py      # Package init
│   ├── __main__.py      # CLI entry point
│   ├── models.py        # Data models (DatePattern, FileEntry, etc.)
│   ├── utils.py         # Utility functions (date validation, etc.)
│   ├── date_detector.py # Pattern detection logic
│   ├── file_classifier.py # Classification workflow
│   ├── file_operations.py # Safe file operations
│   ├── logger.py        # Logging setup
│   └── cli.py           # Command-line interface
├── tests/               # Test suite
│   ├── unit/            # Unit tests (test individual functions)
│   ├── integration/     # Integration tests (test workflows)
│   └── fixtures/        # Test data
├── docs/                # Documentation
└── specs/               # Feature specifications
```

---

## Implementation Order (Following Spec Priorities)

### Phase 1: Core Models and Utilities (Foundation)
**Estimated time**: 1-2 days

1. **models.py**: Define dataclasses
   - DatePattern
   - FileEntry
   - ClassificationJob
   - OperationLog
   - ClassificationReport

2. **utils.py**: Basic utilities
   - validate_date()
   - convert_two_digit_year()
   - get_file_dates()
   - format_file_size()
   - format_duration()

**Test files**: `tests/unit/test_models.py`, `tests/unit/test_utils.py`

### Phase 2: Date Detection (P1 - User Story 1)
**Estimated time**: 2-3 days

3. **date_detector.py**: Pattern detection
   - get_supported_formats()
   - detect_pattern()
   - extract_date_from_filename()

**Test files**: `tests/unit/test_date_detector.py`

### Phase 3: File Operations (P1 - User Story 4)
**Estimated time**: 2 days

4. **logger.py**: Audit logging
   - setup_logging()
   - log_operation()

5. **file_operations.py**: Safe operations
   - safe_copy_file()
   - safe_move_file()
   - create_year_directory()
   - check_destination_conflict()

**Test files**: `tests/unit/test_logger.py`, `tests/unit/test_file_operations.py`

### Phase 4: Classification Logic (P2 - User Stories 2 & 3)
**Estimated time**: 3-4 days

6. **file_classifier.py**: Workflow coordination
   - create_classification_job()
   - build_classification_plan()
   - execute_classification()
   - generate_classification_report()

**Test files**: `tests/unit/test_file_classifier.py`, `tests/integration/test_full_workflow.py`

### Phase 5: CLI Interface (P3 - User Story 5)
**Estimated time**: 2-3 days

7. **cli.py**: Command-line interface
   - Command: analyze
   - Command: preview
   - Command: classify
   - Argument parsing
   - Progress display

**Test files**: `tests/integration/test_cli.py`

### Phase 6: Cross-Platform Testing
**Estimated time**: 1-2 days

8. **tests/integration/test_cross_platform.py**
   - Path handling on Windows vs Linux
   - Unicode filename support
   - Permission handling

---

## Running Tests

```bash
# Run all tests
pytest

# Run with coverage report
pytest --cov=filesbyyear --cov-report=html

# Run specific test file
pytest tests/unit/test_utils.py

# Run specific test function
pytest tests/unit/test_utils.py::test_validate_date

# Run with verbose output
pytest -v

# Run integration tests only
pytest tests/integration/

# Run unit tests only
pytest tests/unit/
```

---

## Code Style and Standards

### Type Hints
All functions must have type hints:
```python
from pathlib import Path
from typing import Optional

def example_function(
    required: str,
    optional: Optional[int] = None
) -> tuple[bool, str]:
    """Docstring describing function."""
    return True, "success"
```

### Docstrings
Use Google-style docstrings:
```python
def validate_date(year: int, month: int, day: int) -> bool:
    """
    Validate that a date is valid.

    Args:
        year: Year (1900-2099)
        month: Month (1-12)
        day: Day (1-31, depending on month)

    Returns:
        True if date is valid, False otherwise

    Examples:
        >>> validate_date(2024, 2, 29)
        True
        >>> validate_date(2023, 2, 29)
        False
    """
    ...
```

### Error Handling
- Use specific exceptions (ValueError, FileNotFoundError, etc.)
- Include descriptive error messages
- For batch operations, collect errors and continue processing

```python
def process_files(files: list[Path]) -> tuple[list[Result], list[Error]]:
    """Process files, collecting errors instead of failing fast."""
    results = []
    errors = []

    for file in files:
        try:
            result = process_file(file)
            results.append(result)
        except Exception as e:
            logger.error(f"Failed to process {file}: {e}")
            errors.append(Error(file, str(e)))

    return results, errors
```

---

## Testing Patterns

### Unit Test Example
```python
# tests/unit/test_utils.py
import pytest
from filesbyyear.utils import convert_two_digit_year

def test_convert_two_digit_year_2000s():
    """Years 00-49 should map to 2000-2049."""
    assert convert_two_digit_year(24) == 2024
    assert convert_two_digit_year(0) == 2000
    assert convert_two_digit_year(49) == 2049

def test_convert_two_digit_year_1900s():
    """Years 50-99 should map to 1950-1999."""
    assert convert_two_digit_year(95) == 1995
    assert convert_two_digit_year(50) == 1950
    assert convert_two_digit_year(99) == 1999

def test_convert_two_digit_year_invalid():
    """Should raise ValueError for invalid input."""
    with pytest.raises(ValueError):
        convert_two_digit_year(100)
```

### Integration Test Example
```python
# tests/integration/test_full_workflow.py
import pytest
from pathlib import Path
from filesbyyear.file_classifier import (
    create_classification_job,
    build_classification_plan,
    execute_classification
)

def test_full_workflow_with_copy_mode(tmp_path):
    """Test complete workflow from analysis to execution."""
    # Setup: Create test files
    (tmp_path / "240315_report.pdf").touch()
    (tmp_path / "240316_data.xlsx").touch()

    # Create job
    job = create_classification_job(
        tmp_path,
        date_source="filename",
        operation_mode="copy"
    )

    # Build plan
    job = build_classification_plan(job)
    assert len(job.file_entries) == 2
    assert job.job_status == "ready"

    # Execute
    job = execute_classification(job)
    assert job.job_status == "completed"
    assert job.processed_files == 2

    # Verify results
    assert (tmp_path / "2024").exists()
    assert (tmp_path / "2024" / "240315_report.pdf").exists()
    assert (tmp_path / "2024" / "240316_data.xlsx").exists()

    # Original files should still exist (copy mode)
    assert (tmp_path / "240315_report.pdf").exists()
```

---

## Debugging Tips

### Enable Debug Logging
```python
from filesbyyear.logger import setup_logging

setup_logging(console_level="DEBUG", file_level="DEBUG")
```

### Print Data Model State
```python
from dataclasses import asdict
import json

# Pretty-print any dataclass
print(json.dumps(asdict(file_entry), indent=2, default=str))
```

### Use pytest fixtures for common setup
```python
# conftest.py
import pytest
from pathlib import Path

@pytest.fixture
def sample_files(tmp_path):
    """Create sample files for testing."""
    files = [
        "240315_report.pdf",
        "240316_data.xlsx",
        "241201_notes.txt"
    ]
    for filename in files:
        (tmp_path / filename).touch()
    return tmp_path
```

---

## Common Development Tasks

### Add a New Date Format

1. **Add pattern to date_detector.py**:
```python
SUPPORTED_FORMATS = {
    "YYYYMMDD": r"(\d{4})(\d{2})(\d{2})",
    "YYMMDD": r"(\d{2})(\d{2})(\d{2})",
    # Add new format here
    "YYYY_MM_DD": r"(\d{4})_(\d{2})_(\d{2})",
}
```

2. **Write tests first**:
```python
def test_detect_pattern_yyyy_mm_dd(tmp_path):
    # Create test files with new format
    (tmp_path / "2024_03_15_report.pdf").touch()
    # Test detection...
```

3. **Run tests and implement**

### Add a New CLI Option

1. **Update cli.py argument parser**
2. **Write integration test for new option**
3. **Update contracts/cli-interface.md documentation**

---

## Resources

- **Specifications**: `specs/FilesSortByYear/`
  - spec.md - Feature specification
  - plan.md - Implementation plan (this is generated by /speckit.plan)
  - research.md - Technical research
  - data-model.md - Data structures
  - contracts/ - API contracts

- **Constitution**: `.specify/memory/constitution.md`
  - Core principles guiding development
  - Non-negotiable requirements (e.g., safety-first)

- **Python Documentation**:
  - pathlib: https://docs.python.org/3/library/pathlib.html
  - datetime: https://docs.python.org/3/library/datetime.html
  - argparse: https://docs.python.org/3/library/argparse.html

---

## Getting Help

- Review specification documents in `specs/FilesSortByYear/`
- Check constitution for design principles
- Look at existing test examples in `tests/`
- Refer to contracts for API signatures

---

## Next Steps

1. ✅ Read this quickstart
2. ✅ Set up development environment
3. ✅ Review spec.md and plan.md
4. ✅ Review data-model.md for data structures
5. → Start with Phase 1: Implement models.py and utils.py (TDD)
6. → Continue with Phase 2: Date detection
7. → Follow implementation order in this guide

Remember: **Tests first, always!** This is non-negotiable per the constitution.
