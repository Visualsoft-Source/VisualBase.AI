AI Framework Migration Playbook
1. Create a New Module
Procedure: frwAI_CreateModule
Parameters:
@ModuleName = Desired module code (e.g., PP4)
@CaptionEn = English caption
@CaptionAr = Arabic caption
Example:

EXEC frwAI_CreateModule @ModuleName='PP4', @CaptionEn='PP4 Module', @CaptionAr='الوحدة PP4';
Expected Output: OBJECTCAT_ID and Status = Created.
2. Move or Copy Objects
Move Object: frwAI_MoveObject

EXEC frwAI_MoveObject @ObjectName='Chart of accounts', @TargetObjectCatID=1388;
Copy Object: frwAI_CopyObject

EXEC frwAI_CopyObject @ObjectName='Chart of accounts', @TargetObjectCatID=1388;
Expected Output: UpdatedRows, NewObjectCatID, WebMenuID.
3. Verify Operations
Verify Module Creation: frwAI_VerifyCreateModule_JSON
Verify Object Move: frwAI_VerifyMoveObject_JSON

EXEC frwAI_VerifyMoveObject_JSON @ObjectName='Chart of accounts', @ExpectedTargetObjectCatID=1388;
Check: IsInExpectedModule = 1 (True).
4. Manage WebMenu and Metadata
Ensure WebMenuParent and WebMenu IDs are correctly assigned.
Validate DefinitionsCount, ReportsCount, and PermissionsCount.
5. Optional: Find Best Match for Object Names
Procedure: frwAI_FindBestObjectMatch_JSON
Useful for fuzzy matching when object names differ slightly.
6. Document and Log
Record OBJECTCAT_ID, WebMenuID, and verification results.
Maintain a migration log for audit and rollback purposes.
