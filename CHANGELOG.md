## [2.3.0] – 2025-08-19

### Added
- **Useless facts with timeout**  
  – The background request for the “useless fact” at startup now has a strict timeout of 2 seconds for both connection and read, ensuring the UI loads instantly even if the external API is slow or unreachable.  
  – Failure cases now log a clear message without blocking the interface.
- **Database Backup system**  
  – Added remote-to-file backup functionality for the SQLiteCloud database, saving `.db` files locally.  
  – Suggested filename is automatically pre-filled with current date (e.g., `2025-08-14_RetMan.db`).  
  – Added progress bar during backup operations for better feedback.  
  – Introduced **Backup Explorer** allowing inspection of backup files without restoring them.  
  – In Backup Explorer, added the ability to view linked comments, products, and logs for each return.

### Changed
- **KPI module cleanup**  
  – Removed unused imports, redundant variables, and outdated comments for better maintainability.  
  – Streamlined chart rendering logic to avoid unnecessary recomputation when switching tabs.  
  – Unified SQL queries to reduce duplicate database calls for overlapping metrics.
- **UI code consistency**  
  – Reorganized product section widget initialization in `WHForm` to ensure consistent button creation before event binding.  
  – Minor style adjustments for better alignment of product grid elements.
- **Table default scroll position**  
  – Returns table now automatically scrolls to the last record when loaded, instead of starting from the first row.
- **Temporary removal of Restore button**  
  – The database restore button in the Admin Panel backup tab has been temporarily removed (feature flag disabled) while keeping the restore logic intact for future development.
- **QR scan editability**  
  – When a QR code is loaded in **WHForm**, the **Status** dropdown remains locked, while **Return Reason**, **OEM**, and **Open/Unopen** are now enabled and can be edited.

### Fixed
- **Startup white-screen issue**  
  – Resolved a bug where `WHForm` could open with a blank window if certain UI components were accessed before `build_ui()` completed execution.  
  – Initialization order is now enforced so that dropdowns, tables, and buttons are fully built before data loading begins.
- **Potential product-section crashes**  
  – Fixed cases where product edit/delete buttons could be connected before the `product_*_btns` lists were populated, leading to `IndexError` on click.


## [2.2.2] – 2025-08-13

### Added
- **Full-name tooltip in Owner column**  
  – Hovering over the `Owner` column in the returns table now displays the full name(s) corresponding to the initials shown in the cell.  
  – For returns with both creator and modifier, both names are shown separated by `|`.  
  – Tooltip also merges with any existing row warnings (e.g., “Missing Customer ID or Invoice”).
- **VIEW-mode safeguards for Products**  
  – In “VIEW” mode, clicking ✏️ or 🗑️ in the products section no longer triggers any edit or delete actions.  
  – This safeguard is enforced both via button disabling and in the event handlers to prevent accidental modifications.
- **Delete confirmation dialog**  
  – In “EDIT” mode, clicking 🗑️ now shows a confirmation popup before removing a product from the list.

### Changed
- **Product name font style**  
  – Product names in the products section are now displayed in a normal-weight font instead of bold, for improved readability and consistency.  
  – QTY labels remain bold to retain quick quantity recognition.
- **Internal handler structure for Products**  
  – Refactored `edit_product` and `delete_product` methods in `WHForm` to include mode checks and popup logic without altering existing product popup workflows.

### Fixed
- **Accidental product changes in VIEW mode**  
  – Previously, if the product edit/delete buttons were manually re-enabled, clicking them could still modify the return. Now the code enforces no-op behavior in VIEW mode regardless of button state.
- **Consistent product slot rendering**  
  – Ensured that font style changes for product names apply to all 12 slots and persist through return loading, editing, and saving.


## [2.2.1] – 2025-07-25

### Added
- **“View” product button**  
  – A permanent 🔍 icon appears in every product slot (qty+name+✏️+🗑️+🔍), even if empty.  
  – Opens a new **ProductViewDialog** showing SKU, product name and EAN list, all text selectable and copy-able.  
- **Copy-SKU interactions**  
  – Right-click or double-click on any product name label to copy its SKU to clipboard.  
  – A tooltip on each label (“Right-click or double-click to copy SKU”) guides operators.
  - **Automatic update check (hourly)**  
  – A background timer checks once per hour **only during weekdays (Mon–Fri)** and **only between 07:00 and 18:00**.  
  – If a new version is available, a popup informs the user to close and reopen the app to update.
- **Centralized version constant**  
  – Introduced a new `version.py` module that defines `CURRENT_VERSION`, now used by both `login.py` and `update_utils.py`.


### Changed
- **Products grid layout**  
  – Expanded each product slot to 5 columns for qty, name, ✏️, 🗑️ and 🔍.  
  – Uniform column-stretch ensures no empty gaps; “Add Products” spans full width of the products section.  
- **Initialization order**  
  – `product_view_btns` is populated in `wh_ui_sections.py` builder, then only enabled/disabled in `update_products_section()`, avoiding repeated creation or double-binding of click handlers.
  - **Update check moved to utility module**  
  – The function `check_for_update_gui()` was extracted from `login.py` and now resides in a dedicated `update_utils.py` for modularity and reuse.

### Fixed
- **Double-popup issue**  
  – Removed redundant click-handler binding in `WHForm.__init__`, so each 🔍 opens the dialog exactly once.  
- **IndexError crash**  
  – Ensured `self.product_view_btns` always contains 12 buttons by building them in the UI constructor, preventing out-of-range accesses.  
- **Empty-space gap**  
  – Corrected grid indexing to use `(i % 4) * 5 + 0–4` for widget placements, eliminating misaligned or hidden 🔍 buttons.


## [2.1.0] – 2025-07-17

### Added
- **New product entry workflow in WHForm**
  - The "Products" section in `WHForm` now features a dedicated **Add Products** button that opens a popup for rapid barcode/manual entry.
  - The popup supports scanning EAN or entering SKU, with autocompletion and live validation against the products table (SKU/EAN → product name).
  - Users can add multiple products in a row without closing the popup, and each addition updates the products list in real-time in the main form.
  - All product edits are performed via the popup (pre-filled for edits); direct modification of products in the grid is disabled.
  - Added support for deleting or editing individual products via new icons (✏️/🗑️) in each row, visible for all 12 product slots.
  - The products section now shows **QTY** and **Product Name** (in bold), instead of SKU, for better operator clarity.

- **Robust SKU/EAN validation**
  - Prevents the insertion of products not found in the `products` table (either by SKU or EAN).
  - When loading historical returns, if a product SKU is missing from the current database, the app displays `"Unknown product (SKU)"` in the product name column.

- **Product search validation in SEARCH dialog**
  - The SKU search field in the advanced SEARCH popup now offers autocompletion and live validation.
  - Shows product name in green if found, or “Not found” in red, improving the accuracy of filtered queries.

- **Filter for returns with missing customer data**
  - Advanced **SEARCH** popup now includes a checkbox **“⚠ Only returns with missing Customer / Invoice”**.
  - When enabled, the results list only returns where either `Customer ID` or `Invoice` is empty.
  - Corresponding logic added to `wh_table.py` to build the SQL `WHERE` clause.

### Changed
- **Read-only products section**
  - QTY and product name fields in the products section are no longer directly editable; all insertions and modifications are handled via the popup for data consistency.
  - Product edit/delete icons are disabled in "VIEW" mode, preventing accidental changes when reviewing past returns.

- **Popup UX improvements**
  - In the product entry popup, the **"Add"** button now immediately adds a product to the form (and updates the visible list), clearing the fields for the next entry, while the **"Close"** button simply closes the dialog.
  - Product selection workflow ensures the operator always sees exactly what has been added before finalizing a return.

- **Visual marking of incomplete returns**
  - `wh_table.py` now highlights rows with missing `Customer ID` **or** `Invoice` in light-red (`#ffbbbb`) and adds a tooltip “Missing Customer ID or Invoice”.
  - These returns remain visible in other filter modes and are easily identifiable.

### Fixed
- **Legacy `qty_sku` field usage**
  - Removed all references to the obsolete `qty_sku` logic in favor of the new product_rows structure, eliminating `AttributeError` and related errors in loading, editing, or saving returns.

- **Visual and interaction bugs in product grid**
  - All 12 product slots now consistently display both the edit and delete icons, regardless of grid position.
  - UI coloring for "Unknown product" or "Not found" SKUs improved for operator clarity.

### Compatibility
- These changes are fully compatible with existing saved returns; historical returns display unavailable SKUs as "Unknown product (SKU)" but can still be reviewed, edited (if valid SKU chosen), or saved.


## [2.0.8] – 2025-07-11

### Fixed
- **Status column mapping in WH table**
  - The `Status` column in the `WHForm` main table now correctly reflects the value from `return_status.status`.
  - Resolved an indexing bug where `user_modifier` was mistakenly shown in place of the actual return status.
  - Updated `load_returns_table()` in `wh_table.py` to explicitly inject `status` at the correct table index.

### Removed
- **Heartbeat mechanism**
  - Completely removed the periodic `heartbeat_tick()` logic introduced in version 2.0.7.
  - This feature caused `[HEARTBEAT ERROR]` logs in environments where the `users.fullname` field did not exist.
  - Deleted `heartbeat_timer` initialization and `heartbeat_tick()` method from `wh_form.py`.

---

## [2.0.7] – 2025-06-18

### Fixed
- **Persistent connection failure workaround**  
  - Solved recurring `An error occurred while reading command length from the socket` by redesigning the database access model to **open and close a new connection per operation**.
  - `load_returns_table()` now creates a **temporary connection and cursor** for every call, eliminating reliance on persistent socket-based sessions.
  - Removed all instances of `Skipped refresh (inactive window)` logic by making auto-refresh run unconditionally.

### Added
- **Heartbeat mechanism for session preservation**  
  - A new periodic task updates the `users.heartbeat` field every 4 minutes for the logged-in user (`fullname`), keeping the database session alive and reducing idle timeouts.
  - The process also performs a `SELECT` before the update to simulate continuous usage.
  - A `[HEARTBEAT]` log message is added to the interface for each cycle.

### Changed
- **Auto-refresh frequency lowered to 5 minutes**  
  - Reduced interval from 10 minutes (600,000 ms) to 5 minutes (300,000 ms) to avoid disconnection on idle.
  - Auto-refresh now works regardless of window focus or activity status.

---

## [2.0.6] – 2025-06-17

### Fixed
- **Refresh crash handling**:
  - Addressed intermittent `[Load Table Error]` messages caused by socket failures during table refresh.
  - `load_returns_table()` now includes automatic reconnection logic using the current user’s API key if a `sqlitecloud` or socket-related error is encountered.
  - Added recovery logging to show reconnect attempts and fatal fallback if retry fails.

### Changed
- **Extended numeric input limits**:
  - Both **Customer ID** and **Invoice Number** fields in the `WHForm` and `RetQRForm` can now accept up to 20 digits instead of being limited to 9.
  - `QIntValidator` was replaced with `QRegularExpressionValidator(r"\d{1,20}")` to support long customer and invoice numbers without truncation or validation failure.
  - Applied consistently in both the main form (`wh_form.py`) and the external QR generator (`retqr.py`).

---

## [2.0.5] – 2025-06-16

### Changed
- **Renamed buttons in the main WH form interface**:
  - The `"SAVE"` button is now labeled `"CREATE"`.
  - The `"MODIFY"` button is now labeled `"SAVE"`.
- **Removed the `"CLOSE"` button** from the main UI controls section to reduce interface clutter and avoid confusion with window close functionality.
- **Updated admin context menu logic**:
  - When right-clicking on a return and selecting `"Hide"` or `"Delete"`, the `archive_status` field is now correctly updated instead of the (no longer existing) `status` column.
- **Refined logging behavior**:
  - Upon hiding, deleting, or archiving a return, a success message (e.g. `✔ Return 12345 archived as HID`) is now shown in the log box to confirm the administrative action.

---

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
