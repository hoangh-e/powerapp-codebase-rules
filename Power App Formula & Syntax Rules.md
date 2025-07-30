# POWER APP FORMULA & SYNTAX RULES

> **CRITICAL**: Quy tắc chi tiết về formula syntax, YAML compliance, và best practices cho Power App development.

---

## 📋 MỤC LỤC
1. [Formula Syntax Rules](#1-formula-syntax-rules)
2. [YAML Compliance](#2-yaml-compliance)
3. [Icon Guidelines](#3-icon-guidelines)
4. [Color Properties](#4-color-properties)
5. [Text Formatting](#5-text-formatting)
6. [Common Mistakes](#6-common-mistakes)

**NEW DataTable & SharePoint Rules:**
- [DataTable Filter Context Rules](#68-datatable-filter-context-rules)
- [SharePoint Lookup Patterns](#69-sharepoint-lookup-patterns)
- [SharePoint UserID Generation Rules](#610-sharepoint-userid-generation-rules)

---

## 1. FORMULA SYNTAX RULES

### 1.1 Leading Equals Sign
**BẮT BUỘC**: Tất cả expressions phải bắt đầu với dấu equals `=`:

```yaml
# ✅ ĐÚNG
Visible: =true
X: =34
Text: ="Hello, World"
Fill: =RGBA(255, 255, 255, 1)

# ❌ SAI - Thiếu dấu =
Visible: true
X: 34
Text: "Hello, World"
```

### 1.2 Single-line vs Multi-line Formulas
**QUY TẮC**: 
- **≤120 characters**: Sử dụng single line format
- **>120 characters**: Sử dụng pipe operator (`|`) cho multi-line format

```yaml
# ✅ ĐÚNG - Single line cho formula ngắn
OnSelect: =Set(varExample, true); Navigate(NextScreen)
Text: =Concatenate("Hello ", varUserName)

# ✅ ĐÚNG - Multi-line formula với pipe operator
OnVisible: |
  =Set(varActiveScreen, "Dashboard"); Set(varCurrentUser, {Name: "User", Role: "Admin"}); Set(varIsLoading, false)

OnSelect: |
  =Set(varIsLoading, true); If(Not(IsBlank(TextInput.Text)), Set(varSearchTerm, TextInput.Text); Navigate(SearchResultsScreen), Notify("Please enter search term", NotificationType.Warning)); Set(varIsLoading, false)
```

### 1.3 Action Properties - BẮT BUỘC Pipe Operator
**CRITICAL**: LUÔN LUÔN sử dụng pipe operator (`|`) cho TẤT CẢ action properties:

```yaml
# ✅ ĐÚNG - Pipe operator cho tất cả action properties
OnVisible: |
  =Set(varCurrentScreen, "Dashboard")

OnSelect: |
  =Navigate(NextScreen)

OnChange: |
  =Set(varInputValue, Self.Text)

OnHover: |
  =UpdateContext({isHovered: true})

# ❌ SAI - Single line cho action properties
OnVisible: =Set(varCurrentScreen, "Dashboard")
OnSelect: =Navigate(NextScreen)
```

### 1.4 Control References với Dots
**CRITICAL**: Wrap control names có dots bằng single quotes:

```yaml
# ✅ ĐÚNG - Single quotes cho control names có dots
Text: ='LoginCard.FormSection.UsernameInput'.Text
Visible: ='Header.Navigation.MenuButton'.Pressed

# ❌ SAI - Dot operator error
Text: =LoginCard.FormSection.UsernameInput.Text
```

---

## 2. YAML COMPLIANCE

### 2.1 Comment Rules
**CRITICAL**: 
- **KHÔNG BAO GIỜ** sử dụng inline comments trong YAML property values
- **LUÔN** đặt comments trên dòng riêng biệt

```yaml
# ✅ ĐÚNG - Comments trên dòng riêng
# Đây là comment cho Width property
Width: =Parent.Width * 0.8

# Đây là comment cho Height property  
Height: =Parent.Height / 2

# ❌ SAI - Inline comments phá vỡ YAML parsing
Width: =Parent.Width * 0.8  # This comment breaks YAML
Height: =Parent.Height / 2 // This also breaks YAML
```

### 2.2 YAML Indentation
**BẮT BUỘC**: Maintain proper YAML indentation để tránh "did not find expected key" parsing errors:

```yaml
# ✅ ĐÚNG - Proper indentation hierarchy
Children:
  - 'Container1':
      Control: GroupContainer
      Children:
        - 'Child1':
            Control: Label
            Properties:
              Text: ="Hello"
  - 'Container2':      # Consistent indentation level
      Control: Rectangle

# ❌ SAI - Incorrect indentation
Children:
  - 'Container1':
      Control: GroupContainer
- 'Container2':        # Wrong indentation level
    Control: Rectangle
```

### 2.3 Unicode Character Restrictions
**CRITICAL**: KHÔNG BAO GIỜ sử dụng emoji characters hoặc special Unicode symbols trong YAML text values:

```yaml
# ❌ SAI - Unicode emoji gây YAML parsing errors
Text: ="👤 Người dùng"    # Emoji gây parsing error
Text: ="📊 Thống kê"      # Emoji gây parsing error  
Text: ="⚠️ Cảnh báo"      # Emoji gây parsing error

# ✅ ĐÚNG - Chỉ sử dụng text, tránh emojis
Text: ="Người dùng"       # Text only, no emoji
Text: ="Thống kê"         # Text only, no emoji
Text: ="Cảnh báo"         # Text only, no emoji
```


> **CRITICAL**: Quy tắc chi tiết về formula syntax, YAML compliance, và best practices cho Power App development.

---

## 3. ICON GUIDELINES

### 3.1 Valid Icon References
**CHỈ** sử dụng icon references hợp lệ:

```yaml
# ✅ ĐÚNG - Valid icon references
Icon: =Icon.CalendarBlank
Icon: =Icon.CheckBadge
Icon: =Icon.Clock
Icon: =Icon.Cancel
Icon: =Icon.Person
Icon: =Icon.Settings
Icon: =Icon.Search
Icon: =Icon.Home
Icon: =Icon.Money
Icon: =Icon.Currency
Icon: =Icon.BarChart

# ❌ SAI - Invalid icon references
Icon: =Icon.Calendar        # Không tồn tại, sử dụng Icon.CalendarBlank
Icon: =Icon.BarChart        # Không tồn tại, sử dụng Icon.BarChart4
Icon: =Icon.Document        # Không tồn tại, sử dụng Icon.DocumentText
Icon: =Icon.CustomIcon      # Không có trong Power Apps
```

### 3.2 Icon Usage Best Practices
- Luôn verify icon names tồn tại trong Power Apps
- Sử dụng `Icon.CalendarBlank` thay vì `Icon.Calendar`
- Sử dụng `Icon.CheckBadge` cho checkmarks
- Sử dụng `Icon.Cancel` cho cancel/close actions

---

## 4. COLOR PROPERTIES

### 4.1 RGBA Format - BẮT BUỘC
**LUÔN LUÔN** sử dụng RGBA format cho tất cả color properties:

```yaml
# ✅ ĐÚNG - RGBA format
Fill: =RGBA(255, 255, 255, 1)
Color: =RGBA(32, 54, 71, 1)
BorderColor: =RGBA(230, 230, 230, 1)
HoverFill: =RGBA(0, 120, 212, 0.1)

# ❌ SAI - Other color formats
Fill: =Color.White
Color: =#205A47
BorderColor: =ColorValue("#E6E6E6")
```

### 4.2 DropShadow Properties
**CHỈ** sử dụng những values này cho DropShadow:

```yaml
# ✅ ĐÚNG - Valid DropShadow values
DropShadow: =DropShadow.Light
DropShadow: =DropShadow.Regular
DropShadow: =DropShadow.Bold
DropShadow: =DropShadow.ExtraBold
DropShadow: =DropShadow.Semilight
DropShadow: =DropShadow.None

# ❌ SAI - Invalid DropShadow values
DropShadow: =Light
DropShadow: ="Light"
DropShadow: =1
```

### 4.3 Text Function với RGBA Values
**CRITICAL**: KHÔNG BAO GIỜ sử dụng Text() function với RGBA values:

```yaml
# ❌ SAI - Text() không thể convert RGBA values
Text: =Concatenate("Color: ", Text(RGBA(255, 0, 0, 1)))
Text: =Text(RGBA(59, 130, 246, 1))

# ✅ ĐÚNG - Sử dụng string literals cho RGBA trong text
Text: =Concatenate("Color: RGBA(255, 0, 0, 1)")
Text: ="Primary: RGBA(59, 130, 246, 1)"
```

---

## 5. TEXT FORMATTING

### 5.1 Text Concatenation - PROPER SPACING
**BẮT BUỘC**: Luôn tách colon và space trong text concatenation:

```yaml
# ✅ ĐÚNG - Proper spacing
Text: ="Email:" & " " & ThisItem.Email
Text: ="Status:" & " " & ThisItem.Status
Text: ="Name:" & " " & ThisItem.Name

# ❌ SAI - Space bên trong quotes
Text: ="Email: " & ThisItem.Email
Text: ="Status: " & ThisItem.Status
Text: ="Name: " & ThisItem.Name
```

### 5.2 Font Properties
```yaml
# ✅ ĐÚNG - Font properties
FontWeight: =FontWeight.Bold
FontWeight: =FontWeight.Semibold
FontWeight: =FontWeight.Normal
Font: =Font.'Open Sans'
Font: =Font.'Segoe UI'

# Size recommendations
Size: =12    # Base size
Size: =14    # Medium size
Size: =16    # Large size
Size: =18    # XLarge size
```

---

## 6. COMMON MISTAKES

### 6.1 Invalid Self Properties
**KHÔNG BAO GIỜ SỬ DỤNG** những Self properties không hợp lệ:

```yaml
# ❌ SAI - Invalid Self properties
BorderColor: =If(Self.Focused, RGBA(0, 120, 212, 1), RGBA(200, 200, 200, 1))
Fill: =If(Self.IsHovered, RGBA(240, 240, 240, 1), RGBA(255, 255, 255, 1))

# ✅ ĐÚNG - Sử dụng properties hợp lệ hoặc events
BorderColor: =If(Self.Focus, RGBA(0, 120, 212, 1), RGBA(200, 200, 200, 1))
# Sử dụng OnHover events thay vì Self.IsHovered
```

### 6.2 Classic/TextInput Focus Property Error - CORRECTED
**CRITICAL**: Classic/TextInput chỉ hỗ trợ .Focus property, KHÔNG hỗ trợ .Focused:

```yaml
# ✅ ĐÚNG - Sử dụng .Focus cho Classic/TextInput
Visible: ='MyTextInput'.Focus
BorderColor: =If('MyTextInput'.Focus, RGBA(0, 120, 212, 1), RGBA(200, 200, 200, 1))

# ❌ SAI - .Focused không được hỗ trợ cho Classic/TextInput
Visible: ='MyTextInput'.Focused  # PA2108 Error
BorderColor: =If('MyTextInput'.Focused, RGBA(0, 120, 212, 1), RGBA(200, 200, 200, 1))

# NOTE: Classic/TextInput .Focus = Supported, .Focused = NOT Supported
```

### 6.3 Component Event Call Syntax Errors
**CRITICAL**: Sử dụng correct event call syntax:

```yaml
# ❌ SAI - Gọi component events trực tiếp từ screens
OnSelect: ='MyComponent'.OnNavigate(); Set(varAction, true)    # ERROR

# ✅ ĐÚNG - Binding actions to component events
OnNavigate: |
  =Set(varAction, true)  # Bind action to event

# ❌ SAI - Event call không có parentheses
OnSelect: =NavigationComponent.OnNavigate; Set(varActiveScreen, "Dashboard")

# ✅ ĐÚNG - Event call với parentheses
OnSelect: =NavigationComponent.OnNavigate(); Set(varActiveScreen, "Dashboard")
```

### 6.4 YAML Indentation Errors
**CRITICAL**: Tránh indentation errors gây "did not find expected key":

```yaml
# ❌ SAI - Inconsistent indentation
Children:
  - 'Container1':
      Control: GroupContainer
      Children:
        - 'Child1':
            Control: Label
- 'Container2':        # SAI - Inconsistent indentation
    Control: Rectangle

# ✅ ĐÚNG - Consistent indentation
Children:
  - 'Container1':
      Control: GroupContainer
      Children:
        - 'Child1':
            Control: Label
  - 'Container2':      # ĐÚNG - Consistent indentation level
      Control: Rectangle
```

### 6.5 Import Function Restrictions
**CRITICAL**: `Import()` function không được hỗ trợ trong Power Apps formulas:

```yaml
# ❌ SAI - Import() function không được hỗ trợ
OnSelect: =Set(varData, Import())

# ✅ ĐÚNG - Sử dụng simulated data với counters cho demo
OnSelect: |
  =Set(varImportedDataCount, Rand() * 10 + 5); Set(varDisplayText, Text(Round(varImportedDataCount, 0)) & " records imported")
```

### 6.6 Table Schema Compatibility
**CRITICAL**: KHÔNG BAO GIỜ sử dụng `Set(varName, Table(...))` cho complex data structures:

```yaml
# ❌ SAI - Set với Table có thể gây "Incompatible type" errors
OnVisible: =Set(varMyData, Table({Name: "John", Age: 30}, {Name: "Jane", Age: 25}))

# ✅ ĐÚNG - Sử dụng ClearCollect thay thế
OnVisible: |
  =ClearCollect(colMyData, {Name: "John", Age: 30}, {Name: "Jane", Age: 25})
```

### 6.7 Collection Naming Conventions
**CRITICAL**: Sử dụng naming conventions đúng:

```yaml
# ✅ ĐÚNG - Collection naming với 'col' prefix
ClearCollect(colAllUsers, ...)
ClearCollect(colMenuItems, ...)
ClearCollect(colWorkflowData, ...)

# ✅ ĐÚNG - Variable naming với 'var' prefix cho simple variables
Set(varCurrentUser, ...)
Set(varIsLoading, ...)
Set(varSelectedItem, ...)

# ❌ SAI - Confusion giữa collections và variables
Set(AllUsers, Table(...))  # Nên là ClearCollect(colAllUsers, ...)
ClearCollect(CurrentUser, ...)  # Nên là Set(varCurrentUser, ...)
```

### 6.8 DataTable Filter Context Rules - CRITICAL
**CRITICAL**: Trong Filter() context, sử dụng ThisRecord để reference current row, KHÔNG dùng disambiguation operator [@]:

```yaml
# ✅ ĐÚNG - Sử dụng ThisRecord trong Filter context
Items: |
  =Filter(User_1, 
    And(
      Or(IsBlank(varSearchText), varSearchText in fullname, varSearchText in email),
      Or(IsBlank(varSelectedDepartment), 
        LookUp(Department_1, departmentID = ThisRecord.departmentID).name = varSelectedDepartment),
      Or(IsBlank(varSelectedRole), 
        LookUp(Role_1, roleID = LookUp(User_Role, userID = ThisRecord.userID).roleID).name = varSelectedRole)
    )
  )

# ❌ SAI - Disambiguation operator [@] không đúng context trong Filter
Items: |
  =Filter(User_1,
    LookUp(Department_1, departmentID = User_1[@departmentID]).name = varSelectedDepartment
  )

# NOTE: Filter() tạo ra ThisRecord context cho mỗi row, KHÔNG phải table[@field] context
```

### 6.9 SharePoint Lookup Patterns - CRITICAL
**CRITICAL**: Proper SharePoint lookup patterns trong complex filters:

```yaml
# ✅ ĐÚNG - Nested LookUp pattern cho SharePoint related tables
Items: |
  =Filter(Users, 
    And(
      searchText in fullname,
      LookUp(Department, departmentID = ThisRecord.departmentID).name = selectedDept,
      LookUp(Role, roleID = LookUp(User_Role, userID = ThisRecord.userID).roleID).name = selectedRole
    )
  )

# ❌ SAI - Direct field access không work cho related tables
Items: |
  =Filter(Users,
    And(
      searchText in fullname,
      ThisRecord.Department.name = selectedDept  # Error - không có direct relationship
    )
  )

# ✅ ĐÚNG - ComboBox Selected property access
departmentID: ='Department.Dropdown'.Selected.departmentID
roleID: ='Role.Dropdown'.Selected.roleID

# ❌ SAI - Indirect lookup when direct access available
departmentID: =LookUp(Department, name = 'Department.Dropdown'.Selected.Name).departmentID
```

### 6.10 SharePoint UserID Generation Rules - CRITICAL
**CRITICAL**: SharePoint tables cần unique userID, KHÔNG sử dụng user input làm primary key:

```yaml
# ✅ ĐÚNG - Generate unique userID cho SharePoint
OnSelect: |
  =With({
    newUserID: Concatenate("USER_", Text(Rand() * 1000000, "000000"))
  },
    Patch(User_1, Defaults(User_1), {
      userID: newUserID,
      fullname: 'FullName.Input'.Text,
      email: 'Email.Input'.Text
    })
  )

# ✅ ĐÚNG - Alternative với timestamp
newUserID: =Concatenate("USR_", Text(Now(), "yyyymmddhhmmss"))

# ❌ SAI - Sử dụng user input làm primary key
OnSelect: |
  =Patch(User_1, Defaults(User_1), {
    userID: 'FullName.Input'.Text,  # Có thể duplicate, không unique
    fullname: 'FullName.Input'.Text
  })

# NOTE: SharePoint cần unique primary keys, user input không guarantee uniqueness
```

---
