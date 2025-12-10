# VisualBase / InnovBase frw* Metadata Playbook (Site-specific)

Note: VisualBase is the current product name (rebrand of InnovBase). Use "VisualBase" as the canonical product name in scripts, documentation and grants; include "InnovBase" only in historical notes if needed.

This is the canonical internal playbook for safe operations on frw* metadata in VisualBase (aka InnovBase). It includes mandatory initialization, discovery queries, the frwObjects schema observed in this database, safe patterns for creating modules, moving and copying objects (two-step copy), rewrite rules, verification queries and rollback templates, plus site-specific changes performed.

---

## 1. Purpose & Assumptions
- frw* tables store the application metadata (modules, objects, fields/controls, tabs, reports, menus, permissions, workflows, validations, exceptions, tutorials, collections).
- Always discover (read-only) before DML, preview intended changes with SELECTs, and run DML inside transactions with verification and rollback prepared.
- Use the DefaultConnection via MCP Tools: call `mssql_initialize_connection('DefaultConnection')` before queries.

---

## 2. Mandatory Initialization
Always run:
```
EXEC mssql_initialize_connection('DefaultConnection');
```

---

## 3. Core Discovery Queries (Read-only)
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
- Get column list for a table:
```
SELECT COLUMN_NAME, DATA_TYPE, IS_NULLABLE, COLUMN_DEFAULT, CHARACTER_MAXIMUM_LENGTH
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_NAME = 'YourTableName'
ORDER BY ORDINAL_POSITION;
```
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

## 4. How to Inspect a Stored Procedure (metadata + parameters)
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
- Aggregated signature (single row):
```
SELECT
  SCHEMA_NAME(p.schema_id) AS SchemaName,
  p.name AS ProcedureName,
  m.definition,
  STUFF((
    SELECT ', ' + ISNULL(TYPE_NAME(prm.user_type_id), 'sql_variant') + ' ' + prm.name +
           CASE WHEN prm.is_output = 1 THEN ' OUTPUT' ELSE '' END +
           CASE WHEN prm.has_default_value = 1 THEN ' = ' + ISNULL(CONVERT(nvarchar(4000), prm.default_value), 'NULL') ELSE '' END
    FROM sys.parameters prm
    WHERE prm.object_id = p.object_id
    ORDER BY prm.parameter_id
    FOR XML PATH(''), TYPE).value('.', 'nvarchar(max)'), 1, 2, '') AS ParametersSignature
FROM sys.procedures p
LEFT JOIN sys.sql_modules m ON p.object_id = m.object_id
WHERE SCHEMA_NAME(p.schema_id) = 'dbo'
  AND p.name = 'YourProcName';
```

---

## 5. Key frw Tables & Important Columns
- frwObjectCat (modules):
  - OBJECTCAT_ID (int, may not be identity)
  - OBJECTCAT_NAME (varchar)
  - Description
  - OBJECTCAT_Sort (float)
  - GUID (uniqueidentifier, default NEWID())
  - Icon / IconClass
  - Category (varchar, e.g., 'Core')
- frwObjects (primary metadata)
  - Primary key in this DB: Object_Name (varchar(50))
  - Key columns: Object_Name, OBJECT (short technical name), OBJECTCAT_ID, OBJECTCAT_Sort, GUID, Web, WebMenuParent, WebMenu, SourceType, OriginalTable, many event/procedure fields, security flags.
  - Full column list (observed): Object_Name, Show, Add, Edit, Delete, Print, Approval, LAST_UPDATE, Object_NameAR, OBJECTCAT_ID, OBJECTCAT_Sort, OBJECT, GUID, Created Event, Modified Event, Deleted Event, WizardID, TreeView_FieldID, TreeView_FieldName, TreeView_Formate, QuickInfo, Approval_TypeID, SearchView, Run Audit Trail, Relations, AfterApproval, Description, Spelling Check, Parameters, ParameterObjectID, GridParameters, GridParameterObjectID, DefaultPrint, PrerequisiteObjectID, PrerequisiteParameters, ClassName, TreeView_RightToLeftFieldName, QuickInfoRightToLeft, SearchViewRightToLeft, ReportPrerequisite, HugeData, DrillView, Requester, OriginalTable, AuditSubTable, BeforeDeleteEvent, PasteEvent, Title, SortBy, SortAToZ, DetailEvent, DetailInfo, Assembly, Justification, DrillFirstPage, BeforeModifyEvent, ApprovalApplied, RequestApplied, RejectionApplied, PrerequisiteCancelEvent, Version, Ownership, Notes, AfteRejection, IgnoreIdentifier, At Server Event, Thread Event, SendRequestEvent, DrillView Prerequisite, Apply LegalEntityID, ImportData, Open Event, Created Parameter, Thread Parameter, BeforeModify Parameter, Modified Parameter, BeforeDelete Parameter, Deleted Parameter, Open Parameter, SourceType, UpdateCommand, InsertCommand, Status, Misspelling, SampleData, BBTesting, WBTesting, RTLLanguage, Performance, Web, WebMenu, locks, SecurityGroup1..5, DrilldownEditable, ClientVersion, PreviewField, AfterApprovalWebService, AfteRejectionWebService, OpenEventWebService, SmartPhones, WebServiceName, WebServiceAddress, WebServiceResult, DefaultGroupBy, UseEmail, UseSMS, UseVoice, Overwrite, HugeCalculation, HistorianMethod, HistorianRecursiveLevel, HistorianTimeField, HistorianCondtion, WebMenuParent, AdvancedEditForm, SyncEnable, SyncUseDelete, SyncPriority, SyncTwoWays, SyncClientEvent, SyncServerEvent, SyncStatus, Icon, WebColSpan, EnableSOA, WebServiceFormatXML, WebServiceFormatJSON, WebServiceEncryptionKey, BeforeCreateEvent, BeforeCreate Parameter, Indicator, Print Event.
  - Operational note: Always confirm current columns with INFORMATION_SCHEMA before using them.
- frwDefinitions:
  - Field, Control, LinkTable, LinkField, DisplayField, TabName, TabIndex, Enabled, Invisible, GUID, Formula, Reminder_Report (uniqueidentifier), Default, Caption, [Object]
- frwReports:
  - Report_FileName, Report_Description, Report_ObjectName, GUID, Report_Web, Report_ShowSelection
- frwPermissions:
  - PERMISSION_USERNAME, PERMISSION_OBJECT, PERMISSION_ACCESS, GUID, Select/Insert/Update/Delete bits
- frwWebMenu, frwWebMenuParent:
  - frwWebMenu: ID (int, may not be identity), Menu, Sort, Icon, GUID
  - frwWebMenuParent keyed by OBJECTCAT_ID — needed for module to appear on web menu
- frwTabs, frwFields, frwValidations, frwRestrictions, frwExceptions, frwTutorials, frwObjects_Collection — include these when copying subordinate metadata.

---

## 6. Safety & Transactional Rules
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
- Use `sp_getapplock` or sequences for concurrency if `MAX+1` is used in concurrent environments.
- Avoid `mssql_get_table_metadata` / `mssql_get_database_objects_metadata` per site rule — use INFORMATION_SCHEMA and sys.*.

---

## 7. frwAI_CopyObject: Rewrite & Remap Rules
- Two-step pattern: Step A (create object/view only) then Step B (copy subordinate metadata).
- Subordinate metadata copy:
  - New GUIDs generated with `NEWID()`.
  - `frwDefinitions.[Object]` set to TargetValue (short name).
  - Field rewrite:
    - Dot case: `prefix.field` → `TargetValue.field`
    - No-dot: `field` → `TargetValue.field`
  - `Reminder_Report` remapped via OldGUID->NewGUID table when reports are copied; otherwise set NULL.
  - Copy frwReports, frwDefinitions, frwPermissions, frwValidations, frwRestrictions, frwExceptions, frwTutorials, frwObjects_Collection.
  - If `CreateTargetViewIfNotExists = 1`, a view `dbo.TargetValue` may be created.

---

## 8. Two-step Safe Copy Pattern (Recommended)
- Step A — Create object + view only:
```
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
```
- Verify UI; when satisfied run Step B — full subordinate metadata:
```
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
```

---

## 9. Move Object Best Practices (frwAI_MoveObject)
- Preferred: call `frwAI_MoveObject`.
- Ensure target `OBJECTCAT_ID` exists.
- Ensure `frwWebMenuParent` exists for target `OBJECTCAT_ID`.
- Ensure `frwWebMenu` entry exists for the menu; if missing compute `ID = ISNULL(MAX(ID),0)+1`.
- Compute `OBJECTCAT_Sort = ISNULL(MAX(OBJECTCAT_Sort),0)+1` for destination and update `frwObjects`:
  - `OBJECTCAT_ID`, `OBJECTCAT_Sort`, `WebMenuParent`, `WebMenu`.
- Procedure returns `UpdatedRows`, `NewObjectCatID`, `WebMenuID`.
- Rename `Object_Name` with an UPDATE if you need a new display name.

---

## 10. Web Menu / Parent Rules
- `frwWebMenuParent.OBJECTCAT_ID` = `frwObjectCat.OBJECTCAT_ID`.
- `frwObjects.WebMenuParent` must be the module `OBJECTCAT_ID`.
- `frwObjects.WebMenu` should refer to `frwWebMenu.ID`.
- `frwWebMenu.ID` might not be identity — compute `MAX(ID)+1` for inserts.

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
- Verify definitions count:
```
SELECT COUNT(*) FROM dbo.frwDefinitions WHERE [Object] = 'PPM_LedgerTable';
```
- Verify reports:
```
SELECT COUNT(*) FROM dbo.frwReports WHERE Report_ObjectName = 'PPM - Chart of Accounts';
```
- Verify permissions:
```
SELECT * FROM dbo.frwPermissions WHERE PERMISSION_OBJECT = 'PPM_LedgerTable';
```

---

## 12. JSON verification procedures (NEW — recommended)
Purpose: return one machine‑readable JSON result after create/move/copy operations so automation can gate subsequent steps.

- dbo.frwAI_VerifyCreateModule_JSON(@ModuleName)  
  - Returns JSON: ModuleName, OBJECTCAT_ID, ModuleExists, WebMenuParentCount, WebMenuParentRows, WebMenuRows

- dbo.frwAI_VerifyMoveObject_JSON(@ObjectName, @ExpectedTargetObjectCatID)  
  - Returns JSON: InputName, ShortObject, Found, ObjectName, OBJECTCAT_ID, OBJECTCAT_Sort, WebMenuParent, WebMenu, DefinitionsCount, ReportsCount, PermissionsCount, IsInExpectedModule

- dbo.frwAI_VerifyCopyObject_JSON(@TargetValue, @TargetName, @SourceName)  
  - Returns JSON: TargetValue, TargetName, SourceName, ObjectExists, ViewExists, DefinitionsCount, ReportsCount, PermissionsCount, ValidationsCount, RestrictionsCount, ExceptionsCount, TutorialsCount, ObjectsCollectionCount, DefinitionsSample (array), ReportsSample (array), PermissionsSample (array)

Use the verify procs immediately after frwAI_CopyObject Step A and Step B. Example:
```
EXEC dbo.frwAI_VerifyCopyObject_JSON @TargetValue='PPM_LedgerTable', @TargetName='PPM - Chart of Accounts', @SourceName='Chart of Accounts';
```
Parse the returned JSON and require ObjectExists = 1 and ViewExists = 1 (and expected counts) before proceeding.

---

## 13. Best-match helper (NEW)
- dbo.frwAI_FindBestObjectMatch_JSON(@UserInput,@Top)  
  - Returns a JSON array of best candidate objects for ambiguous names using heuristics (exact match, prefix, contains, SOUNDEX, length proximity). Each item includes Object_Name, ShortObject, OBJECTCAT_ID, OBJECTCAT_NAME, Score.
  - Example usage:
```
EXEC dbo.frwAI_FindBestObjectMatch_JSON @UserInput = N'Direct Sales', @Top = 10;
```
- Use this when the user provides an ambiguous object name; show top results to the operator or let automation pick the top-scored match.

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
  - Re-run Step A (`@CopySubrMetadata = 0`) to isolate view/object creation.
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
- Module `PPM`: `OBJECTCAT_ID = 1389`, GUID `d60dea45-4630-4c2b-b2ab-fc29052370c9`.
- Employees:
  - Source: `Object_Name = 'Employees Full File'`, `OBJECT = 'HrEmpTable'`, `OBJECTCAT_ID = 600`, GUID `9a4d627b-...`
  - Action: Moved to `OBJECTCAT_ID = 1389` and renamed to `PPM - Employees Full File`. `frwAI_MoveObject` returned `UpdatedRows = 1`, `NewObjectCatID = 1389`, `WebMenuID = 415`.
- Chart of Accounts:
  - Source: `Object_Name = 'Chart of Accounts'`, `OBJECT = 'LedgerTable'`, `OBJECTCAT_ID = 1388`, GUID `1c6642dd-...`
  - Action: Copied to PPM as `PPM - Chart of Accounts` with `TargetValue = PPM_LedgerTable` and subordinate metadata copied. `frwAI_CopyObject` returned `Status = OK`. `VisualBase` granted permissions.

---

## 19. How I Will Behave Next Time
- Initialize connection, inspect schema, run preview SELECTs and show results for user confirmation.
- Use `frwAI_*` procedures for standard operations unless user requests custom behavior.
- For custom flows produce commented SQL for review and only run after explicit confirmation.
- Always return created IDs, proc status and verification SELECTs; provide rollback SQL if requested.

---

## 20. Lessons learned (added)
- Ensure CREATE/ALTER is first statement in a batch (use CREATE OR ALTER in its own call).  
- Use JSON verify procs to return single machine‑readable summaries for automation gating.  
- Preserve GUID mapping during copy operations and expose mapping in verify output.  
- Use proper concurrency patterns (sp_getapplock / SEQUENCE) in production.  
- Always wrap DML in transactions and keep rollback templates handy.

---

## 21. Deliverables Available
- Markdown file (this document).
- PDF (if requested).
- Site-specific rollback script for the PPM changes.
- GUID mapping report for Chart of Accounts copy and list of frwDefinitions copied.

---

## 22. Contact / Safety Note
- Do not share clear-text credentials or secrets.
- Backup or snapshot the DB before running large or irreversible changes in production.



