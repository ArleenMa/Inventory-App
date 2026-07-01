# Architecture

This document explains the system at a high level without exposing private source code.

## System Overview

```mermaid
flowchart LR
    Employee["Employee browser"] --> UI["Flask-rendered web UI"]
    Admin["Admin browser"] --> UI

    UI --> Auth["Authentication and role checks"]
    Auth --> Routes["Flask blueprints / route modules"]

    Routes --> Orders["Order services"]
    Routes --> Import["CSV/XLSX import services"]
    Routes --> Users["User management"]
    Routes --> Sync["Shopify sync services"]
    Routes --> Export["Excel export"]

    Orders --> DB[("SQLite database")]
    Import --> DB
    Users --> DB
    Sync --> DB
    Export --> Files["Generated .xlsx files"]

    Sync --> Shopify["Shopify Admin API"]
    Backup["Monthly backup job"] --> DB
    Backup --> BackupFiles["Local backup spreadsheets"]
    Logger["Request, backend, frontend logs"] --> LogFiles["Local log files"]

    Routes --> Logger
```

## Application Layers

| Layer | Responsibility |
| --- | --- |
| Browser UI | Tables, forms, filters, preview editing, admin screens, and user feedback. |
| Flask app | Routing, session handling, role checks, request logging, error handling, and page rendering. |
| Domain workflows | Order CRUD, spreadsheet import, soft delete, export, backups, and Shopify sync. |
| Persistence | SQLite database through SQLAlchemy models. |
| Integration | Shopify API calls for product and inventory synchronization. |
| Packaging | Waitress server plus PyInstaller distribution for Windows deployment. |

## Main Modules

The private source is organized around feature modules:

| Area | Purpose |
| --- | --- |
| App factory | Creates the Flask app, configures logging, registers routes, initializes the database, runs migrations, and starts scheduled backup checks. |
| Auth | Login, logout, default admin seeding, and admin-only route protection. |
| Orders | Main order table, filters, add/edit/delete/restore flows, similar-product lookup, bulk delete, and export. |
| CSV/XLSX import | File decoding, column matching, row validation, preview refresh, draft saving, and import commit. |
| Shopify | Worklists, connection status, payload preparation, product sync, failed-sync handling, and manual mark-as-synced flow. |
| Users | Admin user creation, listing, role assignment, and deletion. |
| Validators | Shared parsing and validation for dates, quantities, money, currency, and booleans. |
| Backup | Monthly backup decision logic and spreadsheet generation trigger. |
| Logging | Backend, frontend, and server log formatting. |

## Data Model

```mermaid
erDiagram
    USER ||--o{ ORDER : creates
    USER ||--o{ ORDER : updates
    USER ||--o{ ORDER_HISTORY : changes
    USER ||--o| CSV_IMPORT_DRAFT : owns
    ORDER ||--o{ ORDER_HISTORY : records

    USER {
        integer id PK
        string username
        string password_hash
        string role
        datetime created_at
    }

    ORDER {
        integer id PK
        date order_date
        string status
        boolean paid
        string item
        string ordered_quantity
        string unit
        float cost
        string cost_currency
        date receiving_date
        string barcode
        string retail_price
        string retail_price_currency
        boolean require_auto_inventory
        string english_name
        string batch_number
        string source
        boolean is_deleted
        string csv_source
        string sku
        string product_state
        string shopify_sync_status
        datetime created_at
        datetime updated_at
    }

    ORDER_HISTORY {
        integer id PK
        integer order_id FK
        string field_name
        text old_value
        text new_value
        integer changed_by FK
        datetime changed_at
    }

    CSV_IMPORT_DRAFT {
        integer id PK
        integer user_id FK
        text payload
        datetime created_at
        datetime updated_at
    }
```

Sensitive fields such as real product names, barcodes, supplier notes, pricing, production Shopify IDs, and credentials are not included in this public display.

## Request Flow

```mermaid
sequenceDiagram
    participant Browser
    participant Flask
    participant Auth
    participant Domain as Domain Workflow
    participant DB as SQLite
    participant Logs

    Browser->>Flask: HTTP request
    Flask->>Logs: Assign request ID and log request start
    Flask->>Auth: Check session and role
    Auth-->>Flask: Allowed or rejected
    Flask->>Domain: Run feature workflow
    Domain->>DB: Read or write records
    DB-->>Domain: Results
    Domain-->>Flask: JSON, HTML, or file response
    Flask->>Logs: Log status and duration
    Flask-->>Browser: Response
```

## Packaged Deployment Model

```mermaid
flowchart TB
    subgraph DeployRoot["Client machine deploy folder"]
        subgraph Program["Inventory program folder - replaceable on update"]
            Exe["Inventory.exe"]
            Assets["Templates and static assets"]
        end

        subgraph Runtime["inventory_database folder - preserved on update"]
            DB[("inventory.db")]
            BackupState["backup_state.json"]
            Backups["backups/*.xlsx"]
        end
    end

    Env[".env configuration"] --> Exe
    Exe --> Waitress["Waitress local server"]
    Waitress --> Browser["Browser on same computer or LAN"]
    Exe --> DB
    Exe --> Backups
```

The important design choice is that runtime data lives outside the replaceable program folder. Updating the app means replacing the program files while leaving the database and backups untouched.

## Security Boundaries

```mermaid
flowchart LR
    Public["Public portfolio repo"] -->|contains| Docs["Sanitized docs"]
    Public --> Screens["Redacted screenshots"]
    Public --> Diagrams["Architecture and workflow diagrams"]

    Private["Private company repo"] -->|contains| Source["Application source"]
    Private --> Secrets[".env values and credentials"]
    Private --> Data["Database, logs, exports, product records"]
    Private --> Integrations["Real Shopify configuration"]

    Public -. must not include .-> Source
    Public -. must not include .-> Secrets
    Public -. must not include .-> Data
    Public -. must not include .-> Integrations
```
