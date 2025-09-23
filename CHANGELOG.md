## [2.3.2] – 2025-09-22

### Added
- **Updater window icon**: All updater windows and the taskbar now display the custom RetManUpdaterIcon.ico icon.
- **Fun Fact reload on logo double-click**: Double-clicking the logo (top-right corner) fetches and appends a new Fun Fact directly into the log box.
- **RetQR – Apply to all pages**: New "Apply QR to all pages" checkbox to stamp the QR on every page of the PDF.
- **RetQR – Non-blocking operations**: Preview rendering and saving now run on worker threads with a QProgressDialog.
- **RetQR – Optional OpenGL viewport**: If available, preview uses QOpenGLWidget for smoother pan/zoom.
- **RetQR – Auto Save shortcut**: Ctrl+Shift+A toggles Auto Save.
- **RetQR – HiDPI policy**: PassThrough rounding policy for crisp rendering on high-DPI displays.
- **Search dialog — User filters**: Added two dropdowns: Created by and Modified/Closed by.
  - Dropdowns show users.username while filtering by initials stored in returns.user_creator and returns.user_modifier.
  - User list is loaded from the users table and ordered by username.
- **Warehouse — Selling price per product**: Product dialog now includes a mandatory **Selling price** field.
  - Accepts inputs like `12`, `12.3`, `12,3`, `12.345`; automatically converts comma to dot and rounds **half-up** to **two decimals**.
  - Prices are stored normalized as `123.45` and persisted alongside qty/sku for each product.

### Changed
- **Updater destination folder**: Update packages are now always extracted into the RetManV2 folder (created automatically if missing).
  - Before extraction, the updater removes all contents of RetManV2 except:
    - RetQR/ (kept if not present in the update package).
    - All versioned updaters RetManUpdaterV*.exe (never deleted during update).
- **Updater self-update strategy**: Each new updater version is saved as RetManUpdaterV{N}.exe; only the two most recent versions are kept.
- **Shortcut creation**: The updater always recreates RetMan.lnk pointing to RetMan.exe inside RetManV2.
- **Progress bar**: Thicker progress bar with visible percentage text.
- **RetQR – Window startup**: The window now opens always maximized with post-show enforcement and timed retries (more reliable even from the executable).
- **RetQR – Lighter preview**: Preview DPI set to 150 to reduce memory usage.
- **RetQR – Preview engine**: Prefer PyMuPDF (fitz); pdf2image is used only as a lazy-import fallback inside the function.
- **RetQR – Auto Save & filename**: Auto Save defaults ON (persisted via QSettings); save dialog pre-fills filename from invoice/customer when used.
- **RetQR – Cleanup**: Centralized scene cleanup and temporary directory removal on app exit.
- **Search dialog — Admin controls layout**: HID/DEL admin controls moved below the new user filters and set to span both columns to keep the grid tidy.
- **Warehouse — Save/Modify validation**: Saving a return now requires a valid selling price (two decimals) for each product; input is auto-normalized.

### Fixed
- **Rename exclusions**: The rename step excludes RetManUpdater.exe, any RetManUpdaterV*.exe, the RetQR folder, and other critical files/folders.
- **Extraction cleanup**: Fixed cases where leftover files in RetManV2 could persist between updates; irrelevant files are now reliably removed.
- **RetQR – Crash after "NEW QR" → "Load PDF"**: Resolved crash caused by operating on a deleted QGraphicsPixmapItem (reference cleanup + sip.isdeleted guards).
- **RetQR – Preview/Save errors**: Corrected QR path checks (no more "Missing PDF or QR code image" / "No PDF available to preview" in normal flows).
- **RetQR – Script startup**: Fixed NameError: win is not defined in main (proper window creation and maximize sequence).
- **Search dialog — Overlapping controls**: Resolved overlap between the new Created/Modified dropdowns and the HID/DEL admin controls.
- **Warehouse — Product dialog tab-order warning**: Removed stray setTabOrder calls that caused “QWidget::setTabOrder: 'first' and 'second' must be in the same window”.

### Packaging
- **RetQR – Leaner build & faster start**: Recommended --onedir build: prefer PyMuPDF and exclude pdf2image/Poppler plus unused Qt modules (QtWebEngine*, QtQml, QtQuick, QtDesigner, QtTest).
- **Removed top-level import of pdf2image** (now only a fallback inside the function) to avoid startup crashes when excluded from the build.

### Database
- **return_products.selling_price (TEXT)**: New column storing per-product selling prices as a semicolon-separated list aligned with qtys/skus.
  - Migration:
    - `ALTER TABLE return_products ADD COLUMN selling_price TEXT;`
    - `UPDATE return_products SET selling_price = '' WHERE selling_price IS NULL;`


## [2.3.1] – 2025-09-05

### Added
- **Product name resolver (DB-backed)**  
  – Introduced a centralized product lookup with module-level cache to resolve product names by SKU across the app.  
  – Normalizes SKUs (`strip().upper()`) before querying, removing case/spacing mismatches and reducing duplicate DB hits.
- **Admin Panel — Product Import live preview**  
  – Added a preview table that shows sample rows as they are imported from Excel.  
  – Log box now streams created/updated/skipped entries live.  
  – Progress bar is fully functional during the import of large files (60k+ rows).  
- **Admin Panel — Backup progress**  
  – Backup process now runs in a worker thread with a responsive **progress dialog**.  
  – User can cancel the operation mid-way.  
  – Success and error messages are clearer.
- **KPI Module — Date range controls & quick presets**  
  – Added date range selectors and quick presets (e.g., last 7/30/90 days, YTD) for **Overview** and **User KPIs** tabs.  
  – Charts and tables are refreshed only with data within the selected range.  
  – Switching range or applying presets now clears the old data traces before rendering new ones, fixing label overlap issues.  
- **KPI Module — Enhanced Customer/Product popups**  
  – Customer details popup now shows **Return ID**, **Invoice Number** and related **products per invoice**, all with copy-to-clipboard support.  
  – Product details popup similarly supports copying values and ensures invoice numbers are displayed as integers.  
- **Duplicate Invoice check**  
  – When creating a return, the system now checks if the same **Invoice** already exists in the database.  
  – If matches are found, a popup warns the user, listing the existing Return IDs (and related info), and asks whether to proceed anyway.  
- **Search dialog — Admin-only HID/DEL controls**  
  – Added two admin-only checkboxes: **Include HID** and **Include DEL**.  
  – Added **VIEW** buttons next to each checkbox to quickly show a list containing **only** HID or **only** DEL returns.

### Changed
- **“Add Products” popup — Product Code field**  
  – The **Product Code** (SKU) field is now **read-only**. It’s populated via scan/autofill and can’t be edited manually, preventing accidental SKU corruption.  
- **Consistent SKU normalization**  
  – Added a small helper to normalize SKUs and applied it wherever products are rendered or viewed, ensuring consistent lookups and UI text.  
- **Minor UX polish in product area**  
  – When product names are refreshed from the DB, the cleaned name is kept in memory so subsequent popups and views display exactly the same value.  
- **Admin Panel — Optimized Product Import**  
  – Import now uses **batched transactions** and `ON CONFLICT(sku) DO UPDATE`, dramatically reducing the time for large imports.  
  – Merging of EANs is handled efficiently in Python before commit.  
  – Default batch size set to 1000 rows (configurable in code).  
- **KPI Module — Filtering logic**  
  – All KPIs exclude returns with archive status `HID` or `DEL`.  
  – Top customers table now hides entries with customer_code `0` or empty.
- **Repository & Table — HID/DEL handling**  
  – Repository search now dynamically includes or excludes `HID`/`DEL` based on the new admin filters, and supports an `only_arch` mode used by the **VIEW** buttons.  
  – Table highlights included `HID` and `DEL` rows with distinct font colors (HID = blue, DEL = red) when filters are active.

### Fixed
- **“Unknown product (SKU)” after the first item**  
  – Resolved an issue where, in **VIEW**/**MODIFY** mode, only the first product showed the correct name while subsequent items appeared as “Unknown product (XXX)”.  
  – Each slot now resolves its name independently using the normalized SKU and the product cache, with a single, consistent fallback only when the SKU truly isn’t found.  
- **View Product dialog name mismatch**  
  – The **Product Details** popup now always shows the correct product name (synced with the list) and retrieves the EANs reliably from the database.  
- **Stability on edge data**  
  – Hardened product rendering against partial rows and `None` values to avoid intermittent errors during UI refresh.  
- **Admin Panel — Progress bars not updating**  
  – Fixed an issue where progress bars for Backup and Product Import were not updating because the operations were blocking the UI thread.  
  – Now both tasks run in dedicated workers, keeping the interface responsive.  
- **KPI Module — Visual refresh issues**  
  – Old labels and chart data no longer persist when changing date ranges or quick presets.  
  – Popups in **Customers & Products** tab now display integers consistently for invoice numbers.  
- **Counts error with some drivers**  
  – Fixed `no such column: status` by joining the `return_status` table in counters and fully qualifying columns; counters also consistently exclude `HID`/`DEL` unless explicitly included by admin filters.


## [2.3.0] – 2025-08-21

### Added
- **Fun facts with timeout**  
  – The background request for the “useless fact” at startup now has a strict timeout of 2 seconds for both connection and read, ensuring the UI loads instantly even if the external API is slow or unreachable.  
  – Failure cases now log a clear message without blocking the interface.
- **Database Backup system**  
  – Added remote-to-file backup functionality for the SQLiteCloud database, saving `.db` files locally.  
  – Suggested filename is automatically pre-filled with current date (e.g., `2025-08-14_RetMan.db`).  
  – Introduced **Backup Explorer** allowing inspection of backup files without restoring them.  
  – In Backup Explorer, added the ability to view linked comments, products, and logs for each return.
- **Product Add dialog UX improvements**  
  – In the “Add Products” popup, the **Scan/Barcode field** is now the first input with initial focus, followed automatically by the **Quantity field**.  
  – Pressing **Enter** in the popup now triggers the product addition, equivalent to clicking the **ADD** button.

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
- **QR reset behavior**  
  – The QR input field is now automatically cleared after pressing **RESET** or after creating a new return, avoiding manual cleanup.
- **Owner column display**  
  – The **Owner** column in the returns table now shows only the **initials** of the creator and last modifier.  
  – Hovering over the field shows a tooltip with the full names of both users for clarity.

### Fixed
- **Startup white-screen issue**  
  – Resolved a bug where `WHForm` could open with a blank window if certain UI components were accessed before `build_ui()` completed execution.  
  – Initialization order is now enforced so that dropdowns, tables, and buttons are fully built before data loading begins.
- **Potential product-section crashes**  
  – Fixed cases where product edit/delete buttons could be connected before the `product_*_btns` lists were populated, leading to `IndexError` on click.
- **Lock release after save**  
  – When a return is modified and saved, its **lock** is now correctly released and the form switches back to **VIEW** mode.
- **Customer/Invoice zero-padding**  
  – Fixed a formatting issue where leading zeros in Customer ID or Invoice were dropped (e.g., `0002345` shown as `2345`). They now retain full formatting as entered.
- **Row coloring in returns table**  
  – Status-based row coloring (yellow for **NEW!**, green for **CLOSED**) has been restored. Missing Customer/Invoice rows are also highlighted in light red.



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
