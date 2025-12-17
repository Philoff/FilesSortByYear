# Python API Contract: FilesByYear Internal Modules

**Feature**: FilesSortByYear
**Date**: 2025-12-15
**Purpose**: Define internal Python API contracts between FilesByYear modules

## Overview

This document specifies the function signatures, input/output contracts, and error handling for FilesByYear's internal Python modules. These are not public APIs but internal contracts between modules.

---

## Module: date_detector.py

### detect_pattern()

**Purpose**: Analyze files and detect dominant date pattern in filenames.

**Signature**:
```python
def detect_pattern(
    directory: Path,
    sample_size: int | None = None,
    min_confidence: float = 0.7
) -> DatePattern | None
```

**Parameters**:
- `directory`: Path to directory containing files
- `sample_size`: Number of files to sample (None = auto-calculate)
- `min_confidence`: Minimum confidence threshold (0.0-1.0)

**Returns**:
- `DatePattern` object if pattern detected with confidence ≥ min_confidence
- `None` if no pattern detected

**Raises**:
- `ValueError`: If directory doesn't exist or is not accessible
- `ValueError`: If min_confidence not in range [0.0, 1.0]

**Example**:
```python
pattern = detect_pattern(Path("/documents"), sample_size=100, min_confidence=0.7)
if pattern:
    print(f"Detected: {pattern.format_string} (confidence: {pattern.confidence_score})")
```

---

### extract_date_from_filename()

**Purpose**: Extract date components from filename using a specific pattern.

**Signature**:
```python
def extract_date_from_filename(
    filename: str,
    pattern: DatePattern
) -> tuple[int, int, int] | None
```

**Parameters**:
- `filename`: Filename to parse
- `pattern`: DatePattern object specifying format

**Returns**:
- `(year, month, day)` tuple if successfully extracted and valid
- `None` if extraction failed or date invalid

**Raises**:
- None (returns None on failure)

**Example**:
```python
date = extract_date_from_filename("240315_report.pdf", pattern)
if date:
    year, month, day = date
    print(f"Extracted: {year}-{month:02d}-{day:02d}")
```

---

### get_supported_formats()

**Purpose**: Get list of all supported date format patterns.

**Signature**:
```python
def get_supported_formats() -> list[str]
```

**Returns**:
- List of format strings (e.g., ["YYYYMMDD", "YYMMDD", "DDMMYYYY", ...])

**Example**:
```python
formats = get_supported_formats()
print(f"Supported formats: {', '.join(formats)}")
```

---

## Module: file_classifier.py

### create_classification_job()

**Purpose**: Initialize a new classification job.

**Signature**:
```python
def create_classification_job(
    source_directory: Path,
    date_source: str = "auto",
    operation_mode: str = "copy",
    detected_pattern: DatePattern | None = None,
    manual_pattern: str | None = None,
    preview_mode: bool = False
) -> ClassificationJob
```

**Parameters**:
- `source_directory`: Directory to classify
- `date_source`: One of "auto", "filename", "ctime", "mtime"
- `operation_mode`: "copy" or "move"
- `detected_pattern`: DatePattern if auto-detected
- `manual_pattern`: Manual format override (e.g., "YYYYMMDD")
- `preview_mode`: If True, no actual file operations

**Returns**:
- `ClassificationJob` object in "created" state

**Raises**:
- `ValueError`: If parameters invalid
- `FileNotFoundError`: If source_directory doesn't exist

**Example**:
```python
job = create_classification_job(
    Path("/documents"),
    date_source="auto",
    operation_mode="copy",
    detected_pattern=pattern,
    preview_mode=False
)
```

---

### build_classification_plan()

**Purpose**: Analyze all files and build operation plan.

**Signature**:
```python
def build_classification_plan(job: ClassificationJob) -> ClassificationJob
```

**Parameters**:
- `job`: ClassificationJob in "created" state

**Returns**:
- Updated ClassificationJob in "ready" state with all FileEntry objects populated

**Raises**:
- `ValueError`: If job not in "created" state
- `RuntimeError`: If pattern detection fails and no fallback available

**Side Effects**:
- Scans source_directory
- Creates FileEntry for each file
- Extracts dates and determines destinations
- Detects conflicts

**Example**:
```python
job = create_classification_job(...)
job = build_classification_plan(job)
print(f"Plan created: {len(job.file_entries)} files")
```

---

### execute_classification()

**Purpose**: Execute the classification plan (copy/move files).

**Signature**:
```python
def execute_classification(job: ClassificationJob) -> ClassificationJob
```

**Parameters**:
- `job`: ClassificationJob in "ready" state

**Returns**:
- Updated ClassificationJob in "completed" state

**Raises**:
- `ValueError`: If job not in "ready" state
- `ValueError`: If job.preview_mode is True

**Side Effects**:
- Creates year subdirectories
- Copies or moves files
- Generates OperationLog entries
- Updates FileEntry statuses
- Writes to audit log file

**Example**:
```python
job = build_classification_plan(job)
# User reviews plan
job = execute_classification(job)
print(f"Completed: {job.processed_files}/{job.total_files} files")
```

---

### generate_classification_report()

**Purpose**: Create human-readable summary report from job results.

**Signature**:
```python
def generate_classification_report(job: ClassificationJob) -> ClassificationReport
```

**Parameters**:
- `job`: Completed ClassificationJob

**Returns**:
- ClassificationReport object

**Raises**:
- None

**Example**:
```python
report = generate_classification_report(job)
print(f"Success rate: {report.successful_count}/{report.total_files}")
```

---

## Module: file_operations.py

### safe_copy_file()

**Purpose**: Safely copy a file with verification.

**Signature**:
```python
def safe_copy_file(
    source: Path,
    destination: Path,
    verify: bool = True
) -> OperationLog
```

**Parameters**:
- `source`: Source file path
- `destination`: Destination file path
- `verify`: If True, verify copy succeeded

**Returns**:
- OperationLog entry recording the operation

**Raises**:
- None (errors captured in OperationLog)

**Side Effects**:
- Creates destination directory if needed
- Copies file with metadata preservation
- Optionally verifies file integrity

**Example**:
```python
log = safe_copy_file(
    Path("/documents/file.pdf"),
    Path("/documents/2024/file.pdf"),
    verify=True
)
if log.success:
    print("Copy successful")
```

---

### safe_move_file()

**Purpose**: Safely move a file (copy + verify + delete).

**Signature**:
```python
def safe_move_file(
    source: Path,
    destination: Path
) -> OperationLog
```

**Parameters**:
- `source`: Source file path
- `destination`: Destination file path

**Returns**:
- OperationLog entry recording the operation

**Raises**:
- None (errors captured in OperationLog)

**Side Effects**:
- Copies file to destination
- Verifies copy succeeded
- Deletes source file only if copy verified
- Creates destination directory if needed

**Example**:
```python
log = safe_move_file(
    Path("/documents/file.pdf"),
    Path("/documents/2024/file.pdf")
)
```

---

### create_year_directory()

**Purpose**: Create a year subdirectory if it doesn't exist.

**Signature**:
```python
def create_year_directory(
    base_path: Path,
    year: int
) -> Path
```

**Parameters**:
- `base_path`: Parent directory
- `year`: Year for subdirectory name

**Returns**:
- Path to year directory (created or existing)

**Raises**:
- `PermissionError`: If cannot create directory
- `ValueError`: If year invalid (not 1900-2099)

**Side Effects**:
- Creates directory if doesn't exist

**Example**:
```python
year_dir = create_year_directory(Path("/documents"), 2024)
# Returns: /documents/2024/
```

---

### check_destination_conflict()

**Purpose**: Check if destination file already exists.

**Signature**:
```python
def check_destination_conflict(destination: Path) -> bool
```

**Parameters**:
- `destination`: Proposed destination path

**Returns**:
- `True` if file exists at destination
- `False` if destination is available

**Example**:
```python
if check_destination_conflict(Path("/documents/2024/file.pdf")):
    print("Warning: Destination already exists")
```

---

## Module: utils.py

### validate_date()

**Purpose**: Validate that a date is valid (handles leap years, month lengths).

**Signature**:
```python
def validate_date(year: int, month: int, day: int) -> bool
```

**Parameters**:
- `year`: Year (1900-2099)
- `month`: Month (1-12)
- `day`: Day (1-31, depending on month)

**Returns**:
- `True` if date is valid
- `False` if date is invalid

**Example**:
```python
assert validate_date(2024, 2, 29) == True  # Leap year
assert validate_date(2023, 2, 29) == False # Not leap year
assert validate_date(2024, 13, 1) == False # Invalid month
```

---

### convert_two_digit_year()

**Purpose**: Convert 2-digit year to 4-digit year using century logic.

**Signature**:
```python
def convert_two_digit_year(year: int) -> int
```

**Parameters**:
- `year`: 2-digit year (0-99)

**Returns**:
- 4-digit year following rule: 00-49 → 2000-2049, 50-99 → 1950-1999

**Raises**:
- `ValueError`: If year not in range 0-99

**Example**:
```python
assert convert_two_digit_year(24) == 2024
assert convert_two_digit_year(95) == 1995
```

---

### get_file_dates()

**Purpose**: Get creation and modification timestamps from file.

**Signature**:
```python
def get_file_dates(file_path: Path) -> tuple[datetime, datetime]
```

**Parameters**:
- `file_path`: Path to file

**Returns**:
- `(creation_time, modification_time)` tuple of datetime objects

**Raises**:
- `FileNotFoundError`: If file doesn't exist
- `PermissionError`: If file not accessible

**Example**:
```python
ctime, mtime = get_file_dates(Path("/documents/file.pdf"))
print(f"Created: {ctime}, Modified: {mtime}")
```

---

### format_file_size()

**Purpose**: Format file size in human-readable format (KB, MB, GB).

**Signature**:
```python
def format_file_size(size_bytes: int) -> str
```

**Parameters**:
- `size_bytes`: Size in bytes

**Returns**:
- Formatted string (e.g., "1.5 MB", "234 KB")

**Example**:
```python
assert format_file_size(1536) == "1.5 KB"
assert format_file_size(1048576) == "1.0 MB"
```

---

### format_duration()

**Purpose**: Format duration in human-readable format.

**Signature**:
```python
def format_duration(seconds: float) -> str
```

**Parameters**:
- `seconds`: Duration in seconds

**Returns**:
- Formatted string (e.g., "2m 30s", "1h 15m")

**Example**:
```python
assert format_duration(150) == "2m 30s"
assert format_duration(3665) == "1h 1m 5s"
```

---

## Module: logger.py

### setup_logging()

**Purpose**: Initialize logging system with console and file handlers.

**Signature**:
```python
def setup_logging(
    log_file: Path | None = None,
    console_level: str = "INFO",
    file_level: str = "DEBUG"
) -> None
```

**Parameters**:
- `log_file`: Path to log file (None = default location)
- `console_level`: Console log level ("DEBUG", "INFO", "WARNING", "ERROR")
- `file_level`: File log level

**Returns**:
- None

**Side Effects**:
- Configures Python logging system
- Creates log file if doesn't exist

**Example**:
```python
setup_logging(
    log_file=Path("./filesbyyear.log"),
    console_level="INFO",
    file_level="DEBUG"
)
```

---

### log_operation()

**Purpose**: Log a file operation to audit trail.

**Signature**:
```python
def log_operation(operation_log: OperationLog) -> None
```

**Parameters**:
- `operation_log`: OperationLog entry to write

**Returns**:
- None

**Side Effects**:
- Appends to audit log file (CSV format)
- Writes to Python logger

**Example**:
```python
log = OperationLog(...)
log_operation(log)
```

---

## Error Handling Patterns

### Recoverable Errors
Functions that process multiple items (e.g., file operations) should:
1. Catch exceptions for individual items
2. Log the error
3. Continue processing
4. Return error details in result

Example:
```python
def process_files(files: list[Path]) -> tuple[list[Success], list[Error]]:
    successes = []
    errors = []
    for file in files:
        try:
            result = process_file(file)
            successes.append(result)
        except Exception as e:
            errors.append(Error(file, str(e)))
            logger.error(f"Failed to process {file}: {e}")
    return successes, errors
```

### Fatal Errors
Functions that must succeed or stop should raise exceptions:
- `FileNotFoundError`: File/directory doesn't exist
- `PermissionError`: Access denied
- `ValueError`: Invalid parameters
- `RuntimeError`: Unexpected state

---

## Type Hints

All functions use type hints:
```python
from typing import Optional, Tuple, List
from pathlib import Path
from datetime import datetime

def example_function(
    required_param: str,
    optional_param: Optional[int] = None,
    list_param: List[Path] = []
) -> Tuple[bool, str]:
    ...
```

---

## Testing Contracts

Each function should have:
1. **Unit tests**: Test function in isolation with mocked dependencies
2. **Parametrized tests**: Test with multiple input variations
3. **Edge case tests**: Empty inputs, invalid inputs, boundary values
4. **Error tests**: Verify correct exceptions raised

Example:
```python
@pytest.mark.parametrize("year,month,day,expected", [
    (2024, 2, 29, True),   # Leap year
    (2023, 2, 29, False),  # Not leap year
    (2024, 13, 1, False),  # Invalid month
])
def test_validate_date(year, month, day, expected):
    assert validate_date(year, month, day) == expected
```

---

## Module Dependencies

```
cli.py
  ├─→ date_detector.py
  ├─→ file_classifier.py
  └─→ logger.py

file_classifier.py
  ├─→ date_detector.py
  ├─→ file_operations.py
  ├─→ utils.py
  ├─→ logger.py
  └─→ models.py

file_operations.py
  ├─→ logger.py
  └─→ models.py

date_detector.py
  ├─→ utils.py
  └─→ models.py

utils.py
  └─→ (no dependencies - leaf module)

logger.py
  └─→ (no dependencies - leaf module)

models.py
  └─→ (dataclasses only - no dependencies)
```

**Design principle**: Lower modules don't depend on higher modules (dependency inversion).
