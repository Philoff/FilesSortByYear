# Research: File Year Classifier (FilesByYear)

**Feature**: FilesSortByYear
**Date**: 2025-12-15
**Purpose**: Research technical decisions and best practices for implementation

## 1. Date Pattern Detection Strategy

### Decision
Use regex-based pattern matching with statistical confidence scoring based on match frequency and consistency.

### Rationale
- **Regex efficiency**: Python's `re` module is optimized for pattern matching, perfect for identifying date sequences
- **Statistical approach**: By analyzing multiple files, we can determine dominant patterns with confidence scores
- **Flexibility**: Regex patterns can easily accommodate different separators and formats
- **No ML overhead**: Simple statistical analysis avoids complexity of machine learning

### Implementation Approach
1. Define regex patterns for each supported format (YYYYMMDD, YYMMDD, DDMMYYYY, etc.)
2. Scan sample of filenames (adaptive: 50-200 files depending on folder size)
3. Calculate match frequency for each pattern
4. Compute confidence score based on:
   - Match percentage (matches / total files scanned)
   - Consistency (variance in date positions within matches)
   - Date validity (percentage of matched dates that are valid)
5. Recommend pattern with highest confidence if > 70%

### Alternatives Considered
- **Machine Learning**: Rejected - overly complex for this use case, requires training data
- **Heuristic rules only**: Rejected - less accurate than statistical approach
- **Manual pattern only**: Rejected - defeats automation purpose

---

## 2. Cross-Platform Path Handling

### Decision
Use `pathlib.Path` from Python standard library for all path operations.

### Rationale
- **Native cross-platform**: Automatically handles Windows (`\`) vs Unix (`/`) path separators
- **Type-safe**: Path object methods prevent common path manipulation errors
- **Standard library**: No external dependencies
- **Rich API**: Built-in methods for exists(), is_file(), stat(), etc.

### Implementation Approach
```python
from pathlib import Path

# Convert user input to Path object immediately
source_dir = Path(user_input).resolve()

# All operations use Path methods
for file_path in source_dir.iterdir():
    if file_path.is_file():
        # Process file
```

### Alternatives Considered
- **os.path module**: Rejected - more error-prone, requires manual separator handling
- **Third-party path library**: Rejected - unnecessary dependency

---

## 3. Safe File Operations

### Decision
Implement multi-stage safety checks with atomic operations and rollback capability.

### Rationale
- **User trust**: File operations are irreversible - safety is paramount
- **Preview first**: Dry-run shows exactly what will happen
- **Atomic operations**: Use `shutil.copy2()` (preserves metadata) and verify before any deletes
- **Audit trail**: Log every operation with timestamp

### Implementation Approach
1. **Preview Phase**:
   - Build complete operation plan
   - Check for conflicts (duplicate filenames)
   - Calculate space requirements
   - Display plan to user for approval

2. **Execution Phase**:
   - For each file operation:
     - Verify source exists and is readable
     - Check destination space available
     - Create destination directory if needed
     - Copy file (shutil.copy2 preserves timestamps)
     - Verify copy succeeded (compare size/checksum if critical)
     - Log success
     - Only delete source if move mode AND copy verified

3. **Error Handling**:
   - Continue processing on individual file errors
   - Collect all errors for final report
   - Never leave partial operations

### Alternatives Considered
- **Direct shutil.move()**: Rejected - less safe, no verification step
- **Transaction log with rollback**: Rejected - overly complex for file operations
- **Copy-all-then-verify**: Rejected - requires double the space

---

## 4. Date Validation Strategy

### Decision
Use Python's `datetime` module for date validation with custom rules for edge cases.

### Rationale
- **Built-in validation**: `datetime.date()` raises ValueError for invalid dates
- **Standards-compliant**: Handles leap years, month lengths correctly
- **Performance**: Fast enough for thousands of files

### Implementation Approach
```python
from datetime import datetime

def validate_date(year, month, day):
    try:
        datetime(year, month, day)
        return True
    except ValueError:
        return False
```

**Special handling**:
- 2-digit years: 00-49 → 2000-2049, 50-99 → 1950-1999
- Invalid dates (e.g., Feb 30) → rejected, file reported in errors
- Future dates: Allowed (user might be planning ahead)

### Alternatives Considered
- **Simple range checks**: Rejected - doesn't handle leap years correctly
- **Third-party date library**: Rejected - unnecessary dependency

---

## 5. Performance Optimization

### Decision
Use sampling for large directories and lazy iteration for file processing.

### Rationale
- **Sampling efficiency**: Don't need to analyze ALL files to detect pattern
- **Memory efficiency**: Process files one at a time, don't load entire directory
- **User feedback**: Show progress for long operations

### Implementation Approach
1. **Pattern Detection Sampling**:
   - Directories < 100 files: Analyze all
   - Directories 100-1000 files: Sample 100 files (random)
   - Directories > 1000 files: Sample 200 files (random)

2. **File Processing**:
   - Use `Path.iterdir()` generator (lazy iteration)
   - Process one file at a time
   - Show progress bar (simple text-based: "Processing 523/1000")

3. **Memory Management**:
   - Don't load file contents (only work with paths and metadata)
   - Clear processed file entries after logging

### Alternatives Considered
- **Analyze all files always**: Rejected - unnecessary for large directories
- **Parallel processing**: Rejected - adds complexity, I/O bound anyway
- **Database for tracking**: Rejected - overkill, simple list is sufficient

---

## 6. CLI Interface Design

### Decision
Use `argparse` with subcommands pattern: `filesbyyear analyze`, `filesbyyear classify`, `filesbyyear preview`.

### Rationale
- **Clear intent**: Subcommands make action explicit
- **Standard library**: argparse is battle-tested
- **Help system**: Built-in `--help` for each subcommand
- **Extensibility**: Easy to add new commands later

### Implementation Approach
```bash
# Analyze folder to detect pattern
filesbyyear analyze /path/to/folder

# Classify with detected pattern
filesbyyear classify /path/to/folder --mode copy

# Preview without executing
filesbyyear preview /path/to/folder

# Classify with manual pattern
filesbyyear classify /path/to/folder --pattern YYYYMMDD --mode move

# Use filesystem dates
filesbyyear classify /path/to/folder --source ctime
```

**User Flow**:
1. User runs `filesbyyear analyze <folder>`
2. System shows detected pattern with confidence
3. User runs `filesbyyear preview <folder>` to see plan
4. User runs `filesbyyear classify <folder>` to execute

### Alternatives Considered
- **Single command with flags**: Rejected - less discoverable
- **Interactive wizard**: Considered for future enhancement
- **Configuration file**: Rejected - unnecessary for one-time operations

---

## 7. Logging and Audit Trail

### Decision
Use Python's `logging` module with dual output: console (INFO+) and file (DEBUG+).

### Rationale
- **Standard library**: Built-in logging is comprehensive
- **Dual output**: Users see important info, detailed log for troubleshooting
- **Structured**: Log format includes timestamp, level, module, message

### Implementation Approach
```python
import logging

# Console handler - INFO and above
console_handler = logging.StreamHandler()
console_handler.setLevel(logging.INFO)

# File handler - DEBUG and above
file_handler = logging.FileHandler('filesbyyear_operations.log')
file_handler.setLevel(logging.DEBUG)

# Format: 2025-12-15 10:30:45 - INFO - filesbyyear.file_operations - Copied: file.txt -> 2024/file.txt
```

**Log entries**:
- Every file operation (copy/move) with source and destination
- Pattern detection results
- Errors and warnings
- Performance metrics (files processed, time taken)

### Alternatives Considered
- **Simple print statements**: Rejected - not structured, can't filter by level
- **Third-party logging library**: Rejected - standard logging is sufficient

---

## 8. Error Handling Philosophy

### Decision
"Continue on error" with comprehensive error reporting at the end.

### Rationale
- **User productivity**: One bad file shouldn't stop processing of 1000 good files
- **Transparency**: Collect and report all errors together
- **Actionable**: Errors include file path and specific reason

### Implementation Approach
1. Wrap each file operation in try-except
2. Log error with details
3. Add to error collection
4. Continue processing
5. Display error summary at end:
   ```
   Classification complete: 950/1000 files processed

   Errors (50 files):
   - /path/file1.txt: Permission denied
   - /path/file2.txt: Invalid date detected (20251332)
   - /path/file3.txt: Destination conflict
   ```

### Alternatives Considered
- **Fail-fast**: Rejected - poor user experience
- **Automatic retry**: Rejected - errors are usually permanent (permissions, invalid data)

---

## 9. Testing Strategy

### Decision
Comprehensive pytest-based testing with fixtures for file system mocking.

### Rationale
- **pytest power**: Fixtures, parametrization, clear assertion messages
- **File system testing**: Use `tmp_path` fixture for isolated test directories
- **Test categories**:
  - Unit tests: Individual functions (date validation, pattern matching)
  - Integration tests: Full workflows (analyze → classify)
  - Cross-platform tests: Path handling on Windows vs Unix

### Implementation Approach
```python
# Unit test example
@pytest.mark.parametrize("year,month,day,expected", [
    (2024, 2, 29, True),   # Leap year
    (2023, 2, 29, False),  # Not leap year
    (2024, 13, 1, False),  # Invalid month
])
def test_date_validation(year, month, day, expected):
    assert validate_date(year, month, day) == expected

# Integration test example
def test_full_classify_workflow(tmp_path):
    # Create test files
    (tmp_path / "240315_report.pdf").touch()
    (tmp_path / "240316_data.xlsx").touch()

    # Run classification
    result = classify_files(tmp_path, mode="copy")

    # Verify year folders created
    assert (tmp_path / "2024").exists()
    assert len(list((tmp_path / "2024").iterdir())) == 2
```

### Alternatives Considered
- **unittest module**: Rejected - pytest is more powerful and concise
- **Manual testing only**: Rejected - insufficient for reliability

---

## 10. Unicode and Special Characters

### Decision
Use UTF-8 encoding throughout, rely on pathlib's native Unicode support.

### Rationale
- **Modern standard**: UTF-8 is universal
- **pathlib handles it**: Path objects work with Unicode on all platforms
- **Windows support**: Python 3.9+ handles Windows Unicode paths correctly

### Implementation Approach
- All string operations assume UTF-8
- No explicit encoding/decoding needed with pathlib
- Test with Unicode filenames: "rapport_été_2024.pdf", "文件_240315.txt"

### Alternatives Considered
- **ASCII-only**: Rejected - too restrictive
- **Explicit encoding management**: Rejected - pathlib abstracts this

---

## Summary of Key Technologies

| Component | Technology | Justification |
|-----------|-----------|---------------|
| Language | Python 3.9+ | Cross-platform, rich standard library, readable |
| Path handling | pathlib | Native cross-platform support |
| Pattern matching | re (regex) | Efficient, flexible, standard library |
| Date validation | datetime | Standards-compliant, built-in |
| CLI framework | argparse | Standard library, powerful, familiar |
| Logging | logging module | Structured, dual-output capability |
| Testing | pytest | Best-in-class Python testing framework |
| File operations | shutil | Safe, metadata-preserving operations |

**Total external dependencies**: 2 (pytest, pytest-cov) - both dev-only

---

## Open Questions
**None** - All technical decisions have been resolved through this research phase.
