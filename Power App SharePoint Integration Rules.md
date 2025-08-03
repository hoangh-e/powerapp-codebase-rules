# POWER APP SHAREPOINT INTEGRATION RULES

> **CRITICAL**: Comprehensive rules để avoid "invalid string in filter query" và PA2108/PA2109 errors dựa trên actual errors từ UserManagementScreen project.

---

## 📋 MỤC LỤC
1. [SharePoint Lookup Restrictions](#1-sharepoint-lookup-restrictions)
2. [Collection Creation Rules](#2-collection-creation-rules)
3. [OnVisible Performance Rules](#3-onvisible-performance-rules)
4. [Filter Query Restrictions](#4-filter-query-restrictions)
5. [Delegation Handling Rules](#5-delegation-handling-rules)
6. [Control Version Restrictions](#6-control-version-restrictions)
7. [Implementation Checklist](#7-implementation-checklist)
8. [Real-World Examples](#8-real-world-examples)

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

## 5. DELEGATION HANDLING RULES - LARGE DATASET OPTIMIZATION

### Rule 11: AVOID SEARCH() FUNCTION FOR LARGE DATASETS - Delegation Issue
**NEVER** sử dụng Search() function trên large datasets vì nó không support delegation:

```yaml
# ❌ FORBIDDEN - Search() không support delegation
Items: =Filter(SharePointList, 
  Search(varSearchText, Title, Description, Comments)
)

# ✅ CORRECT - Use StartsWith() for delegable search
Items: =Filter(SharePointList,
  Or(
    StartsWith(Title, varSearchText),
    StartsWith(Description, varSearchText),
    StartsWith(Comments, varSearchText)
  )
)

# ✅ ALTERNATIVE - Use "in" operator for delegation
Items: =Filter(SharePointList,
  Or(
    varSearchText in Title,
    varSearchText in Description,
    varSearchText in Comments
  )
)
```

**WHY**: Search() function executes client-side và chỉ process 500 records đầu tiên. StartsWith() và "in" operator support delegation và can process unlimited records on server-side.

### Rule 12: PRE-FILTER LARGE DATASETS - Delegation Prevention
**ALWAYS** lọc dữ liệu trước khi cho phép tìm kiếm để thu gọn tập dữ liệu:

```yaml
# ✅ CORRECT - Pre-filter with delegation-friendly conditions
Items: =Filter(SharePointList,
  And(
    // Pre-filter: Recent records only (delegable)
    'Created Date' >= DateAdd(Today(), -30, Days),
    
    // Pre-filter: Active status only (delegable) 
    Status = "Active",
    
    // Then apply search (on smaller dataset)
    Or(
      IsBlank(varSearchText),
      varSearchText in Title,
      varSearchText in Description
    )
  )
)

# ✅ ALTERNATIVE - Use collections for complex search
OnVisible: |
  =// Load filtered data into collection first
  ClearCollect(colFilteredData,
    Filter(SharePointList,
      And(
        'Created Date' >= DateAdd(Today(), -30, Days),
        Status = "Active"
      )
    )
  );

# Then search in collection (no delegation issues)
Items: =Filter(colFilteredData,
  Or(
    IsBlank(varSearchText),
    Search(varSearchText, Title, Description, Comments) <> ""
  )
)
```

**WHY**: Pre-filtering reduces dataset size below delegation warning threshold (typically 500-2000 records), allowing complex search operations to work reliably.

### Rule 13: DELEGATION-FRIENDLY SEARCH PATTERNS - Best Practices
**ALWAYS** design search patterns that work with SharePoint delegation:

```yaml
# ✅ CORRECT - Multi-field delegable search
Items: =Filter(SharePointList,
  And(
    // Category filter (delegable lookup)
    Or(
      IsBlank(varSelectedCategory),
      varSelectedCategory = "All",
      Category.Value = varSelectedCategory
    ),
    
    // Date range filter (delegable)
    'Created Date' >= varStartDate,
    'Created Date' <= varEndDate,
    
    // Text search (delegable with StartsWith or "in")
    Or(
      IsBlank(varSearchText),
      StartsWith(Title, varSearchText),
      StartsWith(CustomerName, varSearchText)
    )
  )
)

# ✅ ALTERNATIVE - Progressive filtering approach
OnTextChanged (TextInput): |
  =If(
    Len(Self.Text) >= 3,  // Start search after 3 characters
    ClearCollect(colSearchResults,
      Filter(SharePointList,
        And(
          StartsWith(Title, Self.Text),
          'Created Date' >= DateAdd(Today(), -90, Days)  // Recent only
        )
      )
    ),
    ClearCollect(colSearchResults, Blank())  // Clear if less than 3 chars
  )
```

**WHY**: Progressive filtering và minimum character requirements reduce server load và ensure responsive user experience with large datasets.

### Rule 14: DELEGATION WARNING MONITORING - Critical Performance
**ALWAYS** monitor và address delegation warnings:

```yaml
# ✅ REQUIRED - Check for delegation warnings in Power Apps Studio
# Look for blue "delegation warning" indicators in formula bar

# ✅ REQUIRED - Use Top() function for large result sets
Items: =Top(
  Filter(SharePointList,
    StartsWith(Title, varSearchText)
  ),
  100  // Limit results for performance
)

# ✅ REQUIRED - Implement "Load More" pattern for pagination
Items: =Top(
  Filter(SharePointList,
    And(
      StartsWith(Title, varSearchText),
      ID > varLastLoadedID  // Pagination key
    )
  ),
  varPageSize
)
```

**WHY**: Delegation warnings indicate potential performance issues với large datasets. Addressing them ensures consistent app performance regardless of data size.

---

## 6. CONTROL VERSION RESTRICTIONS - CRITICAL ERRORS

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
- [ ] Delegation analysis completed for large datasets (>500 records)
- [ ] Search strategy planned with delegation-friendly functions
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
- [ ] Search() function avoided for large datasets - use StartsWith() or "in" (Rule 11)
- [ ] Pre-filtering implemented for datasets >500 records (Rule 12)
- [ ] Delegation-friendly search patterns implemented (Rule 13)
- [ ] Delegation warnings monitored and addressed (Rule 14)

### Testing Requirements:
- [ ] OnVisible loads without "invalid string in filter query" errors
- [ ] Filter operations work with actual SharePoint data và real field names
- [ ] Collections populate correctly with proper schemas
- [ ] Error recovery triggers appropriately with meaningful messages
- [ ] Performance acceptable với large datasets (>100 records)
- [ ] No delegation warnings in Power Apps studio for large datasets
- [ ] Search operations work efficiently with >500 records
- [ ] Pre-filtering reduces dataset size appropriately
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

### Example 4: Delegation-Friendly Search Implementation
**PROPER DELEGATION HANDLING PATTERN:**

```yaml
# ✅ CORRECT - Delegation-friendly search for large SharePoint lists
# Text Input - Search Box
OnChange: |
  =If(
    Len(Self.Text) >= 3,  // Only start search after 3 characters
    Set(varSearchText, Self.Text),
    Set(varSearchText, "")
  )

# Gallery Items - Using delegation-friendly functions
Items: |
  =If(
    IsBlank(varSearchText),
    // No search - show recent records only (pre-filtered)
    Top(
      Filter(SharePointList,
        And(
          'Created Date' >= DateAdd(Today(), -30, Days),
          Status = "Active"
        )
      ),
      100
    ),
    
    // Search mode - delegation-friendly pattern
    Top(
      Filter(SharePointList,
        And(
          // Pre-filter for better performance
          'Created Date' >= DateAdd(Today(), -90, Days),
          Status = "Active",
          
          // Delegation-friendly search (avoid Search() function)
          Or(
            StartsWith(Title, varSearchText),
            StartsWith(CustomerName, varSearchText),
            varSearchText in Description
          )
        )
      ),
      50  // Limit results for performance
    )
  )

# Alternative Pattern - Collection-based for complex searches
OnVisible: |
  =// Load recent active data into collection (delegation-friendly)
  ClearCollect(colRecentData,
    Filter(SharePointList,
      And(
        'Created Date' >= DateAdd(Today(), -60, Days),
        Status = "Active"
      )
    )
  );

# Then use collection for complex search (no delegation issues)
Items: |
  =If(
    IsBlank(varSearchText),
    colRecentData,
    Filter(colRecentData,
      Or(
        Search(varSearchText, Title, Description, CustomerName) <> "",
        StartsWith(Title, varSearchText)
      )
    )
  )
```

### Example 5: Progressive Loading Pattern for Large Datasets
**LOAD MORE IMPLEMENTATION:**

```yaml
# Button - Load More Records
OnSelect: |
  =Collect(colDisplayData,
    Top(
      Filter(SharePointList,
        And(
          ID > Last(colDisplayData).ID,  // Pagination key
          Status = "Active",
          Or(
            IsBlank(varSearchText),
            StartsWith(Title, varSearchText)
          )
        )
      ),
      20  // Load 20 more records
    )
  );
  Set(varCanLoadMore, CountRows(
    Top(Filter(SharePointList, ID > Last(colDisplayData).ID), 1)
  ) > 0)

# Initial Load
OnVisible: |
  =ClearCollect(colDisplayData,
    Top(
      Filter(SharePointList,
        And(
          Status = "Active",
          'Created Date' >= DateAdd(Today(), -30, Days)
        )
      ),
      20  // Initial load - 20 records
    )
  );
  Set(varCanLoadMore, CountRows(SharePointList) > 20)
```

---

## 🚨 CRITICAL REMINDER

**These rules are based on ACTUAL errors encountered trong production SharePoint environments. Non-compliance WILL result in:**

1. **"invalid string in filter query"** errors (Rules 1-3, 8-9)
2. **PA2108 errors** (Rule 10) 
3. **PA2109 errors** (Rule 10)
4. **Type mismatch errors** (Rules 4-5)
5. **Performance degradation** (Rules 6-7)
6. **Delegation warnings và poor performance** với large datasets (Rules 11-14)
7. **Timeout errors** when Search() function used on >500 records
8. **Inconsistent results** due to delegation limits

**SUCCESS METRICS:**
- ✅ Zero "invalid string in filter query" errors
- ✅ Zero PA2108/PA2109 parsing errors  
- ✅ Proper error recovery với meaningful user messages
- ✅ Fast OnVisible loading (< 3 seconds for 100+ records)
- ✅ Reliable filter operations với actual SharePoint data
- ✅ Zero delegation warnings cho large datasets (>500 records)
- ✅ Consistent search performance regardless of dataset size
- ✅ Efficient search operations using StartsWith() và "in" operators

**ENFORCEMENT:** These rules MUST be followed trong all SharePoint-connected Power App projects. No exceptions.