# Feature Specification: File Year Classifier (FilesByYear)

**Feature Branch**: `FilesSortByYear`
**Created**: 2025-12-15
**Status**: Draft
**Input**: User description: "fait un programme que je puisse lancer depuis mon PC windows ou en ligne de commande sous linux pour classer tous les fichiers d'un dossier en sous dossier par année. L'année peut au choix de l'utilisation être basé sur la date du fichier ou à partir du nom du fichier une analyse de quelques fichiers permet de trouver la facon dont la date est encodé dans le nom selon les différente forme de date possible sur 6 ou 8 car analyse assez de fichier pour être certain de la règle à proposer l'utilisateur peut indiquer le codage analysé par la saisie du format par ex AAMMDD"

## User Scenarios & Testing *(mandatory)*

<!--
  IMPORTANT: User stories should be PRIORITIZED as user journeys ordered by importance.
  Each user story/journey must be INDEPENDENTLY TESTABLE - meaning if you implement just ONE of them,
  you should still have a viable MVP (Minimum Viable Product) that delivers value.
  
  Assign priorities (P1, P2, P3, etc.) to each story, where P1 is the most critical.
  Think of each story as a standalone slice of functionality that can be:
  - Developed independently
  - Tested independently
  - Deployed independently
  - Demonstrated to users independently
-->

### User Story 1 - Automatic Date Pattern Detection (Priority: P1)

As a user, I want the program to automatically analyze my files and detect how dates are encoded in filenames, so I don't have to manually figure out the date format pattern.

**Why this priority**: This is the core value proposition - intelligent automatic detection saves users time and prevents classification errors. Without this, the tool is just a basic file mover.

**Independent Test**: Can be fully tested by running the program in analysis mode on a sample directory and verifying it correctly identifies the date pattern (e.g., "YYMMDD detected with 95% confidence"). Delivers immediate value by showing users what pattern was found.

**Acceptance Scenarios**:

1. **Given** a folder containing 50 files with names like "240315_report.pdf", "240316_data.xlsx", **When** I run the analysis, **Then** the system detects the pattern "YYMMDD" with high confidence (>90%) and shows me example matches
2. **Given** files with mixed date formats (some YYYYMMDD, some YYMMDD), **When** I run the analysis, **Then** the system identifies both patterns, shows their frequency, and recommends the dominant one
3. **Given** a folder with files that have dates in various positions (prefix, suffix, middle), **When** I run the analysis, **Then** the system detects the position and format of dates correctly
4. **Given** files with no detectable date pattern in names, **When** I run the analysis, **Then** the system indicates no pattern found and suggests using file system dates instead

---

### User Story 2 - Classify by File System Date (Priority: P2)

As a user, I want to organize files by year based on their creation or modification date when filenames don't contain dates, so I can still organize my files chronologically.

**Why this priority**: Essential fallback when filename analysis fails. Many files (photos, documents) have meaningful filesystem dates even without dates in names.

**Independent Test**: Can be tested by creating files with specific creation/modification dates and verifying they're organized into correct year folders. Works independently of filename detection.

**Acceptance Scenarios**:

1. **Given** a folder with files having various creation dates from 2020-2025, **When** I choose to classify by creation date, **Then** files are organized into folders "2020/", "2021/", ... "2025/"
2. **Given** the option to choose between creation date or modification date, **When** I select modification date, **Then** the system uses file modification timestamps for classification
3. **Given** files spanning multiple years, **When** classification completes, **Then** I receive a summary report showing how many files were placed in each year folder

---

### User Story 3 - Classify by Filename Date Pattern (Priority: P2)

As a user, I want to organize files by extracting year from filenames based on a detected or manually specified pattern, so files are grouped by the date encoded in their names.

**Why this priority**: Primary use case for files with systematic naming conventions. Equal priority to P2 as it's an alternative to filesystem date classification.

**Independent Test**: Can be tested by providing files with dates in names and verifying correct extraction and organization. Delivers value independently of other features.

**Acceptance Scenarios**:

1. **Given** files with pattern "YYMMDD_filename.ext" and detected pattern confirmed, **When** I run classification, **Then** files are organized by extracted year (converting 2-digit years appropriately)
2. **Given** the detected pattern is incorrect, **When** I manually specify the correct pattern (e.g., "YYYYMMDD"), **Then** the system uses my specified pattern for all files
3. **Given** some files match the pattern and others don't, **When** classification runs, **Then** matching files are organized by year and non-matching files are reported separately
4. **Given** ambiguous 2-digit years (e.g., "25" could be 1925 or 2025), **When** extracting dates, **Then** system applies century conversion logic: years 00-49 are interpreted as 2000-2049, years 50-99 are interpreted as 1950-1999

---

### User Story 4 - Safe File Organization with Preview (Priority: P1)

As a user, I want to see what changes will be made before any files are moved or copied, so I can verify the organization is correct and my files stay safe.

**Why this priority**: Critical for user trust and data safety. Users need confidence their files won't be lost or incorrectly organized. Should be P1 as it's non-negotiable for safety.

**Independent Test**: Can be tested by running preview mode and verifying no actual file operations occur, only preview output is shown. Essential safety feature that works independently.

**Acceptance Scenarios**:

1. **Given** a classification is ready to execute, **When** I run in preview mode, **Then** I see a list of all proposed moves/copies with source and destination paths, but no files are modified
2. **Given** the preview shows the organization plan, **When** I approve and execute, **Then** files are organized exactly as shown in preview
3. **Given** I'm concerned about data safety, **When** I choose copy mode instead of move mode, **Then** original files remain in place and copies are created in year folders
4. **Given** potential filename conflicts (same name in destination), **When** preview runs, **Then** conflicts are identified with proposed resolution strategy

---

### User Story 5 - Cross-Platform Operation (Priority: P3)

As a user, I want to run the same program on both Windows and Linux, so I can use it across my different computers with consistent behavior.

**Why this priority**: Nice to have for flexibility but lower priority than core functionality. Most users will primarily use one OS.

**Independent Test**: Can be tested by running the same command on Windows and Linux with identical input and verifying identical output and behavior.

**Acceptance Scenarios**:

1. **Given** the program installed on Windows, **When** I run `filesbyyear <folder>`, **Then** the program operates correctly with Windows path separators
2. **Given** the program installed on Linux, **When** I run `filesbyyear <folder>`, **Then** the program operates correctly with Unix path separators
3. **Given** a folder path with spaces or special characters, **When** I run on either platform, **Then** paths are correctly handled without errors

---

### Edge Cases

- What happens when a file already exists in the destination year folder with the same name?
- How does the system handle files with invalid dates (e.g., "20251332" - December 32)?
- What happens when analyzing a folder with very few files (< 10)?
- How does the system handle files without read permissions?
- What happens with hidden files or system files?
- How does the system handle symbolic links or shortcuts?
- What happens when the destination year folder cannot be created (permission denied)?
- How does the system handle extremely large folders (10,000+ files)?
- What happens with files that have no extension?
- How does the system handle non-ASCII characters in filenames?

## Requirements *(mandatory)*

<!--
  ACTION REQUIRED: The content in this section represents placeholders.
  Fill them out with the right functional requirements.
-->

### Functional Requirements

- **FR-001**: System MUST analyze a sample of filenames from a specified directory to detect date patterns
- **FR-002**: System MUST support detecting dates in formats with 6 digits (YYMMDD, MMDDYY, DDMMYY) and 8 digits (YYYYMMDD, MMDDYYYY, DDMMYYYY)
- **FR-003**: System MUST detect common date separators in filenames (hyphen, underscore, dot, no separator)
- **FR-004**: System MUST calculate and display confidence level for detected date patterns
- **FR-005**: System MUST allow users to manually specify the date format pattern (e.g., "YYMMDD", "YYYYMMDD")
- **FR-006**: System MUST support classification by file system creation date as an alternative to filename-based dates
- **FR-007**: System MUST support classification by file system modification date as an alternative to filename-based dates
- **FR-008**: System MUST organize files into subdirectories named by year (e.g., "2024/", "2025/")
- **FR-009**: System MUST provide a preview/dry-run mode that shows proposed changes without modifying files
- **FR-010**: System MUST support both copy and move modes for file organization
- **FR-011**: System MUST default to copy mode for safety, with explicit option to enable move mode
- **FR-012**: System MUST validate extracted dates and reject impossible dates (e.g., month > 12, day > 31)
- **FR-013**: System MUST handle 2-digit year conversion with configurable century cutoff rule
- **FR-014**: System MUST detect and report filename conflicts when multiple files would have the same destination path
- **FR-015**: System MUST provide a summary report after classification showing counts of files processed, errors, and skipped files
- **FR-016**: System MUST log all file operations with timestamps for audit purposes
- **FR-017**: System MUST run on Windows command line and Linux terminal with consistent behavior
- **FR-018**: System MUST handle Unicode filenames correctly on both platforms
- **FR-019**: System MUST skip files that cannot be read due to permissions and report them
- **FR-020**: System MUST require user confirmation before executing file operations in non-preview mode

### Key Entities

- **File Entry**: Represents a file to be classified, containing original path, detected/extracted year, proposed destination path, and classification status
- **Date Pattern**: Represents a detected date format pattern, including format string (e.g., "YYMMDD"), separator character, position in filename, confidence score, and example matches
- **Classification Job**: Represents a complete classification operation, including source directory, selected date source (filename/creation/modification), operation mode (copy/move), list of file entries, and operation status
- **Operation Log**: Represents a record of file operations performed, including timestamp, source path, destination path, operation type, and success/failure status

## Success Criteria *(mandatory)*

<!--
  ACTION REQUIRED: Define measurable success criteria.
  These must be technology-agnostic and measurable.
-->

### Measurable Outcomes

- **SC-001**: Users can analyze a folder and receive a detected date pattern recommendation within 5 seconds for folders containing up to 1000 files
- **SC-002**: Pattern detection achieves >90% accuracy when analyzing folders where >70% of files follow a consistent date naming convention
- **SC-003**: Users can classify 1000 files into year folders in under 30 seconds (excluding actual file copy/move time)
- **SC-004**: System correctly handles 100% of valid date formats specified in requirements without misclassification
- **SC-005**: Zero files are lost or corrupted during classification operations when following recommended workflow (preview first, then execute)
- **SC-006**: Users can complete the entire workflow (analyze, preview, execute) in under 10 user interactions for typical use case
- **SC-007**: System produces identical classification results when run on Windows and Linux with the same input files
- **SC-008**: 95% of classification attempts complete without errors for folders with standard file permissions and naming
- **SC-009**: Users can understand the classification report without consulting documentation (report is self-explanatory)
- **SC-010**: System handles folders containing up to 10,000 files without performance degradation or memory issues
