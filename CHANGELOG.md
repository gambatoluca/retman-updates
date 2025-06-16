## [2.0.4] – 2025-06-16

### Added
- Support for dynamic database connection via external JSON file `db_con.json`, using the key `connection_string`.  
- Utility function `_load_config()` in `db.py` to read and validate that file before opening the connection.  
- Sample template for `db_con.json` bundled in the project.  
- **New module** `kpi_module.py`:
  - **Overview** tab with:
    - Time-series charts for total returns (daily & weekly).  
    - Table of average processing time (days) for each period (rounded to 2 decimals).  
    - Pie chart of return reasons.  
    - Bar chart of top-10 returned products showing both quantity returned and number of return transactions (clickable to copy SKU).  
    - Monthly KPIs table.  
  - **User KPIs** tab with:
    - Side-by-side pie charts for returns **created** and **closed** per user (full names + percentages).  
    - Totals label (“Total Created | Total Closed”).  
    - Two tables showing counts per user.  
  - **Customers & Products** tab with:
    - Two side-by-side tables of top customers (≥ 2 returns) and top products (≥ 2 returns).  
    - Click-to-popup detail lists of invoices and return reasons (values copy-able).  
  - Integrated into the main GUI via the “KPIs” button in `wh_ui_sections.py`.

### Changed
- Refactored `db.py` to remove the hard-coded connection string and load the `connection_string` from `db_con.json`, with error handling for missing keys.  
- Updated `get_data_conn()` to centralize and return the configured connection.  
- Modified all database-accessing modules (`wh_dropdowns.py`, `wh_table.py`, `admin_panel.py`, `login.py`, etc.) to consume `form.conn` instead of calling `sqlitecloud.connect` or `get_data_conn()` directly.  
- Removed direct imports of `get_data_conn()` in favor of passing the connection from the main application, ensuring consistent permission handling.

### Fixed
- Eliminated accidental double-open/close of the database connection by delegating connection management to `WHForm`.  
- Resolved login crashes when `db_con.json` was missing or malformed.

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
