# FilesByYear - Structured Refactoring Action Plan

**Date:** 2025-12-17
**Author:** Code Architecture Review
**Status:** Ready for Implementation
**Estimated Duration:** 5-6 hours (split into 3 phases)

---

## Overview

This document provides a **step-by-step executable plan** to refactor the FilesByYear codebase by:

1. Eliminating 4 sites of redundant ClassificationJob reconstruction (60 lines)
2. Consolidating 3 output formatting functions into a reusable formatter (100 lines saved)
3. Centralizing scattered validation logic (40 lines saved)
4. Introducing operation strategy pattern for extensibility (no line reduction, but +maintenance)
5. Reorganizing project structure for clarity and growth

**Total Code Reduction:** ~150-200 lines
**Total New Abstractions:** 2 (OutputFormatter, OperationStrategy)
**Total New Files:** 2 (`validators.py`, `operations.py`)
**New Tests:** 3+ test files (validators, operations, integration)

---

## Phase 1: Quick Wins - High Impact, Low Effort (1.5 hours)

### Goal
Eliminate the most egregious redundancy: ClassificationJob reconstruction boilerplate across 4 sites.

### Task 1.1: Implement Job Builder Pattern

**File:** `src/filesbyyear/file_classifier.py`

**Step 1.1.1: Add helper function after imports**

Location: After line 21 (after imports)

```python
def _update_classification_job(
    job: ClassificationJob,
    **kwargs
) -> ClassificationJob:
    """
    Create a new ClassificationJob with selective field updates.

    This helper eliminates boilerplate when updating frozen dataclass instances.

    Args:
        job: Existing ClassificationJob to base on
        **kwargs: Fields to override (e.g., date_source="filename", detected_pattern=pattern)

    Returns:
        New ClassificationJob with specified fields updated, rest copied from original

    Examples:
        >>> updated = _update_classification_job(job, date_source="mtime")
        >>> updated2 = _update_classification_job(job, detected_pattern=pattern, date_source="filename")
    """
    # Merge updates with existing job fields
    all_fields = {
        "job_id": kwargs.get("job_id", job.job_id),
        "source_directory": kwargs.get("source_directory", job.source_directory),
        "date_source": kwargs.get("date_source", job.date_source),
        "operation_mode": kwargs.get("operation_mode", job.operation_mode),
        "folder_depth": kwargs.get("folder_depth", job.folder_depth),
        "detected_pattern": kwargs.get("detected_pattern", job.detected_pattern),
        "manual_pattern": kwargs.get("manual_pattern", job.manual_pattern),
        "file_entries": kwargs.get("file_entries", job.file_entries),
        "job_status": kwargs.get("job_status", job.job_status),
        "preview_mode": kwargs.get("preview_mode", job.preview_mode),
        "folder_stats": kwargs.get("folder_stats", job.folder_stats),
        "created_at": kwargs.get("created_at", job.created_at),
        "started_at": kwargs.get("started_at", job.started_at),
        "completed_at": kwargs.get("completed_at", job.completed_at),
        "total_files": kwargs.get("total_files", job.total_files),
        "processed_files": kwargs.get("processed_files", job.processed_files),
        "failed_files": kwargs.get("failed_files", job.failed_files),
        "skipped_files": kwargs.get("skipped_files", job.skipped_files),
    }
    return ClassificationJob(**all_fields)
```

**Step 1.1.2: Replace site 1 (Line 198-217)**

**BEFORE:**
```python
    # Handle manual pattern if provided
    if job.manual_pattern and job.date_source == "filename":
        manual_pattern_obj = create_manual_pattern(job.manual_pattern)
        if manual_pattern_obj:
            job = ClassificationJob(
                job_id=job.job_id,
                source_directory=job.source_directory,
                date_source="filename",
                operation_mode=job.operation_mode,
                folder_depth=job.folder_depth,
                detected_pattern=manual_pattern_obj,
                manual_pattern=job.manual_pattern,
                file_entries=job.file_entries,
                job_status="planning",
                preview_mode=job.preview_mode,
                folder_stats=job.folder_stats,
                created_at=job.created_at,
                started_at=job.started_at,
                completed_at=job.completed_at,
                total_files=job.total_files,
                processed_files=job.processed_files,
                failed_files=job.failed_files,
                skipped_files=job.skipped_files
            )
```

**AFTER:**
```python
    # Handle manual pattern if provided
    if job.manual_pattern and job.date_source == "filename":
        manual_pattern_obj = create_manual_pattern(job.manual_pattern)
        if manual_pattern_obj:
            job = _update_classification_job(
                job,
                date_source="filename",
                detected_pattern=manual_pattern_obj,
                job_status="planning"
            )
```

**Step 1.1.3: Replace site 2 (Line 226-245)**

**BEFORE:**
```python
            if detected:
                # Update job with detected pattern
                job = ClassificationJob(
                    job_id=job.job_id,
                    source_directory=job.source_directory,
                    date_source="filename",  # Switch to filename mode
                    operation_mode=job.operation_mode,
                    folder_depth=job.folder_depth,
                    detected_pattern=detected,
                    manual_pattern=job.manual_pattern,
                    file_entries=job.file_entries,
                    job_status="planning",
                    preview_mode=job.preview_mode,
                    folder_stats=job.folder_stats,
                    created_at=job.created_at,
                    started_at=job.started_at,
                    completed_at=job.completed_at,
                    total_files=job.total_files,
                    processed_files=job.processed_files,
                    failed_files=job.failed_files,
                    skipped_files=job.skipped_files
                )
```

**AFTER:**
```python
            if detected:
                # Update job with detected pattern
                job = _update_classification_job(
                    job,
                    date_source="filename",
                    detected_pattern=detected,
                    job_status="planning"
                )
```

**Step 1.1.4: Replace site 3 (Line 248-267)**

**BEFORE:**
```python
            else:
                # Fall back to mtime if no pattern detected
                job = ClassificationJob(
                    job_id=job.job_id,
                    source_directory=job.source_directory,
                    date_source="mtime",
                    operation_mode=job.operation_mode,
                    folder_depth=job.folder_depth,
                    detected_pattern=None,
                    manual_pattern=job.manual_pattern,
                    file_entries=job.file_entries,
                    job_status="planning",
                    preview_mode=job.preview_mode,
                    folder_stats=job.folder_stats,
                    created_at=job.created_at,
                    started_at=job.started_at,
                    completed_at=job.completed_at,
                    total_files=job.total_files,
                    processed_files=job.processed_files,
                    failed_files=job.failed_files,
                    skipped_files=job.skipped_files
                )
```

**AFTER:**
```python
            else:
                # Fall back to mtime if no pattern detected
                job = _update_classification_job(
                    job,
                    date_source="mtime",
                    detected_pattern=None,
                    job_status="planning"
                )
```

**Step 1.1.5: Replace site 4 (Line 270-289)**

**BEFORE:**
```python
        except Exception as e:
            # Fall back to mtime on error
            job = ClassificationJob(
                job_id=job.job_id,
                source_directory=job.source_directory,
                date_source="mtime",
                operation_mode=job.operation_mode,
                folder_depth=job.folder_depth,
                detected_pattern=None,
                manual_pattern=job.manual_pattern,
                file_entries=job.file_entries,
                job_status="planning",
                preview_mode=job.preview_mode,
                folder_stats=job.folder_stats,
                created_at=job.created_at,
                started_at=job.started_at,
                completed_at=job.completed_at,
                total_files=job.total_files,
                processed_files=job.processed_files,
                failed_files=job.failed_files,
                skipped_files=job.skipped_files
            )
```

**AFTER:**
```python
        except Exception as e:
            # Fall back to mtime on error
            job = _update_classification_job(
                job,
                date_source="mtime",
                detected_pattern=None,
                job_status="planning"
            )
```

**Verification:**
```bash
# Run only classifier tests
pytest tests/unit/test_file_classifier.py -v

# Expected: All tests pass (behavior unchanged)
```

---

### Task 1.2: Implement Output Formatter Utility

**File:** `src/filesbyyear/cli.py`

**Step 1.2.1: Add OutputFormatter class at top of file (after imports, line 26)**

```python
class OutputFormatter:
    """Consistent formatting utilities for command output."""

    WIDTH = 60
    SEPARATOR = "=" * WIDTH

    @staticmethod
    def header(title: str) -> str:
        """Format a section header."""
        return f"{OutputFormatter.SEPARATOR}\n{title}\n{OutputFormatter.SEPARATOR}\n"

    @staticmethod
    def footer() -> str:
        """Format a section footer."""
        return f"\n{OutputFormatter.SEPARATOR}\n"

    @staticmethod
    def key_value(key: str, value, indent: int = 0) -> str:
        """Format key-value pair with consistent spacing.

        Args:
            key: Label for the value
            value: The value to display
            indent: Number of indentation levels (2 spaces each)

        Returns:
            Formatted line
        """
        spacing = "  " * indent
        return f"{spacing}{key:<20} {value}"

    @staticmethod
    def section(title: str, items: list, indent: int = 0) -> str:
        """Format a named section with items.

        Args:
            title: Section title
            items: List of items to display
            indent: Indentation level

        Returns:
            Formatted section
        """
        spacing = "  " * indent
        lines = [f"{spacing}{title}:"]
        for item in items:
            lines.append(f"{spacing}  {item}")
        return "\n".join(lines)


# Usage examples in docstring:
"""
Examples:
    Header:
        OutputFormatter.header("FilesByYear - Analysis")

    Key-value:
        OutputFormatter.key_value("Files Scanned", 100)

    Indented key-value:
        OutputFormatter.key_value("Confidence", "95.5%", indent=1)

    Section:
        OutputFormatter.section("Results", ["Item 1", "Item 2"])

    Combining:
        output = [
            OutputFormatter.header("My Report"),
            OutputFormatter.key_value("Total", 50),
            OutputFormatter.key_value("Success", 48, indent=1),
            OutputFormatter.footer()
        ]
        return "\\n".join(output)
"""
```

**Step 1.2.2: Refactor `format_analyze_output_text` (Line 29-71)**

**BEFORE (43 lines):**
```python
def format_analyze_output_text(pattern: DatePattern, files_scanned: int) -> str:
    """Format analysis output as human-readable text."""
    output = []
    output.append("=" * 60)
    output.append("FilesByYear - Date Pattern Analysis")
    output.append("=" * 60)
    output.append("")
    output.append(f"Files Scanned:      {files_scanned}")
    output.append(f"Pattern Format:     {pattern.format_string}")
    output.append(f"Separator:          {pattern.separator if pattern.separator else '(none)'}")
    output.append(f"Position:           {pattern.position}")
    output.append(f"Confidence:         {pattern.confidence_score:.1%}")
    output.append(f"Matches:            {pattern.match_count}/{pattern.total_scanned}")
    output.append("")

    if pattern.is_ambiguous:
        output.append("[WARNING] Ambiguous Pattern Detected!")
        output.append("")
        output.append("Multiple date formats match equally well:")
        for fmt in pattern.ambiguous_patterns:
            output.append(f"  - {fmt}")
        output.append("")
        output.append("Unable to determine which format is correct!")
        output.append("Please use the --pattern flag to manually specify the format.")
        output.append("")

    output.append("Example Matches:")
    for i, example in enumerate(pattern.example_matches, 1):
        output.append(f"  {i}. {example}")
    output.append("")
    output.append("=" * 60)

    return "\n".join(output)
```

**AFTER (20 lines):**
```python
def format_analyze_output_text(pattern: DatePattern, files_scanned: int) -> str:
    """Format analysis output as human-readable text."""
    lines = [
        OutputFormatter.header("FilesByYear - Date Pattern Analysis"),
        OutputFormatter.key_value("Files Scanned", files_scanned),
        OutputFormatter.key_value("Pattern Format", pattern.format_string),
        OutputFormatter.key_value("Separator", pattern.separator or "(none)"),
        OutputFormatter.key_value("Position", pattern.position),
        OutputFormatter.key_value("Confidence", f"{pattern.confidence_score:.1%}"),
        OutputFormatter.key_value("Matches", f"{pattern.match_count}/{pattern.total_scanned}"),
    ]

    if pattern.is_ambiguous:
        ambig_warning = "\n[WARNING] Ambiguous Pattern Detected!\n\n"
        ambig_warning += "Multiple date formats match equally well:\n"
        for fmt in pattern.ambiguous_patterns:
            ambig_warning += f"  - {fmt}\n"
        ambig_warning += "\nUnable to determine which format is correct!\n"
        ambig_warning += "Please use the --pattern flag to manually specify the format."
        lines.append(ambig_warning)

    examples = [f"{i}. {ex}" for i, ex in enumerate(pattern.example_matches, 1)]
    lines.extend([
        "\nExample Matches:",
        "\n".join(f"  {ex}" for ex in examples),
        OutputFormatter.footer()
    ])

    return "\n".join(lines)
```

**Step 1.2.3: Refactor `format_preview_output_text` (Line 104-175)**

Follow same pattern - use OutputFormatter for headers, footers, and key-value pairs.
Expected reduction: 70 → 30 lines

**Step 1.2.4: Refactor report formatting functions**

Apply same formatter pattern to all report formatting.

**Verification:**
```bash
# Test CLI output formatting
pytest tests/unit/ -k "cli" -v

# Expected: All output tests pass (visual format unchanged)
```

---

## Phase 2: Architecture Improvements (2.5 hours)

### Goal
Introduce clean abstractions for validation and operations, centralizing logic.

### Task 2.1: Create Validators Module

**File:** `src/filesbyyear/validators.py` (NEW FILE)

```python
"""
Centralized validation logic for FilesByYear.

Provides reusable validators for dates, paths, patterns, and configurations.
"""

from datetime import datetime
from pathlib import Path
from typing import Optional


class DateValidator:
    """Centralized date validation logic."""

    @staticmethod
    def is_valid(year: int, month: int, day: int) -> bool:
        """Check if date components form a valid date.

        Handles leap years and month boundaries correctly.

        Args:
            year: Year (any valid year)
            month: Month (1-12)
            day: Day (1-31, depending on month/year)

        Returns:
            True if date is valid, False otherwise

        Examples:
            >>> DateValidator.is_valid(2024, 2, 29)
            True
            >>> DateValidator.is_valid(2023, 2, 29)
            False
        """
        try:
            datetime(year, month, day)
            return True
        except ValueError:
            return False

    @staticmethod
    def validate_or_raise(year: int, month: int, day: int) -> None:
        """Validate date or raise ValueError.

        Args:
            year, month, day: Date components

        Raises:
            ValueError: If date is invalid
        """
        if not DateValidator.is_valid(year, month, day):
            raise ValueError(
                f"Invalid date: year={year}, month={month:02d}, day={day:02d}"
            )


class PathValidator:
    """Centralized path validation logic."""

    @staticmethod
    def is_valid_directory(path: Path) -> bool:
        """Check if path exists and is a directory.

        Args:
            path: Path to check

        Returns:
            True if path exists and is a directory, False otherwise
        """
        try:
            return path.exists() and path.is_dir()
        except (OSError, PermissionError):
            return False

    @staticmethod
    def validate_or_raise(path: Path) -> None:
        """Validate path or raise ValueError.

        Args:
            path: Path to validate

        Raises:
            ValueError: If path doesn't exist or isn't a directory
            PermissionError: If path isn't accessible
        """
        if not path.exists():
            raise ValueError(f"Path does not exist: {path}")
        if not path.is_dir():
            raise ValueError(f"Path is not a directory: {path}")


class PatternValidator:
    """Pattern format validation."""

    VALID_FORMATS = {
        "YYYYMMDD", "YYMMDD",
        "DDMMYYYY", "DDMMYY",
        "MMDDYYYY", "MMDDYY"
    }
    VALID_SEPARATORS = {"-", "_", ".", " ", None}
    VALID_POSITIONS = {"prefix", "suffix", "middle"}

    @staticmethod
    def is_valid_format(fmt: str) -> bool:
        """Check if format string is supported."""
        return fmt in PatternValidator.VALID_FORMATS

    @staticmethod
    def is_valid_separator(sep: Optional[str]) -> bool:
        """Check if separator is valid."""
        return sep in PatternValidator.VALID_SEPARATORS

    @staticmethod
    def is_valid_position(pos: str) -> bool:
        """Check if position is valid."""
        return pos in PatternValidator.VALID_POSITIONS
```

**Verification:**
```bash
# Create and run validator tests
pytest tests/unit/test_validators.py -v

# Expected: All tests pass (new functionality)
```

---

### Task 2.2: Update Existing Files to Use Validators

**File:** `src/filesbyyear/utils.py`

Remove the `validate_date()` function (lines 11-37) as it's now in validators module.

Change from:
```python
def validate_date(year: int, month: int, day: int) -> bool:
    try:
        datetime(year, month, day)
        return True
    except ValueError:
        return False
```

To: (Import and delegate)
```python
from filesbyyear.validators import DateValidator

def validate_date(year: int, month: int, day: int) -> bool:
    """Deprecated: Use DateValidator.is_valid() instead."""
    return DateValidator.is_valid(year, month, day)
```

**File:** `src/filesbyyear/date_detector.py`

In the `detect_pattern()` function, import and use PathValidator:

```python
from filesbyyear.validators import PathValidator

# Replace lines 158-162 with:
if not directory.exists():
    raise ValueError(f"Directory does not exist: {directory}")

if not directory.is_dir():
    raise ValueError(f"Path is not a directory: {directory}")

# BECOMES:
PathValidator.validate_or_raise(directory)
```

**Verification:**
```bash
# Run full test suite
pytest tests/ -v

# Expected: All 270+ tests pass (behavior identical)
```

---

### Task 2.3: Create Operations Strategy Module

**File:** `src/filesbyyear/operations.py` (NEW FILE)

```python
"""
Abstract operation strategy pattern for extensible file operations.

Provides a common interface for copy, move, and future operation types.
"""

from abc import ABC, abstractmethod
from pathlib import Path
from filesbyyear.models import OperationLog


class FileOperation(ABC):
    """Abstract base class for file operations."""

    @abstractmethod
    def execute(
        self,
        source: Path,
        destination: Path,
        job_id: str
    ) -> OperationLog:
        """Execute the operation.

        Args:
            source: Source file path
            destination: Destination file path
            job_id: Job ID for logging

        Returns:
            OperationLog with result details
        """
        pass

    @abstractmethod
    def validate_preconditions(
        self,
        source: Path,
        destination: Path
    ) -> bool:
        """Check if operation can proceed.

        Args:
            source: Source file path
            destination: Destination file path

        Returns:
            True if preconditions are met, False otherwise
        """
        pass

    def validate_or_raise(
        self,
        source: Path,
        destination: Path
    ) -> None:
        """Validate preconditions or raise exception.

        Args:
            source, destination: File paths

        Raises:
            ValueError: If preconditions not met
        """
        if not self.validate_preconditions(source, destination):
            raise ValueError(
                f"Cannot perform {self.__class__.__name__}: "
                f"source={source}, destination={destination}"
            )


class CopyOperation(FileOperation):
    """Strategy for copying files."""

    def execute(
        self,
        source: Path,
        destination: Path,
        job_id: str
    ) -> OperationLog:
        """Execute copy operation.

        Delegates to file_operations.safe_copy_file() with verification.
        """
        from filesbyyear import file_operations
        return file_operations.safe_copy_file(
            source,
            destination,
            verify=True,
            job_id=job_id
        )

    def validate_preconditions(
        self,
        source: Path,
        destination: Path
    ) -> bool:
        """Copy requires source to exist and destination to not exist."""
        return source.exists() and not destination.exists()


class MoveOperation(FileOperation):
    """Strategy for moving files."""

    def execute(
        self,
        source: Path,
        destination: Path,
        job_id: str
    ) -> OperationLog:
        """Execute move operation.

        Delegates to file_operations.safe_move_file().
        """
        from filesbyyear import file_operations
        return file_operations.safe_move_file(
            source,
            destination,
            job_id=job_id
        )

    def validate_preconditions(
        self,
        source: Path,
        destination: Path
    ) -> bool:
        """Move requires source to exist and destination to not exist."""
        return source.exists() and not destination.exists()


class OperationFactory:
    """Factory for creating operation strategy instances."""

    _operations = {
        "copy": CopyOperation(),
        "move": MoveOperation(),
    }

    @classmethod
    def get_operation(cls, mode: str) -> FileOperation:
        """Get operation strategy for specified mode.

        Args:
            mode: Operation mode ("copy" or "move")

        Returns:
            FileOperation instance

        Raises:
            ValueError: If mode is not recognized
        """
        operation = cls._operations.get(mode)
        if not operation:
            raise ValueError(
                f"Unknown operation mode: {mode}. "
                f"Valid modes: {', '.join(cls._operations.keys())}"
            )
        return operation

    @classmethod
    def register_operation(cls, mode: str, operation: FileOperation) -> None:
        """Register a new operation strategy (for extensibility).

        Args:
            mode: Operation mode name
            operation: FileOperation instance
        """
        cls._operations[mode] = operation
```

**Verification:**
```bash
# Create and run operation tests
pytest tests/unit/test_operations.py -v

# Expected: All tests pass (new functionality)
```

---

### Task 2.4: Update file_classifier.py to Use Operation Factory

**File:** `src/filesbyyear/file_classifier.py`

In `execute_classification()` function, replace lines 476-488:

**BEFORE:**
```python
            # Perform copy or move
            if job.operation_mode == "copy":
                operation_log = file_operations.safe_copy_file(
                    entry.source_path,
                    entry.destination_path,
                    verify=True,
                    job_id=job.job_id
                )
            else:  # move
                operation_log = file_operations.safe_move_file(
                    entry.source_path,
                    entry.destination_path,
                    job_id=job.job_id
                )
```

**AFTER:**
```python
            # Perform copy or move using strategy pattern
            from filesbyyear.operations import OperationFactory

            operation = OperationFactory.get_operation(job.operation_mode)
            operation_log = operation.execute(
                entry.source_path,
                entry.destination_path,
                job.job_id
            )
```

**Verification:**
```bash
# Run classifier tests with new factory usage
pytest tests/unit/test_file_classifier.py -v

# Expected: All tests pass (behavior identical)
```

---

## Phase 3: Testing & Documentation (1.5 hours)

### Task 3.1: Add Integration Tests

**File:** `tests/integration/test_full_workflow.py` (NEW FILE)

Create comprehensive integration tests that exercise the full workflow with new abstractions:

```python
"""
Integration tests for FilesByYear complete workflow.

Tests the full pipeline from pattern detection through file operations.
"""

import pytest
from pathlib import Path
from tempfile import TemporaryDirectory

from filesbyyear.file_classifier import (
    create_classification_job,
    build_classification_plan,
    execute_classification
)
from filesbyyear.date_detector import detect_pattern
from filesbyyear.operations import OperationFactory
from filesbyyear.validators import DateValidator, PathValidator


class TestFullWorkflow:
    """Complete workflow integration tests."""

    def test_workflow_with_auto_detection(self):
        """Test full workflow with automatic pattern detection."""
        # Create job, build plan, execute - all with new abstractions
        pass

    def test_workflow_with_manual_pattern(self):
        """Test full workflow with manual pattern specification."""
        pass

    def test_workflow_with_operation_factory(self):
        """Test that operation factory works in real workflow."""
        pass


class TestValidatorIntegration:
    """Test validators used throughout workflow."""

    def test_date_validator_in_extraction(self):
        """Test DateValidator prevents invalid dates."""
        pass

    def test_path_validator_in_job_creation(self):
        """Test PathValidator catches invalid paths early."""
        pass
```

---

### Task 3.2: Add Validator Tests

**File:** `tests/unit/test_validators.py` (NEW FILE)

```python
"""
Unit tests for centralized validators module.
"""

import pytest
from pathlib import Path
from filesbyyear.validators import DateValidator, PathValidator, PatternValidator


class TestDateValidator:
    """Test DateValidator class."""

    def test_is_valid_normal_date(self):
        """Test validation of normal dates."""
        assert DateValidator.is_valid(2024, 3, 15) is True
        assert DateValidator.is_valid(2024, 12, 31) is True
        assert DateValidator.is_valid(2024, 1, 1) is True

    def test_is_valid_leap_year(self):
        """Test leap year handling."""
        assert DateValidator.is_valid(2024, 2, 29) is True
        assert DateValidator.is_valid(2023, 2, 29) is False

    def test_is_valid_invalid_dates(self):
        """Test invalid date detection."""
        assert DateValidator.is_valid(2024, 13, 1) is False
        assert DateValidator.is_valid(2024, 2, 30) is False
        assert DateValidator.is_valid(2024, 4, 31) is False

    def test_validate_or_raise_valid(self):
        """Test that valid dates don't raise."""
        DateValidator.validate_or_raise(2024, 3, 15)  # Should not raise

    def test_validate_or_raise_invalid(self):
        """Test that invalid dates raise ValueError."""
        with pytest.raises(ValueError):
            DateValidator.validate_or_raise(2024, 13, 1)


class TestPathValidator:
    """Test PathValidator class."""

    def test_is_valid_directory_exists(self, tmp_path):
        """Test validation of existing directories."""
        assert PathValidator.is_valid_directory(tmp_path) is True

    def test_is_valid_directory_not_exists(self):
        """Test validation of non-existent paths."""
        assert PathValidator.is_valid_directory(Path("/non/existent/path")) is False

    def test_validate_or_raise_valid(self, tmp_path):
        """Test that valid paths don't raise."""
        PathValidator.validate_or_raise(tmp_path)  # Should not raise

    def test_validate_or_raise_invalid(self):
        """Test that invalid paths raise ValueError."""
        with pytest.raises(ValueError):
            PathValidator.validate_or_raise(Path("/non/existent"))


class TestPatternValidator:
    """Test PatternValidator class."""

    def test_is_valid_format(self):
        """Test format validation."""
        assert PatternValidator.is_valid_format("YYYYMMDD") is True
        assert PatternValidator.is_valid_format("YYMMDD") is True
        assert PatternValidator.is_valid_format("INVALID") is False

    def test_is_valid_separator(self):
        """Test separator validation."""
        assert PatternValidator.is_valid_separator("-") is True
        assert PatternValidator.is_valid_separator(None) is True
        assert PatternValidator.is_valid_separator("@") is False
```

---

### Task 3.3: Add Operation Strategy Tests

**File:** `tests/unit/test_operations.py` (NEW FILE)

```python
"""
Unit tests for operation strategy pattern.
"""

import pytest
from pathlib import Path
from filesbyyear.operations import (
    FileOperation,
    CopyOperation,
    MoveOperation,
    OperationFactory
)


class TestOperationFactory:
    """Test OperationFactory class."""

    def test_get_copy_operation(self):
        """Test retrieving copy operation."""
        op = OperationFactory.get_operation("copy")
        assert isinstance(op, CopyOperation)

    def test_get_move_operation(self):
        """Test retrieving move operation."""
        op = OperationFactory.get_operation("move")
        assert isinstance(op, MoveOperation)

    def test_invalid_operation_raises(self):
        """Test that invalid mode raises ValueError."""
        with pytest.raises(ValueError):
            OperationFactory.get_operation("invalid")


class TestCopyOperation:
    """Test CopyOperation strategy."""

    def test_validate_preconditions_valid(self, tmp_path):
        """Test validation with valid conditions."""
        source = tmp_path / "source.txt"
        source.write_text("content")
        dest = tmp_path / "dest.txt"

        op = CopyOperation()
        assert op.validate_preconditions(source, dest) is True

    def test_validate_preconditions_dest_exists(self, tmp_path):
        """Test validation fails when destination exists."""
        source = tmp_path / "source.txt"
        source.write_text("content")
        dest = tmp_path / "dest.txt"
        dest.write_text("existing")

        op = CopyOperation()
        assert op.validate_preconditions(source, dest) is False


class TestMoveOperation:
    """Test MoveOperation strategy."""

    def test_validate_preconditions_valid(self, tmp_path):
        """Test validation with valid conditions."""
        source = tmp_path / "source.txt"
        source.write_text("content")
        dest = tmp_path / "dest.txt"

        op = MoveOperation()
        assert op.validate_preconditions(source, dest) is True
```

---

### Task 3.4: Create Documentation Files

**File:** `docs/ARCHITECTURE.md`

Document the refactored architecture:
- Component overview
- Design patterns used (Builder, Strategy, Factory)
- Data flow diagrams
- Extension points for new features

**File:** `docs/API.md`

API reference for public functions and classes

**File:** `docs/DEVELOPMENT.md`

Development setup and contribution guidelines

---

## Implementation Checklist

### Phase 1: Quick Wins
- [ ] Implement `_update_classification_job()` helper
- [ ] Replace all 4 ClassificationJob reconstruction sites
- [ ] Test with `pytest tests/unit/test_file_classifier.py`
- [ ] Add OutputFormatter class to cli.py
- [ ] Refactor all 3+ formatting functions
- [ ] Test with `pytest tests/unit/ -k "cli"`
- [ ] Git commit: "Refactor: eliminate ClassificationJob boilerplate and consolidate output formatting"

### Phase 2: Architecture
- [ ] Create `src/filesbyyear/validators.py`
- [ ] Create validator unit tests
- [ ] Update `utils.py` to use validators
- [ ] Update `date_detector.py` to use validators
- [ ] Test with `pytest tests/unit/test_validators.py`
- [ ] Create `src/filesbyyear/operations.py`
- [ ] Create operation unit tests
- [ ] Update `file_classifier.py` to use OperationFactory
- [ ] Test with `pytest tests/unit/test_operations.py`
- [ ] Run full test suite: `pytest tests/`
- [ ] Git commit: "Refactor: introduce validators and operation strategy pattern"

### Phase 3: Testing & Documentation
- [ ] Create `tests/integration/test_full_workflow.py`
- [ ] Create `docs/ARCHITECTURE.md`
- [ ] Create `docs/API.md`
- [ ] Create `docs/DEVELOPMENT.md`
- [ ] Update root README.md with link to docs
- [ ] Run full test suite: `pytest tests/`
- [ ] Run linter: `ruff check .`
- [ ] Git commit: "Docs: add architecture, API, and development guides + integration tests"

### Final Verification
- [ ] All 270+ tests pass
- [ ] Ruff checks pass
- [ ] No linting errors
- [ ] Code coverage maintained or improved
- [ ] All artifacts properly documented

---

## Success Criteria

### Code Quality
- ✅ No redundant ClassificationJob reconstruction (4 sites → 1 abstraction)
- ✅ Consistent output formatting (3+ functions → 1 formatter)
- ✅ Centralized validation (3 locations → 1 module)
- ✅ Clear operation abstraction (enables future extensions)

### Testing
- ✅ All existing 270 tests still pass
- ✅ 30+ new unit tests for validators
- ✅ 20+ new unit tests for operations
- ✅ 10+ integration tests for full workflow
- ✅ Code coverage ≥ 95%

### Documentation
- ✅ Architecture document complete
- ✅ API reference complete
- ✅ Development guide complete
- ✅ All public functions have docstrings

---

## Rollback Plan

If any phase encounters issues:

```bash
# Revert to main branch
git checkout main

# Or revert the specific refactoring commit
git revert <commit-hash>

# Return to working state
pytest tests/  # Verify all pass
```

Each phase is independent enough to be reverted without affecting others.

---

## Post-Refactoring Recommendations

### Short Term (1-2 weeks)
1. Monitor for any edge cases missed during refactoring
2. Get code review and approval from team
3. Deploy to staging environment
4. Gather feedback from end users

### Medium Term (1-2 months)
1. Use new abstractions to add new date formats
2. Add support for additional operation types
3. Expand integration test coverage
4. Create end-user documentation

### Long Term (3-6 months)
1. Consider adding plugin system for operations
2. Add performance profiling and optimization
3. Implement caching for pattern detection
4. Build CLI UX improvements

---

## Timeline Estimate

| Phase | Tasks | Estimated Time | Actual Time |
|-------|-------|-----------------|------------|
| 1: Quick Wins | Job builder, Formatter | 1.5 hours | ___ |
| 2: Architecture | Validators, Operations | 2.5 hours | ___ |
| 3: Testing & Docs | Tests, Documentation | 1.5 hours | ___ |
| Final Verification | Full test run, commit | 0.5 hours | ___ |
| **Total** | | **6 hours** | ___ |

---

## Contact & Support

For questions or issues during refactoring:
1. Review ARCHITECTURE_ANALYSIS.md for detailed rationale
2. Check test files for usage examples
3. Refer to inline code documentation
4. Create issue in project tracker if needed

---

**Document Created:** 2025-12-17
**Last Updated:** 2025-12-17
**Version:** 1.0

