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

## 8. POWER APP FORM RULES - QUY TẮC FORM ĐẦY ĐỦ

> **CRITICAL**: Quy tắc chi tiết về Form controls (FormViewer, Form) và TypedDataCard trong Power App.

---

## 📋 MỤC LỤC FORM RULES
1. [FormViewer vs Form Controls](#81-formviewer-vs-form-controls)
2. [Form Properties Complete](#82-form-properties-complete)
3. [TypedDataCard Structure](#83-typeddatacard-structure)
4. [DataCard Variants](#84-datacard-variants)
5. [Form Event Properties](#85-form-event-properties)
6. [SharePoint Data Binding](#86-sharepoint-data-binding)
7. [Form Field References](#87-form-field-references)

---

## 8.1 FORMVIEWER VS FORM CONTROLS

### 8.1.1 FormViewer Control - READ-ONLY Display
**CRITICAL**: FormViewer dùng để HIỂN THỊ data, không thể edit:

```yaml
# ✅ ĐÚNG - FormViewer cho read-only display
- FormViewer1:
    Control: FormViewer
    Layout: Vertical                # REQUIRED - MUST be outside Properties (prevents PA1011)
    Properties:
      # DATA Section
      DataSource: =User_1           # SharePoint table reference
      Item: =varSelectedRecord      # Record để display (optional)
      
      # DESIGN Section  
      BorderColor: =RGBA(245, 245, 245, 1)
      BorderStyle: =BorderStyle.Solid
      BorderThickness: =0
      Fill: =RGBA(0, 0, 0, 0)
      FocusedBorderColor: =Self.BorderColor
      FocusedBorderThickness: =4
      Height: =500
      Width: =800
      X: =40
      Y: =40
      Visible: =true
      
      # LAYOUT Section
      Columns: =3                   # Number of columns
      AcceptsFocus: =false          # Focus behavior
    Children:
      # TypedDataCard với Variant: ClassicTextualView (read-only)
```

### 8.1.2 Form Control - EDITABLE Forms
**CRITICAL**: Form control dùng để EDIT/CREATE data:

```yaml
# ✅ ĐÚNG - Form cho edit/create operations
- Form4:
    Control: Form
    Variant: Classic              # REQUIRED - MUST be outside Properties
    Layout: Vertical              # REQUIRED - MUST be outside Properties (prevents PA1011)
    Properties:
      # ACTION Section
      OnFailure: =false            # Action when form submission fails
      OnReset: =false              # Action when form is reset
      OnSuccess: =false            # Action when form submission succeeds
      
      # DATA Section
      DataSource: =User_1          # SharePoint table reference
      DefaultMode: =FormMode.Edit  # Edit, New, or View mode
      Item: =varSelectedRecord     # Record để edit (cho Edit mode)
      
      # DESIGN Section
      BorderColor: =RGBA(245, 245, 245, 1)
      BorderStyle: =BorderStyle.Solid
      BorderThickness: =0
      Fill: =RGBA(0, 0, 0, 0)
      FocusedBorderColor: =Self.BorderColor
      FocusedBorderThickness: =4
      Height: =500
      Width: =800
      X: =80
      Y: =80
      Visible: =true
      AcceptsFocus: =false         # Focus behavior
      
      # LAYOUT Section
      Columns: =3                  # Number of columns
    Children:
      # TypedDataCard với Variant: ClassicTextualEdit (editable)
```

### 8.1.3 Form Control Decision Rules
**KHI NÀO SỬ DỤNG:**

| Scenario | Control | Variant | Purpose |
|----------|---------|---------|---------|
| **View Details** | FormViewer | N/A | Read-only display của record |
| **Edit Record** | Form | Classic | Edit existing record |
| **Create Record** | Form | Classic | Create new record |
| **Approval Workflow** | FormViewer | N/A | View-only cho approval |

---

## 8.2 FORM PROPERTIES COMPLETE

### 8.2.1 Form Required Properties - CRITICAL
**BẮT BUỘC**: Form PHẢI có những properties này:

```yaml
# CRITICAL: Layout và Variant PHẢI nằm ngoài Properties section
Control: Form
Variant: Classic                  # REQUIRED - MUST be outside Properties
Layout: Vertical                  # REQUIRED - MUST be outside Properties (prevents PA1011)

Properties:
  # REQUIRED CORE PROPERTIES
  DataSource: =SharePointTable    # REQUIRED - Data source
  
  # POSITION & SIZE
  X: =position                    # Horizontal position
  Y: =position                    # Vertical position
  Width: =800                     # Form width
  Height: =500                    # Form height
  
  # FORM MODE (cho Form control only)
  DefaultMode: =FormMode.Edit     # Edit/New/View mode
  Item: =varSelectedRecord        # Record cho Edit mode
```

### 8.2.2 FormViewer Required Properties - CRITICAL
**BẮT BUỘC**: FormViewer PHẢI có những properties này:

```yaml
# CRITICAL: Layout PHẢI nằm ngoài Properties section
Control: FormViewer
Layout: Vertical                  # REQUIRED - MUST be outside Properties (prevents PA1011)

Properties:
  # REQUIRED CORE PROPERTIES
  DataSource: =SharePointTable    # REQUIRED - Data source
  
  # POSITION & SIZE
  X: =position                    # Horizontal position
  Y: =position                    # Vertical position
  Width: =800                     # Form width
  Height: =500                    # Form height
  
  # DISPLAY PROPERTIES
  Item: =varSelectedRecord        # Record để display (optional)
```

### 8.2.3 Common Form Properties
**OPTIONAL**: Các properties có thể customize:

```yaml
Properties:
  # LAYOUT OPTIONS
  Columns: =3                     # Number of columns (1-3)
  Layout: =Layout.Vertical        # Vertical (default) hoặc Horizontal
  
  # DESIGN STYLING
  BorderColor: =RGBA(245, 245, 245, 1)
  BorderStyle: =BorderStyle.Solid
  BorderThickness: =0
  Fill: =RGBA(0, 0, 0, 0)        # Transparent background
  FocusedBorderColor: =Self.BorderColor
  FocusedBorderThickness: =4
  
  # BEHAVIOR
  AcceptsFocus: =false           # Whether form accepts focus
  Visible: =true                 # Form visibility
```

---

## 8.3 TYPEDDATACARD STRUCTURE

### 8.3.1 TypedDataCard - BẮT BUỘC Structure
**CRITICAL**: Mọi field trong Form/FormViewer PHẢI sử dụng TypedDataCard:

```yaml
# ✅ ĐÚNG - TypedDataCard structure
- fullname_DataCard3:
    Control: TypedDataCard
    Variant: ClassicTextualView    # hoặc ClassicTextualEdit
    IsLocked: true                 # ALWAYS true
    Properties:
      BorderColor: =RGBA(245, 245, 245, 1)
      DataField: ="fullname"       # Field name trong SharePoint
      Default: =ThisItem.fullname  # Default value binding
      DisplayName: =DataSourceInfo([@User_1],DataSourceInfo.DisplayName,'fullname')
      Required: =true              # hoặc false
      Width: =266
      X: =0
      Y: =0
      
      # FOR EDIT CARDS ONLY
      Update: =DataCardValue23.Text           # Value để update (Edit mode)
      MaxLength: =DataSourceInfo([@User_1], DataSourceInfo.MaxLength, 'fullname')
    Children:
      # Label cho field name (DataCardKey)
      # Control cho field value (DataCardValue) 
      # Error message label (ErrorMessage)
      # Required indicator (StarVisible)
```

### 8.3.2 TypedDataCard Required Properties
**BẮT BUỘC**: Mọi TypedDataCard PHẢI có những properties này:

```yaml
Properties:
  BorderColor: =RGBA(245, 245, 245, 1)  # Border styling
  DataField: ="field_name"               # SharePoint field name
  Default: =ThisItem.field_name          # Default value
  DisplayName: =DataSourceInfo([@Table],DataSourceInfo.DisplayName,'field')
  Required: =true/false                  # Field requirement
  Width: =266                           # Card width
  X: =position                          # X position
  Y: =position                          # Y position
  
  # FOR EDIT VARIANTS ONLY
  Update: =DataCardValue.Text           # Update value cho edit mode
  MaxLength: =DataSourceInfo([@Table], DataSourceInfo.MaxLength, 'field')
```

### 8.3.3 TypedDataCard Children Structure
**CRITICAL**: Mọi TypedDataCard PHẢI có 4 children controls:

```yaml
Children:
  # 1. DataCardKey - Field label (REQUIRED)
  - DataCardKey15:
      Control: Label
      MetadataKey: FieldName
      Properties:
        AutoHeight: =true
        BorderColor: =RGBA(0, 0, 0, 0)
        BorderStyle: =BorderStyle.None
        BorderThickness: =2
        Color: =RGBA(50, 49, 48, 1)
        DisabledBorderColor: =RGBA(0, 0, 0, 0)
        DisabledColor: =RGBA(161, 159, 157, 1)
        FocusedBorderThickness: =4
        Font: =Font.'Segoe UI'
        FontWeight: =FontWeight.Semibold
        Height: =34
        PaddingLeft: =0
        Text: =Parent.DisplayName
        Width: =Parent.Width - 60
        Wrap: =false
        X: =30
        Y: =10

  # 2. DataCardValue - Field input/display (REQUIRED)  
  - DataCardValue15:
      Control: Label  # hoặc Classic/TextInput, Classic/ComboBox
      MetadataKey: FieldValue
      Properties:
        # Properties depend on control type và variant

  # 3. ErrorMessage - Validation errors (REQUIRED)
  - ErrorMessage15:
      Control: Label
      MetadataKey: ErrorMessage
      Properties:
        AutoHeight: =true
        BorderColor: =RGBA(0, 0, 0, 0)
        BorderStyle: =BorderStyle.None
        BorderThickness: =2
        Color: =RGBA(168, 0, 0, 1)
        DisabledBorderColor: =RGBA(0, 0, 0, 0)
        DisabledColor: =RGBA(168, 0, 0, 1)
        FocusedBorderThickness: =4
        Font: =Font.'Segoe UI'
        FontWeight: =FontWeight.Semibold
        Height: =10
        Live: =Live.Assertive
        PaddingBottom: =0
        PaddingLeft: =0
        PaddingRight: =0
        PaddingTop: =0
        Text: =Parent.Error
        Visible: =Parent.DisplayMode=DisplayMode.Edit
        Width: =Parent.Width - 60
        X: =30
        Y: =DataCardValue15.Y + DataCardValue15.Height

  # 4. StarVisible - Required indicator (REQUIRED)
  - StarVisible15:
      Control: Label
      MetadataKey: FieldRequired
      Properties:
        Align: =Align.Center
        BorderColor: =RGBA(0, 0, 0, 0)
        BorderStyle: =BorderStyle.None
        BorderThickness: =2
        Color: =RGBA(50, 49, 48, 1)
        DisabledBorderColor: =RGBA(0, 0, 0, 0)
        DisabledColor: =RGBA(161, 159, 157, 1)
        FocusedBorderThickness: =4
        Font: =Font.'Segoe UI'
        FontWeight: =FontWeight.Semibold
        Height: =DataCardKey15.Height
        PaddingLeft: =0
        Text: ="*"
        Visible: =And(Parent.Required, Parent.DisplayMode=DisplayMode.Edit)
        Width: =30
        Wrap: =false
        Y: =DataCardKey15.Y
```

---

## 8.4 DATACARD VARIANTS

### 8.4.1 FormViewer DataCard Variants (Read-Only)
**CHỈ** sử dụng những variants này cho FormViewer:

```yaml
# Text fields - read-only display
Variant: ClassicTextualView

# ComboBox fields - read-only display  
Variant: ClassicComboBoxView
```

### 8.4.2 Form DataCard Variants (Editable)
**CHỈ** sử dụng những variants này cho Form:

```yaml
# Text fields - editable input
Variant: ClassicTextualEdit

# ComboBox fields - editable selection
Variant: ClassicComboBoxEdit
```

### 8.4.3 DataCard Control Types by Variant
**CRITICAL**: Control types PHẢI match variant:

```yaml
# ClassicTextualView - Read-only text display
- DataCardValue:
    Control: Label
    MetadataKey: FieldValue
    Properties:
      AutoHeight: =true
      BorderColor: =RGBA(0, 0, 0, 0)
      BorderStyle: =BorderStyle.None
      BorderThickness: =2
      Color: =RGBA(50, 49, 48, 1)
      DisabledBorderColor: =RGBA(0, 0, 0, 0)
      DisabledColor: =RGBA(161, 159, 157, 1)
      DisplayMode: =Parent.DisplayMode
      FocusedBorderThickness: =4
      Font: =Font.'Segoe UI'
      Height: =27
      PaddingLeft: =0
      PaddingRight: =0
      PaddingTop: =0
      Text: =Parent.Default
      Width: =Parent.Width - 60
      X: =30
      Y: =DataCardKey.Y + DataCardKey.Height + 5

# ClassicTextualEdit - Editable text input
- DataCardValue:
    Control: Classic/TextInput
    MetadataKey: FieldValue
    Properties:
      BorderColor: =If(IsBlank(Parent.Error), Parent.BorderColor, Color.Red)
      Color: =RGBA(50, 49, 48, 1)
      Default: =Parent.Default
      DelayOutput: =true
      DisabledBorderColor: =RGBA(0, 0, 0, 0)
      DisabledColor: =RGBA(161, 159, 157, 1)
      DisabledFill: =RGBA(242, 242, 241, 0)
      DisplayMode: =Parent.DisplayMode
      Font: =Font.'Segoe UI'
      HoverBorderColor: =RGBA(16, 110, 190, 1)
      HoverColor: =RGBA(50, 49, 48, 1)
      HoverFill: =RGBA(255, 255, 255, 1)
      MaxLength: =Parent.MaxLength
      PaddingLeft: =5
      PressedBorderColor: =RGBA(0, 120, 212, 1)
      PressedColor: =RGBA(50, 49, 48, 1)
      PressedFill: =RGBA(255, 255, 255, 1)
      RadiusBottomLeft: =0
      RadiusBottomRight: =0
      RadiusTopLeft: =0
      RadiusTopRight: =0
      Tooltip: =Parent.DisplayName
      Width: =Parent.Width - 60
      X: =30
      Y: =DataCardKey.Y + DataCardKey.Height + 5

# ClassicComboBoxView - Read-only selection display (using Label)
- DataCardValue:
    Control: Label
    MetadataKey: FieldValue
    Properties:
      Text: =Parent.Default
      # Similar properties như ClassicTextualView

# ClassicComboBoxEdit - Editable selection
- DataCardValue:
    Control: Classic/ComboBox
    MetadataKey: FieldValue
    Properties:
      BorderColor: =If(IsBlank(Parent.Error), Parent.BorderColor, Color.Red)
      ChevronBackground: =RGBA(245, 245, 245, 1)
      ChevronDisabledBackground: =RGBA(242, 242, 241, 0)
      ChevronDisabledFill: =RGBA(161, 159, 157, 1)
      ChevronFill: =RGBA(50, 49, 48, 1)
      ChevronHoverBackground: =RGBA(245, 245, 245, 1)
      ChevronHoverFill: =RGBA(50, 49, 48, 1)
      Color: =RGBA(50, 49, 48, 1)
      DefaultSelectedItems: =Parent.Default
      DisabledBorderColor: =RGBA(0, 0, 0, 0)
      DisabledColor: =RGBA(161, 159, 157, 1)
      DisabledFill: =RGBA(242, 242, 241, 0)
      DisplayFields: =["Value"]           # hoặc ["Claims"] cho lookup fields
      DisplayMode: =Parent.DisplayMode
      Fill: =RGBA(245, 245, 245, 1)
      Font: =Font.'Segoe UI'
      HoverBorderColor: =RGBA(16, 110, 190, 1)
      HoverColor: =RGBA(50, 49, 48, 1)
      HoverFill: =RGBA(245, 245, 245, 1)
      Items: =Choices([@User_1].'departmentID')  # SharePoint choices
      PaddingLeft: =If(Self.DisplayMode = DisplayMode.Edit, 5, 0)
      PressedBorderColor: =RGBA(16, 110, 190, 1)
      PressedColor: =RGBA(255, 255, 255, 1)
      PressedFill: =RGBA(0, 120, 212, 1)
      SearchFields: =["Value"]            # hoặc ["Claims"] cho lookup fields
      SelectMultiple: =false
      SelectionColor: =RGBA(255, 255, 255, 1)
      SelectionFill: =RGBA(0, 120, 212, 1)
      Tooltip: =Parent.DisplayName
      Width: =Parent.Width - 60
      X: =30
      Y: =DataCardKey.Y + DataCardKey.Height + 5
```

---

## 8.5 FORM EVENT PROPERTIES

### 8.5.1 Form Action Properties - CRITICAL
**CRITICAL**: Form control có những event properties này:

```yaml
# Form Events (chỉ cho Form control, KHÔNG cho FormViewer)
Properties:
  OnFailure: =false               # Action when form submission fails
  OnReset: =false                 # Action when form is reset  
  OnSuccess: =false               # Action when form submission succeeds

# Example với actual logic
Properties:
  OnFailure: |
    =Notify("Có lỗi khi lưu dữ liệu", NotificationType.Error)
  OnReset: |
    =Set(varFormMode, FormMode.New); Set(varSelectedRecord, Blank())
  OnSuccess: |
    =Notify("Đã lưu thành công!", NotificationType.Success); Navigate(ListScreen)
```

### 8.5.2 Form Mode Management
**CRITICAL**: Form modes và cách sử dụng:

```yaml
# Form Mode Values
DefaultMode: =FormMode.Edit      # Edit existing record
DefaultMode: =FormMode.New       # Create new record  
DefaultMode: =FormMode.View      # View-only mode

# Form Mode với Item binding
Properties:
  DefaultMode: =FormMode.Edit
  Item: =varSelectedRecord       # Record để edit

Properties:
  DefaultMode: =FormMode.New
  Item: =Defaults(User_1)        # Defaults cho new record

Properties:
  DefaultMode: =FormMode.View
  Item: =varSelectedRecord       # Record để view
```

---

## 8.6 SHAREPOINT DATA BINDING

### 8.6.1 SharePoint Field References
**CRITICAL**: Proper SharePoint field binding patterns:

```yaml
# ✅ ĐÚNG - SharePoint field binding
Properties:
  DataField: ="fullname"                    # SharePoint logical field name
  Default: =ThisItem.fullname               # Current record value
  DisplayName: =DataSourceInfo([@User_1],DataSourceInfo.DisplayName,'fullname')
  
# ComboBox cho lookup fields
Properties:
  DataField: ="departmentID"
  Default: =ThisItem.departmentID
  Items: =Choices([@User_1].'departmentID') # SharePoint choices
  DisplayFields: =["Value"]
  SearchFields: =["Value"]

# Special SharePoint fields
Properties:
  DataField: ="Title"                       # SharePoint Title field
  Default: =ThisItem.userID                 # Custom field mapped to Title
  DataField: ="{ContentType}"              # SharePoint Content Type
  Default: =ThisItem.'Content type'
  Items: =Choices([@User_1].'{ContentType}')
  DisplayFields: =["Id"]
  SearchFields: =["Id"]
```

### 8.6.2 SharePoint System Fields
**AUTOMATIC**: SharePoint system fields auto-generated:

```yaml
# System fields - auto-populated by SharePoint
- Created_DataCard:
    Control: TypedDataCard
    Variant: ClassicTextualView
    Properties:
      DataField: ="Created"
      Default: =ThisItem.Created
      DisplayName: =DataSourceInfo([@User_1],DataSourceInfo.DisplayName,'Created')
      Required: =false              # System fields not required

- Modified_DataCard:
    Control: TypedDataCard
    Variant: ClassicTextualView
    Properties:
      DataField: ="Modified" 
      Default: =ThisItem.Modified
      DisplayName: =DataSourceInfo([@User_1],DataSourceInfo.DisplayName,'Modified')
      Required: =false

- Author_DataCard:
    Control: TypedDataCard
    Variant: ClassicComboBoxView
    Properties:
      DataField: ="Author"
      Default: =ThisItem.'Created By'
      DisplayName: =DataSourceInfo([@User_1],DataSourceInfo.DisplayName,'Author')
      Items: =Choices([@User_1].'Author')
      Required: =false

- Editor_DataCard:
    Control: TypedDataCard
    Variant: ClassicComboBoxView
    Properties:
      DataField: ="Editor"
      Default: =ThisItem.'Modified By'
      DisplayName: =DataSourceInfo([@User_1],DataSourceInfo.DisplayName,'Editor') 
      Items: =Choices([@User_1].'Editor')
      Required: =false
```

### 8.6.3 SharePoint Field Mapping Patterns
**CRITICAL**: Common SharePoint field mappings:

```yaml
# Text Fields
DataField: ="fullname"     → Default: =ThisItem.fullname
DataField: ="email"        → Default: =ThisItem.email
DataField: ="jobTitle"     → Default: =ThisItem.jobTitle
DataField: ="phone"        → Default: =ThisItem.phone

# Number Fields  
DataField: ="departmentID" → Default: =ThisItem.departmentID

# Title Field (Special)
DataField: ="Title"        → Default: =ThisItem.userID

# Lookup Fields
DataField: ="departmentID" → Items: =Choices([@User_1].'departmentID')
                           → DisplayFields: =["Value"]
                           → SearchFields: =["Value"]

# Person Fields  
DataField: ="Author"       → Items: =Choices([@User_1].'Author')
                           → DisplayFields: =["Claims"]
                           → SearchFields: =["Claims"]

# Content Type (Special)
DataField: ="{ContentType}" → Items: =Choices([@User_1].'{ContentType}')
                            → DisplayFields: =["Id"]
                            → SearchFields: =["Id"]
```

---

## 8.7 FORM FIELD REFERENCES

### 8.7.1 Field Access Patterns
**CRITICAL**: Correct patterns để access form field values:

```yaml
# ✅ ĐÚNG - Form field value access pattern
FormName.DataCardName.DataCardValueName.Property

# Examples:
Form4.fullname_DataCard4.DataCardValue23.Text           # TextInput value
Form4.email_DataCard4.DataCardValue24.Text              # TextInput value
Form4.departmentID_DataCard3.DataCardValue27.Selected   # ComboBox selected
Form4.departmentID_DataCard3.DataCardValue27.Selected.Value  # ComboBox value

# Submit button example
- SubmitButton:
    Control: Classic/Button
    Properties:
      OnSelect: |
        =Patch(User_1, Defaults(User_1), {
          fullname: Form4.fullname_DataCard4.DataCardValue23.Text,
          email: Form4.email_DataCard4.DataCardValue24.Text,
          jobTitle: Form4.jobTitle_DataCard4.DataCardValue25.Text,
          phone: Form4.phone_DataCard4.DataCardValue26.Text,
          departmentID: Form4.departmentID_DataCard3.DataCardValue27.Selected.Value
        })
```

### 8.7.2 Form Validation Patterns
**CRITICAL**: Form-level validation patterns:

```yaml
# ✅ ĐÚNG - Form validation before submit
OnSelect: |
  =If(And(
    Not(IsBlank(Form4.fullname_DataCard4.DataCardValue23.Text)),
    Not(IsBlank(Form4.email_DataCard4.DataCardValue24.Text)),
    IsMatch(Form4.email_DataCard4.DataCardValue24.Text, Email),
    Not(IsBlank(Form4.jobTitle_DataCard4.DataCardValue25.Text)),
    Not(IsBlank(Form4.phone_DataCard4.DataCardValue26.Text))
  ),
    // Submit logic
    SubmitForm(Form4),
    // Validation error
    Notify("Vui lòng điền đầy đủ thông tin hợp lệ", NotificationType.Warning)
  )

# ✅ ĐÚNG - Individual field validation
BorderColor: =If(
  And(
    Not(IsBlank(Self.Text)),
    IsMatch(Self.Text, Email)
  ),
  RGBA(16, 124, 16, 1),      # Green cho valid
  If(
    IsBlank(Self.Text),
    RGBA(200, 200, 200, 1),   # Gray cho empty
    RGBA(220, 53, 69, 1)      # Red cho invalid
  )
)
```

### 8.7.3 Form State Management
**CRITICAL**: Form mode và state management:

```yaml
# ✅ ĐÚNG - Form mode management
Properties:
  DefaultMode: =FormMode.New     # New record mode
  DefaultMode: =FormMode.Edit    # Edit record mode
  DefaultMode: =FormMode.View    # View-only mode
  
  Item: =varSelectedRecord       # Record được edit
  Item: =Defaults(User_1)        # New record defaults

# Form operations
OnSelect: |
  =SubmitForm(Form4)             # Submit form

OnSelect: |
  =ResetForm(Form4)              # Reset form

OnSelect: |
  =EditForm(Form4)               # Switch to edit mode

OnSelect: |
  =NewForm(Form4)                # Switch to new mode

# Form reset pattern
OnReset: |
  =Set(varSelectedRecord, Blank());
  Set(varFormMode, FormMode.New);
  Set(varErrorMessage, "")
```

### 8.7.4 FormViewer vs Form Field Access
**CRITICAL**: Different patterns cho FormViewer vs Form:

```yaml
# FormViewer - Read-only access
FormViewer1.fullname_DataCard3.DataCardValue15.Text  # Always Label.Text

# Form - Edit/Input access  
Form4.fullname_DataCard4.DataCardValue23.Text        # TextInput.Text
Form4.fullname_DataCard4.DataCardValue23.Default     # TextInput.Default

# ComboBox trong Form
Form4.departmentID_DataCard3.DataCardValue27.Selected
Form4.departmentID_DataCard3.DataCardValue27.Selected.Value
Form4.departmentID_DataCard3.DataCardValue27.SelectedItems
```

---

## 🚨 CRITICAL FORM ERRORS TO AVOID

### ⚠️ PA1011 ERROR - Missing Layout Property
**CRITICAL**: Layout property PHẢI nằm ngoài Properties section, nếu không sẽ gây PA1011 error:

```yaml
# ERROR: PA1011 : The keyword 'Layout' is required but is missing or empty.

# ❌ SAI - Layout trong Properties section hoặc missing
- MyForm:
    Control: Form
    Variant: Classic
    Properties:
      Layout: Vertical          # ERROR - Layout không được trong Properties
      DataSource: =User_1

- MyFormViewer:
    Control: FormViewer
    Properties:                 # ERROR - Missing Layout completely
      DataSource: =User_1

# ✅ ĐÚNG - Layout ngoài Properties section
- MyForm:
    Control: Form
    Variant: Classic
    Layout: Vertical            # CORRECT - Outside Properties
    Properties:
      DataSource: =User_1

- MyFormViewer:
    Control: FormViewer
    Layout: Vertical            # CORRECT - Outside Properties
    Properties:
      DataSource: =User_1
```

### 1. Missing Required Properties - PA1011 Error
```yaml
# ❌ SAI - Form missing required properties
- MyForm:
    Control: Form
    Properties:
      DataSource: =User_1        # ERROR - Missing Variant và Layout

# ERROR MESSAGE: PA1011 : The keyword 'Layout' is required but is missing or empty.

# ✅ ĐÚNG - Complete required properties
- MyForm:
    Control: Form
    Variant: Classic            # REQUIRED
    Layout: Vertical            # REQUIRED - MUST be outside Properties
    Properties:
      DataSource: =User_1
      DefaultMode: =FormMode.Edit

# ✅ ĐÚNG - FormViewer với Layout required
- MyFormViewer:
    Control: FormViewer
    Layout: Vertical            # REQUIRED - MUST be outside Properties
    Properties:
      DataSource: =User_1
```

### 2. Wrong Variant Usage
```yaml
# ❌ SAI - Wrong variant cho form type
FormViewer:
  Children:
    - DataCard:
        Variant: ClassicTextualEdit  # ERROR - FormViewer should use View variants

Form:
  Children:
    - DataCard:
        Variant: ClassicTextualView  # ERROR - Form should use Edit variants

# ✅ ĐÚNG - Correct variants
FormViewer:
  Children:
    - DataCard:
        Variant: ClassicTextualView  # Correct for read-only

Form:
  Children:
    - DataCard:
        Variant: ClassicTextualEdit  # Correct for editable
```

### 3. Missing Required DataCard Properties
```yaml
# ❌ SAI - TypedDataCard missing required properties
- MyDataCard:
    Control: TypedDataCard
    Variant: ClassicTextualEdit
    Properties:
      DataField: ="fullname"     # ERROR - Missing other required properties

# ✅ ĐÚNG - Complete required properties
- MyDataCard:
    Control: TypedDataCard
    Variant: ClassicTextualEdit
    IsLocked: true              # REQUIRED
    Properties:
      BorderColor: =RGBA(245, 245, 245, 1)
      DataField: ="fullname"    # REQUIRED
      Default: =ThisItem.fullname  # REQUIRED
      DisplayName: =DataSourceInfo([@User_1],DataSourceInfo.DisplayName,'fullname')
      Required: =true           # REQUIRED
      Update: =DataCardValue.Text  # REQUIRED cho Edit variants
      Width: =266
      X: =0
      Y: =0
```

### 4. Missing Required Children
```yaml
# ❌ SAI - TypedDataCard thiếu required children
- MyDataCard:
    Control: TypedDataCard
    Children:
      - DataCardValue:           # ERROR - Missing DataCardKey, ErrorMessage, StarVisible

# ✅ ĐÚNG - Complete children structure
- MyDataCard:
    Control: TypedDataCard
    Children:
      - DataCardKey:             # Field label - REQUIRED
      - DataCardValue:           # Field input/display - REQUIRED  
      - ErrorMessage:            # Error display - REQUIRED
      - StarVisible:             # Required indicator - REQUIRED
```

### 5. Wrong Control Types for Variants
```yaml
# ❌ SAI - Wrong control type cho variant
Variant: ClassicTextualEdit
Children:
  - DataCardValue:
      Control: Label            # ERROR - Edit variant needs TextInput

Variant: ClassicTextualView  
Children:
  - DataCardValue:
      Control: Classic/TextInput # ERROR - View variant should use Label

# ✅ ĐÚNG - Correct control types
Variant: ClassicTextualEdit
Children:
  - DataCardValue:
      Control: Classic/TextInput # Correct for edit

Variant: ClassicTextualView
Children:
  - DataCardValue:
      Control: Label            # Correct for view
```

### 6. Missing Update Property for Edit Cards
```yaml
# ❌ SAI - Edit DataCard missing Update property
- EditDataCard:
    Control: TypedDataCard
    Variant: ClassicTextualEdit
    Properties:
      DataField: ="fullname"
      Default: =ThisItem.fullname  # ERROR - Missing Update property

# ✅ ĐÚNG - Edit DataCard with Update property
- EditDataCard:
    Control: TypedDataCard
    Variant: ClassicTextualEdit
    Properties:
      DataField: ="fullname"
      Default: =ThisItem.fullname
      Update: =DataCardValue.Text  # REQUIRED cho Edit variants
```

---

## 🔑 FORM BEST PRACTICES

### 1. Use FormViewer cho Read-Only Scenarios
- Detail views và record display dialogs
- Approval workflows where data shouldn't be edited
- Report displays và data review screens
- **Variant**: ClassicTextualView, ClassicComboBoxView

### 2. Use Form cho Edit/Create Scenarios  
- Data entry forms và user input collection
- Record editing và update operations
- New record creation
- **Variant**: ClassicTextualEdit, ClassicComboBoxEdit

### 3. Always Include All 4 DataCard Children
- **DataCardKey** (field label) - REQUIRED
- **DataCardValue** (input/display control) - REQUIRED
- **ErrorMessage** (validation feedback) - REQUIRED
- **StarVisible** (required field indicator) - REQUIRED

### 4. Proper SharePoint Field Binding
- Use `DataSourceInfo()` cho DisplayName
- Use `Choices()` cho ComboBox Items
- Map custom fields correctly to SharePoint logical names
- Handle system fields (Created, Modified, Author, Editor) appropriately

### 5. Form State Management
- Use appropriate FormMode (New/Edit/View)
- Bind Item property correctly cho edit scenarios
- Implement proper form validation before submission
- Handle form events (OnSuccess, OnFailure, OnReset) appropriately

### 6. Field Access Patterns
- Use full path: `FormName.DataCardName.DataCardValueName.Property`
- Different patterns cho TextInput (.Text) vs ComboBox (.Selected)
- Validate all required fields before form submission

---

**Agent PHẢI tuân thủ những Form rules này TUYỆT ĐỐI khi tạo Power App forms với FormViewer, Form, và TypedDataCard controls.**
