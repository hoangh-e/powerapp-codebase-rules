# POWER APP CONTROL RULES - QUY TẮC CONTROLS

> **CRITICAL**: Quy tắc chi tiết về các controls được phép và cách sử dụng chúng trong Power App YAML.

---

## 📋 MỤC LỤC
1. [Allowed Controls](#1-allowed-controls)
2. [Control Properties Restrictions](#2-control-properties-restrictions)
3. [Control Z-Index & Rendering Order](#3-control-z-index--rendering-order)
4. [Control Variants](#4-control-variants)

**UPDATED Control Property Rules:**
- [Control Properties Restrictions - Rule 10](#20-control-properties-restrictions---critical-rule-10)
- [Version Number Prohibition](#201-version-number-prohibition---all-controls)

**NEW DataTable Rules:**
- [DataTable vs Gallery Decision Rules](#18-datatable-vs-gallery-decision-rules)
- [DataTable Control Structure](#19-datatable-control-structure)
- [DataTableColumn Rules](#110-datatablecolumn-rules)
- [Critical DataTable Errors](#111-critical-datatable-errors)
- [DataTable Performance Guidelines](#112-datatable-performance-guidelines)
- [DataTable Decision Flowchart](#113-datatable-decision-flowchart)
- [DataTable Selection Handling](#114-datatable-selection-handling)

**NEW Control Property Rules:**
- [Classic/TextInput Focus Properties](#28-classictextinput-focus-properties)
- [Input & Dropdown Focus Properties](#29-input--dropdown-focus-properties---critical)
- [Collection vs Text Function Rules](#210-collection-vs-text-function-rules)
- [Table() Function Restrictions](#211-table-function-restrictions---critical)

---

## 1. ALLOWED CONTROLS

> **⚠️ CRITICAL FOCUS RULE**: TẤT CẢ Input controls (bao gồm cả `Classic/TextInput`) KHÔNG được hỗ trợ `.Focus` hoặc `.Focused` properties. Sử dụng `FocusedBorderColor` và `FocusedBorderThickness` thay thế.

### 1.1 Input Controls
```yaml
# Text Input
Control: Classic/TextInput

# Buttons  
Control: Classic/Button

# Date Selection
Control: Classic/DatePicker

# Dropdowns
Control: Classic/DropDown
Control: Classic/ComboBox

# Selection Controls
Control: Classic/CheckBox
Control: Classic/Radio
Control: Classic/Toggle
Control: Classic/Slider
Control: Rating
Control: PenInput
```

### 1.2 Display Controls
```yaml
# Text Display
Control: Label

# Media Display
Control: Image
Control: HtmlViewer
Control: RichTextEditor
Control: PDFViewer

# Icons
Control: Classic/Icon
```

### 1.3 Layout Controls
```yaml
# Containers
Control: GroupContainer
Control: Rectangle

# Data Display
Control: Gallery
Control: Form
Control: FormViewer
Control: DataTable
```

### 1.4 Component Controls
```yaml
# Canvas Components
Control: CanvasComponent
ComponentName: ComponentName
```

### 1.5 Shape Controls
```yaml
Control: Circle
Control: Triangle
Control: Pentagon
Control: Octagon
Control: Star
Control: Arrow
Control: PartialCircle
```

### 1.6 Chart Controls

**CRITICAL**: `ColumnChart` control KHÔNG TỒN TẠI. Sử dụng `BarChart` thay thế.

#### 1.6.1 Các Chart Control được hỗ trợ
```yaml
# Bar/Column Charts
Control: BarChart          # Dùng cho column charts và bar charts

# Line Charts  
Control: LineChart

# Pie Charts
Control: PieChart

# Chart Legend
Control: Legend             # Hiển thị legend cho charts
```

#### 1.6.2 Cấu trúc Chart Composite (BẮT BUỘC)
**MANDATORY**: Charts PHẢI được tổ chức trong composite structure với Title, Chart, và Legend:

```yaml
# ✅ ĐÚNG - Composite Chart Structure
- Title1:
    Control: Label
    Group: CompositeColumnChart1    # Group name để liên kết components
    Properties:
      Align: =Align.Center
      BorderColor: =RGBA(0, 0, 0, 1)
      Color: =RGBA(50, 49, 48, 1)
      Font: =Font.'Segoe UI'
      Height: =25
      Size: =14
      Text: ="Chart Title"
      Width: =400
      X: =40
      Y: =40

- ColumnChart1:
    Control: BarChart              # ❗ QUAN TRỌNG: Dùng BarChart, KHÔNG phải ColumnChart
    Group: CompositeColumnChart1    # Cùng group name
    Properties:
      BorderColor: =RGBA(0, 0, 0, 0)
      BorderStyle: =BorderStyle.None
      BorderThickness: =2
      Color: =RGBA(50, 49, 48, 1)
      Font: =Font.'Segoe UI'
      ItemColorSet: =[RGBA(17, 141, 255, 1),RGBA(18,35,158, 1), RGBA(230,108,55,1), RGBA(107,0,123,1), RGBA(224,68,167,1), RGBA(116,78,194,1), RGBA(217,179,0,1), RGBA(214,69,80,1), RGBA(25, 114, 120, 1), RGBA(26, 171, 64, 1), RGBA(21, 198, 244, 1)]
      Items.Labels: =City            # Chart labels data
      Items.Series1: =Area           # Primary data series
      Items.Series2: =Density        # Additional series (optional)
      Items.Series3: =Population     # Additional series (optional)
      Size: =15
      X: =40
      Y: =70

- Legend1:
    Control: Legend
    Group: CompositeColumnChart1    # Cùng group name
    Properties:
      BorderColor: =RGBA(0, 0, 0, 0)
      BorderStyle: =BorderStyle.None
      Color: =RGBA(50, 49, 48, 1)
      Font: =Font.'Segoe UI'
      Height: =80
      ItemColorSet: =ColumnChart1.ItemColorSet  # Reference đến chart colors
      Items: =ColumnChart1.SeriesLabels         # Reference đến chart series
      Items.Value: =Value
      Width: =320
      X: =80
      Y: =480
```

#### 1.6.3 Chart Control Errors & Solutions
**COMMON ERROR**: `Unknown control type 'ColumnChart'`
```yaml
# ❌ SAI - Control không tồn tại
Control: ColumnChart

# ✅ ĐÚNG - Sử dụng BarChart
Control: BarChart
```

#### 1.6.4 Chart Properties Requirements
**BẮT BUỘC** properties cho chart controls:
- `Items.Labels` - Data source cho chart labels
- `Items.Series1` - Primary data series (minimum requirement)
- `ItemColorSet` - Color palette cho chart
- `Group` - Group name để liên kết với Title và Legend

**TÙY CHỌN** properties:
- `Items.Series2`, `Items.Series3`, etc. - Additional data series (chỉ cho BarChart/LineChart)
- `BorderStyle`, `BorderColor`, `BorderThickness` - Chart border styling

#### 1.6.5 PieChart Composite Structure (BẮT BUỘC)
**MANDATORY**: PieChart cũng PHẢI được tổ chức trong composite structure:

```yaml
# ✅ ĐÚNG - Composite PieChart Structure
- Title3:
    Control: Label
    Group: CompositePieChart1       # Group name để liên kết components
    Properties:
      Align: =Align.Center
      BorderColor: =RGBA(0, 0, 0, 1)
      Color: =RGBA(50, 49, 48, 1)
      DisabledColor: =RGBA(161, 159, 157, 1)
      Font: =Font.'Segoe UI'
      Height: =25
      Size: =14
      Text: ="Chart Title"
      Width: =400
      X: =60
      Y: =60

- PieChart1:
    Control: PieChart               # PieChart control (không có version number)
    Group: CompositePieChart1       # Cùng group name
    Properties:
      BorderColor: =RGBA(0, 0, 0, 0)
      BorderStyle: =BorderStyle.None
      BorderThickness: =2
      Color: =RGBA(50, 49, 48, 1)
      DisabledBorderColor: =RGBA(0, 0, 0, 0)
      Font: =Font.'Segoe UI'
      HoverBorderColor: =RGBA(0, 0, 0, 0)
      ItemColorSet: =[RGBA(17, 141, 255, 1),RGBA(18,35,158, 1), RGBA(230,108,55,1), RGBA(107,0,123,1), RGBA(224,68,167,1), RGBA(116,78,194,1), RGBA(217,179,0,1), RGBA(214,69,80,1), RGBA(25, 114, 120, 1), RGBA(26, 171, 64, 1), RGBA(21, 198, 244, 1)]
      Items.Labels: =City           # Chart labels data
      Items.Series: =Area           # ❗ QUAN TRỌNG: PieChart chỉ dùng Items.Series (không có Series1, Series2)
      PressedBorderColor: =RGBA(0, 0, 0, 0)
      Size: =15
      X: =60
      Y: =90

- Legend3:
    Control: Legend
    Group: CompositePieChart1       # Cùng group name
    Properties:
      BorderColor: =RGBA(0, 0, 0, 0)
      BorderStyle: =BorderStyle.None
      BorderThickness: =2
      Color: =RGBA(50, 49, 48, 1)
      DisabledBorderColor: =RGBA(0, 0, 0, 0)
      DisabledColor: =RGBA(161, 159, 157, 1)
      DisabledFill: =RGBA(225, 223, 221, 1)
      Font: =Font.'Segoe UI'
      Height: =200
      HoverBorderColor: =RGBA(0, 0, 0, 0)
      ItemColorSet: =PieChart1.ItemColorSet    # Reference đến pie chart colors
      Items: =PieChart1.SeriesLabels           # Reference đến pie chart series
      Items.Value: =Value
      PressedBorderColor: =RGBA(0, 0, 0, 0)
      Size: =16
      Width: =400
      X: =60
      Y: =490
```

#### 1.6.6 Khác biệt giữa BarChart và PieChart Properties
**CRITICAL DIFFERENCES**:

**BarChart (Column Charts)**:
- Sử dụng `Items.Series1`, `Items.Series2`, etc. cho multiple data series
- Hỗ trợ multiple data series trên cùng một chart

**PieChart**:
- Chỉ sử dụng `Items.Series` (singular) cho single data series  
- KHÔNG hỗ trợ multiple series (Items.Series1, Items.Series2 sẽ gây lỗi)

```yaml
# ✅ ĐÚNG - BarChart multiple series
Items.Series1: =Area
Items.Series2: =Population
Items.Series3: =Density

# ✅ ĐÚNG - PieChart single series
Items.Series: =Area

# ❌ SAI - PieChart với multiple series
Items.Series1: =Area      # Sẽ gây lỗi
Items.Series2: =Population
```

### 1.7 Media & Data Controls
```yaml
# Media
Control: AddMedia
Control: Timer

# Data Import/Export
Control: Import
Control: Export
Control: ListBox
Control: PowerBI
```

### 1.8 DATATABLE VS GALLERY DECISION RULES

#### 1.8.1 KHI NÀO SỬ DỤNG DATATABLE - BẮT BUỘC
**CRITICAL**: LUÔN LUÔN ưu tiên sử dụng DataTable thay vì Gallery trong những trường hợp sau:

##### Database/SharePoint Data Lists
```yaml
# ✅ ĐÚNG - DataTable cho database data
Items: =User_1  # SharePoint table
Items: =Filter(NamecardRequest, Status = "Pending")
Items: =Department_1

# ❌ SAI - Gallery cho tabular database data  
# Gallery chỉ dùng cho custom layouts, không phải tabular data
```

##### Forms có chức năng Chỉnh sửa và Tạo mới - CRITICAL
**MANDATORY**: DataTable PHẢI được sử dụng khi screen có các form/modal để:
- **Tạo mới** record (Create/Add forms)
- **Chỉnh sửa** existing records (Edit/Update forms)
- **Xóa** records với confirmation
- **Quản lý** data với CRUD operations

```yaml
# ✅ ĐÚNG - DataTable với Edit/Create/Delete functionality
- UserManagementScreen với:
  - DataTable hiển thị users
  - Add User form/modal
  - Edit User form/modal  
  - Delete confirmation modal

# ✅ ĐÚNG - DataTable với CRUD operations
- NamecardRequestScreen với:
  - DataTable hiển thị requests
  - Create Request form
  - Edit Request details
  - Approve/Reject actions

# ❌ SAI - Gallery cho data management screens
# Gallery không phù hợp khi cần edit/create/delete data
```

##### Tabular Data với Multiple Columns
```yaml
# ✅ ĐÚNG - DataTable cho data có nhiều columns cần hiển thị
- Users: fullname, email, department, role, phone, actions
- Orders: orderID, customerName, orderDate, status, total, actions
- Products: name, category, price, quantity, supplier, actions

# ❌ SAI - Gallery cho structured tabular data với action buttons
```

##### Data cần Sorting/Filtering Built-in
```yaml
# ✅ ĐÚNG - DataTable có built-in sorting và selection
Control: DataTable
Properties:
  Items: =MyData
  # Built-in sorting by clicking column headers
  # Built-in row selection automatically handled
# User có thể click column header để sort và click row để select

# ❌ SAI - Gallery cần custom sorting/selection implementation
```

##### Management Screens Pattern - MANDATORY
**CRITICAL**: Tất cả screens có pattern "Management" PHẢI sử dụng DataTable:

```yaml
# ✅ ĐÚNG - Management screens với DataTable
- UserManagementScreen    # Quản lý người dùng
- RequestManagementScreen # Quản lý yêu cầu  
- OrderManagementScreen   # Quản lý đơn hàng
- ProductManagementScreen # Quản lý sản phẩm

# Pattern: DataTable + Add/Edit/Delete forms + Search/Filter
```

#### 1.8.2 KHI NÀO SỬ DỤNG GALLERY
**CHỈ** sử dụng Gallery trong những trường hợp này:

##### Custom Visual Layouts
```yaml
# ✅ ĐÚNG - Gallery cho custom card layouts
- Image thumbnails với text
- Dashboard cards với icons và metrics
- Chat message bubbles
- Product cards với images
```

##### Non-Tabular Data Presentation
```yaml
# ✅ ĐÚNG - Gallery cho non-structured presentation
- Photo galleries
- News article cards  
- Social media posts
- Dashboard widgets
```

### 1.9 DATATABLE CONTROL STRUCTURE

#### 1.9.1 Basic DataTable Structure - BẮT BUỘC
```yaml
# ✅ ĐÚNG - DataTable structure (NO version numbers, NO OnSelect)
- MyDataTable:
    Control: DataTable
    Properties:
      Items: =MyDataSource
      # DataTable automatically handles selection - no OnSelect property
    Children:
      - Column1:
          Control: DataTableColumn
          Variant: Textual
          Properties:
            FieldDisplayName: ="Display Name"
            FieldName: ="LogicalName"
            Order: =1
            Text: =ThisItem.LogicalName

# ❌ SAI - Version numbers và OnSelect KHÔNG được phép
- MyDataTable:
    Control: DataTable@1.0.3  # PA2108 Error - No version numbers
    Properties:
      OnSelect: =Set(varSelected, Self.Selected)  # PA2108 Error - Not supported
```

#### 1.9.2 DataTable với SharePoint Data
```yaml
# ✅ ĐÚNG - DataTable cho SharePoint tables (NO OnSelect property)
- UsersTable:
    Control: DataTable
    Properties:
      Items: =User_1  # SharePoint table reference
      # Selection automatically handled by DataTable control
    Children:
      - FullName.Column:
          Control: DataTableColumn
          Variant: Textual
          Properties:
            FieldDisplayName: ="Họ và tên"
            FieldName: ="fullname"
            Order: =1
            Text: =ThisItem.fullname
      - Email.Column:
          Control: DataTableColumn
          Variant: Textual
          Properties:
            FieldDisplayName: ="Email"
            FieldName: ="email"
            Order: =2
            Text: =ThisItem.email
```

### 1.10 DATATABLECOLUMN RULES

#### 1.10.1 Required DataTableColumn Properties
**BẮT BUỘC**: Mọi DataTableColumn PHẢI có những properties này:

```yaml
Properties:
  FieldDisplayName: ="User Friendly Name"  # Tên hiển thị cho user
  FieldName: ="logical_name"               # Tên field trong data source
  Order: =1                                # Thứ tự column (bắt đầu từ 1)
  Text: =ThisItem.field_name               # Giá trị hiển thị
```

#### 1.10.2 DataTableColumn Variants - SUPPORTED ONLY
**CHỈ** sử dụng những variants này (PA2109 Error nếu sử dụng variants khác):

```yaml
# Text data - SUPPORTED
Variant: Textual

# Number data - SUPPORTED
Variant: Number  

# Date/Time data - SUPPORTED
Variant: DateTime

# Boolean data - SUPPORTED
Variant: Boolean

# ❌ KHÔNG HỖ TRỢ - Custom variant causes PA2109 Error
# Variant: Custom  # ERROR - Not supported
```

#### 1.10.3 DataTableColumn Width Management
```yaml
# ✅ ĐÚNG - Specify column widths
Properties:
  Width: =200      # Fixed width
  Width: =Parent.Width * 0.25  # Relative width

# ✅ ĐÚNG - Let DataTable auto-size
# Không specify Width property cho auto-sizing
```

### 1.11 CRITICAL DATATABLE ERRORS

#### 1.11.1 Version Number Prohibition - PA2108/PA2109 Prevention (Rule 10)
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

**CRITICAL**: This applies to ALL controls, not just DataTable. Version numbers cause parsing errors and are never supported trong Power Apps YAML format.

#### 1.11.2 Missing Required Properties
```yaml
# ❌ SAI - Missing required properties
- MyColumn:
    Control: DataTableColumn
    Properties:
      Text: =ThisItem.Name  # Missing FieldName, FieldDisplayName, Order

# ✅ ĐÚNG - All required properties
- MyColumn:
    Control: DataTableColumn
    Variant: Textual
    Properties:
      FieldDisplayName: ="Name"
      FieldName: ="Name"  
      Order: =1
      Text: =ThisItem.Name
```

#### 1.11.3 DataTable Screen Children Only - CRITICAL RULE
**CRITICAL**: DataTable controls MUST be direct Children of Screens ONLY. Never nest DataTable inside GroupContainer or other containers.

```yaml
# ✅ ĐÚNG - DataTable as direct Screen child
Screens:
  MyScreen:
    Children:
      - MyDataTable:
          Control: DataTable
          Properties:
            X: =Parent.Width * 0.03
            Y: =Parent.Height * 0.3  
            Items: =MyDataSource

# ❌ SAI - DataTable nested in containers
Screens:
  MyScreen:
    Children:
      - Container:
          Control: GroupContainer
          Children:
            - MyDataTable:  # ERROR - Cannot nest DataTable
                Control: DataTable
```

#### 1.11.4 DataTable Event Properties - CRITICAL ERROR
**CRITICAL**: DataTable control does NOT support OnSelect OR OnSelectionChange properties (PA2108 error).

```yaml
# ❌ SAI - DataTable không hỗ trợ OnSelect
- MyDataTable:
    Control: DataTable
    Properties:
      Items: =MyDataSource
      OnSelect: |
        =Set(varSelectedItem, Self.Selected)  # PA2108 Error

# ❌ SAI - DataTable không hỗ trợ OnSelectionChange  
- MyDataTable:
    Control: DataTable
    Properties:
      Items: =MyDataSource
      OnSelectionChange: |
        =Set(varSelectedItem, Self.Selected)  # PA2108 Error

# ✅ ĐÚNG - DataTable chỉ có Items property, selection via overlay handler
- MyDataTable:
    Control: DataTable
    Properties:
      Items: =MyDataSource
      # Access selection via Self.Selected trong separate overlay handler
```

#### 1.11.5 DataTableColumn Custom Variant - PA2109 ERROR
**CRITICAL**: DataTableColumn does NOT support 'Custom' variant. Use only supported variants.

```yaml
# ❌ SAI - Custom variant không được hỗ trợ
- ActionColumn:
    Control: DataTableColumn
    Variant: Custom  # PA2109 Error
    Properties:
      FieldDisplayName: ="Actions"

# ✅ ĐÚNG - Chỉ sử dụng supported variants
- ActionColumn:
    Control: DataTableColumn
    Variant: Textual  # Supported variant
    Properties:
      FieldDisplayName: ="Actions"
      FieldName: ="Actions"
      Order: =5
      Text: ="Edit | Delete"
```

#### 1.11.6 DataTableColumn Reference Rules - CRITICAL
**CRITICAL**: In DataTableColumn context, use proper data references.

```yaml
# ❌ SAI - Sai reference context
- NameColumn:
    Control: DataTableColumn
    Properties:
      OnSelect: |
        =Set(varSelectedItem, ThisItem)  # ERROR - ThisItem not available in column

# ✅ ĐÚNG - DataTableColumn chỉ hiển thị data, không handle events
- NameColumn:
    Control: DataTableColumn
    Variant: Textual
    Properties:
      FieldDisplayName: ="Full Name"
      FieldName: ="fullname"
      Order: =1
      Text: =ThisItem.fullname  # Correct for display
```

#### 1.11.7 DataTable OnSelectionChange Error - PA2108 CRITICAL FIX
**CRITICAL**: PA2108 Error "Unknown property 'OnSelectionChange' for control type 'DataTable'" - DataTable does NOT support this property.

```yaml
# ❌ SAI - OnSelectionChange causes PA2108 error
- MyDataTable:
    Control: DataTable
    Properties:
      Items: =MyDataSource
      OnSelectionChange: |  # PA2108 ERROR - Property not supported
        =Set(varSelectedItem, Self.Selected)

# ✅ ĐÚNG - Use overlay handler for selection detection
- MyDataTable:
    Control: DataTable
    Properties:
      Items: =MyDataSource
      # Only Items property supported for DataTable

# Separate overlay for selection handling
- DataTable.SelectionOverlay:
    Control: Rectangle
    Properties:
      Fill: =RGBA(0, 0, 0, 0)  # Invisible
      Height: =MyDataTable.Height
      Width: =MyDataTable.Width  
      X: =MyDataTable.X
      Y: =MyDataTable.Y
      OnSelect: |
        =If(Not(IsBlank(MyDataTable.Selected)),
          Set(varSelectedItem, MyDataTable.Selected);
          # Additional selection logic here
        )
```

**SOLUTION PATTERN**: Always use separate Rectangle overlay positioned over DataTable for selection event handling.

### 1.12 DATATABLE PERFORMANCE GUIDELINES

#### 1.12.1 Filter at Data Source Level
```yaml
# ✅ ĐÚNG - Filter trước khi pass vào DataTable
Items: |
  =Filter(Users, 
    And(
      Status = "Active",
      Department = varSelectedDepartment
    )
  )

# ❌ SAI - Load all data then rely on DataTable filtering
Items: =Users  # Load everything
```

#### 1.12.2 Use Proper Column Data Types
```yaml
# ✅ ĐÚNG - Sử dụng proper variants
- DateColumn:
    Variant: DateTime
    Properties:
      Text: =Text(ThisItem.CreatedDate, DateTimeFormat.ShortDate)

- NumberColumn:
    Variant: Number  
    Properties:
      Text: =Text(ThisItem.Amount, "$#,##0.00")

# ❌ SAI - All columns as Textual
- DateColumn:
    Variant: Textual  # SAI - should be DateTime
```

#### 1.12.3 DataTable vs Gallery Performance
**DataTable WINS khi:**
- Tabular data với > 100 rows
- Multiple sortable columns
- Need built-in selection
- Professional table appearance

**Gallery WINS khi:**
- Custom visual layouts
- Rich media content (images, videos)
- Card-based presentation
- < 50 items với complex layouts

### 1.13 DATATABLE DECISION FLOWCHART

```
Có phải screen Management (CRUD operations)?
├── YES → ALWAYS use DataTable ✅
│   ├── Create/Add forms → DataTable required
│   ├── Edit/Update forms → DataTable required  
│   └── Delete operations → DataTable required
└── NO → Is it database/SharePoint data?
    ├── YES → Is it tabular (rows/columns)?
    │   ├── YES → Use DataTable ✅
    │   └── NO → Consider Gallery
    └── NO → Is it custom visual layout?
        ├── YES → Use Gallery ✅
        └── NO → Use DataTable for structured data
```

### 1.14 DATATABLE SELECTION HANDLING - CRITICAL

#### 1.14.1 DataTable Selection Handling - CRITICAL ERROR FIXED
**CRITICAL**: DataTable does NOT support OnSelectionChange property (PA2108 error). Use separate overlay handler for selection detection.

```yaml
# ❌ SAI - DataTable không hỗ trợ OnSelectionChange property
- MyDataTable:
    Control: DataTable
    Properties:
      Items: =MyDataSource
      OnSelectionChange: |
        =Set(varSelectedItem, Self.Selected)  # PA2108 Error - Property not supported

# ✅ ĐÚNG - DataTable without OnSelectionChange, use overlay for selection
- MyDataTable:
    Control: DataTable
    Properties:
      Items: =MyDataSource
      # DataTable selection accessed via Self.Selected in overlay handler

# ✅ ĐÚNG - Separate overlay handler for selection detection
- DataTable.Selection.Handler:
    Control: Rectangle
    Properties:
      Fill: =RGBA(0, 0, 0, 0)  # Invisible overlay
      Height: =MyDataTable.Height
      Width: =MyDataTable.Width
      X: =MyDataTable.X
      Y: =MyDataTable.Y
      OnSelect: |
        =If(Not(IsBlank(MyDataTable.Selected)),
          Set(varSelectedItem, MyDataTable.Selected);
          Set(varShowContextMenu, true)
        )
```

#### 1.14.2 DataTable Context Menu Pattern - CORRECTED
**CRITICAL**: Context menus require separate overlay handler since DataTable does NOT support OnSelectionChange.

```yaml
# ✅ ĐÚNG - DataTable with proper context menu support
- Users.DataTable:
    Control: DataTable
    Properties:
      Items: =FilteredUsers
      # NO OnSelectionChange property - not supported
    Children:
      # DataTable columns here...

# ✅ ĐÚNG - Separate overlay handler for context menu
- DataTable.Context.Handler:
    Control: Rectangle
    Properties:
      Fill: =RGBA(0, 0, 0, 0)  # Invisible overlay
      Height: ='Users.DataTable'.Height
      Width: ='Users.DataTable'.Width
      X: ='Users.DataTable'.X
      Y: ='Users.DataTable'.Y
      OnSelect: |
        =If(Not(IsBlank('Users.DataTable'.Selected)),
          Set(varSelectedUser, 'Users.DataTable'.Selected);
          Set(varContextMenuX, Min(App.ActiveScreen.Width - 150, App.MouseX));
          Set(varContextMenuY, Min(App.ActiveScreen.Height - 120, App.MouseY));
          Set(varShowContextMenu, true)
        )

# ❌ SAI - DataTable with OnSelectionChange causes PA2108 error
- Users.DataTable:
    Control: DataTable
    Properties:
      OnSelectionChange: =Set(varSelected, Self.Selected)  # ERROR - Not supported
```

#### 1.14.3 Context Menu Z-Index Rules
**CRITICAL**: Context menus MUST be positioned last trong Children array để ensure highest z-index.

```yaml
# ✅ ĐÚNG - Context menu positioning trong Children array
Children:
  # 1. Background elements first
  - Background:
      Control: Rectangle
  
  # 2. Data display elements
  - Users.DataTable:
      Control: DataTable
      
  # 3. Interactive overlays
  - DataTable.Overlay.Handler:
      Control: Rectangle
      
  # 4. Context menus LAST (highest z-index)
  - User.Context.Menu:
      Control: GroupContainer
      Properties:
        Visible: =varShowContextMenu

# ❌ SAI - Context menu positioned early trong Children array
Children:
  - User.Context.Menu:  # ERROR - Sẽ bị other elements cover
      Control: GroupContainer
  - Users.DataTable:
      Control: DataTable
```

#### 1.13.1 DataTable Decision Matrix

| Screen Type | Has CRUD Forms | Data Source | Layout | Recommendation |
|-------------|----------------|-------------|---------|----------------|
| **UserManagement** | ✅ Create/Edit/Delete | SharePoint | Tabular | **DataTable** ✅ |
| **RequestManagement** | ✅ Create/Edit/Approve | SharePoint | Tabular | **DataTable** ✅ |
| **OrderManagement** | ✅ Create/Edit/Delete | Database | Tabular | **DataTable** ✅ |
| **Dashboard Stats** | ❌ View only | API | Cards | **Gallery** ✅ |
| **Photo Gallery** | ❌ View only | Files | Grid | **Gallery** ✅ |
| **News Feed** | ❌ View only | API | Cards | **Gallery** ✅ |
| **Reports List** | ✅ Create/Edit | SharePoint | Tabular | **DataTable** ✅ |

#### 1.13.2 CRUD Pattern Recognition
**IDENTIFY** these patterns requiring DataTable:

```yaml
# PATTERN 1: Management Screen với Add/Edit/Delete
Screen có:
- Add/Create button
- Edit action trong rows  
- Delete confirmation
- Search/filter functionality
→ USE DataTable

# PATTERN 2: Admin/Configuration Screen  
Screen có:
- Settings management
- User role management  
- System configuration
→ USE DataTable

# PATTERN 3: Workflow Management
Screen có:
- Approval workflows
- Status updates
- Bulk operations
→ USE DataTable
```

---

## 2. CONTROL PROPERTIES RESTRICTIONS

### 2.0 Control Properties Restrictions - CRITICAL (Rule 10)
**MANDATORY**: Follow these universal control restrictions để prevent PA2108/PA2109 errors:

#### 2.0.1 Version Number Prohibition - ALL CONTROLS
**NEVER** include version numbers trong ANY control declarations:

```yaml
# ❌ FORBIDDEN - Causes PA2108/PA2109 errors
Control: DataTable@1.0.3        # Error: PA2108
Control: DataTableColumn@1.0.1   # Error: PA2109  
Control: GroupContainer@1.3.0    # Error: PA2108
Control: Classic/Button@2.2.0    # Error: PA2108
Control: Classic/TextInput@1.5.1 # Error: PA2108
Control: Rectangle@2.1.0         # Error: PA2108

# ✅ REQUIRED - Clean control names only
Control: DataTable
Control: DataTableColumn
Control: GroupContainer  
Control: Classic/Button
Control: Classic/TextInput
Control: Rectangle
```

#### 2.0.2 Universal Property Restrictions
**NEVER** use these properties on ANY control unless explicitly documented:

```yaml
# ❌ FORBIDDEN - Universal restrictions
Columns: =3          # Not supported on Form/FormViewer controls
Version: ="1.0"      # Never specify version properties
@version: ="1.0.3"   # Never use @ syntax with controls
Variant: ="Custom"   # Custom variants not supported (PA2109)
```

### 2.1 Rectangle Control - CẤM SỬ DỤNG
**Rectangle Control** - KHÔNG BAO GIỜ sử dụng những properties này:
- `RadiusBottomLeft` - Không được hỗ trợ
- `RadiusBottomRight` - Không được hỗ trợ  
- `RadiusTopLeft` - Không được hỗ trợ
- `RadiusTopRight` - Không được hỗ trợ
- `Variant` - Rectangle KHÔNG hỗ trợ Variant property
- `DropShadow` - Rectangle KHÔNG hỗ trợ DropShadow property
- `BorderRadius` - Rectangle KHÔNG hỗ trợ BorderRadius property

```yaml
# ❌ SAI - Rectangle với properties không hỗ trợ
- 'MyRectangle':
    Control: Rectangle
    Properties:
      RadiusTopLeft: =8        # PA2108 Error
      Variant: ManualLayout    # PA2108 Error
      DropShadow: =DropShadow.Light  # PA2108 Error

# ✅ ĐÚNG - Rectangle với properties hỗ trợ
- 'MyRectangle':
    Control: Rectangle
    Properties:
      Fill: =RGBA(255, 255, 255, 1)
      BorderColor: =RGBA(230, 230, 230, 1)
      BorderThickness: =2
```

### 2.2 Classic/Button Control - CẤM SỬ DỤNG
**Classic/Button Control** - KHÔNG BAO GIỜ sử dụng những properties này:
- `BorderRadius` - Không được hỗ trợ cho Classic/Button
- `Disabled` - Sử dụng `DisplayMode` thay thế
- `Align` - Không được hỗ trợ cho buttons
- `Children` - Classic/Button không hỗ trợ Children property 

```yaml
# ❌ SAI - Classic/Button với properties không hỗ trợ
- 'MyButton':
    Control: Classic/Button
    Properties:
      BorderRadius: =8           # PA2108 Error
      Disabled: =true            # PA2108 Error
      Align: =Align.Center       # PA2108 Error

# ✅ ĐÚNG - Classic/Button với properties hỗ trợ
- 'MyButton':
    Control: Classic/Button
    Properties:
      DisplayMode: =If(varIsDisabled, DisplayMode.Disabled, DisplayMode.Edit)
      Fill: =RGBA(0, 120, 212, 1)
      Color: =RGBA(255, 255, 255, 1)
```

### 2.3 Classic/TextInput Control - QUY TẮC ĐÚNG
**Classic/TextInput Control** - Sử dụng properties ĐÚNG:

```yaml
# ✅ ĐÚNG - Classic/TextInput properties
- 'MyTextInput':
    Control: Classic/TextInput
    Properties:
      Default: ="Default value"        # Sử dụng Default (KHÔNG phải Text)
      HintText: ="Enter text here"     # Sử dụng HintText (KHÔNG phải Placeholder)
      Mode: =TextMode.Password         # Sử dụng Mode (KHÔNG phải TextMode)

# ❌ SAI - Classic/TextInput với properties sai
- 'MyTextInput':
    Control: Classic/TextInput
    Properties:
      Text: ="Default value"          # PA2108 Error - sử dụng Default
      Placeholder: ="Enter text"      # PA2108 Error - sử dụng HintText
      TextMode: =TextMode.Password    # PA2108 Error - sử dụng Mode
```

### 2.4 GroupContainer Control - CẤM SỬ DỤNG
**GroupContainer Control** - KHÔNG BAO GIỜ sử dụng những properties này:
- `OnSelect` - GroupContainer không hỗ trợ event properties
- `OnClick` - GroupContainer không hỗ trợ event properties
- `BorderRadius` - GroupContainer không hỗ trợ BorderRadius
- `HoverFill` - GroupContainer không hỗ trợ hover states
- `HoverColor` - GroupContainer không hỗ trợ hover states
- `HoverBorderColor` - GroupContainer không hỗ trợ hover states
- `PressedFill` - GroupContainer không hỗ trợ pressed states
- `PressedColor` - GroupContainer không hỗ trợ pressed states
- `PressedBorderColor` - GroupContainer không hỗ trợ pressed states

```yaml
# ❌ SAI - GroupContainer với event properties
- 'MyContainer':
    Control: GroupContainer
    Properties:
      OnSelect: =Navigate(NextScreen)  # PA2108 Error
      BorderRadius: =8                 # PA2108 Error

# ✅ ĐÚNG - Sử dụng Rectangle cho interactive containers
- 'MyContainer':
    Control: Rectangle
    Properties:
      OnSelect: =Navigate(NextScreen)  # Rectangle hỗ trợ events
      Fill: =RGBA(255, 255, 255, 1)
```

### 2.5 Gallery Control - VARIANT RESTRICTIONS
**Gallery Control** - Restrictions theo Variant:

```yaml
# ❌ SAI - VariableHeight không hỗ trợ WrapCount
- 'MyGallery':
    Control: Gallery
    Variant: VariableHeight
    Properties:
      WrapCount: =7  # PA2108 Error cho VariableHeight

# ✅ ĐÚNG - WrapCount chỉ cho Horizontal/Vertical variants
- 'MyGallery':
    Control: Gallery
    Variant: Horizontal
    Properties:
      WrapCount: =7  # Hỗ trợ cho Horizontal/Vertical variants
      TemplateSize: =Self.Height / 5  # Bắt buộc cho tất cả Gallery variants
```

### 2.6 DataTable Control - RULES
**DataTable Control** - Quy tắc sử dụng:
- **ALWAYS** use as direct Screen children only
- **NEVER** nest inside GroupContainer or other containers
- **REQUIRED** properties: Items (only)
- **FORBIDDEN** properties: OnSelect, OnSelectionChange (causes PA2108 error)
- **VERSION NUMBERS**: Never use @1.0.3 syntax
- **CHILDREN**: Must contain DataTableColumn controls only
- **SELECTION HANDLING**: Use separate overlay Rectangle with OnSelect to detect DataTable.Selected

### 2.7 DataTableColumn Control - RULES  
**DataTableColumn Control** - Properties BẮT BUỘC:
- `FieldDisplayName` - ALWAYS required for user display
- `FieldName` - ALWAYS required for data binding
- `Order` - ALWAYS required for column sequence
- `Text` - ALWAYS required for content display
- `Variant` - ALWAYS specify (Textual, Number, DateTime, Boolean ONLY)

**DataTableColumn Control** - FORBIDDEN Properties:
- `Variant: Custom` - Causes PA2109 error, NOT supported
- `OnSelect` - DataTableColumn không hỗ trợ event properties

**DataTableColumn Control** - Reference Rules:
- **USE**: `ThisItem.fieldname` for data display only
- **NEVER USE**: Event properties trong DataTableColumn
- **CONTEXT**: DataTableColumn chỉ để hiển thị data, không handle events

### 2.8 Classic/TextInput Focus Properties - CRITICAL
**Classic/TextInput Control** - Focus Property Rules:

```yaml
# ❌ SAI - Classic/TextInput KHÔNG được hỗ trợ .Focus
- MyTextInput:
    Control: Classic/TextInput
    Properties:
      BorderColor: =If(Self.Focus, Colors.Primary, Colors.Border)  # PA2108 Error
      Fill: =If(Self.Focus, Colors.LightBlue, Colors.White)        # PA2108 Error

# ❌ SAI - Classic/TextInput KHÔNG được hỗ trợ .Focused
- MyTextInput:
    Control: Classic/TextInput
    Properties:
      BorderColor: =If(Self.Focused, Colors.Primary, Colors.Border)  # PA2108 Error

# ✅ ĐÚNG - Sử dụng FocusedBorderColor thay thế
- MyTextInput:
    Control: Classic/TextInput
    Properties:
      FocusedBorderColor: =Colors.Primary
      FocusedBorderThickness: =2
      BorderColor: =Colors.Border

# NOTE: Classic/TextInput KHÔNG hỗ trợ .Focus và KHÔNG hỗ trợ .Focused
```

**Classic/TextInput Control** - Validation Patterns:
```yaml
# ✅ ĐÚNG - Real-time validation KHÔNG sử dụng Focus state
BorderColor: =If(IsBlank(Self.Text), Colors.Border, 
    If(IsMatch(Self.Text, Email), Colors.Success, Colors.Error)
  )

# Visual validation KHÔNG sử dụng Focus detection
ValidationIcon.Visible: =Not(IsBlank(Self.Text))
ValidationIcon.Color: =If(IsBlank(Self.Text), Colors.Error, Colors.Success)
```

### 2.9 Input & Dropdown Focus Properties - CRITICAL
**Input và Dropdown Controls** - Focus Property Restrictions:

**CRITICAL**: TẤT CẢ Input controls (bao gồm cả "input" nói chung và Classic/TextInput) KHÔNG được hỗ trợ .Focus hoặc .Focused properties.

```yaml
# ❌ SAI - TẤT CẢ Input controls (bao gồm Classic/TextInput) KHÔNG có focus/focused properties
- MyTextInput:
    Control: Classic/TextInput
    Properties:
      BorderColor: =If(Self.Focus, Colors.Primary, Colors.Border)    # PA2108 Error
      Fill: =If(Self.Focused, Colors.LightBlue, Colors.White)        # PA2108 Error

- MyDropDown:
    Control: Classic/DropDown
    Properties:
      BorderColor: =If(Self.Focus, Colors.Primary, Colors.Border)    # PA2108 Error
      Fill: =If(Self.Focused, Colors.LightBlue, Colors.White)        # PA2108 Error

- MyComboBox:
    Control: Classic/ComboBox  
    Properties:
      BorderColor: =If(Self.Focus, Colors.Primary, Colors.Border)    # PA2108 Error

- MyDatePicker:
    Control: Classic/DatePicker
    Properties:
      Fill: =If(Self.Focused, Colors.LightBlue, Colors.White)        # PA2108 Error

# ✅ ĐÚNG - Sử dụng OnSelect để theo dõi và FocusedBorderColor/FocusedBorderThickness cho focus styling
- MyTextInput:
    Control: Classic/TextInput
    Properties:
      FocusedBorderColor: =Colors.Primary
      FocusedBorderThickness: =2
      BorderColor: =Colors.Border
      Fill: =Colors.White
    OnChange: |
      =Set(varTextInputChanged, true)

- MyDropDown:
    Control: Classic/DropDown
    Properties:
      FocusedBorderColor: =Colors.Primary
      FocusedBorderThickness: =2
      BorderColor: =Colors.Border
      Fill: =Colors.White
    OnSelect: |
      =Set(varDropdownSelected, true)
    
# ✅ ĐÚNG - Sử dụng FocusedBorderColor thay vì .Focus property
- MyComboBox:
    Control: Classic/ComboBox
    Properties:
      FocusedBorderColor: =Colors.Primary
      FocusedBorderThickness: =2
      HoverFill: =Colors.LightBlue
      PressedFill: =Colors.Primary
      BorderColor: =Colors.Border
    OnSelect: |
      =Set(varComboBoxSelected, true)

# ✅ ĐÚNG - DatePicker với focus styling
- MyDatePicker:
    Control: Classic/DatePicker
    Properties:
      FocusedBorderColor: =Colors.Primary
      FocusedBorderThickness: =2
      BorderColor: =Colors.Border
    OnSelect: |
      =Set(varDateSelected, true)

# NOTE: TẤT CẢ input controls (bao gồm Classic/TextInput và "input" nói chung) KHÔNG được hỗ trợ .Focus hoặc .Focused
# THAY THẾ: Sử dụng FocusedBorderColor + FocusedBorderThickness cho focus styling và OnSelect cho tracking
```

**Input Controls KHÔNG được hỗ trợ Focus/Focused Properties:**
- `Classic/TextInput` - KHÔNG được hỗ trợ .Focus hoặc .Focused
- `Classic/DropDown` - KHÔNG được hỗ trợ .Focus hoặc .Focused
- `Classic/ComboBox` - KHÔNG được hỗ trợ .Focus hoặc .Focused  
- `Classic/DatePicker` - KHÔNG được hỗ trợ .Focus hoặc .Focused
- `Classic/CheckBox` - KHÔNG được hỗ trợ .Focus hoặc .Focused
- `Classic/Radio` - KHÔNG được hỗ trợ .Focus hoặc .Focused
- `Classic/Toggle` - KHÔNG được hỗ trợ .Focus hoặc .Focused
- `Classic/Slider` - KHÔNG được hỗ trợ .Focus hoặc .Focused
- `Rating` - KHÔNG được hỗ trợ .Focus hoặc .Focused
- `PenInput` - KHÔNG được hỗ trợ .Focus hoặc .Focused
- **Tất cả "input" controls khác** - KHÔNG được hỗ trợ .Focus hoặc .Focused

**KHÔNG CÓ EXCEPTION:** TẤT CẢ Input controls bao gồm `Classic/TextInput` đều KHÔNG được hỗ trợ `.Focus` và `.Focused`.

**RECOMMENDED APPROACH cho Input/Dropdown Focus:**
- **Border Styling**: Sử dụng `FocusedBorderColor` và `FocusedBorderThickness` properties
- **Tracking**: Sử dụng `OnSelect` event để set variables cho state management
- **Alternative**: `HoverFill`, `PressedFill` cho visual feedback

### 2.10 Collection vs Text Function Rules - CRITICAL
**Collection Functions** - Proper Usage:

```yaml
# ✅ ĐÚNG - Concat() cho collections/tables
Items: |
  =Concat(
    Table({Name: "Tất cả"}),
    AddColumns(Department_1, "Name", name)
  )

# ✅ ĐÚNG - Concatenate() cho text strings  
Text: =Concatenate("Hello ", userName, "!")

# ❌ SAI - Concatenate() cho tables causes error
Items: |
  =Concatenate(
    Table({Name: "Tất cả"}),
    Department_1  # Invalid argument type error
  )

# ❌ SAI - Concat() cho text strings không optimal
Text: =Concat(Table({Value: "Hello"}), Table({Value: userName}))
```

**Function Selection Rules**:
- **Concatenate()** = Text function ONLY
- **Concat()** = Collection/Table function ONLY  
- **& operator** = Text concatenation alternative
- **Union()** = Table combination alternative

### 2.11 Table() Function Restrictions - CRITICAL
**ALWAYS AVOID Table()** - Use Collection Management Functions Instead:

```yaml
# ❌ SAI - Table() function tạo temporary data, không efficient
OnSelect: |
  =Set(varOptions, Table(
    {ID: 1, Name: "Option 1"},
    {ID: 2, Name: "Option 2"},
    {ID: 3, Name: "Option 3"}
  ))

# ✅ ĐÚNG - ClearCollect() cho collection initialization
OnSelect: |
  =ClearCollect(colOptions,
    {ID: 1, Name: "Option 1"},
    {ID: 2, Name: "Option 2"},
    {ID: 3, Name: "Option 3"}
  )

# ❌ SAI - Table() trong Items property
Items: =Table(
  {Value: "All Departments"},
  {Value: "IT"},
  {Value: "HR"}
)

# ✅ ĐÚNG - Sử dụng pre-populated collection
Items: =colDepartmentOptions  # Populated via ClearCollect trong OnStart
```

**Collection Management Priorities**:
1. **ClearCollect()** - Replace entire collection (preferred for initialization)
2. **Collect()** - Add records to existing collection
3. **Remove()/RemoveIf()** - Remove specific records
4. **Update()/UpdateIf()** - Modify existing records
5. **AVOID Table()** - Only use when absolutely necessary for single-use scenarios

**Performance Benefits của Collection Functions**:
```yaml
# ✅ ĐÚNG - Collections maintain state và performance
OnStart: |
  =Concurrent(
    ClearCollect(colDepartments, Department_1),
    ClearCollect(colUsers, User_1),
    ClearCollect(colStatuses, 
      {ID: 1, Name: "Active", Color: "Green"},
      {ID: 2, Name: "Inactive", Color: "Red"},
      {ID: 3, Name: "Pending", Color: "Orange"}
    )
  )

# Items references cached collections
MyDropdown.Items: =colDepartments
MyGallery.Items: =Filter(colUsers, Status = "Active")

# ❌ SAI - Table() recreated every time, poor performance
MyDropdown.Items: =Table({Name: "IT"}, {Name: "HR"}, {Name: "Finance"})
```

**Table() Function - Limited Use Cases Only**:
```yaml
# ✅ CHẤP NHẬN - Table() cho single-use lookup trong formulas
LookUp: =LookUp(
  Table({Min: 0, Max: 100, Grade: "A"}, {Min: 80, Max: 99, Grade: "B"}),
  score >= Min And score <= Max
).Grade

# ✅ CHẤP NHẬN - Table() trong complex calculations
Result: =Sum(
  Table({Factor: 1.2}, {Factor: 1.5}, {Factor: 0.8}),
  baseValue * Factor
)

# ❌ SAI - Table() cho dropdown options, gallery items, any UI data binding
```

---

## 3. CONTROL Z-INDEX & RENDERING ORDER

### 3.1 Critical Z-Index Rules
**BẮT BUỘC**: Controls được render theo thứ tự xuất hiện trong YAML. Controls sau xuất hiện **TRÊN TOP** của controls trước.

```yaml
# ✅ ĐÚNG - Thứ tự layering đúng
Children:
  # 1. BACKGROUND LAYERS (render trước, ở bottom)
  - Background:
      Control: Rectangle
      
  # 2. CONTAINER LAYERS 
  - MainContainer:
      Control: GroupContainer
      
  # 3. CONTENT LAYERS
  - ContentArea:
      Control: Rectangle
      Children:
        - TextLabel:
            Control: Label
        - InputField:
            Control: Classic/TextInput
            
  # 4. INTERACTIVE LAYERS
  - ActionButton:
      Control: Classic/Button
      
  # 5. OVERLAY LAYERS (render cuối, ở top)
  - HoverOverlay:
      Control: Rectangle
  - LoadingSpinner:
      Control: Classic/Icon
```

### 3.2 Mandatory Layer Hierarchy
**CRITICAL**: Luôn tuân thủ thứ tự này cho control declarations:

#### Level 1: Background & Foundation (Đầu tiên)
- Background controls - render ở bottom layer

#### Level 2: Layout Containers  
- Container controls - structural layout

#### Level 3: Static Content
- Static display elements

#### Level 4: Interactive Content
- Form inputs và data displays

#### Level 5: Action Controls
- Buttons và action triggers

#### Level 6: Status & Feedback (Cuối cùng)
- Status indicators và overlays - render ở top layer

---

## 4. CONTROL VARIANTS

### 4.1 GroupContainer Variants
```yaml
Variant: ManualLayout
Variant: AutoLayout
```

### 4.2 Gallery Variants
```yaml
Variant: Vertical
Variant: Horizontal
Variant: VariableHeight
```

### 4.3 Form Variants
```yaml
Variant: Classic
```

### 4.4 Arrow Variants
```yaml
Variant: BackArrow
```

### 4.5 Star Variants
```yaml
Variant: 5Points
Variant: 6Points
Variant: 8Points
Variant: 12Points
```

---

## 🚨 CRITICAL ERROR FIXES

### PA2108 Error Prevention

#### Rectangle với Event Properties
```yaml
# ❌ WRONG - Rectangle không hỗ trợ event properties như OnSelect
# ✅ CORRECT - Sử dụng Rectangle cho visual, GroupContainer cho layout, Classic/Button cho interaction
```

#### Gallery VariableHeight với WrapCount
```yaml
# ❌ WRONG - VariableHeight không hỗ trợ WrapCount
# ✅ CORRECT - Xóa WrapCount cho VariableHeight hoặc đổi variant
```

#### Classic/Button Invalid Properties
```yaml
# ❌ WRONG - Classic/Button không hỗ trợ BorderRadius, Disabled, Align
# ✅ CORRECT - Sử dụng DisplayMode thay Disabled, bỏ các properties không hỗ trợ
```

#### DataTable với Event Properties
```yaml
# ❌ WRONG - DataTable không hỗ trợ event properties như OnSelect
# ✅ CORRECT - Sử dụng GroupContainer cho layout, Classic/Button cho interaction
```

#### DataTable với Properties sai
```yaml
# ❌ WRONG - DataTable với properties sai
- 'MyDataTable':
    Control: DataTable
    Properties:
      Items: =MyDataSource
      OnSelect: |
        =Set(varSelectedItem, ThisItem)  # ERROR - ThisItem not available

# ✅ ĐÚNG - DataTable với properties đúng
- 'MyDataTable':
    Control: DataTable
    Properties:
      Items: =MyDataSource
      OnSelect: |
        =Set(varSelectedItem, Self.Selected)
```

---

**CRITICAL REMINDERS:**

70. **DATATABLE CRUD PRIORITIZATION** - CRITICAL: Always use DataTable when screen has Create/Edit/Delete forms or CRUD operations. Gallery is NOT suitable for data management screens with forms
71. **DATATABLE MANAGEMENT SCREENS** - CRITICAL: All "Management" pattern screens (UserManagement, RequestManagement, etc.) MUST use DataTable with proper Add/Edit/Delete functionality
72. **DATATABLE SCREEN CHILDREN** - CRITICAL: DataTable controls must be direct children of Screens only. Never nest inside GroupContainer or other containers
73. **DATATABLECOLUMN REQUIRED PROPERTIES** - CRITICAL: Every DataTableColumn must have FieldDisplayName, FieldName, Order, and Text properties. Missing any causes display errors
74. **DATATABLE ONSELECT FORBIDDEN** - CRITICAL: DataTable does NOT support OnSelect property (PA2108 error). Selection is handled automatically
75. **DATATABLECOLUMN CUSTOM VARIANT FORBIDDEN** - CRITICAL: DataTableColumn does NOT support Custom variant (PA2109 error). Use only Textual, Number, DateTime, Boolean
76. **DATATABLE VERSION RESTRICTIONS** - CRITICAL: Never use version numbers (@1.0.3) with DataTable or DataTableColumn controls
77. **DATATABLE PERFORMANCE FILTERING** - CRITICAL: Always filter data at source level before passing to DataTable Items property. Never rely on DataTable internal filtering for large datasets
78. **DATATABLECOLUMN DISPLAY ONLY** - CRITICAL: DataTableColumn is for data display only. No OnSelect or event properties. Use ThisItem.fieldname for data binding
79. **DATATABLE SELECTION HANDLING** - CRITICAL: DataTable does NOT support OnSelectionChange (PA2108 error). Use separate overlay Rectangle with OnSelect to detect DataTable.Selected for context menus
80. **CONTEXT MENU Z-INDEX** - CRITICAL: Context menus must be positioned LAST trong Children array để ensure highest z-index
81. **CLASSIC/TEXTINPUT FOCUS PROPERTY** - CRITICAL: Classic/TextInput chỉ hỗ trợ .Focus, KHÔNG hỗ trợ .Focused (PA2108 error)
82. **INPUT & DROPDOWN FOCUS RESTRICTIONS** - CRITICAL: TẤT CẢ Input controls (bao gồm "input" nói chung) KHÔNG có .Focus hoặc .Focused properties (PA2108 error), CHỈ NGOẠI TRỪ Classic/TextInput. SỬ DỤNG: FocusedBorderColor + FocusedBorderThickness cho styling, OnSelect cho tracking. Includes Classic/DropDown, Classic/ComboBox, Classic/DatePicker, Classic/CheckBox, Classic/Radio, Classic/Toggle, Classic/Slider, Rating, PenInput
83. **COLLECTION VS TEXT FUNCTIONS** - CRITICAL: Concatenate() cho text only, Concat() cho collections/tables only. Wrong usage causes invalid argument errors
84. **TABLE() FUNCTION RESTRICTIONS** - CRITICAL: ALWAYS AVOID Table() function. Use ClearCollect/Collect/Remove/Update instead for better performance và state management. Table() only acceptable for single-use lookups/calculations
85. **DATATABLE ONSELECTIONCHANGE PA2108 ERROR** - CRITICAL: DataTable does NOT support OnSelectionChange property (PA2108 error). Use separate Rectangle overlay with OnSelect positioned over DataTable for selection handling

---

**Agent phải tuân thủ những quy tắc controls này TUYỆT ĐỐI khi tạo Power App YAML code.**