# Feature Tour

This document describes the application as a product, using public-safe language and demo data only.

## 1. Role-Based Access

The app supports two user roles:

| Role | Capabilities |
| --- | --- |
| Employee | View orders, search, filter, add new orders, and export visible records. |
| Administrator | Full order management, spreadsheet import, Shopify sync, deleted-order recovery, source cleanup, and user management. |

This keeps everyday data entry simple while reserving destructive or integration-related actions for administrators.

## 2. Orders Dashboard

The orders screen is the primary workspace. It provides:

- Quick keyword search.
- Advanced filters for order status, payment state, currency, barcode, year, and date range.
- Sortable table columns.
- Pagination for large data sets.
- New order creation.
- Admin-only edit and delete controls.
- Bulk deletion.
- Excel export of the currently filtered view.

Each order detail view includes the full record, edit history, and a list of similar products to help users avoid creating duplicates.

## 3. Validation

The system validates key fields before saving:

- Required text fields.
- Dates.
- Integer-like quantities.
- Money fields.
- Currency fields.
- Boolean-style import values.

The goal is to catch mistakes at entry time and keep imported spreadsheet data consistent with manually entered records.

## 4. Spreadsheet Import

Administrators can upload CSV or Excel files and review them before adding records to the live order table.

Import features include:

- CSV and XLSX support.
- Automatic column matching.
- Manual column mapping review.
- Row-by-row validation.
- Issue counters.
- Editable preview cells.
- Filter to show only rows with problems.
- Draft recovery when an import is not finished.
- Duplicate source detection when a file was imported previously.
- Import history by source file.
- Option to import valid rows while leaving invalid rows for correction.

## 5. Audit History

Order updates create field-level history entries. Admins can review what changed, who changed it, and when it changed.

This is useful in small-team environments where multiple people may update order statuses, quantities, received dates, notes, or inventory-related fields.

## 6. Soft Delete & Recovery

Deleting an order moves it to a deleted-orders area instead of removing it immediately.

Administrators can:

- Restore a deleted order.
- Permanently delete one order.
- Permanently delete all deleted orders.
- Delete records tied to a specific imported source file.

This two-step approach protects the team from accidental data loss while still allowing cleanup.

## 7. Shopify Sync Workflow

The app separates unsynced records into worklists:

- New products selected for automatic Shopify inventory creation.
- Existing products that should be handled manually.
- Failed sync records that need review.

Successful sync actions update each record's inventory state. Failed syncs remain visible with error information so admins can fix the source data and retry.

## 8. User Management

Administrators can create accounts, assign roles, view users, and remove non-protected accounts.

The default administrator account is protected from deletion so the team cannot accidentally lock itself out.

## 9. Export & Backup

Users can export the current order view to Excel. Filters and searches are respected, so the export matches what the user is looking at.

The app also runs an automatic monthly backup process that writes local spreadsheet backups for recovery and reporting.

## 10. Packaging

The system can be packaged as a Windows onedir application using PyInstaller. The packaged app runs a local Waitress server and opens in the browser, while storing runtime data outside the replaceable program folder so updates do not overwrite the database.
