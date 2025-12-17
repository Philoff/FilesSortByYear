# Data Model: File Year Classifier (FilesByYear)

**Feature**: FilesSortByYear
**Date**: 2025-12-15
**Purpose**: Define core data structures and their relationships

## Overview

FilesByYear uses a lightweight, in-memory data model with no persistent storage beyond log files. All entities are Python dataclasses for type safety and clarity.

---

## Core Entities

### 1. DatePattern

Represents a detected date format pattern in filenames.

**Purpose**: Capture the result of pattern detection analysis with confidence metrics.

**Fields**:
- `format_string`: str - The detected format (e.g., "YYYYMMDD", "YYMMDD", "DDMMYYYY")
- `separator`: str | None - Separator character ("-", "_", ".", " ", or None)
- `position`: str - Where date appears ("prefix", "suffix", "middle")
- `regex_pattern`: str - Compiled regex for matching this pattern
- `confidence_score`: float - Statistical confidence (0.0-1.0)
- `match_count`: int - Number of files that matched
- `example_matches`: list[str] - Sample filenames that matched (up to 5)
- `total_scanned`: int - Total files scanned during detection

**Validation Rules**:
- `confidence_score` must be 0.0 ≤ score ≤ 1.0
- `format_string` must be one of the supported formats
- `match_count` ≤ `total_scanned`

**Relationships**:
- Used by `ClassificationJob` to determine how to extract dates

**State Transitions**: Immutable once created

**Example**:
```python
DatePattern(
    format_string="YYMMDD",
    separator="_",
    position="prefix",
    regex_pattern=r"(\d{2})_(\d{2})_(\d{2})",
    confidence_score=0.94,
    match_count=47,
    example_matches=["240315_report.pdf", "240316_data.xlsx", ...],
    total_scanned=50
)
```

---

### 2. FileEntry

Represents a single file to be classified.

**Purpose**: Track all information about a file through the classification workflow.

**Fields**:
- `source_path`: Path - Original file path (pathlib.Path object)
- `filename`: str - Filename without directory
- `extracted_year`: int | None - Year extracted from filename or filesystem
- `extraction_method`: str - How year was extracted ("pattern", "ctime", "mtime", "manual", "failed")
- `destination_path`: Path | None - Target path after classification
- `classification_status`: str - Current status (see states below)
- `error_message`: str | None - Error details if classification failed
- `file_size`: int - Size in bytes
- `file_modified_time`: datetime - Last modification timestamp

**Validation Rules**:
- `source_path` must exist and be a file
- `extracted_year` must be 1900 ≤ year ≤ 2099 if not None
- `classification_status` must be valid state
- `file_size` ≥ 0

**Relationships**:
- Contains in `ClassificationJob.file_entries`
- References `DatePattern` (implicitly via extraction_method)

**State Transitions**:
```
pending → analyzing → classified → completed
                  ↓
                failed (terminal)
                  ↓
              skipped (terminal)
```

**States**:
- `pending`: Initial state, not yet processed
- `analyzing`: Currently being analyzed
- `classified`: Year extracted, destination determined
- `completed`: File operation successfully executed
- `failed`: Error occurred during processing
- `skipped`: Intentionally skipped (permissions, invalid date, etc.)

**Example**:
```python
FileEntry(
    source_path=Path("/documents/240315_report.pdf"),
    filename="240315_report.pdf",
    extracted_year=2024,
    extraction_method="pattern",
    destination_path=Path("/documents/2024/240315_report.pdf"),
    classification_status="classified",
    error_message=None,
    file_size=1024576,
    file_modified_time=datetime(2024, 3, 15, 14, 30)
)
```

---

### 3. ClassificationJob

Represents a complete classification operation from start to finish.

**Purpose**: Coordinate the entire workflow and maintain operation state.

**Fields**:
- `job_id`: str - Unique identifier (UUID)
- `source_directory`: Path - Root directory being classified
- `date_source`: str - Where to get dates ("auto", "filename", "ctime", "mtime")
- `operation_mode`: str - "copy" or "move"
- `detected_pattern`: DatePattern | None - Pattern if auto-detected
- `manual_pattern`: str | None - User-specified pattern override
- `file_entries`: list[FileEntry] - All files being processed
- `job_status`: str - Overall job state (see states below)
- `preview_mode`: bool - If True, no actual file operations
- `created_at`: datetime - When job was created
- `started_at`: datetime | None - When processing started
- `completed_at`: datetime | None - When processing finished
- `total_files`: int - Total files to process
- `processed_files`: int - Files successfully processed
- `failed_files`: int - Files that failed
- `skipped_files`: int - Files intentionally skipped

**Validation Rules**:
- `date_source` must be one of: "auto", "filename", "ctime", "mtime"
- `operation_mode` must be "copy" or "move"
- If `date_source` == "filename", either `detected_pattern` or `manual_pattern` required
- `total_files` == len(`file_entries`)
- `processed_files` + `failed_files` + `skipped_files` ≤ `total_files`

**Relationships**:
- Contains many `FileEntry` objects
- References `DatePattern` if pattern detected
- Generates `OperationLog` entries

**State Transitions**:
```
created → detecting → planning → ready → executing → completed
                                               ↓
                                          failed (terminal)
```

**States**:
- `created`: Job initialized, no processing yet
- `detecting`: Analyzing files for date patterns
- `planning`: Building operation plan (preview)
- `ready`: Plan approved, waiting for execution
- `executing`: File operations in progress
- `completed`: All operations finished (may have individual failures)
- `failed`: Job-level failure (e.g., source directory not accessible)

**Example**:
```python
ClassificationJob(
    job_id="550e8400-e29b-41d4-a716-446655440000",
    source_directory=Path("/documents"),
    date_source="auto",
    operation_mode="copy",
    detected_pattern=DatePattern(...),
    manual_pattern=None,
    file_entries=[FileEntry(...), FileEntry(...), ...],
    job_status="completed",
    preview_mode=False,
    created_at=datetime(2025, 12, 15, 10, 0),
    started_at=datetime(2025, 12, 15, 10, 0, 5),
    completed_at=datetime(2025, 12, 15, 10, 2, 30),
    total_files=1000,
    processed_files=950,
    failed_files=30,
    skipped_files=20
)
```

---

### 4. OperationLog

Represents a single logged file operation for audit trail.

**Purpose**: Provide complete audit trail of all file system changes.

**Fields**:
- `log_id`: str - Unique identifier (UUID)
- `job_id`: str - Reference to ClassificationJob
- `timestamp`: datetime - When operation occurred
- `operation_type`: str - "copy", "move", "mkdir", "skip"
- `source_path`: Path - Original file location
- `destination_path`: Path | None - Target location (None for skip)
- `success`: bool - Whether operation succeeded
- `error_message`: str | None - Error details if failed
- `file_size`: int - Size of file operated on
- `checksum`: str | None - File hash for verification (optional)

**Validation Rules**:
- `operation_type` must be valid type
- If `operation_type` in ["copy", "move"], `destination_path` required
- `timestamp` must be ≤ current time
- If `success` == False, `error_message` required

**Relationships**:
- References `ClassificationJob` via `job_id`
- One per `FileEntry` operation

**State Transitions**: Immutable once written (append-only log)

**Example**:
```python
OperationLog(
    log_id="660f9511-f39c-52e5-b827-557766551111",
    job_id="550e8400-e29b-41d4-a716-446655440000",
    timestamp=datetime(2025, 12, 15, 10, 1, 15),
    operation_type="copy",
    source_path=Path("/documents/240315_report.pdf"),
    destination_path=Path("/documents/2024/240315_report.pdf"),
    success=True,
    error_message=None,
    file_size=1024576,
    checksum="a3b2c1d4e5f6..."
)
```

---

### 5. ClassificationReport

Represents the final summary report presented to the user.

**Purpose**: Provide human-readable summary of classification results.

**Fields**:
- `job_id`: str - Reference to ClassificationJob
- `total_files`: int - Total files processed
- `successful_count`: int - Successfully classified
- `failed_count`: int - Failed to classify
- `skipped_count`: int - Intentionally skipped
- `years_created`: list[int] - Year folders created
- `errors_by_type`: dict[str, int] - Error counts by category
- `execution_time_seconds`: float - Total time taken
- `preview_mode`: bool - Whether this was a preview run
- `operation_mode`: str - "copy" or "move"

**Validation Rules**:
- `total_files` == `successful_count` + `failed_count` + `skipped_count`
- `years_created` must be sorted
- `execution_time_seconds` ≥ 0

**Relationships**:
- Generated from `ClassificationJob`
- Aggregates `FileEntry` statuses
- Aggregates `OperationLog` results

**State Transitions**: Immutable once generated

**Example**:
```python
ClassificationReport(
    job_id="550e8400-e29b-41d4-a716-446655440000",
    total_files=1000,
    successful_count=950,
    failed_count=30,
    skipped_count=20,
    years_created=[2020, 2021, 2022, 2023, 2024, 2025],
    errors_by_type={
        "permission_denied": 15,
        "invalid_date": 10,
        "destination_conflict": 5
    },
    execution_time_seconds=145.3,
    preview_mode=False,
    operation_mode="copy"
)
```

---

## Entity Relationship Diagram

```
ClassificationJob (1) ─┬─ contains ─→ (many) FileEntry
                       │
                       ├─ references ─→ (0..1) DatePattern
                       │
                       └─ generates ─→ (many) OperationLog
                       │
                       └─ produces ─→ (1) ClassificationReport
```

---

## Data Flow

1. **Job Creation**:
   - User provides source directory and preferences
   - `ClassificationJob` created with status="created"
   - Source directory scanned, `FileEntry` objects created for each file

2. **Pattern Detection** (if date_source="auto"):
   - Sample files analyzed
   - `DatePattern` object created with confidence score
   - Job status → "detecting"

3. **Planning**:
   - For each `FileEntry`:
     - Extract year using pattern or filesystem dates
     - Determine destination path
     - Check for conflicts
     - Update entry status to "classified"
   - Job status → "planning" → "ready"

4. **Execution** (if not preview_mode):
   - Job status → "executing"
   - For each `FileEntry`:
     - Perform file operation
     - Create `OperationLog` entry
     - Update entry status to "completed" or "failed"
   - Job status → "completed"

5. **Reporting**:
   - Generate `ClassificationReport` from job results
   - Display to user
   - Write audit log to file

---

## Serialization

**JSON Export** (for preview and debugging):
```json
{
  "job_id": "550e8400-e29b-41d4-a716-446655440000",
  "source_directory": "/documents",
  "date_source": "auto",
  "detected_pattern": {
    "format": "YYMMDD",
    "confidence": 0.94
  },
  "total_files": 1000,
  "status": "completed",
  "summary": {
    "successful": 950,
    "failed": 30,
    "skipped": 20
  }
}
```

**Log File Format** (CSV for audit trail):
```csv
timestamp,job_id,operation,source,destination,success,error
2025-12-15T10:01:15,550e8400-...,copy,/documents/file1.pdf,/documents/2024/file1.pdf,true,
2025-12-15T10:01:16,550e8400-...,copy,/documents/file2.pdf,/documents/2024/file2.pdf,false,permission_denied
```

---

## Memory Considerations

**For 10,000 files**:
- `FileEntry`: ~500 bytes each → ~5 MB
- `OperationLog`: ~300 bytes each → ~3 MB
- Total in-memory: ~10 MB (well within constraints)

**Optimization**:
- Process files in batches if memory becomes an issue
- Clear completed `OperationLog` entries from memory after writing to disk
- Use generators for file iteration

---

## Validation Summary

All entities include validation at construction time:
- Type checking via dataclass annotations
- Range validation for numeric fields
- State consistency checks
- Required field enforcement

Invalid data raises `ValueError` with descriptive message.
