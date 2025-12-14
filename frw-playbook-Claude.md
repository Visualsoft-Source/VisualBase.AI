Final — Complete, comprehensive playbook for VisualBase (frw*) metadata
Ready to copy‑paste

Source: canonical frw-playbook.md (merged + required additions: migration logging, post-commit verify, error handling, frwAI_Documentation rules, integration impact checks)

========================================================================
ONE-LINE PURPOSE
Canonical internal playbook for safe discovery and operations on VisualBase (InnovBase) frw* metadata (modules, objects, fields, menus, reports, workflows, approvals, etc.) — always discover first, preview in transaction, verify with JSON procs, keep rollback script ready.

QUICK PRE-FLIGHT CHECKLIST (copy-paste)
1. Initialize connection (client/tool) — run as a separate client call (do NOT embed):
   EXEC mssql_initialize_connection('DefaultConnection');
2. Run discovery SELECTs (INFORMATION_SCHEMA + sys views) for all target tables/columns.
3. Resolve ambiguous names:
   EXEC dbo.frwAI_FindBestObjectMatch_JSON(@UserInput,@Top) — require explicit user selection if not exact.
4. Prepare DML inside:
   BEGIN TRAN;
     -- Exec frwAI_* or run reviewed DML
     -- Exec Verify JSON procs (frwAI_Verify*JSON)
   ROLLBACK;  -- test
   -- After verification + human approval: re-run and COMMIT;
5. Save OBJECTCAT_ID / WebMenuID / GUID mappings to migration log (see schema below).
6. Keep the rollback script ready and peer-reviewed.

========================================================================
1. MANDATORY INITIALIZATION (always separate client call)
Do NOT embed inside a T‑SQL batch. Run once per session:
```sql
EXEC mssql_initialize_connection('DefaultConnection');
```
Rationale: embedding caused syntax errors historically.

========================================================================
2. SAFETY & TRANSACTIONAL RULES (must follow)
- Discover first (read-only). Confirm table/column names via INFORMATION_SCHEMA/sys views.
- Wrap all metadata changes in transactions. Test with ROLLBACK before COMMIT.
- Use NEWID() for GUIDs.
- Do not assume numeric columns are IDENTITY. Compute IDs via ISNULL(MAX(col),0)+1 and protect with sp_getapplock or use SQL SEQUENCE under concurrency.
- Keep a reviewed rollback script available before COMMIT.
- Prefer two‑step copy: Step A = create object/view only; Step B = copy subordinate metadata.
- Verify existence of helper procs (frwAI_CreateModule, frwAI_CopyObject, frwAI_MoveObject, frwAI_Verify*). If missing, use manual, reviewed SQL flows.

========================================================================
3. CORE DISCOVERY QUERIES (run BEFORE any DML)
Always confirm via INFORMATION_SCHEMA and sys.* queries.

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
Rationale: sites differ in column names (example: frwObjectCat.Sort vs OBJECTCAT_Sort). Always confirm.

========================================================================
4. HOW TO INSPECT STORED PROCEDURES (always run before calling)
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

========================================================================
5. FRW SCHEMA REFERENCE (key tables — always validate columns per site)
Core tables to confirm via INFORMATION_SCHEMA:
- frwObjectCat (modules): OBJECTCAT_ID, OBJECTCAT_NAME, Description, Sort, GUID, Icon, IconClass, Category, OBJECTCAT_Parent, OBJECTCAT_NAMEAR, Dashboard, DescriptionAR.
- frwObjects: Object_Name (display PK commonly), OBJECT (short name), OBJECTCAT_ID, OBJECTCAT_Sort, GUID, Web, WebMenuParent, WebMenu, SourceType, OriginalTable, many event/proc fields, security flags.
- frwDefinitions: Field, Control, LinkTable, LinkField, DisplayField, TabName, TabIndex, Enabled, Invisible, GUID, Formula, Reminder_Report (GUID), Default, Caption, [Object].
- frwReports: Report_FileName, Report_Description, Report_ObjectName, GUID, Report_Web, Report_ShowSelection.
- frwPermissions: PERMISSION_USERNAME, PERMISSION_OBJECT, PERMISSION_ACCESS, GUID, select/insert/update/delete bits.
- frwWebMenu, frwWebMenuParent: frwWebMenu.ID (may not be identity), Menu, Sort, Icon, GUID; frwWebMenuParent keyed by OBJECTCAT_ID.
- frwTabs, frwFields, frwValidations, frwRestrictions, frwExceptions, frwTutorials, frwObjects_Collection.

========================================================================
6. WORKFLOW & PROCEDURE TABLES (discovery & use)
- frwProcedures, frwProcedure_Objects, frwProcedure_ObjectsIn, frwProcedure_ObjectsOut, frwProcedure_ProcessCategories, frwProcedureApprovals, frwApproval_Type (0=Not Required,1=Ad hoc,2=All must,3=Majority,4=Percentage), frwObject_Approvals, frwWorkflow, frwWorkflow_Approvals.
Note: frwValidations may use ObjectID (not Object_Name) — confirm before queries.

========================================================================
7. APPROVAL FORMULA PATTERNS (common)
- [FieldName] — dynamic approver from a record field.
- [CreatedUser] — record creator.
- @Skip — skip sequence.
- IIF(condition, value, @Skip)
Document formulas from frwObject_Approvals and frwWorkflow_Approvals and test in non‑prod.

========================================================================
8. frwAI PROCEDURES — USAGE, PARAMETERS, SAFE TEMPLATES (MANDATORY)
General safety wrapper (must use):
1. Initialize connection (separate client call).
2. BEGIN TRAN;
3. EXEC dbo.frwAI_* (with parameters)
4. EXEC dbo.frwAI_Verify*JSON(...) — inspect JSON
5. ROLLBACK; (test)
6. Repeat until verify JSON is acceptable; then COMMIT after human approval.
Use JSON verify procs as machine gates; save outputs.

8.1 frwAI_CreateModule (example)
```sql
BEGIN TRAN;
EXEC dbo.frwAI_CreateModule
  @ModuleName = N'PP4',
  @CaptionEn   = N'PP4 Module',
  @CaptionAr   = N'الوحدة PP4';
EXEC dbo.frwAI_VerifyCreateModule_JSON @ModuleName = N'PP4';
ROLLBACK;
```
Expected output: OBJECTCAT_ID, Status. Record OBJECTCAT_ID in migration log.

8.2 frwAI_CopyObject (two‑step recommended — REQUIRED)
Step A — object/view only (CopySubrMetadata = 0):
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
EXEC dbo.frwAI_VerifyCopyObject_JSON @TargetValue = N'PPM_LedgerTable', @TargetName = N'PPM - Chart of Accounts', @SourceName = N'Chart of Accounts';
ROLLBACK;
```
Step B — copy subordinate metadata (CopySubrMetadata = 1) — run only after Step A verification:
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
EXEC dbo.frwAI_VerifyCopyObject_JSON @TargetValue = N'PPM_LedgerTable', @TargetName = N'PPM - Chart of Accounts', @SourceName = N'Chart of Accounts';
COMMIT;
```
Field remap rules:
- Generate new GUIDs via NEWID().
- frwDefinitions.[Object] = TargetValue.
- Field values: if contains dot (prefix.field) replace prefix with TargetValue; if no dot, set Field = TargetValue + '.' + Field.
- Reminder_Report remapped via OldGUID->NewGUID when reports copied; otherwise set NULL.
- Copy subordinate tables: frwReports, frwDefinitions, frwPermissions, frwValidations, frwRestrictions, frwExceptions, frwTutorials, frwObjects_Collection.
- If CreateTargetViewIfNotExists = 1, create the view dbo.TargetValue.

8.3 frwAI_MoveObject (example)
Pre-checks: ensure target OBJECTCAT_ID exists, frwWebMenuParent exists or create, compute WebMenu ID safely (ISNULL(MAX(ID),0)+1 with applock/sequence).
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

========================================================================
9. VERIFICATION PROCEDURES & USE (MACHINE GATES)
- dbo.frwAI_VerifyCreateModule_JSON(@ModuleName) — returns ModuleName, OBJECTCAT_ID, ModuleExists, WebMenuParentCount, WebMenuParentRows, WebMenuRows.
- dbo.frwAI_VerifyMoveObject_JSON(@ObjectName, @ExpectedTargetObjectCatID) — returns counts and WebMenu info; check IsInExpectedModule = 1.
- dbo.frwAI_VerifyCopyObject_JSON(@TargetValue, @TargetName, @SourceName) — returns per-subsystem counts and sample rows.
- dbo.frwAI_FindBestObjectMatch_JSON(@UserInput,@Top) — fuzzy helper to avoid ambiguous operations.

Use these JSON returns as machine‑readable gates before COMMIT. Save all JSON outputs to migration log.

========================================================================
10. VERIFICATION QUERIES (examples — run after operations)
- Verify module created:
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
Compare results to verify JSON.

========================================================================
11. ROLLBACK & CLEANUP TEMPLATES (customize before running)
Undo a copy (example PPM_LedgerTable):
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
Always tailor rollback scripts to recorded GUID/ID mappings and test in non‑prod.

========================================================================
12. TROUBLESHOOTING FLOW
- frwAI_CopyObject fails:
  - Re-run Step A (CopySubrMetadata = 0) to isolate view/object creation issues.
  - Inspect frwAI_VerifyCopyObject_JSON for subsystem counts and sample rows.
  - Ensure frwWebMenuParent / frwWebMenu exist; create if needed.
  - Verify executing account privileges; use @InteractiveGrant = 1 if necessary.
  - Use sp_getapplock or SEQUENCE for ID generation concurrency.

- Missing/modified stored-procs: inspect sys.sql_modules and adapt manual SQL.

========================================================================
13. CONCURRENCY & ID GENERATION GUIDANCE
- Low-concurrency admin tasks: ISNULL(MAX(col),0)+1 acceptable.
- High-concurrency: use sp_getapplock or SQL SEQUENCE or dedicated ID allocation table. Protect WebMenu ID and OBJECTCAT_ID allocations similarly.

========================================================================
14. INPUTS EXPECTED FROM USERS (pre-action)
- Create module: ModuleName, CaptionEn, optional CaptionAr, Category, IconClass/Icon binary, confirm menu creation.
- Move object: ObjectName (exact), TargetObjectCatID, whether to set WebMenuParent/WebMenu, MenuName/MenuIcon.
- Copy object: SourceName (Object_Name or [OBJECT]), TargetName (display), TargetValue (internal short name), Target module OBJECTCAT_ID, CopySubrMetadata (1/0), CreateTargetViewIfNotExists (1/0), GrantUser.
Always require confirmation after discovery SQL and before COMMIT.

========================================================================
15. AUDIT / GUID MAPPING & LOGGING (MANDATORY)
- Save each operation outcome: OBJECTCAT_ID, WebMenuID, UpdatedRows, new GUIDs, and OldGUID→NewGUID mappings to migration log.
- Example migration log row format stored in dbo.frwAI_MigrationLog (required schema shown above). AI MUST insert the migration log row after COMMIT, including VerifyJSON and GUIDMapPath where applicable.

========================================================================
16. frwAI_DOCUMENTATION TABLE (operational note — MUST be used)
frwAI_Documentation stores short docs & runbook snippets. Use it to store site notes and quick references.

Examples:
```sql
-- List docs
SELECT DocName FROM dbo.frwAI_Documentation;

-- Load doc
SELECT DocContent FROM dbo.frwAI_Documentation WHERE DocName = 'AI Agent Self-Reference Guide';

-- Add doc
INSERT INTO dbo.frwAI_Documentation (DocName, DocCategory, DocContent, CreatedBy, Version)
VALUES ('New Doc Name','Category',N'Content here','AI Agent','1.0');
```
Requirement: AI must write a short operational doc entry per major operation summarizing: operation, returned IDs, VerifyJSON path, GUIDMapPath, rollback script path, CreatedBy, Version. Store these entries after COMMIT.

========================================================================
17. POST-COMMIT VERIFICATION (REQUIRED)
- After COMMIT, AI MUST:
  1. Re-run the same frwAI_Verify*JSON used pre-commit and the verification SELECTs.
  2. Compare post-commit VerifyJSON + SELECTs to pre-commit ROLLBACK verify outputs.
  3. Insert post‑commit VerifyJSON into frwAI_MigrationLog.VerifyJSON and mark migration Completed.
- If any mismatch, run prepared rollback immediately and notify approver.

========================================================================
18. ERROR HANDLING & NOTIFICATIONS (REQUIRED)
If Verify JSON shows CRITICAL issues or an error occurs:
1. ROLLBACK (if in transaction).
2. INSERT into dbo.frwAI_ErrorLog (schema below) with details.
3. Notify approver(s) with full VerifyJSON and suggested rollback steps.
4. Require human ACK before retry.

Error log schema:
```sql
CREATE TABLE dbo.frwAI_ErrorLog (
  ErrorID INT IDENTITY(1,1) PRIMARY KEY,
  CreatedAt DATETIME2 DEFAULT SYSUTCDATETIME(),
  CreatedBy NVARCHAR(100),
  Operation NVARCHAR(50),
  ErrorMessage NVARCHAR(MAX),
  VerifyJSON NVARCHAR(MAX),
  StackTrace NVARCHAR(MAX),
  Status NVARCHAR(50) DEFAULT 'OPEN'
);
```
Notification template (AI must use):
- Subject: [frwAI][ERROR] <Operation> on <Object_Name> — <ShortObject>
- Body: Timestamp, Operation, User, ShortObject, Object_Name, ErrorMessage, VerifyJSON (attach), Suggested next steps, Rollback script path.
Require human ACK.

========================================================================
19. IMPACT & INTEGRATION CHECKLIST (SITE-SPECIFIC; REQUIRED)
Before create/copy/move:
- Search for references to OBJECT/OBJECT_NAME in routines, jobs, integrations, APIs:
```sql
SELECT ROUTINE_SCHEMA, ROUTINE_NAME, ROUTINE_DEFINITION
FROM INFORMATION_SCHEMA.ROUTINES
WHERE ROUTINE_DEFINITION LIKE '%YourShortObject%' OR ROUTINE_DEFINITION LIKE '%YourDisplayName%';
```
- Check SQL Agent jobs, SSIS packages, external API consumers. If references exist, notify owners and include them in approval.

========================================================================
20. PERMISSIONS ARCHITECTURE & REFRESH
- Two-table model:
  - frwWebPermissions = definition (what the user SHOULD have).
  - frwPermissions = runtime (what the app reads).
- After changes run:
```sql
DECLARE @g UNIQUEIDENTIFIER;
SELECT @g = [GUID] FROM dbo.frwUsers WHERE USER_NAME = 'VisualBase';
EXEC dbo.frwWeb_InsertPermissions @GUID = @g;
```
- Use Object_Name (display) for permissions (not short table names).

========================================================================
21. FIELD & REPORT REMAP (must run during copy Step B)
- Replace old short object name references across:
  - frwDefinitions.Field and Formula
  - frwReports selection criteria and parameters
  - Stored-proc event code and formulas
- Use OldGUID→NewGUID mapping for Reminder_Report remap when copying reports.

========================================================================
22. ACCEPTANCE TEST (what to require before AI marks operation complete)
1. Pre-discovery SELECTs shown and signed off.
2. frwAI_FindBestObjectMatch_JSON used if name ambiguous — user selected candidate.
3. Two-step copy used for copies.
4. Pre-commit VerifyJSON PASSED (CRITICAL=0) or WARNINGS approved.
5. Post-commit VerifyJSON + SELECTs match pre-commit expectations.
6. Migration log row inserted and rollback script saved.
7. frwAI_Documentation updated with operation summary.

========================================================================
23. DELIVERABLES AI MUST RETURN (after each operation)
- Discovery SELECT outputs, pre‑commit VerifyJSON, post‑commit VerifyJSON, migration log id + contents, rollback script path, frwAI_Documentation entry path, human-readable summary with IDs and GUID map location.

========================================================================
APPENDIX: EXAMPLES & TEMPLATES (included)
All original examples for CreateObject, CreateModule, Two‑step Copy, MoveObject, Verify JSON calls, verification SELECTs, rollback templates, troubleshooting flows remain valid; use them, adapt per site schema, and follow the added mandatory steps above.

========================================================================
CONTACT / SAFETY NOTES
- Do not share credentials/secrets in clear text.
- Backup or snapshot DB before large irreversible changes in production.
- Keep an audit trail of OBJECTCAT_ID / GUID mappings for all copies/moves.

