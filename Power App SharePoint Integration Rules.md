# POWER APP SHAREPOINT INTEGRATION RULES

> **CRITICAL**: Comprehensive rules để avoid "invalid string in filter query" và PA2108/PA2109 errors dựa trên actual errors từ UserManagementScreen project.

---

## 📋 MỤC LỤC
1. [SharePoint Lookup Restrictions](#1-sharepoint-lookup-restrictions)
2. [Collection Creation Rules](#2-collection-creation-rules)
3. [OnVisible Performance Rules](#3-onvisible-performance-rules)
4. [Filter Query Restrictions](#4-filter-query-restrictions)
5. [Control Version Restrictions](#5-control-version-restrictions)
6. [Implementation Checklist](#6-implementation-checklist)
7. [Real-World Examples](#7-real-world-examples)

---

## 1. SHAREPOINT LOOKUP RESTRICTIONS - CRITICAL ERROR SOURCE

### Rule 1: NESTED LOOKUP PROHIBITION - CRITICAL
**NEVER** sử dụng nested LookUp() trong AddColumns() với SharePoint data:

```yaml
# ❌ FORBIDDEN - Causes "invalid string in filter query"
ClearCollect(colUsers, 
  AddColumns(User_1, 
    "Role", 
    LookUp(Role_1, roleID = LookUp(User_Role, userID = ThisRecord.userID).roleID).name
  )
)

# ✅ CORRECT - Step-by-step approach
ClearCollect(colUsers, User_1);
ClearCollect(colUsers,
  AddColumns(colUsers,
    "Role",
    LookUp(Role_1, 
      roleID = LookUp(User_Role, userID = ThisRecord.userID).roleID
    ).name
  )
)
```

**WHY**: SharePoint query engine cannot parse complex nested lookup expressions và generates "invalid string in filter query" error.

### Rule 2: WITH() STATEMENT LIMITATIONS - SharePoint Context
**NEVER** sử dụng nested With() trong SharePoint AddColumns():

```yaml
# ❌ FORBIDDEN - SharePoint parsing error
AddColumns(User_1,
  "Department",
  With({dept: LookUp(Department_1, departmentID = ThisRecord.departmentID)}, 
    If(IsBlank(dept), "Unknown", dept.name)
  )
)

# ✅ CORRECT - Direct lookup with null checking
AddColumns(User_1,
  "Department",
  If(
    Not(IsBlank(LookUp(Department_1, departmentID = ThisRecord.departmentID))),
    LookUp(Department_1, departmentID = ThisRecord.departmentID).name,
    "Unknown"
  )
)
```

**WHY**: With() statements create additional context scoping that SharePoint cannot resolve properly trong complex expressions.

### Rule 3: COMPLEX SHAREPOINT OPERATIONS - Mandatory Separation
**ALWAYS** separate complex SharePoint operations into multiple steps:

```yaml
# ❌ AVOID - Complex single operation
ClearCollect(colUsers, 
  AddColumns(
    AddColumns(User_1, "Department", DeptLookup),
    "Role", RoleLookup
  )
)

# ✅ REQUIRED - Step-by-step approach
ClearCollect(colUsers, User_1);
ClearCollect(colUsers, AddColumns(colUsers, "Department", DeptLookup));
ClearCollect(colUsers, AddColumns(colUsers, "Role", RoleLookup));
```

**WHY**: SharePoint has query complexity limits. Breaking operations into steps allows individual validation và better error handling.

---

## 2. COLLECTION CREATION RULES - ERROR PREVENTION

### Rule 4: CONCAT() FUNCTION RESTRICTIONS - Critical Fix
**NEVER** sử dụng Concat() để combine tables:

```yaml
# ❌ FORBIDDEN - Concat() is for text, not tables
ClearCollect(colFilter,
  Concat(
    Table({name: "Tất cả"}),
    AddColumns(Source_1, "name", name)
  )
)

# ✅ CORRECT - Use separate operations
ClearCollect(colFilter, {name: "Tất cả"});
Collect(colFilter, ForAll(Source_1, {name: name}));

# ✅ ALTERNATIVE - Multiple records in ClearCollect
ClearCollect(colFilter,
  Table({name: "Tất cả"}),
  ForAll(Source_1, {name: name})
)
```

**WHY**: Concat() is designed for text concatenation. Using it with tables causes type mismatch errors.

### Rule 5: TABLE() + FORALL() SCHEMA COMPATIBILITY
**ALWAYS** ensure schema compatibility when combining Table() và ForAll():

```yaml
# ✅ CORRECT - Consistent schema
ClearCollect(colFilter,
  Table({name: "Tất cả"}),      # Schema: {name: Text}
  ForAll(Source_1, {name: name}) # Schema: {name: Text}
)

# ❌ AVOID - Schema mismatch risk
ClearCollect(colFilter,
  {name: "Tất cả"},              # Direct record
  ForAll(Source_1, {name: name}) # Collection of records
)
```

**WHY**: Schema mismatches cause runtime errors when Power Apps tries to combine different data structures.

---

## 3. ONVISIBLE PERFORMANCE RULES - MANDATORY

### Rule 6: SHAREPOINT DATA LOADING PATTERN - Required Sequence
**ALWAYS** follow this exact sequence trong OnVisible:

```yaml
# ✅ MANDATORY SEQUENCE:
OnVisible: |
  =// STEP 1: Load base SharePoint tables
  ClearCollect(colDepartments, Department_1);
  ClearCollect(colRoles, Role_1);
  
  // STEP 2: Create filter collections (simple)
  ClearCollect(colDepartmentFilter, {name: "Tất cả"});
  Collect(colDepartmentFilter, ForAll(Department_1, {name: name}));
  
  // STEP 3: Load base user data ONLY
  ClearCollect(colUsers, User_1);
  
  // STEP 4: Enhance with lookups SEPARATELY
  ClearCollect(colUsers, AddColumns(colUsers, "Department", LookupLogic));
  ClearCollect(colUsers, AddColumns(colUsers, "Role", LookupLogic));
  
  // STEP 5: Reset UI state variables
  Set(varShowUserForm, false);
  // ... other state resets
```

**WHY**: This sequence prevents data dependency issues và ensures each step can be validated independently.

### Rule 7: ERROR RECOVERY PATTERN - Mandatory Implementation
**ALWAYS** implement error recovery cho SharePoint operations:

```yaml
# ✅ REQUIRED - Error-safe SharePoint operations
IfError(
  ClearCollect(colUsers, 
    AddColumns(User_1, "Department", DepartmentLookup)
  ),
  // Fallback: Keep basic user data
  ClearCollect(colUsers, User_1);
  Notify("Could not load department info", NotificationType.Warning)
)
```

**WHY**: SharePoint operations can fail due to network issues, permissions, or data corruption. Fallback ensures app continues functioning.

---

## 4. FILTER QUERY RESTRICTIONS - CRITICAL

### Rule 8: FIELD REFERENCE IN FILTERS - String Literal Prohibition
**NEVER** sử dụng string literals trong filter field references:

```yaml
# ❌ FORBIDDEN - Causes filter parsing error
Items: =Filter(colUsers, 
  Or(varSearchText in "FullName", varSearchText in "Email")
)

# ✅ REQUIRED - Use actual field names
Items: =Filter(colUsers, 
  Or(varSearchText in fullname, varSearchText in email)
)

# ✅ REQUIRED - For added columns
Items: =Filter(colUsers, 
  Or(varSearchText in fullname, varSearchText in Department)
)
```

**WHY**: String literals trong filter expressions are interpreted as literal text, not field references, causing filter parsing failures.

### Rule 9: THISRECORD CONTEXT IN FILTERS - SharePoint Specific
**ALWAYS** sử dụng ThisRecord trong Filter() context với SharePoint:

```yaml
# ✅ CORRECT - Filter với SharePoint lookups
Items: =Filter(User_1, 
  And(
    varSearchText in fullname,
    LookUp(Department_1, departmentID = ThisRecord.departmentID).name = varSelectedDept
  )
)

# ❌ WRONG - Disambiguation operator trong Filter context
Items: =Filter(User_1,
  LookUp(Department_1, departmentID = User_1[@departmentID]).name = varSelectedDept
)
```

**WHY**: Filter() creates a ThisRecord context for each row. Using disambiguation operators breaks this context và causes reference errors.

---

## 5. CONTROL VERSION RESTRICTIONS - CRITICAL ERRORS

### Rule 10: VERSION NUMBER PROHIBITION - PA2108/PA2109 Prevention
**NEVER** include version numbers trong ANY control declarations:

```yaml
# ❌ FORBIDDEN - Causes PA2108 errors
Control: DataTable@1.0.3
Control: DataTableColumn@1.0.1  
Control: GroupContainer@1.3.0
Control: Classic/Button@2.2.0

# ✅ REQUIRED - No version numbers
Control: DataTable
Control: DataTableColumn
Control: GroupContainer  
Control: Classic/Button
```

**WHY**: Power Apps YAML parser does not support version specifications và treats them as invalid control types.

---

## 6. IMPLEMENTATION CHECKLIST - MANDATORY

### Pre-Development Checks:
- [ ] SharePoint lookup complexity assessment completed
- [ ] Collection creation strategy planned với error recovery
- [ ] Filter query field mappings verified against actual SharePoint columns
- [ ] Control declarations cleaned of version numbers
- [ ] OnVisible sequence planned với step-by-step approach

### Development Rules:
- [ ] No nested LookUp() operations trong AddColumns() (Rule 1)
- [ ] No nested With() statements trong SharePoint contexts (Rule 2)  
- [ ] Complex operations separated into multiple steps (Rule 3)
- [ ] Proper Concat() vs Concatenate() usage (Rule 4)
- [ ] Schema compatibility verified for collections (Rule 5)
- [ ] OnVisible follows mandatory sequence pattern (Rule 6)
- [ ] Error recovery implemented for all SharePoint operations (Rule 7)
- [ ] Field references vs string literals trong filters verified (Rule 8)
- [ ] ThisRecord context used correctly trong Filter operations (Rule 9)
- [ ] All control declarations version-free (Rule 10)

### Testing Requirements:
- [ ] OnVisible loads without "invalid string in filter query" errors
- [ ] Filter operations work with actual SharePoint data và real field names
- [ ] Collections populate correctly with proper schemas
- [ ] Error recovery triggers appropriately with meaningful messages
- [ ] Performance acceptable với large datasets (>100 records)
- [ ] No PA2108/PA2109 errors trong Power Apps studio

---

## 7. REAL-WORLD EXAMPLES - USERMANAGEMENTSCREEN

### Example 1: Proper SharePoint Data Loading
**FROM UserManagementScreen PROJECT:**

```yaml
# ✅ CORRECT - Working pattern từ actual project
OnVisible: |
  =// Load base SharePoint tables first
  ClearCollect(colDepartments, Department_1);
  ClearCollect(colRoles, Role_1);
  ClearCollect(colUserRoles, User_Role);
  
  // Create filter options separately
  ClearCollect(colDepartmentFilter, {name: "Tất cả"});
  Collect(colDepartmentFilter, ForAll(Department_1, {name: name}));
  
  ClearCollect(colRoleFilter, {name: "Tất cả"});
  Collect(colRoleFilter, ForAll(Role_1, {name: name}));
  
  // Load base user data
  ClearCollect(colUsers, User_1);
  
  // Add department info step-by-step
  ClearCollect(colUsers,
    AddColumns(colUsers,
      "Department",
      LookUp(Department_1, departmentID = ThisRecord.departmentID).name
    )
  );
  
  // Add role info separately
  ClearCollect(colUsers,
    AddColumns(colUsers,
      "Role", 
      LookUp(Role_1, 
        roleID = LookUp(User_Role, userID = ThisRecord.userID).roleID
      ).name
    )
  );
  
  // Reset UI state
  Set(varShowUserForm, false);
  Set(varShowDetailForm, false);
  Set(varFormMode, "Add");
  Set(varSelectedUser, Blank())
```

### Example 2: Working Filter Pattern
**FROM UserManagementScreen PROJECT:**

```yaml
# ✅ CORRECT - Filter pattern that works
Items: |
  =Filter(colUsers,
    And(
      // Text search trong actual field names (not string literals)
      Or(
        IsBlank(varSearchText),
        varSearchText in fullname,
        varSearchText in email,
        varSearchText in Department,
        varSearchText in Role
      ),
      // Department filter
      Or(
        IsBlank(varSelectedDepartment),
        varSelectedDepartment = "Tất cả",
        Department = varSelectedDepartment
      ),
      // Role filter
      Or(
        IsBlank(varSelectedRole),
        varSelectedRole = "Tất cả", 
        Role = varSelectedRole
      )
    )
  )
```

### Example 3: Error Recovery Implementation
**FROM UserManagementScreen PROJECT:**

```yaml
# ✅ CORRECT - Error recovery for form submission
OnSelect: |
  =Set(varErrorMessage, "");
  IfError(
    With({
      newUserID: Concatenate("USER_", Text(Rand() * 1000000, "000000"))
    },
      // Create user record
      Patch(User_1, Defaults(User_1), {
        userID: newUserID,
        fullname: 'FullName.Input'.Text,
        email: 'Email.Input'.Text,
        departmentID: 'Department.Dropdown'.Selected.departmentID,
        jobTitle: 'JobTitle.Input'.Text,
        phone: 'Phone.Input'.Text
      });
      
      // Create user role mapping
      Patch(User_Role, Defaults(User_Role), {
        userID: newUserID,
        roleID: 'Role.Dropdown'.Selected.roleID
      })
    ),
    
    // Error handling
    Set(varErrorMessage, FirstError.Message);
    Notify("Có lỗi: " & FirstError.Message, NotificationType.Error),
    
    // Success handling
    Set(varShowUserForm, false);
    Notify("Đã tạo người dùng thành công!", NotificationType.Success);
    
    // Refresh data với error recovery
    IfError(
      ClearCollect(colUsers, User_1);
      ClearCollect(colUsers, AddColumns(colUsers, "Department", DeptLookup));
      ClearCollect(colUsers, AddColumns(colUsers, "Role", RoleLookup)),
      
      // Fallback if refresh fails
      ClearCollect(colUsers, User_1);
      Notify("Data refreshed with basic info only", NotificationType.Warning)
    )
  )
```

---

## 🚨 CRITICAL REMINDER

**These rules are based on ACTUAL errors encountered trong production SharePoint environments. Non-compliance WILL result in:**

1. **"invalid string in filter query"** errors (Rules 1-3, 8-9)
2. **PA2108 errors** (Rule 10) 
3. **PA2109 errors** (Rule 10)
4. **Type mismatch errors** (Rules 4-5)
5. **Performance degradation** (Rules 6-7)

**SUCCESS METRICS:**
- ✅ Zero "invalid string in filter query" errors
- ✅ Zero PA2108/PA2109 parsing errors  
- ✅ Proper error recovery với meaningful user messages
- ✅ Fast OnVisible loading (< 3 seconds for 100+ records)
- ✅ Reliable filter operations với actual SharePoint data

**ENFORCEMENT:** These rules MUST be followed trong all SharePoint-connected Power App projects. No exceptions.