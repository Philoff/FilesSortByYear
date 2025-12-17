# FilesByYear - Security Hardening Plan

**Date:** 2025-12-17
**Status:** In Preparation
**Scope:** Complete security audit and hardening of the codebase

---

## Overview

This plan details security improvements to FilesByYear including:
1. Complete error handling for all edge cases
2. Input validation for all user inputs
3. Path traversal attack prevention
4. Enhanced logging and monitoring
5. Edge case handling (long paths, special characters, corrupted files)
6. Defensive programming patterns throughout

---

## Security Audit Summary

### Identified Vulnerabilities

#### High Priority
1. **Path Traversal Attacks**
   - Destination paths not validated for path traversal (../../)
   - File operations could write outside intended directory
   - Requires: Path normalization and containment checks

2. **Insufficient Error Handling**
   - Permission errors not properly handled
   - Corrupted/inaccessible files cause crashes
   - Requires: Try-except blocks, graceful degradation

3. **Input Validation Missing**
   - User-provided patterns not validated
   - Directory paths not validated
   - Requires: Input sanitization and validation

#### Medium Priority
4. **Inadequate Logging**
   - Security-relevant events not logged
   - Error context insufficient
   - Requires: Structured logging with context

5. **Edge Cases Not Handled**
   - Files without extensions
   - Extremely long file paths
   - Special characters in filenames
   - Requires: Edge case handling

#### Low Priority
6. **Hardened Error Messages**
   - Some error messages too verbose
   - Could leak system information
   - Requires: Sanitized error messages

---

## Implementation Plan

### Phase 1: Enhanced Path Security (Priority: HIGH)

#### 1.1 Path Validation Module
Create `src/filesbyyear/path_security.py` with:

```python
class PathSecurityValidator:
    """Validate paths for security and integrity."""

    @staticmethod
    def is_safe_path(path: Path, base_dir: Path) -> bool:
        """Check if path is within base directory (no traversal)."""
        # Resolve to absolute paths
        # Check if path is within base_dir
        # Detect ../../ attempts

    @staticmethod
    def normalize_path(path: Path) -> Path:
        """Normalize and safely canonicalize path."""
        # Remove .., ., redundant separators
        # Resolve symlinks
        # Check for path traversal

    @staticmethod
    def is_safe_filename(filename: str) -> bool:
        """Check if filename is safe (no directory traversal)."""
        # No path separators
        # No null bytes
        # No leading/trailing spaces
```

#### 1.2 Update file_operations.py
- Add path validation before all file operations
- Verify destination is within source directory tree
- Log all path-related operations
- Reject unsafe paths with clear errors

---

### Phase 2: Input Validation (Priority: HIGH)

#### 2.1 User Input Validation
Create validation in cli.py and file_classifier.py:

```python
def validate_directory_input(path: str) -> Path:
    """Validate and sanitize user-provided directory path."""
    # Convert string to Path
    # Check path exists and is directory
    # Check read permissions
    # Return validated Path or raise ValueError

def validate_pattern_input(pattern: str) -> str:
    """Validate date pattern format."""
    # Check against known formats
    # Reject unknown patterns
    # Return validated pattern or raise ValueError

def validate_operation_mode(mode: str) -> str:
    """Validate operation mode."""
    # Must be 'copy' or 'move'
    # Case-insensitive comparison
    # Return normalized mode or raise ValueError
```

#### 2.2 Update CLI Argument Parsing
- Add custom type converters for validation
- Provide better error messages
- Validate before processing

---

### Phase 3: Complete Error Handling (Priority: HIGH)

#### 3.1 File Operation Errors
Handle in file_operations.py:

```python
class FileOperationError(Exception):
    """Base exception for file operations."""

try:
    # Copy/Move operation
except FileNotFoundError:
    # Log: file doesn't exist
    # Return: OperationLog with failure details
except PermissionError:
    # Log: insufficient permissions
    # Return: OperationLog with failure details
except FileExistsError:
    # Log: destination already exists
    # Return: OperationLog with failure details
except OSError as e:
    # Log: general OS error
    # Return: OperationLog with failure details
```

#### 3.2 Date Extraction Errors
Handle in date_detector.py:

```python
try:
    # Extract date from filename
except ValueError:
    # Invalid date components
    # Return: None (skip file)
except Exception:
    # Unexpected error
    # Log and return: None
```

#### 3.3 Path Errors
Handle in all file operations:

```python
try:
    # Path operations
except ValueError:
    # Invalid path
    # Log and propagate
except PermissionError:
    # Permission denied
    # Log and propagate
```

---

### Phase 4: Enhanced Logging (Priority: MEDIUM)

#### 4.1 Structured Logging
Update logger.py with context information:

```python
class SecurityLogger:
    """Logger with security event tracking."""

    def log_operation(self, operation: str, source: Path, dest: Path, result: bool):
        """Log file operation with security context."""
        # Include: timestamp, user, operation, paths, result
        # Sanitize: paths (no sensitive info leak)

    def log_security_event(self, event_type: str, details: dict):
        """Log security-relevant events."""
        # Invalid path attempts
        # Permission errors
        # Suspicious patterns

    def log_validation_failure(self, reason: str, input_value: str):
        """Log input validation failures."""
        # What failed
        # Why it failed
        # What was rejected (sanitized)
```

#### 4.2 Log Levels
Use appropriate logging levels:
- DEBUG: Detailed operation info
- INFO: Normal operations
- WARNING: Validation failures, permission issues
- ERROR: Operation failures
- CRITICAL: Security events, path traversal attempts

---

### Phase 5: Edge Case Handling (Priority: MEDIUM)

#### 5.1 File Extensions
Handle files without extensions:

```python
def extract_extension(filename: str) -> str:
    """Safely extract file extension."""
    # Handle files without extension
    # Handle hidden files (.bashrc)
    # Return: ".ext" or "" if no extension
```

#### 5.2 Path Length
Handle OS-specific path length limits:

```python
MAX_PATH_LENGTH = 260  # Windows limit
MAX_FILENAME_LENGTH = 255

def check_path_length(path: Path) -> bool:
    """Check if path exceeds OS limits."""
    # Check full path length
    # Check individual component lengths
    # Return: True if valid, False if too long
```

#### 5.3 Special Characters
Handle special characters safely:

```python
def sanitize_filename(filename: str) -> str:
    """Remove/escape problematic characters."""
    # Remove null bytes
    # Remove control characters
    # Warn on unusual characters
```

---

### Phase 6: Hardened Error Messages (Priority: LOW)

#### 6.1 Error Message Sanitization
Prevent information leakage:

```python
def get_user_friendly_error(error: Exception) -> str:
    """Convert exception to user-friendly message."""
    # Map internal errors to user messages
    # Don't leak system paths
    # Don't expose internal details
    # Include: what failed, why, what to do
```

---

## Implementation Steps

### Step 1: Create path_security.py
- [ ] Implement PathSecurityValidator class
- [ ] Add path validation methods
- [ ] Add normalization methods
- [ ] Add tests for security checks

### Step 2: Enhance validators.py
- [ ] Add PathValidator improvements
- [ ] Add comprehensive path checks
- [ ] Add filename validation
- [ ] Add edge case handling

### Step 3: Update file_operations.py
- [ ] Add path validation before operations
- [ ] Add comprehensive error handling
- [ ] Add security logging
- [ ] Test permission errors, corrupted files

### Step 4: Update file_classifier.py
- [ ] Add input validation
- [ ] Add error handling
- [ ] Add operation-level logging
- [ ] Test edge cases

### Step 5: Enhance logger.py
- [ ] Add structured logging
- [ ] Add security event logging
- [ ] Add context information
- [ ] Sanitize sensitive data

### Step 6: Update CLI (cli.py)
- [ ] Add argument validation
- [ ] Add custom type converters
- [ ] Improve error messages
- [ ] Validate all user inputs

---

## Security Checklist

### Path Security
- [ ] Path traversal attacks prevented
- [ ] Paths normalized and canonicalized
- [ ] Symlinks resolved safely
- [ ] Destination verified within source tree
- [ ] Path length validated

### Input Validation
- [ ] Directory paths validated
- [ ] Patterns validated
- [ ] Operation modes validated
- [ ] Filenames sanitized
- [ ] Special characters handled

### Error Handling
- [ ] File not found errors handled
- [ ] Permission errors handled
- [ ] Corrupted file errors handled
- [ ] OS errors handled
- [ ] Unexpected errors logged

### Logging
- [ ] All operations logged
- [ ] Security events logged
- [ ] Errors logged with context
- [ ] Validation failures logged
- [ ] Sensitive data sanitized

### Edge Cases
- [ ] Files without extensions handled
- [ ] Long paths handled
- [ ] Special characters handled
- [ ] Empty files handled
- [ ] Large files handled

---

## Testing Strategy

### Security Testing
```python
def test_path_traversal_attack():
    """Ensure ../../ attempts are blocked."""

def test_permission_error_handled():
    """Ensure permission errors are graceful."""

def test_invalid_path_rejected():
    """Ensure invalid paths are rejected."""

def test_malformed_pattern_rejected():
    """Ensure malformed patterns are rejected."""
```

### Edge Case Testing
```python
def test_file_without_extension():
    """Handle files with no extension."""

def test_very_long_filename():
    """Handle extremely long filenames."""

def test_special_characters_in_filename():
    """Handle special characters safely."""

def test_corrupted_file_readable():
    """Handle files that can't be read."""
```

---

## Security Guidelines

### 1. Defense in Depth
- Validate at every layer
- Don't trust external inputs
- Log all security-relevant events
- Fail securely

### 2. Principle of Least Privilege
- Only read/write needed files
- Don't escalate permissions
- Warn on unusual operations

### 3. Secure Defaults
- Default to strictest security checks
- Require explicit allowance for risky operations
- Deny by default

### 4. Clear Error Reporting
- Tell users what failed
- Tell users why it failed
- Tell users what to do
- Don't leak sensitive information

### 5. Logging and Monitoring
- Log all operations
- Log all errors
- Log security events
- Include timestamps and context

---

## Performance Impact

### Security vs. Performance Trade-offs
- Path validation: O(n) overhead (acceptable)
- Input validation: O(1) overhead (negligible)
- Enhanced logging: I/O bound (acceptable)
- Error handling: Try-except (negligible overhead)

**Conclusion:** Security improvements have negligible performance impact

---

## Backward Compatibility

### Breaking Changes
- None planned
- All changes are additive (better error handling)
- CLI interface remains unchanged
- Public API remains unchanged

### Migration Notes
- No user action required
- All existing scripts continue to work
- New security checks are transparent

---

## Success Criteria

✅ All path operations validated for traversal attacks
✅ All input validated with clear error messages
✅ All errors handled gracefully (no crashes)
✅ All operations logged with security context
✅ All edge cases handled safely
✅ All tests passing (including security tests)
✅ 100% backward compatibility maintained
✅ Code review approved for security

---

## Timeline

- **Phase 1 & 2:** 3-4 hours (critical security)
- **Phase 3:** 2-3 hours (error handling)
- **Phase 4 & 5:** 2-3 hours (logging and edge cases)
- **Phase 6:** 1 hour (polish)
- **Testing:** 2 hours (security tests)
- **Total:** ~12-14 hours

---

## Next Steps

1. ✅ Create this plan
2. [ ] Start Phase 1 (path security)
3. [ ] Implement path_security.py
4. [ ] Add comprehensive error handling
5. [ ] Enhanced logging throughout
6. [ ] Write security tests
7. [ ] Code review
8. [ ] Deployment

---

**Status:** Planning Phase Complete
**Ready to:** Start Implementation

