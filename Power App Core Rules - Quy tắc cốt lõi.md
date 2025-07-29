# POWER APP CORE RULES - QUY TẮC CỐT LÕI

> **CRITICAL**: Agent PHẢI tuân thủ các quy tắc này TUYỆT ĐỐI khi tạo hoặc chỉnh sửa file YAML Power App.

---

## 📋 MỤC LỤC
1. [Cấu trúc File](#1-cấu-trúc-file)
2. [Quy tắc Control](#2-quy-tắc-control)
3. [Quy tắc Position & Size](#3-quy-tắc-position--size)
4. [Properties Guidelines](#4-properties-guidelines)
5. [Naming Conventions](#5-naming-conventions)
6. [Formula Rules](#6-formula-rules)

---

## 1. CẤU TRÚC FILE

### 1.1 Screens (Màn hình)
**BẮT BUỘC**: Tất cả file screen PHẢI bắt đầu với `Screens:`

```yaml
Screens:
  ScreenName:
    Properties:
      Fill: =RGBA(248, 250, 252, 1)
      OnVisible: =Set(varExample, "value")
    Children:
      - ControlName:
          Control: ControlType
          Properties:
            # Control properties here
```

### 1.2 Components (Thành phần)
**BẮT BUỘC**: Tất cả file component PHẢI sử dụng cấu trúc ĐÚNG:

```yaml
# ✅ ĐÚNG - Component Definition Structure
ComponentDefinitions:
  ComponentName:
    DefinitionType: CanvasComponent
    CustomProperties:
      PropertyName:
        PropertyKind: Input
        DisplayName: PropertyName
        Description: "Property description"
        DataType: Text
        Default: ="Default value"
    Properties:
      Height: =App.Height
      Width: =App.Width
    Children:
      - ControlName:
          Control: ControlType
          Properties:
            # Control properties here

# ❌ SAI - Cấu trúc cũ/không hợp lệ
ComponentDefinition:
  DefinitionType: CanvasComponent
```

### 1.3 Custom Properties Data Types
**CHỈ** những data types này được phép cho `DataType`:
- `Text` - String values
- `Number` - Numeric values  
- `Boolean` - True/false values
- `Date and time` - Date/time values
- `Screen` - Screen references
- `Record` - Single record
- `Table` - Data tables
- `Image` - Image data
- `Video or audio` - Media files
- `Color` - Color values
- `Currency` - Currency values

### 1.4 Custom Property Kinds
**CHỈ** những property kinds này được phép cho `PropertyKind`:
- `Input` - Input properties (phổ biến nhất)
- `Output` - Output properties
- `Event` - Event properties

---

## 2. QUY TẮC CONTROL

### 2.1 Version Restriction
**CẤM**: Không bao giờ bao gồm version numbers trong Control declarations

```yaml
# ✅ ĐÚNG
Control: GroupContainer

# ❌ SAI
Control: GroupContainer@1.3.0
```

### 2.2 Component Control Declarations
**CRITICAL**: Sử dụng syntax đúng cho component references

```yaml
# ✅ ĐÚNG - Component Usage
Control: CanvasComponent
ComponentName: HeaderComponent

# ❌ SAI - Direct Component Reference
Control: HeaderComponent
```

### 2.3 Forbidden Properties
**KHÔNG BAO GIỜ SỬ DỤNG** những properties này:
- `ZIndex` - Không được hỗ trợ
- `DropShadow` - Chỉ sử dụng khi chắc chắn control hỗ trợ

### 2.4 Geometric Controls Container Restrictions
**CRITICAL**: Các control hình học (Rectangle, Circle, Triangle, etc.) KHÔNG THỂ chứa controls khác như children.

```yaml
# ❌ SAI - Geometric controls với Children
- 'MyContainer':
    Control: Rectangle
    Children:        # ❌ SAI - Rectangle không thể có children
      - 'ChildControl':
          Control: Label

# ✅ ĐÚNG - Sử dụng geometric controls chỉ để trang trí
- 'Background.Shape':
    Control: Rectangle
    Properties:
      Fill: =RGBA(255, 255, 255, 1)
      # Không có Children property

# ✅ ĐÚNG - Sử dụng GroupContainer cho containers
- 'Content.Container':
    Control: GroupContainer
    Variant: ManualLayout
    Children:
      - 'Child.Control':
          Control: Label
```

### 2.5 Required Properties for All Controls
Mọi control PHẢI có những properties này khi applicable:
- `X` - Vị trí ngang (relative to parent)
- `Y` - Vị trí dọc (relative to parent)  
- `Width` - Chiều rộng control (relative to parent)
- `Height` - Chiều cao control (relative to parent)

---

## 3. QUY TẮC POSITION & SIZE

### 3.1 Width và Height - BẮT BUỘC RELATIVE POSITIONING
**KHÔNG BAO GIỜ** sử dụng absolute values. Luôn luôn tham chiếu parent hoặc controls khác.

```yaml
# ✅ ĐÚNG
Width: =Parent.Width * (3/2)
Height: =Parent.Height / 3
Width: =(Parent.Width - Control.Width) / 2
Height: =Self.Width

# ❌ SAI  
Width: 400
Height: 200
Width: =400
```

### 3.2 X và Y Positioning - BẮT BUỘC RELATIVE POSITIONING
**LUÔN LUÔN** position relative to parent hoặc controls khác.

```yaml
# ✅ ĐÚNG
X: =Parent.X /2
Y: =HeaderControl.Y + HeaderControl.Height
X: =(Parent.Width - Self.Width) / 2
Y: =Parent.Height - Self.Height 

# ❌ SAI
X: 100
Y: 50
```

### 3.3 Parent-Child Coordinate System - HIỂU BIẾT CRITICAL
**BẮT BUỘC**: Khi một control nằm trong parent container, tọa độ của nó là RELATIVE đến boundaries của parent.

```yaml
# ✅ ĐÚNG - Child control positioning bên trong parent
- 'Parent.Container':
    Control: Rectangle
    Properties:
      X: =100  # Screen position
      Y: =50   # Screen position
      Width: =400
      Height: =300
    Children:
      - 'Child.Control':
          Control: Label
          Properties:
            X: =0           # = Parent.X (100) - left edge của parent
            Y: =0           # = Parent.Y (50) - top edge của parent
            Width: =Parent.Width    # = 400 - full parent width
            Height: =Parent.Height  # = 300 - full parent height
```

### 3.4 Arithmetic Operations - TRÁNH FIXED NUMBERS
**CRITICAL**: Giảm thiểu việc sử dụng fixed numbers trong tính toán positioning/sizing. Ưu tiên tính toán relative.

```yaml
# ✅ ƯU TIÊN - Tính toán relative
Height: =Parent.Height / 2 - Control1.Height / 4
Width: =Parent.Width * 0.8 - SidePanel.Width
X: =Self.Width + Parent.X

# ⚠️ NẢN NÃ - Fixed numbers (sử dụng tiết kiệm)
Height: =Parent.Height / 2 - 4
Width: =Parent.Width - 20

# ❌ SAI - Pure fixed values
Height: 200
Width: 300
```

---

## 4. PROPERTIES GUIDELINES

### 4.1 Color Properties
**LUÔN LUÔN** sử dụng RGBA format:

```yaml
Fill: =RGBA(255, 255, 255, 1)
Color: =RGBA(32, 54, 71, 1)
BorderColor: =RGBA(230, 230, 230, 1)
```

### 4.2 Font Properties
```yaml
FontWeight: =FontWeight.Bold
FontWeight: =FontWeight.Semibold
FontWeight: =FontWeight.Normal
Font: =Font.'Open Sans'
```

### 4.3 Formula Properties
**TẤT CẢ** dynamic properties PHẢI bắt đầu với `=`:

```yaml
# ✅ ĐÚNG
Visible: =varShowControl
Text: =Concatenate("Hello ", varUserName)
Fill: =If(Self.IsHovered, RGBA(0, 120, 212, 1), RGBA(255, 255, 255, 1))

# ❌ SAI
Visible: varShowControl
Text: "Static text" # Chỉ sử dụng cho text thực sự static
```

---

## 5. NAMING CONVENTIONS

### 5.1 Special Character Handling
**BẮT BUỘC**: Wrap tên có special characters bằng single quotes:

```yaml
# ✅ ĐÚNG - Tên có special characters
- 'Header.Logo':
- 'Login Container':
- 'User-Info':
- 'Data View 1':
- 'My.Control.Name':

# ✅ ĐÚNG - Tên đơn giản (không cần quotes)
- HeaderLogo:
- LoginContainer:
- UserInfo:
```

### 5.2 Naming Best Practices
```yaml
# Sử dụng tên mô tả, phân cấp
- 'Header.UserInfo.Avatar':
- 'LoginForm.UsernameInput':
- 'Dashboard.StatsCard.Title':
- 'Navigation.MenuButton':
```

---

## 6. FORMULA RULES

### 6.1 YAML Syntax cho Action Properties
**CRITICAL**: Sử dụng pipe operator (`|`) cho TẤT CẢ action properties để đảm bảo tính nhất quán và tránh YAML parsing errors:

```yaml
# ✅ ĐÚNG - Luôn sử dụng pipe operator cho action properties
OnVisible: |
  =Set(varActiveScreen, "Dashboard"); Set(varCurrentUser, {Name: "User"})

OnSelect: |
  =Set(varIsLoading, true); Navigate(NextScreen)

OnChange: |
  =Set(varInputValue, Self.Text); UpdateContext({localVar: true})

# ❌ SAI - Single line action properties có thể gây YAML parsing issues
OnVisible: =Set(varActiveScreen, "Dashboard"); Set(varCurrentUser, {Name: "User"})
```

### 6.2 Text Formatting
**ĐÚNG** text concatenation formatting:

```yaml
# ✅ ĐÚNG - Proper spacing
Text: ="Email:" & " " & ThisItem.Email
Text: ="Status:" & " " & ThisItem.Status

# ❌ SAI - Space trong quotes
Text: ="Email: " & ThisItem.Email
Text: ="Status: " & ThisItem.Status
```

---

## 🚨 LỖI THƯỜNG GẶP CẦN TRÁNH

### Component Definition Structure Errors
```yaml
# ❌ SAI - Cấu trúc cũ
ComponentDefinition:
  DefinitionType: CanvasComponent
  CustomProperties:
    - UserRole:
        PropertyType: Data

# ✅ ĐÚNG - Cấu trúc mới
ComponentDefinitions:
  NavigationComponent:
    DefinitionType: CanvasComponent
    CustomProperties:
      UserRole:
        PropertyKind: Input
        DisplayName: UserRole
        DataType: Text
        Default: ="Employee"
```

### Absolute Positioning
```yaml
# ❌ SAI
X: 100
Y: 200
Width: 300

# ✅ ĐÚNG
X: =Parent.X + 20
Y: =HeaderControl.Y + HeaderControl.Height + 10
Width: =Parent.Width - 40
```

### Missing Required Properties
```yaml
# ✅ LUÔN bao gồm cho positioned controls
Properties:
  X: =Parent.X + 20
  Y: =Parent.Y + 10
  Width: =Parent.Width - 40
  Height: =40
```

---

**Agent phải tuân thủ những quy tắc này TUYỆT ĐỐI khi tạo Power App YAML code.**