# Workflows

These diagrams show how the system behaves from a user's point of view.

## Login And Role Routing

```mermaid
flowchart TD
    Start["Open app URL"] --> Login["Enter username and password"]
    Login --> Check{"Credentials valid?"}
    Check -->|No| Error["Show login error"]
    Error --> Login
    Check -->|Yes| Role{"User role"}
    Role -->|Employee| EmployeeUI["Orders tab only"]
    Role -->|Administrator| AdminUI["Orders, Import, Shopify Sync, Deleted Orders, Management"]
```

## Order Lifecycle

```mermaid
stateDiagram-v2
    [*] --> Active: create manually or import
    Active --> Active: edit order
    Active --> Active: export filtered view
    Active --> Deleted: soft delete
    Deleted --> Active: restore
    Deleted --> PermanentlyDeleted: permanent delete
    PermanentlyDeleted --> [*]
```

## Manual Order Entry

```mermaid
flowchart TD
    Open["Open Orders tab"] --> Add["Click Add Order"]
    Add --> Form["Fill required fields"]
    Form --> Validate{"Valid?"}
    Validate -->|No| Fix["Show field errors"]
    Fix --> Form
    Validate -->|Yes| Save["Save order"]
    Save --> History["Create initial record metadata"]
    History --> Table["Order appears in dashboard"]
```

## CSV / XLSX Import

```mermaid
flowchart TD
    Upload["Upload CSV/XLSX file"] --> Decode["Decode file and read rows"]
    Decode --> Map["Detect column mapping"]
    Map --> Analyze["Validate rows"]
    Analyze --> Preview["Show preview with issue counts"]
    Preview --> Edit{"Need edits?"}
    Edit -->|Yes| InlineEdit["Edit cells in preview"]
    InlineEdit --> Refresh["Refresh validation"]
    Refresh --> Preview
    Edit -->|No| Import{"Import valid rows?"}
    Import -->|No| Draft["Save draft for later"]
    Import -->|Yes| Duplicate{"Source file already imported?"}
    Duplicate -->|Yes| ReplaceChoice["Confirm replace or cancel"]
    ReplaceChoice --> Commit["Create order records"]
    Duplicate -->|No| Commit
    Commit --> History["Record import source metadata"]
    History --> Orders["Rows appear in Orders dashboard"]
```

## Shopify Sync

```mermaid
flowchart TD
    Unsynced["Unsynced orders"] --> Split{"Auto inventory?"}
    Split -->|Yes| NewQueue["New products to sync"]
    Split -->|No or blank| ExistingQueue["Existing products review"]

    NewQueue --> Select["Admin selects products"]
    Select --> Payload["Build Shopify product payload"]
    Payload --> Api["Send to Shopify API"]
    Api --> Result{"Success?"}
    Result -->|Yes| Synced["Mark order synced and save Shopify IDs"]
    Result -->|No| Failed["Save sync error and show failed badge"]

    ExistingQueue --> Manual["Admin handles manually in Shopify"]
    Manual --> Mark["Mark as synced"]
    Mark --> Synced
```

## Deleted Order Recovery

```mermaid
flowchart TD
    Delete["Admin deletes order"] --> Soft["Set deleted flag"]
    Soft --> Hidden["Hide from active Orders table"]
    Hidden --> Bin["Show in Deleted Orders"]
    Bin --> Choice{"Admin action"}
    Choice -->|Restore| Restore["Unset deleted flag"]
    Restore --> Active["Return to active table"]
    Choice -->|Permanent delete| Destroy["Remove database record"]
    Choice -->|Delete by source| SourceDelete["Remove records from selected import source"]
```

## Export And Backup

```mermaid
flowchart LR
    Filters["Current search and filters"] --> Query["Query matching orders"]
    Query --> Workbook["Build Excel workbook"]
    Workbook --> Download["User downloads .xlsx"]

    Scheduler["Monthly backup check"] --> Due{"Backup due?"}
    Due -->|No| Skip["Do nothing"]
    Due -->|Yes| BackupQuery["Query active records"]
    BackupQuery --> BackupFile["Write local backup .xlsx"]
```

## Update Workflow

```mermaid
flowchart TD
    Build["Build new packaged version"] --> Replace["Replace program folder"]
    Replace --> Preserve["Leave runtime database folder untouched"]
    Preserve --> Launch["Start new executable"]
    Launch --> Migrate{"Schema updates needed?"}
    Migrate -->|Yes| Upgrade["Run lightweight migrations"]
    Migrate -->|No| Ready["Open app"]
    Upgrade --> Ready
```
