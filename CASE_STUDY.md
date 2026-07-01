# Case Study

## Context

A private company needed a lightweight internal system for managing order and inventory records. The team was using spreadsheet-based workflows, which made it easy to lose track of edits, duplicate products, import inconsistent data, and manually coordinate inventory updates.

The goal was to build a practical browser-based tool that could run locally, support multiple people on the office network, and avoid the operational overhead of a full cloud deployment.

## Problem

The workflow needed to support:

- Centralized order records.
- Admin and employee access levels.
- Bulk import from existing CSV/XLSX spreadsheets.
- Data validation before imported rows become live records.
- Recovery from accidental deletion.
- Exportable spreadsheets for reporting and backup.
- Shopify inventory preparation.
- A packaging approach that non-technical users could run on Windows.

## Constraints

- The app needed to serve a small office environment, not millions of public users.
- Existing business data could not be shared publicly.
- The team needed browser access from devices on the same local network.
- Updates needed to preserve the database automatically.
- The interface needed to be usable for Chinese-language operations.
- The system had to be simple enough to maintain without a large infrastructure footprint.

## Solution

I built a Flask-based internal web app with a SQLite database, role-based authentication, spreadsheet import workflows, order management, soft deletion, audit history, Shopify sync worklists, Excel export, monthly backup behavior, logging, and Windows packaging support.

The app runs as a local Waitress server and can be opened in a browser on the host machine or from other devices on the same LAN, depending on configuration.

## Engineering Decisions

| Decision | Why It Fits |
| --- | --- |
| Flask | Lightweight server framework suitable for a focused internal tool. |
| SQLite | Simple local persistence with minimal deployment overhead. |
| SQLAlchemy | Clear model layer and safer database access than manual SQL scattered through routes. |
| Flask-Login | Established session and user-management pattern. |
| Pandas/OpenPyXL | Practical support for CSV/XLSX import and export workflows. |
| Waitress | Production-style WSGI serving without requiring users to run Flask's dev server. |
| PyInstaller | Makes the app distributable as a Windows folder/executable. |
| Soft delete | Reduces risk of accidental data loss in admin workflows. |
| Field-level history | Provides accountability when multiple users edit shared records. |
| Runtime data outside program folder | Allows app updates without overwriting the database. |

## Notable Product Details

- The spreadsheet import flow does not immediately write uploaded rows into the live table. It first analyzes, validates, previews, and lets admins edit rows.
- Deleting is intentionally two-step: active records move to a deleted-orders area before permanent removal.
- Export uses the current filtered table state, so users can generate targeted reports without building a separate report module.
- Shopify sync is treated as a workflow queue rather than a hidden background task. Admins can see what is ready, what is already existing, and what failed.
- Similar-product lookup helps users catch duplicates before creating new Shopify items.

## Results

The finished system gives the team a single shared operational tool while keeping deployment simple. It replaces fragile spreadsheet-only coordination with structured records, validation, access control, auditability, recoverability, exports, backups, and integration-focused worklists.

## What This Public Folder Shows

Because the real source and data are private, this public folder focuses on:

- Screenshots with sanitized data.
- Architecture diagrams.
- User workflows.
- Technical decision-making.
- Demo guidance.
- Security and sanitization rules.

That gives reviewers a clear understanding of the project without exposing the private company's implementation or business information.
