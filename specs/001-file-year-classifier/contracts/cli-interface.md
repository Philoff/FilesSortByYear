# CLI Interface Contract: FilesByYear

**Feature**: FilesSortByYear
**Date**: 2025-12-15
**Purpose**: Define command-line interface contract for FilesByYear tool

## Overview

FilesByYear provides a command-line interface with three main commands: `analyze`, `preview`, and `classify`. All commands follow standard Unix conventions for arguments, flags, and output.

---

## Command Structure

```bash
filesbyyear <command> [options] <directory>
```

**Global Options** (available for all commands):
- `--version`: Display version and exit
- `--help`, `-h`: Display help message
- `--verbose`, `-v`: Enable verbose output
- `--quiet`, `-q`: Suppress non-error output
- `--log-file <path>`: Specify custom log file location (default: `./filesbyyear_operations.log`)

---

## Commands

### 1. analyze

**Purpose**: Detect date patterns in filenames within a directory.

**Signature**:
```bash
filesbyyear analyze [options] <directory>
```

**Arguments**:
- `<directory>`: Path to directory containing files to analyze (required)

**Options**:
- `--sample-size <n>`: Number of files to sample (default: auto-calculated)
- `--min-confidence <float>`: Minimum confidence threshold 0.0-1.0 (default: 0.7)
- `--output-format <format>`: Output format: `text` or `json` (default: `text`)

**Output** (text format):
```
Analyzing directory: /path/to/documents
Scanned 50 files

Detected Pattern:
  Format: YYMMDD
  Separator: _
  Position: prefix
  Confidence: 94%
  Matches: 47/50 files

Example matches:
  - 240315_report.pdf → 2024-03-15
  - 240316_data.xlsx → 2024-03-16
  - 241201_notes.txt → 2024-12-01

Recommendation: Use this pattern for classification
```

**Output** (json format):
```json
{
  "directory": "/path/to/documents",
  "files_scanned": 50,
  "pattern_detected": true,
  "pattern": {
    "format": "YYMMDD",
    "separator": "_",
    "position": "prefix",
    "confidence": 0.94,
    "matches": 47,
    "examples": [
      {"filename": "240315_report.pdf", "parsed_date": "2024-03-15"},
      {"filename": "240316_data.xlsx", "parsed_date": "2024-03-16"}
    ]
  }
}
```

**Exit Codes**:
- `0`: Pattern detected successfully
- `1`: No pattern detected (confidence < threshold)
- `2`: Directory not found or not accessible
- `3`: Invalid arguments

---

### 2. preview

**Purpose**: Show what would happen during classification without modifying files.

**Signature**:
```bash
filesbyyear preview [options] <directory>
```

**Arguments**:
- `<directory>`: Path to directory containing files to classify (required)

**Options**:
- `--source <method>`: Date source - `auto`, `filename`, `ctime`, `mtime` (default: `auto`)
- `--pattern <format>`: Manual pattern override (e.g., `YYYYMMDD`, `DDMMYY`)
- `--mode <operation>`: Operation mode - `copy` or `move` (default: `copy`)
- `--output-format <format>`: Output format: `text` or `json` (default: `text`)

**Output** (text format):
```
Preview: Classification Plan
Directory: /path/to/documents
Mode: copy
Date source: filename (pattern: YYMMDD_)

Proposed Operations:
  2020/ (5 files)
    └─ copy: 200115_old.pdf → 2020/200115_old.pdf
    └─ copy: 200316_archive.xlsx → 2020/200316_archive.xlsx
    ...

  2024/ (920 files)
    └─ copy: 240315_report.pdf → 2024/240315_report.pdf
    └─ copy: 240316_data.xlsx → 2024/240316_data.xlsx
    ...

  2025/ (75 files)
    └─ copy: 250101_new.txt → 2025/250101_new.txt
    ...

Summary:
  Total files: 1000
  Years: 2020, 2021, 2022, 2023, 2024, 2025
  Estimated space: 245 MB

Warnings:
  - 20 files could not be classified (no date detected)
  - 5 potential conflicts (duplicate filenames in same year)

No files were modified. Run 'filesbyyear classify' to execute this plan.
```

**Exit Codes**:
- `0`: Preview generated successfully
- `1`: Warnings present (files that can't be classified)
- `2`: Directory not found or not accessible
- `3`: Invalid arguments
- `4`: Critical errors (e.g., no files can be classified)

---

### 3. classify

**Purpose**: Execute file classification into year-based subdirectories.

**Signature**:
```bash
filesbyyear classify [options] <directory>
```

**Arguments**:
- `<directory>`: Path to directory containing files to classify (required)

**Options**:
- `--source <method>`: Date source - `auto`, `filename`, `ctime`, `mtime` (default: `auto`)
- `--pattern <format>`: Manual pattern override (e.g., `YYYYMMDD`, `DDMMYY`)
- `--mode <operation>`: Operation mode - `copy` or `move` (default: `copy`)
- `--yes`, `-y`: Auto-confirm (skip confirmation prompt)
- `--dry-run`: Same as `preview` command
- `--output-format <format>`: Output format: `text` or `json` (default: `text`)

**Interactive Flow** (without `--yes`):
```bash
$ filesbyyear classify /documents

Analyzing directory: /documents
Pattern detected: YYMMDD_ (confidence: 94%)

Preview: 1000 files will be copied into 6 year folders
- 2020: 5 files
- 2021: 45 files
- 2022: 120 files
- 2023: 250 files
- 2024: 520 files
- 2025: 60 files

Mode: copy (originals will be preserved)
Estimated space required: 245 MB

Proceed with classification? [y/N]: y

Classifying files...
[████████████████████████████████] 1000/1000 (100%)

Classification complete!
  Successfully classified: 950 files
  Failed: 30 files (see log for details)
  Skipped: 20 files

Detailed log: ./filesbyyear_operations.log
```

**Output** (after completion):
```
Classification Report
=====================

Job ID: 550e8400-e29b-41d4-a716-446655440000
Directory: /path/to/documents
Mode: copy
Execution time: 2m 25s

Results:
  Total files: 1000
  Successfully classified: 950
  Failed: 30
  Skipped: 20

Year Folders Created:
  2020/ (5 files, 2.5 MB)
  2021/ (45 files, 18 MB)
  2022/ (120 files, 45 MB)
  2023/ (250 files, 80 MB)
  2024/ (520 files, 180 MB)
  2025/ (60 files, 20 MB)

Errors (30 files):
  Permission denied (15 files):
    - /documents/locked.pdf
    - /documents/protected.xlsx
    ...

  Invalid date (10 files):
    - /documents/321232_invalid.txt (month > 12)
    - /documents/240230_bad.pdf (Feb 30 doesn't exist)
    ...

  Destination conflict (5 files):
    - /documents/240315_report.pdf (already exists in 2024/)
    ...

Skipped (20 files):
  No date detected:
    - /documents/readme.txt
    - /documents/notes.md
    ...

Detailed audit log: /path/to/filesbyyear_operations.log
```

**Exit Codes**:
- `0`: All files successfully classified
- `1`: Some files failed or skipped (partial success)
- `2`: Directory not found or not accessible
- `3`: Invalid arguments
- `4`: User cancelled operation
- `5`: Critical error (most/all files failed)

---

## Environment Variables

- `FOXCAT_LOG_DIR`: Override default log file directory
- `FOXCAT_LOG_LEVEL`: Set log level (`DEBUG`, `INFO`, `WARNING`, `ERROR`)
- `FOXCAT_NO_COLOR`: Disable colored output (set to `1`)

---

## File Naming for Patterns

### Supported Formats

| Format | Description | Example | Parsed Date |
|--------|-------------|---------|-------------|
| YYYYMMDD | 8-digit, year first | 20240315_report.pdf | 2024-03-15 |
| YYMMDD | 6-digit, year first | 240315_report.pdf | 2024-03-15 |
| DDMMYYYY | 8-digit, day first | 15032024_report.pdf | 2024-03-15 |
| DDMMYY | 6-digit, day first | 150324_report.pdf | 2024-03-15 |
| MMDDYYYY | 8-digit, US format | 03152024_report.pdf | 2024-03-15 |
| MMDDYY | 6-digit, US format | 031524_report.pdf | 2024-03-15 |

### Separators

Supported: `-` (hyphen), `_` (underscore), `.` (dot), ` ` (space), or no separator

Examples:
- `2024-03-15_report.pdf`
- `2024_03_15_report.pdf`
- `2024.03.15.report.pdf`
- `20240315_report.pdf`

---

## Error Handling

**User-Friendly Messages**:
- Directory not found: `Error: Directory '/path' does not exist`
- Permission denied: `Error: Cannot access directory '/path' (permission denied)`
- No files found: `Warning: Directory '/path' contains no files`
- Pattern detection failed: `Warning: Could not detect date pattern (confidence too low)`

**Recovery Suggestions**:
```
Error: Could not detect date pattern (confidence: 45%, threshold: 70%)

Suggestions:
  1. Manually specify pattern: filesbyyear classify --pattern YYYYMMDD /documents
  2. Use filesystem dates: filesbyyear classify --source ctime /documents
  3. Lower confidence threshold: filesbyyear analyze --min-confidence 0.4 /documents
```

---

## Progress Reporting

For long operations (>100 files):
```
Classifying files...
[████████████████░░░░░░░░░░░░░░░░] 523/1000 (52%) - ETA: 1m 15s
```

**Progress indicators**:
- Visual progress bar
- Current/total count
- Percentage
- Estimated time remaining (ETA)

---

## JSON Output Schema

For `--output-format json`:

**analyze command**:
```json
{
  "command": "analyze",
  "directory": "/path/to/documents",
  "files_scanned": 50,
  "pattern_detected": true,
  "pattern": {
    "format": "YYMMDD",
    "separator": "_",
    "position": "prefix",
    "confidence": 0.94,
    "matches": 47,
    "examples": [...]
  }
}
```

**classify command (final report)**:
```json
{
  "command": "classify",
  "job_id": "550e8400-e29b-41d4-a716-446655440000",
  "directory": "/path/to/documents",
  "mode": "copy",
  "execution_time_seconds": 145.3,
  "summary": {
    "total_files": 1000,
    "successful": 950,
    "failed": 30,
    "skipped": 20
  },
  "years_created": [2020, 2021, 2022, 2023, 2024, 2025],
  "errors": [
    {
      "type": "permission_denied",
      "count": 15,
      "files": ["/documents/locked.pdf", ...]
    },
    {
      "type": "invalid_date",
      "count": 10,
      "files": ["/documents/321232_invalid.txt", ...]
    }
  ],
  "log_file": "/path/to/filesbyyear_operations.log"
}
```

---

## Compatibility

- **Shell compatibility**: Works with bash, zsh, fish, PowerShell, cmd.exe
- **Path handling**: Supports absolute and relative paths, handles spaces correctly
- **Unicode**: Full Unicode support for directory names and filenames
- **Exit codes**: Standard Unix conventions for scripting integration

---

## Examples

```bash
# Simple analysis
filesbyyear analyze /documents

# Preview before executing
filesbyyear preview /documents

# Classify with default settings (copy mode, auto-detect)
filesbyyear classify /documents

# Move files (destructive!) with manual pattern
filesbyyear classify --mode move --pattern YYYYMMDD /documents

# Use filesystem modification time
filesbyyear classify --source mtime /documents

# Auto-confirm for scripting
filesbyyear classify --yes /documents

# JSON output for parsing
filesbyyear analyze --output-format json /documents | jq '.pattern.confidence'

# Verbose output for debugging
filesbyyear classify --verbose /documents
```
