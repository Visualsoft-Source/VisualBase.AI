Note: VisualBase is the canonical product name (rebrand of InnovBase). Use "VisualBase" in scripts, docs and grants; include "InnovBase" only in historical notes.

This single-file markdown is the canonical internal playbook for safe operations on frw* metadata in VisualBase. It merges the original frw playbook and the AI Framework Migration Playbook and includes explicit frwAI_CreateModule guidance.

---

## 1. Purpose & Assumptions
- frw* tables store application metadata (modules, objects, fields/controls, tabs, reports, menus, permissions, workflows, validations, exceptions, tutorials, collections).
- Always discover (read-only) before DML, preview intended changes with SELECTs, and run DML inside transactions with verification and rollback prepared.
- Initialize the DefaultConnection via MCP Tools before running SQL batches (see Section 2).

---

## 2. Mandatory Initialization
Always initialize as a separate client/tool call (do not embed in a T-SQL batch):
```sql
EXEC mssql_initialize_connection('DefaultConnection');
```
Rationale: embedding this call inside a SQL batch caused a syntax error during recent runs; initialize once from the client/tool layer then submit SQL batches.

---

## 3. High-level Rules / Safety Checklist
- Discover first (read-only): inspect schema, object names, GUIDs, menus, counts.
- Prepare and preview changes with SELECTs.
- Wrap destructive DML in a transaction and verify (BEGIN TRAN … verification SELECTs … ROLLBACK; then re-run and COMMIT when satisfied).
- Do not assume numeric columns are IDENTITY — compute `ISNULL(MAX(col),0)+1` for manual IDs and protect with `sp_getapplock` or a `SEQUENCE` in concurrent contexts.
- Use `NEWID()` for GUIDs.
- Keep a reviewed rollback script ready before committing.
- Prefer the two-step copy pattern (Step A = object/view only; Step B = subordinate metadata).
- Verify stored-proc existence (frwAI_CreateModule, frwAI_MoveObject, frwAI_CopyObject, frwAI_Verify*). If missing, use a manual, reviewed SQL flow.

---

## 4. Core Discovery Queries (Read-only)
Always validate objects and columns via INFORMATION_SCHEMA before referencing them. Example checks:
- List base tables:
```sql
SELECT TABLE_SCHEMA AS SchemaName, TABLE_NAME AS TableName
FROM INFORMATION_SCHEMA.TABLES
WHERE TABLE_TYPE = 'BASE TABLE'
ORDER BY SchemaName, TableName;
```
- Get column list for a table:
```sql
SELECT COLUMN_NAME, DATA_TYPE, IS_NULLABLE, COLUMN_DEFAULT, CHARACTER_MAXIMUM_LENGTH
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_NAME = 'YourTableName'
ORDER BY ORDINAL_POSITION;
```
Rationale: querying a wrong column name causes "Invalid column name" errors (example: `frwObjectCat.Sort` vs `OBJECTCAT_Sort` differences across sites).

- Get stored procedures:
```sql
SELECT SPECIFIC_SCHEMA AS SchemaName, SPECIFIC_NAME AS ProcedureName
FROM INFORMATION_SCHEMA.ROUTINES
WHERE ROUTINE_TYPE = 'PROCEDURE'
ORDER BY SchemaName, ProcedureName;
```
- Get FK relations (replace TableName):
```sql
-- (see playbook for full sys.foreign_keys query)
```

---

## 5. Inspecting Stored Procedures
- Procedure definition:
```sql
SELECT SCHEMA_NAME(p.schema_id) AS SchemaName, p.name AS ProcedureName,
       p.create_date, p.modify_date, m.definition
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

---

## 6. Key frw Tables & Important Columns
(Always confirm exact columns using INFORMATION_SCHEMA before use.)
- frwObjectCat (modules): OBJECTCAT_ID, OBJECTCAT_NAME, Description, Sort, GUID, Icon, IconClass, Category, OBJECTCAT_Parent, etc.
- frwObjects (primary metadata): Object_Name (PK in many sites), OBJECT (short name), OBJECTCAT_ID, OBJECTCAT_Sort, GUID, WebMenuParent, WebMenu, SourceType, OriginalTable, event/proc fields, security flags.
- frwDefinitions: Field, Control, LinkTable, LinkField, DisplayField, TabName, TabIndex, Enabled, Invisible, GUID, Formula, Reminder_Report, Default, Caption, [Object].
- frwReports, frwPermissions, frwWebMenu, frwWebMenuParent, frwTabs, frwFields, frwValidations, frwRestrictions, frwExceptions, frwTutorials, frwObjects_Collection.

---

## 7. Safety & Transactional Rules (Operational)
- Preview with SELECTs and show approvers before DML.
- Use:
```sql
BEGIN TRAN;
-- actions
-- verification SELECTs
ROLLBACK; -- or COMMIT after confirmation
```
- Use `NEWID()` for GUIDs.
- Use `sp_getapplock` or `SEQUENCE` objects for ID allocation under concurrency.
- Use JSON verification procedures to gate steps (see Section 12).

---

## 8. frwAI_CreateModule (Clear inclusion)
Procedure: dbo.frwAI_CreateModule

Purpose: create a new module (frwObjectCat row) and associated WebMenuParent entries as needed.

Required parameters (typical)
- @ModuleName (module code or short name)
- @CaptionEn (English caption)
- @CaptionAr (optional Arabic caption)
- @Category (optional, e.g., 'Core')
- @IconClass / @Icon (optional)
- @InteractiveGrant (optional — whether to run interactive grants)

Example:
```sql
EXEC dbo.frwAI_CreateModule
  @ModuleName = N'PP4',
  @CaptionEn   = N'PP4 Module',
  @CaptionAr   = N'الوحدة PP4';
```

Expected output:
- OBJECTCAT_ID (new module id)
- Status (e.g., 'Created' or result code)

Recommended flow:
1. Initialize connection (Section 2).
2. BEGIN TRAN; EXEC dbo.frwAI_CreateModule …; SELECT output; EXEC dbo.frwAI_VerifyCreateModule_JSON(@ModuleName); ROLLBACK; — review JSON.
3. If verification is satisfactory: COMMIT.

Notes:
- Run dbo.frwAI_VerifyCreateModule_JSON(@ModuleName) immediately after creation for machine-readable verification and to list WebMenuParent/WebMenu rows created.

---

## 9. frwAI_CopyObject: Rules & Two-step Pattern
- Two-step recommended pattern:
  - Step A (object/view only): create object entry and view (no subordinate metadata). Use `@CopySubrMetadata = 0`.
  - Step B (subordinate metadata): copy frwDefinitions, frwReports, frwPermissions, frwValidations, frwRestrictions, frwExceptions, frwTutorials, frwObjects_Collection. Use `@CopySubrMetadata = 1`.

Field remapping rules:
- Generate new GUIDs with NEWID() for copied records.
- frwDefinitions.[Object] = TargetValue (short object name).
- Field rewriting:
  - If Field contains a dot (prefix.field): replace prefix with TargetValue (TargetValue + '.' + remainder).
  - If no dot: set Field = TargetValue + '.' + Field.
- Reminder_Report remapped via OldGUID->NewGUID mapping when reports are copied; otherwise set NULL.

Example — Step A:
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
-- VERIFY and ROLLBACK for test
ROLLBACK;
```
Step B (after Step A verification):
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
-- Verify subordinate counts
COMMIT;
```

---

## 10. Move Object Best Practices (frwAI_MoveObject)
- Preferred: call dbo.frwAI_MoveObject.
- Pre-checks:
  - Ensure target OBJECTCAT_ID exists.
  - Ensure frwWebMenuParent exists for the destination OBJECTCAT_ID (procedures may create it).
  - Ensure frwWebMenu entry exists for the menu; compute ID as `ISNULL(MAX(ID),0)+1` if needed.
- Update frwObjects: set OBJECTCAT_ID, OBJECTCAT_Sort (`ISNULL(MAX(OBJECTCAT_Sort),0)+1` for destination), WebMenuParent, WebMenu.
- Procedure returns UpdatedRows, NewObjectCatID, WebMenuID.
- If you need to rename Object_Name, do an UPDATE after the move.

Example:
```sql
BEGIN TRAN;
EXEC dbo.frwAI_MoveObject
  @ObjectName = N'Banks',
  @TargetObjectCatID = 1392,
  @SetWebMenu = 1,
  @SetWebMenuParent = 1;
-- Verify via frwAI_VerifyMoveObject_JSON(...)
COMMIT;
```

---

## 11. Verification Queries (Examples)
- Verify module:
```sql
SELECT OBJECTCAT_ID, OBJECTCAT_NAME, GUID, Icon
FROM dbo.frwObjectCat
WHERE OBJECTCAT_NAME = 'PPM';
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

---

## 12. JSON Verification Procedures (Recommended)
- dbo.frwAI_VerifyCreateModule_JSON(@ModuleName)  
  - Returns JSON: ModuleName, OBJECTCAT_ID, ModuleExists, WebMenuParentCount, WebMenuParentRows, WebMenuRows.
- dbo.frwAI_VerifyMoveObject_JSON(@ObjectName, @ExpectedTargetObjectCatID)  
  - Returns JSON: InputName, ShortObject, Found, ObjectName, OBJECTCAT_ID, OBJECTCAT_Sort, WebMenuParent, WebMenu, DefinitionsCount, ReportsCount, PermissionsCount, IsInExpectedModule.
- dbo.frwAI_VerifyCopyObject_JSON(@TargetValue, @TargetName, @SourceName)  
  - Returns JSON: per-subsystem counts and samples.

Use these JSON outputs to gate commit decisions and to produce machine-readable logs for audit.

---

## 13. Best-match Helper
- dbo.frwAI_FindBestObjectMatch_JSON(@UserInput,@Top)
  - Returns a scored JSON array of candidate objects for ambiguous names (heuristics: exact, prefix, contains, SOUNDEX, length proximity).
  - Use this when user input may be ambiguous; present top candidates and require confirmation before running Move/Copy.

---

## 14. Rollback & Cleanup Templates
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

Keep a reviewed rollback script available before committing any change in production.

---

## 15. Troubleshooting Flow
- If frwAI_CopyObject fails:
  - Re-run Step A (`@CopySubrMetadata = 0`) to isolate view/object creation issues.
  - Inspect the verify proc JSON for per-subsystem counts and samples.
  - Ensure frwWebMenuParent / frwWebMenu exist; create if needed.
  - Check the executing account privileges; set `@InteractiveGrant = 1` if permission elevation is needed.
  - For ID generation concurrency issues, consider `sp_getapplock` or `SEQUENCE`.

---

## 16. Concurrency & ID Generation Guidance
- For low-concurrency admin tasks `ISNULL(MAX(col),0)+1` is acceptable.
- For concurrent operations use `sp_getapplock`, SQL `SEQUENCE`, or a dedicated ID allocation table.

---

## 17. Inputs Required from Users Before Acting
- Create module: ModuleName, CaptionEn, optional CaptionAr, Category, IconClass/Icon binary; confirm menu creation.
- Move object: ObjectName (exact), TargetObjectCatID, whether to set WebMenuParent/WebMenu, MenuName/MenuIcon; confirm preview.
- Copy object: SourceName (Object_Name or [OBJECT]), TargetName (display), TargetValue (internal short name), Target module OBJECTCAT_ID, CopySubrMetadata (1/0), CreateTargetViewIfNotExists (1/0), GrantUser; confirm preview.

---

## 18. Site-specific Audit (Example from recent run)
Record operational outputs (OBJECTCAT_ID, WebMenuID, UpdatedRows, GUIDs) for each operation. Example recorded entries:
- Created module `Hisham PPM` → OBJECTCAT_ID = 1392.
- Moved `Banks` (Object_Name = 'Banks', OBJECT = 'BankTable', GUID = cd080d28-71a3-4ffe-acb5-f248cca304ce) from OBJECTCAT_ID = 1391 → 1392. frwAI_MoveObject returned UpdatedRows = 1, NewObjectCatID = 1392, WebMenuID = 418. OBJECTCAT_Sort = 1 for Banks after move.

Keep these mappings in a GUID mapping / change log for rollback and traceability.

---

## 19. How to Operate (Summary Runbook / Checklist)
1. Initialize connection (Section 2).
2. Discovery queries: validate schema and required stored-procs.
3. For module creation:
   - BEGIN TRAN;
   - EXEC dbo.frwAI_CreateModule …;
   - EXEC dbo.frwAI_VerifyCreateModule_JSON(@ModuleName);
   - ROLLBACK (test), then COMMIT when verified.
4. For object copy:
   - Step A: create object/view only (CopySubrMetadata = 0). Verify UI & view. ROLLBACK to test.
   - Step B: copy subordinate metadata (CopySubrMetadata = 1). Verify counts via frwAI_VerifyCopyObject_JSON then COMMIT.
5. For object move:
   - BEGIN TRAN; EXEC dbo.frwAI_MoveObject …; EXEC dbo.frwAI_VerifyMoveObject_JSON …; ROLLBACK to test; COMMIT when verified.
6. Save operation outputs (OBJECTCAT_ID, WebMenuID, UpdatedRows, GUID mappings) to the migration log.
7. Keep rollback templates ready and tested.

---

## 20. Lessons Learned (Operational)
- Do not call mssql_initialize_connection inside a SQL batch; call it separately.
- Always validate column names with INFORMATION_SCHEMA (example: frwObjectCat column name differs).
- Use JSON verify procs immediately after frwAI_* operations to gate steps.
- Wrap frwAI_* calls in preview transactions (ROLLBACK first).
- Preserve GUID mapping during copies and record mappings in audit logs.
- Protect MAX+1 generation with locks or SEQUENCE under concurrency.
