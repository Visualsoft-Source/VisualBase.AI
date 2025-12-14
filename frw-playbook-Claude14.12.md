## One-line purpose
Canonical internal playbook for safe discovery and operations on VisualBase (InnovBase) frw* metadata (modules, objects, fields, menus, reports, workflows, approvals, etc.) — always discover first, preview in transaction, verify with JSON procs, keep rollback script ready. 

---

## Quick Pre-flight Checklist (copy-paste)
1. Initialize connection (client/tool): EXEC mssql_initialize_connection('DefaultConnection');  
2. Run discovery SELECTs (INFORMATION_SCHEMA) for all target tables/columns.  
3. Prepare DML inside:
   BEGIN TRAN;
   -- Exec frwAI_* or run DML
   -- Exec Verify JSON procs
   ROLLBACK;  -- test
   -- After verification: COMMIT;
4. Save OBJECTCAT_ID / WebMenuID / GUID mappings to migration log.  
5. Keep rollback script ready and peer-reviewed. 

---

## 1. Mandatory Initialization (must be performed as separate client call)
Do NOT embed this inside a T-SQL batch. Run from the client/tool once per session:
```sql
EXEC mssql_initialize_connection('DefaultConnection');
```
Rationale: embedding caused syntax errors in past runs. 

---

## 2. Safety & Transactional Rules (always follow)
- Discover first (read-only). Use INFORMATION_SCHEMA to confirm table/column names.  
- Wrap destructive changes in transactions and test via ROLLBACK before COMMIT:  
  BEGIN TRAN; … verification SELECTs … ROLLBACK; then re-run and COMMIT when satisfied.  
- Use NEWID() for GUIDs.  
- Do not assume numeric columns are IDENTITY; compute `ISNULL(MAX(col),0)+1` for manual IDs and protect with `sp_getapplock` or SEQUENCE under concurrency.  
- Keep a reviewed rollback script available before committing.  
- Prefer two-step copy: Step A = create object/view only; Step B = copy subordinate metadata.  
- Verify existence of helper stored procs (frwAI_CreateModule, frwAI_CopyObject, frwAI_MoveObject, frwAI_Verify*). If missing, use manual reviewed SQL flows. 

---

## 3. Core Discovery Queries (run BEFORE any DML)
Always query via INFORMATION_SCHEMA and sys views.

- List base tables:
```sql
SELECT TABLE_SCHEMA AS SchemaName, TABLE_NAME AS TableName
FROM INFORMATION_SCHEMA.TABLES
WHERE TABLE_TYPE = 'BASE TABLE'
ORDER BY SchemaName, TableName;
```

- List views:
```sql
SELECT TABLE_SCHEMA, TABLE_NAME
FROM INFORMATION_SCHEMA.VIEWS
ORDER BY TABLE_SCHEMA, TABLE_NAME;
```

- List stored procedures:
```sql
SELECT SPECIFIC_SCHEMA AS SchemaName, SPECIFIC_NAME AS ProcedureName
FROM INFORMATION_SCHEMA.ROUTINES
WHERE ROUTINE_TYPE = 'PROCEDURE'
ORDER BY SchemaName, ProcedureName;
```

- Get column list for a table:
```sql
SELECT COLUMN_NAME, DATA_TYPE, IS_NULLABLE, COLUMN_DEFAULT, CHARACTER_MAXIMUM_LENGTH
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_NAME = 'YourTableName'
ORDER BY ORDINAL_POSITION;
```

- Foreign key relations (replace TableName):
```sql
SELECT fk.name AS ForeignKeyName, tp.name AS ParentTable, cp.name AS ParentColumn,
       tr.name AS ReferencedTable, cr.name AS ReferencedColumn
FROM sys.foreign_keys AS fk
JOIN sys.foreign_key_columns AS fkc ON fk.object_id = fkc.constraint_object_id
JOIN sys.tables AS tp ON fkc.parent_object_id = tp.object_id
JOIN sys.columns AS cp ON fkc.parent_object_id = cp.object_id AND fkc.parent_column_id = cp.column_id
JOIN sys.tables AS tr ON fkc.referenced_object_id = tr.object_id
JOIN sys.columns AS cr ON fkc.referenced_object_id = cr.object_id AND fkc.referenced_column_id = cr.column_id
WHERE tp.name = 'TableName' OR tr.name = 'TableName'
ORDER BY ParentTable, ForeignKeyName;
```
Rationale: different sites use different column names (example: frwObjectCat.Sort vs OBJECTCAT_Sort). Always confirm. 

---

## 4. How to Inspect Stored Procedures
- Definition + body:
```sql
SELECT SCHEMA_NAME(p.schema_id) AS SchemaName, p.name AS ProcedureName, p.create_date, p.modify_date, m.definition
FROM sys.procedures p
LEFT JOIN sys.sql_modules m ON p.object_id = m.object_id
WHERE p.name = 'YourProcName';
```
- Parameters:
```sql
SELECT prm.parameter_id, prm.name AS ParameterName, TYPE_NAME(prm.user_type_id) AS ParameterType, prm.is_output
FROM sys.procedures p
JOIN sys.parameters prm ON p.object_id = prm.object_id
WHERE p.name = 'YourProcName'
ORDER BY prm.parameter_id;
```
Always confirm parameter names/types before calling. 

---

## 5. frw Schema Reference (key tables & important columns)
Always confirm exact columns via INFORMATION_SCHEMA before use.

Core tables:
- frwObjectCat (modules): OBJECTCAT_ID, OBJECTCAT_NAME, Description, Sort, GUID, Icon, IconClass, Category, OBJECTCAT_Parent, OBJECTCAT_NAMEAR, Dashboard, DescriptionAR.  
- frwObjects (primary metadata): Object_Name (PK in many sites), OBJECT (short name), OBJECTCAT_ID, OBJECTCAT_Sort, GUID, Web, WebMenuParent, WebMenu, SourceType, OriginalTable, event/proc fields, security flags.  
- frwDefinitions (fields/controls): Field, Control, LinkTable, LinkField, DisplayField, TabName, TabIndex, Enabled, Invisible, GUID, Formula, Reminder_Report (GUID), Default, Caption, [Object].  
- frwReports: Report_FileName, Report_Description, Report_ObjectName, GUID, Report_Web, Report_ShowSelection.  
- frwPermissions: PERMISSION_USERNAME, PERMISSION_OBJECT, PERMISSION_ACCESS, GUID, Select/Insert/Update/Delete bits.  
- frwWebMenu, frwWebMenuParent: frwWebMenu.ID (may not be identity), Menu, Sort, Icon, GUID; frwWebMenuParent keyed by OBJECTCAT_ID.  
- frwTabs, frwFields, frwValidations, frwRestrictions, frwExceptions, frwTutorials, frwObjects_Collection. 

---

## 6. Workflow & Procedure Tables (new — discovery & use)
- frwProcedures — high-level business process definitions (e.g., "Procurements").  
- frwProcedure_Objects — process steps/flow (Sequence can have gaps).  
- frwProcedure_ObjectsIn — input objects.  
- frwProcedure_ObjectsOut — output objects.  
- frwProcedure_ProcessCategories — categories (Master Data, Configuration, Reports, Drill Down).  
- frwProcedureApprovals — approval settings tied to procedure objects.  
- frwApproval_Type — lookup: 0=Not Required, 1=Ad hoc, 2=All must, 3=Majority, 4=Percentage.  
- frwObject_Approvals — approval sequences with users, formulas, escalation.  
- frwWorkflow and frwWorkflow_Approvals — workflow definitions and approval configs.  
Note: frwValidations uses ObjectID (not Object_Name) — confirm before queries. 

---

## 7. Approval Formula Patterns (common)
- [FieldName] — dynamic approver from a record field.  
- [CreatedUser] — record creator as approver.  
- @Skip — skip this sequence.  
- IIF(condition, value, @Skip) — conditional approver.  
Document formulas from frwObject_Approvals and frwWorkflow_Approvals and test in a non-production snapshot. 

---

## 8. frwAI Procedures — usage, parameters, examples (safe templates)
General safety wrapper (always use):
1. Initialize connection (separate client call).
2. BEGIN TRAN;
3. EXEC dbo.frwAI_* (with parameters)
4. EXEC dbo.frwAI_Verify*JSON(...) — inspect JSON
5. ROLLBACK; (test)
6. Repeat as needed, then COMMIT when verified.

8.1 frwAI_CreateModule (create module + WebMenuParent)
- Typical params: @ModuleName, @CaptionEn, @CaptionAr, @Category, @IconClass, @InteractiveGrant
Example:
```sql
BEGIN TRAN;
EXEC dbo.frwAI_CreateModule
  @ModuleName = N'PP4',
  @CaptionEn   = N'PP4 Module',
  @CaptionAr   = N'الوحدة PP4';
EXEC dbo.frwAI_VerifyCreateModule_JSON @ModuleName = N'PP4';
ROLLBACK;
```
Expected output: OBJECTCAT_ID, Status (Created). Record OBJECTCAT_ID to migration log. 

8.2 frwAI_CopyObject (two-step recommended)
- Step A — object/view only (CopySubrMetadata = 0):
```sql
BEGIN TRAN;
EXEC dbo.frwAI_CopyObject
  @ModuleName = N'PPM',
  @TargetName = N'PPM - Chart of Accounts',
  @TargetValue = N'PPM_LedgerTable',
  @SourceName = N'Chart of Accounts',
  @GrantUser = N'VisualBase',
  @InteractiveGrant = 1,
  @CopySubrMetadata = 0,
  @CreateTargetViewIfNotExists = 1,
  @SetSourceTypeV = 1;
-- Verify UI & view
EXEC dbo.frwAI_VerifyCopyObject_JSON @TargetValue = N'PPM_LedgerTable', @TargetName = N'PPM - Chart of Accounts', @SourceName = N'Chart of Accounts';
ROLLBACK;
```
- Step B — copy subordinate metadata (CopySubrMetadata = 1):
```sql
BEGIN TRAN;
EXEC dbo.frwAI_CopyObject
  @ModuleName = N'PPM',
  @TargetName = N'PPM - Chart of Accounts',
  @TargetValue = N'PPM_LedgerTable',
  @SourceName = N'Chart of Accounts',
  @GrantUser = N'VisualBase',
  @InteractiveGrant = 0,
  @CopySubrMetadata = 1,
  @CreateTargetViewIfNotExists = 0,
  @SetSourceTypeV = 1;
-- Verify counts and GUID mapping
EXEC dbo.frwAI_VerifyCopyObject_JSON @TargetValue = N'PPM_LedgerTable', @TargetName = N'PPM - Chart of Accounts', @SourceName = N'Chart of Accounts';
COMMIT;
```
Field remap rules:
- Generate new GUIDs via NEWID().  
- frwDefinitions.[Object] = TargetValue.  
- Field values: if contains dot (prefix.field) replace prefix with TargetValue; if no dot, prepend TargetValue + '.' + Field.  
- Reminder_Report remapped via OldGUID->NewGUID when reports copied; otherwise set NULL. 

8.3 frwAI_MoveObject
- Pre-checks: ensure target OBJECTCAT_ID exists, frwWebMenuParent exists or create, compute WebMenu ID if missing with ISNULL(MAX(ID),0)+1 protected by applock/sequence.
Example:
```sql
BEGIN TRAN;
EXEC dbo.frwAI_MoveObject
  @ObjectName = N'Banks',
  @TargetObjectCatID = 1392,
  @SetWebMenu = 1,
  @SetWebMenuParent = 1;
EXEC dbo.frwAI_VerifyMoveObject_JSON @ObjectName = N'Banks', @ExpectedTargetObjectCatID = 1392;
ROLLBACK;
```
Procedure returns: UpdatedRows, NewObjectCatID, WebMenuID. Rename Object_Name afterwards if needed. 

---

## 9. Verification Procedures & Use
- dbo.frwAI_VerifyCreateModule_JSON(@ModuleName) — Returns JSON: ModuleName, OBJECTCAT_ID, ModuleExists, WebMenuParentCount, WebMenuParentRows, WebMenuRows.
- dbo.frwAI_VerifyMoveObject_JSON(@ObjectName, @ExpectedTargetObjectCatID) — Returns JSON with counts and WebMenu info; check IsInExpectedModule = 1.
- dbo.frwAI_VerifyCopyObject_JSON(@TargetValue, @TargetName, @SourceName) — Returns JSON with per-subsystem counts and sample rows (use to validate subordinate metadata).
- dbo.frwAI_FindBestObjectMatch_JSON(@UserInput,@Top) — fuzzy matching helper to avoid ambiguous operations. 

Use these JSON returns as machine-readable gates before COMMIT.

---

## 10. Verification Queries (examples)
- Verify module created:
```sql
SELECT OBJECTCAT_ID, OBJECTCAT_NAME, GUID, Icon FROM dbo.frwObjectCat WHERE OBJECTCAT_NAME = 'PPM';
```
- Verify object:
```sql
SELECT Object_Name, [OBJECT], OBJECTCAT_ID, OBJECTCAT_Sort, WebMenuParent, WebMenu, GUID
FROM dbo.frwObjects
WHERE Object_Name = 'PPM - Chart of Accounts';
```
- Verify view existence:
```sql
SELECT OBJECT_ID('dbo.PPM_LedgerTable') AS View_ObjectID,
       OBJECTPROPERTY(OBJECT_ID('dbo.PPM_LedgerTable'), 'IsView') AS IsView;
```
- Verify subordinate counts:
```sql
SELECT COUNT(*) FROM dbo.frwDefinitions WHERE [Object] = 'PPM_LedgerTable';
SELECT COUNT(*) FROM dbo.frwReports WHERE Report_ObjectName = 'PPM - Chart of Accounts';
SELECT * FROM dbo.frwPermissions WHERE PERMISSION_OBJECT = 'PPM_LedgerTable';
```
Run these after each operation and compare to expected values shown by the verify JSON procs. 

---

## 11. Rollback & Cleanup Templates (ready-to-run, customize before use)
Undo a copy (example `PPM_LedgerTable`):
```sql
BEGIN TRAN;
DELETE FROM dbo.frwDefinitions WHERE [Object] = 'PPM_LedgerTable';
DELETE FROM dbo.frwReports WHERE Report_ObjectName = 'PPM - Chart of Accounts';
DELETE FROM dbo.frwPermissions WHERE PERMISSION_OBJECT = 'PPM_LedgerTable';
DELETE FROM dbo.frwValidations WHERE ObjectID = 'PPM_LedgerTable' OR [Object] = 'PPM_LedgerTable';
DELETE FROM dbo.frwRestrictions WHERE [Object] = 'PPM_LedgerTable';
DELETE FROM dbo.frwExceptions WHERE [Object] = 'PPM_LedgerTable';
DELETE FROM dbo.frwTutorials WHERE Object_Name = 'PPM - Chart of Accounts';
DELETE FROM dbo.frwObjects_Collection WHERE Object_Name = 'PPM - Chart of Accounts';
DELETE FROM dbo.frwObjects WHERE Object_Name = 'PPM - Chart of Accounts' OR [OBJECT] = 'PPM_LedgerTable';
DROP VIEW IF EXISTS dbo.PPM_LedgerTable;
COMMIT;
```

Undo a move + rename:
```sql
BEGIN TRAN;
EXEC dbo.frwAI_MoveObject @ObjectName = N'PPM - Employees Full File', @TargetObjectCatID = <original_OBJECTCAT_ID>, @MenuName = N'<original menu>', @SetWebMenu = 1, @SetWebMenuParent = 1;
UPDATE dbo.frwObjects SET Object_Name = N'Employees Full File' WHERE Object_Name = N'PPM - Employees Full File';
COMMIT;
```
Always tailor rollback scripts to your recorded GUID/ID mappings and test in a non-prod snapshot first. 

---

## 12. Troubleshooting Flow (triage)
- frwAI_CopyObject fails:
  - Re-run Step A (`@CopySubrMetadata = 0`) to isolate view/object creation issues.
  - Inspect frwAI_VerifyCopyObject_JSON for subsystem counts and sample rows.
  - Ensure frwWebMenuParent / frwWebMenu exist; create if needed.
  - Verify executing account privileges; use `@InteractiveGrant = 1` if needed.
  - For ID generation concurrency, use `sp_getapplock` or a SEQUENCE.
- Missing stored-procs: inspect definition via sys.sql_modules and adapt to site differences. 

---

## 13. Concurrency & ID Generation Guidance
- Low-concurrency admin tasks: `ISNULL(MAX(col),0)+1` acceptable.  
- High concurrency: use `sp_getapplock` around MAX+1 logic OR SQL `SEQUENCE` OR dedicated ID allocation table. Protect WebMenu ID / OBJECTCAT_ID allocations similarly. 

---

## 14. Inputs I Expect from Users (pre-action checklist)
- Create module: ModuleName, CaptionEn, optional CaptionAr, Category, IconClass/Icon binary, confirm menu creation.  
- Move object: ObjectName (exact), TargetObjectCatID, whether to set WebMenuParent/WebMenu, MenuName/MenuIcon.  
- Copy object: SourceName (Object_Name or [OBJECT]), TargetName (display), TargetValue (internal short name), Target module OBJECTCAT_ID, CopySubrMetadata (1/0), CreateTargetViewIfNotExists (1/0), GrantUser.  
Always require confirmation after discovery SELECTs and before COMMIT. 

---

## 15. Audit / Logging / GUID mapping conventions
- Save each operation outcome (OBJECTCAT_ID, WebMenuID, UpdatedRows, new GUIDs) to a migration log table or document.  
- Preserve OldGUID->NewGUID mappings during copies for report remapping and rollback.  
- Example audit line format: Timestamp | Operation | Object_Name | ShortObject | OldModuleID | NewModuleID | WebMenuID | GUIDMapFile. 

---

## 16. Workflow & Procedures Example (Procurements)
Sample process flow (sequence numbers may have gaps):
Purchase Requisition (Seq 1) → Purchase RFQ (Seq 2) → Purchase Orders (Seq 3) → Vendor Evaluation (Seq 6) → Purchase Delivery (Seq 7) → Purchase Invoices (Seq 8).  
Record GUIDs and sequences in migration log for traceability. 

---

## 17. Appendix: frwAI_Documentation table (operational note)
The session saved docs into frwAI_Documentation. Example retrieval:
```sql

-- Step 1: Check for my documentation
SELECT DocName FROM frwAI_Documentation;

-- Step 2: Load quick reference
SELECT DocContent FROM frwAI_Documentation 
WHERE DocName = 'AI Agent Self-Reference Guide';

```
Add docs:
```sql
INSERT INTO frwAI_Documentation (DocName, DocCategory, DocContent, CreatedBy, Version)
VALUES ('New Doc Name','Category',N'Content here','AI Agent','1.0');
```
Maintain these docs as quick references. 

---

## 18. How I will behave next time (runbook summary)
1. Initialize (client).  
2. Run discovery (INFORMATION_SCHEMA + stored-proc list).  
3. Present preview SELECTs to approver.  
4. Wrap DML in transaction + run frwAI_Verify* JSON.  
5. ROLLBACK to test; COMMIT only after approval.  
6. Record audit entries and GUID mappings.  
7. Provide rollback script and confirm completion.
8. When starting a new session, I should read the operational note (frwAI_Documentation)
   

##19. After this line find the updates and consider it cearlflly 




═══════════════════════════════════════════════════════════════════════════════
OBJECT CREATION PROCEDURES
═══════════════════════════════════════════════════════════════════════════════

frwAI_CreateObject
──────────────────────────────────────────────────────────────────────────────

PURPOSE:
Automates complete creation of a VisualBase framework object including all 
required table entries, permissions, and menu configuration.

PARAMETERS:
┌────────────────────┬────────────────┬──────────┬─────────────────────────────┐
│ Parameter          │ Type           │ Required │ Default                     │
├────────────────────┼────────────────┼──────────┼─────────────────────────────┤
│ @ObjectTable       │ NVARCHAR(100)  │ YES      │ -                           │
│ @ObjectNameEN      │ NVARCHAR(200)  │ YES      │ -                           │
│ @ObjectNameAR      │ NVARCHAR(200)  │ YES      │ -                           │
│ @ModuleID          │ INT            │ YES      │ -                           │
│ @Icon              │ NVARCHAR(100)  │ NO       │ 'fa fa-file'                │
│ @CopyPermissionsFrom│ NVARCHAR(200) │ NO       │ NULL                        │
│ @DefaultUser       │ NVARCHAR(100)  │ NO       │ 'VisualBase'                │
└────────────────────┴────────────────┴──────────┴─────────────────────────────┘

WHAT IT CREATES:
• frwWebMenu        - Menu entry with EN/AR names and icon
• frwObjects        - Object config (Status=Released, Show/Add/Edit/Delete/Print=1)
• frwTabs           - Default tab: "1.General.عام"
• frwDefinitions    - One per table column with auto-detected control types
• frwGroupPermissions - Copied or default (Admin group)
• frwWebPermissions - Copied + DefaultUser guaranteed
• frwPermissions    - Auto-refreshed for DefaultUser

CONTROL TYPE AUTO-DETECTION:
• int, bigint, decimal, numeric, float, money  →  TextBox
• datetime, datetime2, smalldatetime           →  DateTimePicker
• date                                         →  DateGregorian
• bit                                          →  CheckBox
• uniqueidentifier                             →  Label
• varchar, nvarchar, text                      →  TextBox

USAGE EXAMPLES:

Basic (with defaults):
    -- Step 1: Create table first
    CREATE TABLE dbo.MyNewTable (
        ID INT IDENTITY(1,1) PRIMARY KEY,
        Name NVARCHAR(200),
        Status NVARCHAR(50),
        CreatedDate DATETIME DEFAULT GETDATE(),
        GUID UNIQUEIDENTIFIER DEFAULT NEWID()
    )

    -- Step 2: Create object
    EXEC dbo.frwAI_CreateObject 
        @ObjectTable = 'MyNewTable',
        @ObjectNameEN = 'My New Object',
        @ObjectNameAR = N'الكائن الجديد',
        @ModuleID = 1401

With custom icon and copied permissions:
    EXEC dbo.frwAI_CreateObject 
        @ObjectTable = 'VIPVisitorsRequest',
        @ObjectNameEN = 'VIP Visitors Request',
        @ObjectNameAR = N'طلب زيارة كبار الشخصيات',
        @ModuleID = 1401,
        @Icon = 'fa fa-star',
        @CopyPermissionsFrom = 'Visitors (Basic Information)'

RESPONSE FORMAT:

Success:
    {
      "status": "SUCCESS",
      "message": "Object created & permissions refreshed",
      "objectTable": "VIPVisitorsRequest",
      "objectName": "VIP Visitors Request",
      "webMenuID": 436,
      "defaultUser": "VisualBase",
      "permissionsRefreshed": true
    }

Error:
    {
      "status": "ERROR",
      "message": "Table does not exist. Create table first."
    }

PRE-VALIDATION CHECKS:
1. All required parameters provided
2. Object doesn't already exist in frwObjects
3. Database table exists
4. ModuleID exists in frwWebMenuParent


═══════════════════════════════════════════════════════════════════════════════

frwAI_VerifyCreateObject_JSON
──────────────────────────────────────────────────────────────────────────────

PURPOSE:
Validates an existing object configuration and returns JSON report with issues.
Use after creating objects or to diagnose visibility problems.

PARAMETERS:
┌────────────────────┬────────────────┬──────────┬─────────────────────────────┐
│ Parameter          │ Type           │ Required │ Description                 │
├────────────────────┼────────────────┼──────────┼─────────────────────────────┤
│ @ObjectTable       │ NVARCHAR(100)  │ YES      │ Database table name         │
└────────────────────┴────────────────┴──────────┴─────────────────────────────┘

WHAT IT CHECKS:
┌─────────────────────┬─────────────────────────────────┬──────────┐
│ Category            │ Check                           │ Severity │
├─────────────────────┼─────────────────────────────────┼──────────┤
│ frwObjects          │ Status = 'Released'             │ CRITICAL │
│                     │ Show = 1                        │ CRITICAL │
│                     │ Add = 1                         │ CRITICAL │
│                     │ Edit = 1                        │ CRITICAL │
│                     │ Delete = 1                      │ CRITICAL │
│                     │ Print = 1                       │ HIGH     │
│                     │ OBJECTCAT_Sort not NULL         │ HIGH     │
│                     │ Object_NameAR not NULL          │ HIGH     │
│                     │ OriginalTable not NULL          │ MEDIUM   │
├─────────────────────┼─────────────────────────────────┼──────────┤
│ frwTabs             │ At least one tab exists         │ CRITICAL │
│                     │ All tabs in definitions exist   │ CRITICAL │
├─────────────────────┼─────────────────────────────────┼──────────┤
│ frwDefinitions      │ RighToLeftLanguage set          │ HIGH     │
├─────────────────────┼─────────────────────────────────┼──────────┤
│ frwGroupPermissions │ At least one entry exists       │ CRITICAL │
├─────────────────────┼─────────────────────────────────┼──────────┤
│ frwWebPermissions   │ At least one entry exists       │ HIGH     │
├─────────────────────┼─────────────────────────────────┼──────────┤
│ frwWebMenu          │ MenuAR not NULL                 │ HIGH     │
├─────────────────────┼─────────────────────────────────┼──────────┤
│ frwValidations      │ Compare not NULL                │ HIGH     │
│                     │ ArabicTitle not NULL            │ HIGH     │
└─────────────────────┴─────────────────────────────────┴──────────┘

USAGE:
    EXEC dbo.frwAI_VerifyCreateObject_JSON @ObjectTable = 'VIPVisitorsRequest'

RESPONSE FORMAT:

PASSED (no issues):
    {
      "summary": {
        "objectName": "VIP Visitors Request",
        "objectTable": "VIPVisitorsRequest",
        "critical": 0,
        "high": 0,
        "medium": 0,
        "total": 0,
        "status": "PASSED"
      },
      "issues": []
    }

FAILED (critical issues):
    {
      "summary": {
        "objectName": "Visit Request",
        "objectTable": "VisitRequest",
        "critical": 4,
        "high": 1,
        "medium": 0,
        "total": 5,
        "status": "FAILED"
      },
      "issues": [
        {
          "Category": "frwObjects",
          "Item": "Show",
          "Issue": "Show is disabled",
          "Severity": "CRITICAL",
          "CurrentValue": "0",
          "ExpectedValue": "1"
        }
      ]
    }

STATUS MEANINGS:
• PASSED   - No issues, object ready to use
• WARNINGS - Has HIGH/MEDIUM issues, may work but needs attention
• FAILED   - Has CRITICAL issues, object won't function correctly


═══════════════════════════════════════════════════════════════════════════════

WORKFLOW: CREATE & VERIFY
──────────────────────────────────────────────────────────────────────────────

-- 1. Create table
CREATE TABLE dbo.NewVisitorType (
    ID INT IDENTITY(1,1) PRIMARY KEY,
    TypeName NVARCHAR(100),
    TypeNameAR NVARCHAR(100),
    IsActive BIT DEFAULT 1,
    GUID UNIQUEIDENTIFIER DEFAULT NEWID()
)

-- 2. Create object
EXEC dbo.frwAI_CreateObject 
    @ObjectTable = 'NewVisitorType',
    @ObjectNameEN = 'Visitor Types',
    @ObjectNameAR = N'أنواع الزوار',
    @ModuleID = 1401,
    @Icon = 'fa fa-tags'

-- 3. Verify
EXEC dbo.frwAI_VerifyCreateObject_JSON @ObjectTable = 'NewVisitorType'


═══════════════════════════════════════════════════════════════════════════════

TROUBLESHOOTING
──────────────────────────────────────────────────────────────────────────────

OBJECT NOT VISIBLE IN MENU:
    Cause: frwPermissions table not refreshed
    Fix:
        DECLARE @UserGUID UNIQUEIDENTIFIER
        SELECT @UserGUID = [GUID] FROM dbo.frwUsers WHERE USER_NAME = 'VisualBase'
        EXEC dbo.frwWeb_InsertPermissions @GUID = @UserGUID

PERMISSION OBJECT NAMING:
    IMPORTANT: Use Object_Name (display name), NOT table name!
    
    CORRECT: 'VIP Visitors Request'
    WRONG:   'VIPVisitorsRequest'

FIELD FORMAT IN frwDefinitions:
    IMPORTANT: Field column must use TableName.FieldName format!
    
    CORRECT: 'Visitors.Status'
    WRONG:   'Status'


═══════════════════════════════════════════════════════════════════════════════

QUICK REFERENCE QUERIES
──────────────────────────────────────────────────────────────────────────────

-- List available modules
SELECT OBJECTCAT_ID, OBJECTCAT_NAME FROM dbo.frwWebMenuParent ORDER BY OBJECTCAT_ID

-- Check if object exists
SELECT * FROM dbo.frwObjects WHERE [OBJECT] = 'TableName'

-- Check user permissions (definition)
SELECT * FROM dbo.frwWebPermissions WHERE PERMISSION_USERNAME = 'VisualBase'

-- Check user permissions (runtime)
SELECT * FROM dbo.frwPermissions WHERE PERMISSION_USERNAME = 'VisualBase'

-- Get user GUID for refresh
SELECT USER_NAME, [GUID] FROM dbo.frwUsers WHERE USER_NAME = 'VisualBase'

-- Refresh user permissions
DECLARE @g UNIQUEIDENTIFIER
SELECT @g = [GUID] FROM dbo.frwUsers WHERE USER_NAME = 'VisualBase'
EXEC dbo.frwWeb_InsertPermissions @GUID = @g


═══════════════════════════════════════════════════════════════════════════════

PERMISSIONS ARCHITECTURE
──────────────────────────────────────────────────────────────────────────────

TWO-TABLE SYSTEM:

    ┌─────────────────────┐
    │  frwWebPermissions  │  DEFINITION (what user SHOULD have)
    └──────────┬──────────┘
               │
               │  REFRESH REQUIRED (frwWeb_InsertPermissions)
               ▼
    ┌─────────────────────┐
    │   frwPermissions    │  RUNTIME (what app READS)
    └─────────────────────┘

PERMISSION_ACCESS VALUES:
    1 = Show (View)
    2 = Add (Create)
    3 = Edit (Modify)
    4 = Delete
    5 = Print
    6 = ExportData



    ## ⚠️ CRITICAL LESSON
    frwAI_CreateObject creates BASIC definitions only! You MUST manually configure field definitions afterward.

    ## WHAT frwAI_CreateObject DOES:
    ✅ Creates frwObjects entry
    ✅ Creates frwWebMenu entry
    ✅ Creates frwPermissions for default user (VisualBase)
    ✅ Creates BASIC frwDefinitions (all fields in one tab, no Arabic, TabIndex=0)
    ✅ Creates ONE generic frwTabs entry (1.General.عام)

    ## WHAT YOU MUST DO AFTER:
    ❌ frwAI_CreateObject does NOT properly configure:
    - Multiple tabs (only creates 1.General.عام)
    - Arabic translations (RighToLeftLanguage is column name, not Arabic)
    - Proper TabIndex ordering (all are 0)
    - System fields visibility (not hidden)
    - Mandatory field markers
    - Proper control types for numeric/memo fields

    ## COMPLETE POST-CREATION WORKFLOW:

    ### Step 1: ADD ADDITIONAL TABS
    INSERT INTO frwTabs ([Object], TabName, HasMandatory, GUID)
    VALUES 
        ('ObjectName', N'2.Tab Name.اسم التبويب', 0, NEWID()),
        ('ObjectName', N'3.Another Tab.تبويب آخر', 0, NEWID())

    ### Step 2: UPDATE FIELD DEFINITIONS
    For EACH visible field:
    UPDATE frwDefinitions SET 
        Caption = 'English Caption',
        RighToLeftLanguage = N'التسمية بالعربي',
        TabName = N'1.Tab Name.اسم التبويب',
        TabIndex = 1,
        Mandatory = 1,
        Control = 'ControlType'
    WHERE Field = 'TableName.FieldName'

    ### Step 3: HIDE SYSTEM FIELDS
    UPDATE frwDefinitions SET 
        WebInvisible = 1,
        Invisible = 1,
        TabIndex = 99
    WHERE Field IN (
        'TableName.GUID',
        'TableName.CreatedDate',
        'TableName.CreatedUser',
        'TableName.ModifiedDate',
        'TableName.ModifiedUser'
    )

    ## TAB NAMING FORMAT
    Format: [Order].[English Name].[Arabic Name]
    Examples:
    - 1.General.عام
    - 2.Contact Information.معلومات الاتصال
    - 3.Payment.الدفع
    - 4.Notes.ملاحظات

    ## COMMON CONTROL TYPES
    | Use Case        | Control Type    |
    |-----------------|-----------------|
    | Text input      | TextBox         |
    | Numbers/Amount  | NumericBox      |
    | Multi-line text | MemoBox         |
    | Date only       | DateGregorian   |
    | Date and time   | DateTimePicker  |
    | Yes/No          | CheckBox        |
    | Dropdown        | ComboBox        |
    | Display only    | Label           |

    ⚠️ WARNING: NumericUpDown does NOT exist! Use NumericBox instead.

    ## SYSTEM FIELDS TO ALWAYS HIDE
    - GUID
    - CreatedDate
    - CreatedUser
    - ModifiedDate
    - ModifiedUser

    ## VERIFICATION
    After all steps, run:
    EXEC frwAI_VerifyCreateObject_JSON @ObjectTable = 'TableName'

    Version: 1.0 | Created: 2025-12-14 | Source: Member Subscriptions creation session


