# Changelog

## [2.0.3] - 2025-06-11

### Fixed
- Resolved `Lock check failed: no such table: locks` error by restoring correct locking logic based on the `return_status` table (fields: `locked_by`, `locked_at`, `lock_log`).
- Prevented uninitialized connection usage in lock functions by adding safety checks and proper initialization.

### Changed
- Refactored `release_user_locks()` to correctly integrate with the existing return-level lock system, removing the unintended dependency on a non-existent `locks` table.
- Ensured `form.conn` is properly initialized upon user login for use in locking and unlocking routines.
- Improved error handling and logging in `check_and_apply_lock()` to reflect lock state failures and permission conditions more clearly.

### Added
- Automatic release of locks on application exit via `closeEvent()`, ensuring consistent database state even if the app is closed abruptly.

---

## [2.0.2] - 2025-06-05

### Fixed
- Resolved issues preventing the display of return logs in the executable version.
- Corrected inconsistent data retrieval from SQLiteCloud between `.py` and `.exe` versions.

### Changed
- Switched build system from PyInstaller to Nuitka for improved stability, compatibility, and packaging reliability.

---

## [2.0.1] - 2025-06-05

### Added
- New interactive changelog window: clicking the version label opens a popup displaying the latest changes pulled directly from GitHub.

### Fixed
- Ensured internal versioning is centralized in code to avoid missing version info in update checks and GUI display.

---

## [2.0.0] - 2025-06-05

### Initial Release

This version introduces the core functionalities of **RetMan**, the return management application designed to streamline product return workflows.

### Main Features

- **User Authentication**
  - Secure login system with user identification and role-based access.

- **Return Management**
  - Creation and editing of product returns with fields for customer, invoice, reasons, OEM tags, and internal comments.
  - Status management with indicators like "NEW!", "IN PROGRESS", and resolution tracking.
  - Real-time locking to prevent concurrent editing of the same return.

- **QR Code Integration**
  - Create and read QR codes with embedded return data.
  - QR string includes customer, invoice, reason, OEM, HELA flag, and creator initials.
  - Returns generated from QR codes are automatically prefilled.

- **Logging System**
  - Automatic log entries for creation and modifications, including user details and QR code attribution.
  - "View Log" feature displays return activity history.

- **Admin Panel**
  - Administrative tools for managing return codes, presets, and application configurations.

- **Auto-Updater**
  - Automatic check for new versions on startup.
  - If available, downloads and installs update with progress bar and restart.
