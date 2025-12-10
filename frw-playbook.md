
Note: VisualBase is the current product name (rebrand of InnovBase). Use "VisualBase" as the canonical product name in scripts, documentation and grants; include "InnovBase" only in historical notes if needed.

This is the canonical internal playbook for safe operations on frw* metadata in VisualBase (aka InnovBase). It merges the original playbook and the operational lessons learned during today's activities (creation of the Hisham PPM module and moving Banks). The original playbook contents are preserved and extended here for clarity and future operations .

---

## 1. Purpose & Assumptions
- frw* tables store the application metadata (modules, objects, fields/controls, tabs, reports, menus, permissions, workflows, validations, exceptions, tutorials, collections).
- Always discover (read-only) before DML, preview intended changes with SELECTs, and run DML inside transactions with verification and rollback prepared.
- Use the DefaultConnection via MCP Tools: call `mssql_initialize_connection('DefaultConnection')` before queries (call this separately — see learnings below).

---

## 2. Mandatory Initialization
Always run the initialization as a separate call (do not embed it inside a T-SQL batch):
```
EXEC mssql_initialize_connection('DefaultConnection');
```
Rationale: calling initialization inside a SQL batch caused a syntax error in today's run. Initialize once from the client/tool layer, then submit SQL batches.

---

## 3. High-level rules / safety checklist (always follow)
- Discover first (read-only): inspect schema, object names, GUIDs, menus, counts.
- Prepare and preview changes with SELECTs.
- Wrap every destructive DML in a transaction for verification: BEGIN TRAN; … verification SELECTs … ROLLBACK; then re-run with COMMIT when satisfied.
- Do not assume numeric columns are IDENTITY — use MAX(...) + 1 when inserting manual IDs and consider concurrency (sp_getapplock if needed).
- Use NEWID() for GUIDs on inserts.
- Keep a reviewed rollback script ready before committing.
- Prefer two-step copy pattern: Step A = light (object/view only), Step B = full metadata copy.
- Check stored-proc existence before calling (e.g., frwAI_CreateModule, frwAI_MoveObject, frwAI_CopyObject, frwAI_Verify*). If a helper proc is missing, fall back to a manual, well-reviewed SQL flow.

---

## 4. Core Discovery Queries (Read-only)
- List base tables:
```
SELECT TABLE_SCHEMA AS SchemaName, TABLE_NAME AS TableName
FROM INFORMATION_SCHEMA.TABLES
WHERE TABLE_TYPE = 'BASE TABLE'
ORDER BY SchemaName, TableName;
```
- List views:
```
SELECT TABLE_SCHEMA AS SchemaName, TABLE_NAME AS ViewName
FROM INFORMATION_SCHEMA.VIEWS
ORDER BY SchemaName, ViewName;
```
- List stored procedures:
```
SELECT SPECIFIC_SCHEMA AS SchemaName, SPECIFIC_NAME AS ProcedureName
FROM INFORMATION_SCHEMA.ROUTINES
WHERE ROUTINE_TYPE = 'PROCEDURE'
ORDER BY SchemaName, ProcedureName;
```
- Get column list for a table (always run before referencing columns):
```
SELECT COLUMN_NAME, DATA_TYPE, IS_NULLABLE, COLUMN_DEFAULT, CHARACTER_MAXIMUM_LENGTH
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_NAME = 'YourTableName'
ORDER BY ORDINAL_POSITION;
```
Rationale: Today's failure came from querying a non-existent column name (OBJECTCAT_Sort on frwObjectCat). Always validate columns in INFORMATION_SCHEMA first.

- Get foreign key relations for a table:
(Replace `TableName`)
```
SELECT fk.name AS ForeignKeyName, tp.name AS ParentTable, cp.name AS ParentColumn,
       tr.name AS ReferencedTable, cr.name AS ReferencedColumn
FROM sys.foreign_keys AS fk
INNER JOIN sys.foreign_key_columns AS fkc ON fk.object_id = fkc.constraint_object_id
INNER JOIN sys.tables AS tp ON fkc.parent_object_id = tp.object_id
INNER JOIN sys.columns AS cp ON fkc.parent_object_id = cp.object_id AND fkc.parent_column_id = cp.column_id
INNER JOIN sys.tables AS tr ON fkc.referenced_object_id = tr.object_id
INNER JOIN sys.columns AS cr ON fkc.referenced_object_id = cr.object_id AND fkc.referenced_column_id = cr.column_id
WHERE tp.name = 'TableName' OR tr.name = 'TableName'
ORDER BY ParentTable, ForeignKeyName;
```

---

## 5. How to Inspect a Stored Procedure (metadata + parameters)
- Procedure definition:
```
SELECT
  SCHEMA_NAME(p.schema_id) AS SchemaName,
  p.name AS ProcedureName,
  p.create_date,
  p.modify_date,
  m.definition
FROM sys.procedures p
LEFT JOIN sys.sql_modules m ON p.object_id = m.object_id
WHERE SCHEMA_NAME(p.schema_id) = 'dbo'
  AND p.name = 'YourProcName';
```
- Procedure parameters:
```
SELECT
  p.name AS ProcedureName,
  prm.parameter_id,
  prm.name AS ParameterName,
  TYPE_NAME(prm.user_type_id) AS ParameterType,
  prm.max_length,
  prm.precision,
  prm.scale,
  prm.is_output,
  prm.has_default_value,
  prm.default_value
FROM sys.procedures p
JOIN sys.parameters prm ON p.object_id = prm.object_id
WHERE SCHEMA_NAME(p.schema_id) = 'dbo'
  AND p.name = 'YourProcName'
ORDER BY prm.parameter_id;
```

---

## 6. Key frw Tables & Important Columns
(Validate exact columns via INFORMATION_SCHEMA before use; the names below reflect observed schema but can vary per site.)

- frwObjectCat (modules):
  - OBJECTCAT_ID (int, may not be identity)
  - OBJECTCAT_NAME (varchar)
  - Description
  - Sort (note: column name is Sort in this DB — do not assume OBJECTCAT_Sort)
  - GUID (uniqueidentifier, default NEWID())
  - Icon / IconClass
  - Category (varchar, e.g., 'Core')
  - OBJECTCAT_Parent, OBJECTCAT_NAMEAR, Dashboard, DescriptionAR

- frwObjects (primary metadata)
  - Primary key in this DB: Object_Name (varchar(50))
  - Key columns: Object_Name, OBJECT (short technical name), OBJECTCAT_ID, OBJECTCAT_Sort, GUID, Web, WebMenuParent, WebMenu, SourceType, OriginalTable, many event/procedure fields, security flags.
  - Operational note: Always confirm current columns with INFORMATION_SCHEMA before using them.

- frwDefinitions (field/control metadata)
  - Field, Control, LinkTable, LinkField, DisplayField, TabName, TabIndex, Enabled, Invisible, GUID, Formula, Reminder_Report (uniqueidentifier), Default, Caption, [Object].

- frwReports
  - Report_FileName, Report_Description, Report_ObjectName, GUID, Report_Web, Report_ShowSelection.

- frwPermissions
  - PERMISSION_USERNAME, PERMISSION_OBJECT, PERMISSION_ACCESS, GUID, Select/Insert/Update/Delete bits.

- frwWebMenu, frwWebMenuParent
  - frwWebMenu: ID (may be non-identity), Menu, Sort, Icon, GUID
  - frwWebMenuParent keyed by OBJECTCAT_ID — needed for module parent menu.

- frwTabs, frwFields, frwValidations, frwRestrictions, frwExceptions, frwTutorials, frwObjects_Collection
  - Include these when copying subordinate metadata.

---

## 7. Safety & Transactional Rules
- Always preview with SELECTs and show user before DML.
- Wrap DML in transactions and test via ROLLBACK before COMMIT:
```
BEGIN TRAN;
-- perform actions
-- verification SELECTs
ROLLBACK;  -- test
-- COMMIT;  -- only after user confirmation and verification
```
- Do not assume numeric columns are identity — compute `ISNULL(MAX(col),0)+1` when needed.
- Use `NEWID()` for GUIDs.
- For concurrency use `sp_getapplock` or SEQUENCE objects rather than naive MAX+1 in high-concurrency environments.
- Prefer verifying procedures: use JSON verify procs to gate steps (frwAI_VerifyCreateModule_JSON, frwAI_VerifyMoveObject_JSON, frwAI_VerifyCopyObject_JSON).

---

## 8. frwAI_CopyObject: Rewrite & Remap Rules
- Two-step pattern: Step A (create object/view only) then Step B (copy subordinate metadata).
- Subordinate metadata copy:
  - New GUIDs generated with `NEWID()` for each copied record.
  - `frwDefinitions.[Object]` set to TargetValue (short object name).
  - Field rewriting:
    - If Field contains a dot (prefix.field): replace prefix with TargetValue (TargetValue + '.' + remainder).
    - If no dot: set Field = TargetValue + '.' + Field.
  - `Reminder_Report` is remapped via OldGUID->NewGUID table when reports are copied; otherwise set to NULL.
  - Copy frwReports, frwDefinitions, frwPermissions, frwValidations, frwRestrictions, frwExceptions, frwTutorials, frwObjects_Collection.
  - If `CreateTargetViewIfNotExists = 1`, create the view `dbo.TargetValue`.

---

## 9. Two-step Safe Copy Pattern (Recommended)
- Step A — Create object + view only:
```
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
-- Verify UI and view
ROLLBACK; -- test
```
- Step B — Copy subordinate metadata (run only after Step A verification):
```
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
-- Verify frwDefinitions/frwReports/frwPermissions counts
COMMIT;
```

---

## 10. Move Object Best Practices (frwAI_MoveObject)
- Preferred: call `frwAI_MoveObject`.
- Ensure target `OBJECTCAT_ID` exists.
- Ensure `frwWebMenuParent` exists for target `OBJECTCAT_ID` (procedures may create it if permitted).
- Ensure `frwWebMenu` entry exists for the menu; if missing compute `ID = ISNULL(MAX(ID),0)+1`.
- Compute `OBJECTCAT_Sort = ISNULL(MAX(OBJECTCAT_Sort),0)+1` for destination and update `frwObjects`: `OBJECTCAT_ID`, `OBJECTCAT_Sort`, `WebMenuParent`, `WebMenu`.
- Procedure returns `UpdatedRows`, `NewObjectCatID`, `WebMenuID`.
- Rename `Object_Name` with an UPDATE if you need a new display name.

---

## 11. Verification Queries (Examples)
- Verify module:
```
SELECT OBJECTCAT_ID, OBJECTCAT_NAME, GUID, Icon FROM dbo.frwObjectCat WHERE OBJECTCAT_NAME = 'PPM';
```
- Verify object:
```
SELECT Object_Name, [OBJECT], OBJECTCAT_ID, OBJECTCAT_Sort, WebMenuParent, WebMenu, GUID
FROM dbo.frwObjects
WHERE Object_Name = 'PPM - Chart of Accounts';
```
- Verify view existence:
```
SELECT OBJECT_ID('dbo.PPM_LedgerTable') AS View_ObjectID, OBJECTPROPERTY(OBJECT_ID('dbo.PPM_LedgerTable'), 'IsView') AS IsView;
```
- Verify subordinate counts:
```
SELECT COUNT(*) FROM dbo.frwDefinitions WHERE [Object] = 'PPM_LedgerTable';
SELECT COUNT(*) FROM dbo.frwReports WHERE Report_ObjectName = 'PPM - Chart of Accounts';
SELECT * FROM dbo.frwPermissions WHERE PERMISSION_OBJECT = 'PPM_LedgerTable';
```

---

## 12. JSON verification procedures (NEW — recommended)
- dbo.frwAI_VerifyCreateModule_JSON(@ModuleName)  
  - Returns JSON: ModuleName, OBJECTCAT_ID, ModuleExists, WebMenuParentCount, WebMenuParentRows, WebMenuRows
- dbo.frwAI_VerifyMoveObject_JSON(@ObjectName, @ExpectedTargetObjectCatID)  
  - Returns JSON: InputName, ShortObject, Found, ObjectName, OBJECTCAT_ID, OBJECTCAT_Sort, WebMenuParent, WebMenu, DefinitionsCount, ReportsCount, PermissionsCount, IsInExpectedModule
- dbo.frwAI_VerifyCopyObject_JSON(@TargetValue, @TargetName, @SourceName)  
  - Returns JSON with per-subsystem counts and samples — use immediately after copy operations.

---

## 13. Best-match helper (NEW)
- dbo.frwAI_FindBestObjectMatch_JSON(@UserInput,@Top)  
  - Returns a JSON array of best candidate objects for ambiguous names using heuristics (exact match, prefix, contains, SOUNDEX, length proximity). Each item includes Object_Name, ShortObject, OBJECTCAT_ID, OBJECTCAT_NAME, Score.
  - Use when the user provides an ambiguous object name; show top results for confirmation.

---

## 14. Rollback & Cleanup Templates
- Undo a copy (example `PPM_LedgerTable`):
```
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
- Undo a move + rename:
```
BEGIN TRAN;
EXEC dbo.frwAI_MoveObject @ObjectName = N'PPM - Employees Full File', @TargetObjectCatID = <original_OBJECTCAT_ID>, @MenuName = N'<original menu>', @SetWebMenu = 1, @SetWebMenuParent = 1;
UPDATE dbo.frwObjects SET Object_Name = N'Employees Full File' WHERE Object_Name = N'PPM - Employees Full File';
COMMIT;
```

---

## 15. Troubleshooting Flow
- If frwAI_CopyObject fails:
  - Re-run Step A (`@CopySubrMetadata = 0`) to isolate view/object creation failures.
  - Inspect the verify proc JSON for per-subsystem counts and samples.
  - Ensure frwWebMenuParent/frwWebMenu exist; create if needed.
  - Check executing account privileges; consider `@InteractiveGrant = 1`.
  - For ID generation, consider `sp_getapplock` or SEQUENCE objects for concurrency.
- If stored procs have been modified, inspect sys.sql_modules and adapt.

---

## 16. Concurrency & ID Generation
- `ISNULL(MAX(col),0)+1` is acceptable for low-concurrency admin tasks.
- In concurrent environments use:
  - `sp_getapplock` around MAX+1 logic, or
  - SQL `SEQUENCE` objects, or
  - a dedicated ID allocation table.

---

## 17. Inputs I Expect from the User Before Acting
- Create module: ModuleName, CaptionEn, optional CaptionAr, Category, IconClass/Icon binary, confirm menu creation.
- Move object: ObjectName (exact), TargetObjectCatID, whether to set WebMenuParent/WebMenu, MenuName/MenuIcon, confirm preview.
- Copy object: SourceName (Object_Name or [OBJECT]), TargetName (display), TargetValue (internal short name), Target module (OBJECTCAT_ID), CopySubrMetadata (1/0), CreateTargetViewIfNotExists (1/0), GrantUser, confirm preview.

---

## 18. Site-specific Audit: Changes Performed Today
- Existing audit entries remain.
- New: Module `Hisham PPM` was created and assigned OBJECTCAT_ID = 1392 (created via dbo.frwAI_CreateModule).
- New: `Banks` (Object_Name = 'Banks', OBJECT = 'BankTable', GUID = cd080d28-71a3-4ffe-acb5-f248cca304ce) was moved from OBJECTCAT_ID = 1391 → OBJECTCAT_ID = 1392. frwAI_MoveObject returned UpdatedRows = 1, NewObjectCatID = 1392, WebMenuID = 418. WebMenuParent was set to 1392 and OBJECTCAT_Sort for the Banks row is 1.
- Keep this audit in the GUID mapping / change log for rollback and review.

---

## 19. How I Will Behave Next Time
- Initialize connection as a separate client/tool call, inspect information schema, run preview SELECTs and show results for user confirmation.
- Use `frwAI_*` procedures for standard operations (CreateModule, MoveObject, CopyObject) when present; otherwise produce reviewed manual SQL.
- For custom flows produce commented SQL for review and only run after explicit confirmation.
- Always return created IDs, proc status and verification SELECTs; provide rollback SQL if requested.

---

## 20. Lessons learned (added / updated from today's run)
These are actionable, specific lessons captured from today's attempt and subsequent successful execution:
- Do not call the initialization procedure inside a SQL batch: call `mssql_initialize_connection('DefaultConnection')` separately from the SQL you send to the server. Embedding it caused a syntax error.
- Always validate column names with INFORMATION_SCHEMA before using them in queries. Example: frwObjectCat uses column name `Sort` (not `OBJECTCAT_Sort`) in this DB; querying a wrong column name triggers an "Invalid column name" error.
- Check for the existence of helper stored procedures before calling them (frwAI_CreateModule, frwAI_MoveObject, frwAI_CopyObject, frwAI_Verify*). If present, prefer them; if absent, fall back to the manual, template-based SQL flow.
- Use the JSON verify procs immediately after frwAI_* operations to produce machine-readable verification and gate subsequent steps.
- Wrap each frwAI_* call in a transaction for preview (ROLLBACK) and only COMMIT after verify procs return expected results.
- Preserve and expose GUID mapping during copy operations; record the mapping in the audit/change log for rollback and traceability.
- When computing new menu IDs or OBJECTCAT_ID, always compute using ISNULL(MAX(...),0)+1 and protect with an application lock or sequence if concurrent operations can happen.
- Maintain a short operational log in the playbook of the exact commands and their returned IDs (OBJECTCAT_ID, WebMenuID, UpdatedRows) for easy rollback — recorded above in Section 18.
- Prefer non-destructive discovery queries (INFORMATION_SCHEMA, SELECTs) as the first step; do not assume runtime behaviors or columns.

---

## 21. Deliverables Available
- Markdown file (this document).
- PDF (if requested).
- Site-specific rollback script for the Hisham PPM / Banks changes (see below).
- GUID mapping/audit lines for the Banks move and any copies performed.

Rollback snippet for today's changes (copy/paste and run only after review)
```
-- Move Banks back to previous module (OBJECTCAT_ID = 1391)
BEGIN TRAN;
EXEC dbo.frwAI_MoveObject
  @ObjectName = N'Banks',
  @TargetObjectCatID = 1391,
  @MenuName = N'<original menu if needed>',
  @SetWebMenu = 1,
  @SetWebMenuParent = 1;
COMMIT;

-- If you must remove Hisham PPM (only after confirming no other objects moved there)
BEGIN TRAN;
DELETE FROM dbo.frwWebMenuParent WHERE OBJECTCAT_ID = 1392;
DELETE FROM dbo.frwObjectCat WHERE OBJECTCAT_ID = 1392;
COMMIT;
```

---

## 22. Contact / Safety Note
- Do not share clear-text credentials or secrets.
- Backup or snapshot the DB before running large or irreversible changes in production.
- Keep an audit trail of OBJECTCAT_ID / GUID mappings for every copy/move for traceability and rollback.

---

Appendix: Short operational log (timestamped)
- 2025-12-10 — Initialized DefaultConnection (client).
- 2025-12-10 — Created module Hisham PPM via dbo.frwAI_CreateModule → OBJECTCAT_ID = 1392 (Status: Created).
- 2025-12-10 — Moved Banks (Object_Name = 'Banks', OBJECT='BankTable', GUID=cd080d28-71a3-4ffe-acb5-f248cca304ce) to OBJECTCAT_ID = 1392 via dbo.frwAI_MoveObject → UpdatedRows = 1, NewObjectCatID = 1392, WebMenuID = 418. Verified new frwObjects row shows OBJECTCAT_Sort = 1 and WebMenuParent = 1392.

