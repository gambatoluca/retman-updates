## [2.0.4] - 2025-06-12

### Added
- Support for dynamic database connection configuration via external JSON file `db_con.json`, with key `connection_string`.
- Utility function `_load_config()` in `db.py` to read and validate the configuration file before opening the connection.
- Sample template for `db_con.json` included in the project.
- New `db_apikey` column in the `users` table to store each user’s API key (SQL: `ALTER TABLE users ADD COLUMN db_apikey TEXT;`).
- Automatic generation and regeneration of user API keys in the Admin Panel during user create/edit/delete operations.
- Read‐only connection string loaded from `db_con.json` in `db.py`.

### Changed
- Refactored `db.py` to remove the hard-coded connection string and load `connection_string` from `db_con.json`, with error handling for a missing key.
- Updated `get_data_conn()` to return the correct connection (read-only or user-key) and centralize external configuration usage.
- Modified all database-accessing modules (`wh_dropdowns.py`, `wh_table.py`, `admin_panel.py`, `login.py`, etc.) to always use `form.conn` (user connection) instead of directly importing `sqlitecloud.connect` or calling `get_data_conn()` internally.
- Removed direct imports of `get_data_conn()` in favor of passing the connection from the main app, ensuring consistency and permission control.
- **login.py**:
  - Performs login with read-only connection, retrieves `db_apikey`, closes the read-only connection, and opens the user-key connection.
  - Fixed `AttributeError` by replacing `Qt.SmoothTransformation` with `Qt.TransformationMode.SmoothTransformation` and `Qt.AlignCenter` with `Qt.AlignmentFlag.AlignCenter`.
- **wh_dropdowns.py** & **wh_table.py**:
  - Switched to using `form.conn` (user-key connection) for dropdowns and table loading.
  - Restored definition of `where_clauses` to avoid “not defined” errors.
- **wh_logic.py**:
  - Safe-read of `returns.log` inside a `try/except` to handle prohibited access.
  - Centralized connection handling without closing/reopening mid-flow.

### Fixed
- Eliminated cases of unintended double connection open/close by moving management to the context of `WHForm`.
- Resolved potential login crashes when `db_con.json` was missing or malformed.
- Resolved “`where_clauses` is not defined” error in `wh_table.py`.
- Eliminated “access to … is prohibited” permission errors by using the user’s `db_apikey` connection.

---


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
- Refactored `retqr.py` as a standalone executable (`retqr.exe`) compiled with PyInstaller to bypass Nuitka PDF rendering limitations.  
- GUI integration updated: `retqr.exe` is now called via `subprocess` from within the main application.

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
