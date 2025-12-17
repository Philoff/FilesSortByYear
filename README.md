# FilesByYear

A powerful Python utility that intelligently organizes and classifies files into year-based subdirectories. Perfect for managing large file collections, archives, and document libraries by automatically detecting dates and organizing files accordingly.

## Overview

FilesByYear is a file classification system that:

- **Detects date patterns** in filenames (e.g., `20240315_report.pdf` → 2024)
- **Extracts year information** from file metadata (creation date, modification date)
- **Organizes files** into year-based folder structures automatically
- **Provides safe operations** with preview mode and automatic rollback capability
- **Supports multiple modes**: copy or move files
- **Generates detailed logs** for tracking and auditing

## Use Cases
### Case 1: Organize Historical Documents
```bash
# First, see what patterns exist in your file collection
filesbyyear analyze ~/Documents/incoming/

# Preview how files will be organized
filesbyyear preview ~/Documents/incoming/ --source filename

# Execute the organization
filesbyyear classify ~/Documents/incoming/ --mode move
```
### Case 2: Archive Old Files by Year
```bash
# Copy old files to archive organized by year
filesbyyear classify ~/archive_source/ --mode copy

# This creates: archive_source/2024/, archive_source/2023/, etc.
# Original files remain in archive_source/
```
### Case 3: Use File Metadata When Names Lack Dates
```bash
# When filenames don't contain dates, use modification time
filesbyyear preview ~/photos/ --source mtime
# Or creation time
filesbyyear classify ~/photos/ --source ctime --yes
```
### Case 4: Automated Batch Processing
```bash
#!/bin/bash
# Process multiple directories automatically

for dir in ~/documents/*; do
    if [ -d "$dir" ]; then
        echo "Processing: $dir"
        filesbyyear classify "$dir" --yes --quiet
    fi
done
```

## Features

### Smart Date Detection
- Automatically detects common date patterns in filenames (YYYYMMDD, YYYY-MM-DD, etc.)
- Falls back to file metadata if filename patterns aren't found
- Configurable confidence thresholds for pattern detection

### Safe Operations
- **Preview Mode**: See exactly what would happen before making changes
- **Confirmation Prompt**: Review operations before execution
- **Automatic Rollback Scripts**: Undo operations with a single command
- **Detailed Logging**: Complete audit trail of all operations

### Flexible Configuration
- Choose between **Copy** or **Move** operations
- Use different date sources: filename, creation time, modification time, or auto-detect
- Manual pattern specification for edge cases
- Quiet or verbose output modes

## Installation

### From Source

```bash
# Clone or download the project
cd FilesSortByYear/Sources

# Install in development mode
pip install -e .

# Or install directly
python -m pip install -e .
```

### Requirements
- Python 3.9 or higher
- No external dependencies (uses Python standard library only)

## Usage

### Command Structure

```bash
filesbyyear [COMMAND] [OPTIONS]
```

### Available Commands

#### 1. Analyze - Detect date patterns

Scans a directory to detect date patterns in filenames:

```bash
filesbyyear analyze <directory> [OPTIONS]
```

**Options:**
- `--sample-size N`: Number of files to sample (default: auto)
- `--min-confidence 0.0-1.0`: Confidence threshold (default: 0.7)
- `--output-format text|json`: Output format (default: text)

**Examples:**

```bash
# Analyze current directory
filesbyyear analyze .

# Analyze with custom sample size
filesbyyear analyze /path/to/files --sample-size 100

# Get JSON output
filesbyyear analyze /path/to/files --output-format json
```

#### 2. Preview - Show what would happen
Preview the classification plan without modifying any files:
```bash
filesbyyear preview <directory> [OPTIONS]
```
**Options:**
- `--source auto=detect the best chose|filename=search the year in the name of the file|ctime=use the creation date|mtime=use the modification date`: Date source (default: auto) 
- `--pattern PATTERN`: Manual date pattern (e.g., YYYYMMDD , YYYY-MM-DD, DD.MM.YY)
- `--mode copy|move`: Operation mode (default: move)
- `--output-format text|json`: Output format (default: text)

**Examples:**
```bash
# Preview move with automatic date detection
filesbyyear preview /path/to/files

# Preview with move mode
filesbyyear preview /path/to/files --mode move

# Preview using creation date
filesbyyear preview /path/to/files --source ctime

# Preview with custom pattern
filesbyyear preview /path/to/files --pattern YYYYMMDD

# Get JSON output
filesbyyear preview /path/to/files --output-format json
```

#### 3. Classify - Execute the organization
Execute the classification and organize files:
```bash
filesbyyear classify <directory> [OPTIONS]
```
**Options:**
- `--source auto|filename|ctime|mtime`: Date source (default: auto)
- `--pattern PATTERN`: Manual date pattern (e.g., YYYYMMDD)
- `--mode copy|move`: Operation mode (default: move)
- `--yes|-y`: Skip confirmation prompt
- `--output-format text|json`: Output format (default: text)
- `--log-file PATH`: Custom log file path
- `--quiet|-q`: Suppress output

**Examples:**
```bash
# Classify with confirmation prompt
filesbyyear classify /path/to/files

# Classify in copy mode (keeps originals)
filesbyyear classify /path/to/files --mode copy

# Classify without prompt (automated)
filesbyyear classify /path/to/files --yes

# Classify with custom log file
filesbyyear classify /path/to/files --log-file /var/log/filesbyyear.log

# Classify silently
filesbyyear classify /path/to/files --quiet

# Combine options
filesbyyear classify /path/to/files --mode copy --yes --quiet
```

**Output Example:**
```
Rollback script automatically generated: ./logs/rollback_20251216_233209.sh
To undo this operation, run: bash ./logs/rollback_20251216_233209.sh
```

## Rollback Operations

All classification operations are reversible:

```bash
# After running classify, a rollback script is generated
bash ./logs/rollback_20251216_233209.sh

# The script will:
# 1. Move all files back to their original locations
# 2. Remove empty year directories
# 3. Report the rollback progress
```

## Global Options

Available for all commands:

```bash
filesbyyear [COMMAND] [OPTIONS]
  --version              Show version information
  -v, --verbose          Enable verbose output
  -q, --quiet            Suppress non-error output
  --log-file PATH        Path to log file
```

## Output Formats

### Text Format (default)
readable output with formatted tables and summaries:

```bash
filesbyyear analyze . --output-format text
```

### JSON Format
Machine-readable JSON output for integration:

```bash
filesbyyear analyze . --output-format json
```

**Example JSON:**
```json
{
  "files_scanned": 245,
  "pattern": {
    "format_string": "YYYYMMDD",
    "separator": "_",
    "position": "beginning",
    "confidence_score": 0.953,
    "match_count": 220,
    "total_scanned": 245,
    "example_matches": [
      "20240315_report.pdf",
      "20230620_notes.txt"
    ]
  }
}
```

## Error Handling

### Common Exit Codes

- **0**: Success
- **1**: Processing completed with errors or no pattern found
- **2**: Critical error (file not found, permission denied, etc.)
- **3**: Invalid arguments

### Common Errors

**"Directory not found"**
```bash
# Verify the path exists and is accessible
ls -la /path/to/directory
```

**"No date pattern detected"**
```bash
# Check min-confidence threshold or specify pattern manually
filesbyyear analyze . --min-confidence 0.5
filesbyyear classify . --pattern YYYYMMDD
```

**"Permission denied"**
```bash
# Ensure you have read/write access to the directory
chmod u+rw /path/to/directory
```

## Project Structure

```
FilesSortByYear/
├── Sources/
│   ├── src/filesbyyear/
│   │   ├── __init__.py           # Package initialization
│   │   ├── cli.py                # Command-line interface
│   │   ├── date_detector.py      # Date pattern detection
│   │   ├── file_classifier.py    # File classification logic
│   │   ├── file_operations.py    # File move/copy operations
│   │   ├── models.py             # Data models
│   │   ├── logger.py             # Logging setup
│   │   ├── utils.py              # Utility functions
│   │   └── rollback.py           # Rollback functionality
│   ├── tests/                    # Unit tests
│   ├── setup.py                  # Installation script
│   └── README.md                 # This file
└── logs/                         # Operation logs (auto-created)
```

## Development

### Running Tests

```bash
cd Sources
python -m pytest tests/

# With coverage
python -m pytest tests/ --cov=src/filesbyyear
```

### Code Quality

```bash
cd Sources
ruff check .
```

## Troubleshooting

### Files Not Being Organized

1. Check the detected pattern:
   ```bash
   filesbyyear analyze /path/to/files --min-confidence 0.5
   ```

2. Preview before executing:
   ```bash
   filesbyyear preview /path/to/files
   ```

3. Manually specify the pattern:
   ```bash
   filesbyyear classify /path/to/files --pattern YYYYMMDD
   ```

### Undo Organization

```bash
# Find the rollback script
ls -la logs/rollback_*.sh

# Execute it
bash logs/rollback_TIMESTAMP.sh
```

### Debugging Issues

Enable verbose output:
```bash
filesbyyear classify /path/to/files -v --log-file debug.log
```

## License

MIT License - See project documentation for details

## Support

For issues, feature requests, or documentation, please refer to the project repository.

---

**Version**: 1.0.0
**Python**: 3.9+
**Dependencies**: None (uses Python standard library only)