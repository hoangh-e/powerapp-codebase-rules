# POWER APP COMPONENT RULES - QUY TẮC COMPONENTS

> **CRITICAL**: Quy tắc chi tiết về Component definitions, Custom Properties, và Component usage trong Power App.

---

## 📋 MỤC LỤC
1. [Component Definition Structure](#1-component-definition-structure)
2. [Custom Properties](#2-custom-properties)
3. [Component Sizing Rules](#3-component-sizing-rules)
4. [Component Integration](#4-component-integration)
5. [Event Handling](#5-event-handling)

---

## 1. COMPONENT DEFINITION STRUCTURE

### 1.1 Correct Component Structure - BẮT BUỘC
**CRITICAL**: Sử dụng cấu trúc component ĐÚNG:

```yaml
# ✅ ĐÚNG - Component Definition Structure mới
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
  CustomProperties:
    - UserRole:
        PropertyType: Data
        PropertyDataType: Text
        DefaultValue: ="Employee"
```

### 1.2 Component Usage in Screens
**CRITICAL**: Sử dụng syntax đúng cho component references:

```yaml
# ✅ ĐÚNG - Component Usage trong screens
- 'MyComponentInstance':
    Control: CanvasComponent
    ComponentName: HeaderComponent
    Properties:
      X: =0
      Y: =0
      Width: =Parent.Width
      Height: =Parent.Height * 0.1
      # Custom properties
      UserName: =varCurrentUser.Name
      ShowNotifications: =true

# ❌ SAI - Direct Component Reference
- 'MyHeaderComponent':
    Control: HeaderComponent  # SAI - không sử dụng trực tiếp
    Properties:
      UserName: =varCurrentUser.Name
```

---

## 2. CUSTOM PROPERTIES

### 2.1 Custom Property Structure - BẮT BUỘC
**CRITICAL**: Sử dụng property structure mới:

```yaml
# ✅ ĐÚNG - Custom Property Structure mới
CustomProperties:
  UserRole:
    PropertyKind: Input
    DisplayName: UserRole
    Description: "Vai trò của người dùng"
    DataType: Text
    Default: ="Employee"
  
  IsVisible:
    PropertyKind: Input
    DisplayName: IsVisible
    Description: "Hiển thị component hay không"
    DataType: Boolean
    Default: =true

# ❌ SAI - Cấu trúc cũ KHÔNG được hỗ trợ
CustomProperties:
  - UserRole:
      PropertyType: Data
      PropertyDataType: Text
      DefaultValue: ="Employee"
```

### 2.2 Property Kinds - CHỈ SỬ DỤNG
**CHỈ** những property kinds này được phép cho `PropertyKind`:

- `Input` - Input properties (phổ biến nhất)
- `Output` - Output properties  
- `Event` - Event properties

### 2.3 Data Types - CHỈ SỬ DỤNG
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

### 2.4 Event Properties Structure
**BẮT BUỘC**: Event properties PHẢI tuân thủ cấu trúc này:

```yaml
# ✅ ĐÚNG - Event Property Structure
OnNavigate:
  PropertyKind: Event
  DisplayName: OnNavigate
  Description: "Sự kiện khi chuyển màn hình"
  ReturnType: None
  Default: =
  Parameters:
    - ScreenName:
        Description: "Tên màn hình"
        DataType: Text
        IsOptional: true
        Default: ="Dashboard"

# ❌ SAI - Event Structure cũ
- OnNavigate:
    PropertyType: Event
    PropertyDataType: Text
    DefaultValue: =""
```

---

## 3. COMPONENT SIZING RULES

### 3.1 Component Proportional Sizing - BẮT BUỘC
**CRITICAL**: Components PHẢI sử dụng proportional sizing relative to App dimensions:

```yaml
# ✅ ĐÚNG - Proportional sizing phù hợp với component purpose
ComponentDefinitions:
  NavigationComponent:
    Properties:
      Height: =App.Height           # Full height OK cho navigation
      Width: =App.Width * 0.2       # 20% width cho side navigation
      
  HeaderComponent:
    Properties:
      Height: =App.Height * 0.08    # 8% height cho header
      Width: =App.Width             # Full width OK cho header
      
  ButtonComponent:
    Properties:
      Height: =App.Height * 0.06    # 6% height cho button
      Width: =App.Width * 0.15      # 15% width cho button
      
  CardComponent:
    Properties:
      Height: =App.Height * 0.3     # 30% height cho card
      Width: =App.Width * 0.45      # 45% width cho card

# ❌ SAI - Full App dimensions không phù hợp cho most components
ComponentDefinitions:
  NavigationComponent:
    Properties:
      Height: =App.Height  # Takes full height - sai cho navigation
      Width: =App.Width   # Takes full width - sai cho side navigation
```

### 3.2 Component Sizing Guidelines
**GUIDELINES** cho component sizing:

- **NavigationComponent**: Width = App.Width * 0.15-0.25 (side nav), App.Width (top nav)
- **HeaderComponent**: Height = App.Height * 0.06-0.1, Width = App.Width
- **ButtonComponent**: Height = App.Height * 0.045-0.065, Width = App.Width * 0.12-0.2
- **CardComponent**: Height = App.Height * 0.25-0.4, Width = App.Width * 0.3-0.5
- **InputComponent**: Height = App.Height * 0.05-0.07, Width = App.Width * 0.2-0.8
- **StatsCardComponent**: Height = App.Height * 0.15-0.25, Width = App.Width * 0.2-0.3

### 3.3 Parent-Fitting Rules
**CRITICAL**: Components PHẢI adapt to parent container context:

```yaml
# ✅ ĐÚNG - Component adapts to parent VÀ provides sensible defaults
ComponentDefinitions:
  ButtonComponent:
    Properties:
      Height: =App.Height * 0.06     # Default sizing cho app-level usage
      Width: =App.Width * 0.15       # Default sizing cho app-level usage
    Children:
      - Button.Container:
          Width: =If(Parent.Width > 0,
            Min(Parent.Width, Max(Parent.Width * 0.8, 120)),  # Adapt to parent
            Self.Width)  # Fallback to component default
          Height: =If(Parent.Height > 0,
            Min(Parent.Height, Max(Parent.Height * 0.8, 40)), # Adapt to parent
            Self.Height) # Fallback to component default

# ❌ SAI - Component luôn sử dụng App reference, bỏ qua parent context
ComponentDefinitions:
  ButtonComponent:
    Properties:
      Height: =App.Height * 0.06    # Ignores parent context
      Width: =App.Width * 0.15      # May not fit parent layout
```

---

## 4. COMPONENT INTEGRATION

### 4.1 Component Property Binding - BẮT BUỘC
**CRITICAL**: Khi sử dụng components trong screens, TẤT CẢ custom properties PHẢI được bound properly:

```yaml
# ✅ ĐÚNG - All custom properties được bind
- MyStatsCard:
    Control: CanvasComponent
    ComponentName: StatsCardComponent
    Properties:
      X: =0
      Y: =0
      Width: =Parent.Width / 4
      Height: =Parent.Height
      # Bind ALL custom properties từ component definition
      Title: ="Tổng số đơn"
      Value: =CountRows(MyDataSource)
      Icon: ="DocumentText"
      Color: =RGBA(59, 130, 246, 1)
      ShowTrend: =true
      TrendValue: =5.2

# ❌ SAI - Leaving component custom properties unset
- MyStatsCard:
    Control: CanvasComponent
    ComponentName: StatsCardComponent
    Properties:
      X: =0
      Y: =0
      # Missing Title, Value, Icon, etc. - causes display issues
```

### 4.2 Component Properties Optimization
**CRITICAL**: CHỈ include necessary properties để tránh "Expected operator" errors:

```yaml
# ✅ ĐÚNG - Chỉ essential properties
- MyStatsCard:
    Control: CanvasComponent
    ComponentName: StatsCardComponent
    Properties:
      X: =0
      Y: =0
      Width: =200
      Height: =Parent.Height
      Title: ="Status"
      Value: =42
      Icon: ="Clock"
      Color: =RGBA(251, 191, 36, 1)

# ❌ SAI - Too many unnecessary properties gây conflicts
- MyStatsCard:
    Control: CanvasComponent
    ComponentName: StatsCardComponent
    Properties:
      # ... nhiều properties không cần thiết có thể gây lỗi
      IsClickable: =false          # May not be needed
      MaxValue: =100              # May not be needed
      OnClick: =                  # Empty value có thể gây issues
      TrendText: =""             # Empty string có thể gây issues
```

---

## 5. EVENT HANDLING

### 5.1 Component Event Binding - CRITICAL RESTRICTION
**CRITICAL**: Component events PHẢI được bound to actions trong screens, KHÔNG call trực tiếp:

```yaml
# ✅ ĐÚNG - Binding actions to component events
- 'MyComponent':
    Control: CanvasComponent
    ComponentName: NavigationComponent
    Properties:
      OnNavigate: |
        =Set(varActiveScreen, "Dashboard")  # Bind action to event
      OnToggleCollapse: |
        =Set(varCollapsed, !varCollapsed)   # Bind action to event

# ❌ SAI - Calling component events trực tiếp từ screens
- 'MyComponent':
    Control: CanvasComponent
    ComponentName: NavigationComponent
    Properties:
      OnNavigate: ='MyComponent'.OnNavigate(); Set(varAction, true)    # ERROR
      OnToggleCollapse: ='MyComponent'.OnToggleCollapse()             # ERROR
```

### 5.2 Event Call Syntax Rules
**RULES** cho event handling:

- **In Screen**: Bind actions to component event properties using pipe operator (`|`)
- **In Component**: Call internal events using `ComponentName.EventName()` syntax  
- **NEVER**: Call component events directly từ screens với `ComponentInstance.EventName()` syntax
- **ALWAYS**: Use event binding pattern cho screen-to-component communication

### 5.3 Event Definition Best Practices
```yaml
# ✅ ĐÚNG - Complete event definition
OnCardClick:
  PropertyKind: Event
  DisplayName: OnCardClick
  Description: "Sự kiện khi click vào card"
  ReturnType: None
  Default: =
  Parameters:
    - CardId:
        Description: "ID của card được click"
        DataType: Text
        IsOptional: false
        Default: =""
    - CardData:
        Description: "Dữ liệu của card"
        DataType: Record
        IsOptional: true
        Default: ={}
```

---

## 🚨 COMPONENT CRITICAL ERRORS

### Component Event Call Error
**ERROR**: "Component behavior can only be invoked from within a component"

```yaml
# ❌ SAI - Calling component events directly
OnSelect: ='NavigationComponent'.OnNavigate(); Set(varAction, true)

# ✅ ĐÚNG - Binding actions to events
OnNavigate: |
  =Set(varAction, true)
```

### Component Property Binding Error
**ERROR**: Component display/behavior issues từ unbound properties

```yaml
# ❌ SAI - Missing required custom properties
- MyComponent:
    Control: CanvasComponent
    ComponentName: StatsCard
    Properties:
      X: =0  # Missing Title, Value, etc.

# ✅ ĐÚNG - All required properties bound
- MyComponent:
    Control: CanvasComponent  
    ComponentName: StatsCard
    Properties:
      X: =0
      Title: ="Sales"
      Value: =1234
      Icon: ="Money"
```

### Component Structure Error
**ERROR**: Using old component definition syntax

```yaml
# ❌ SAI - Old syntax
ComponentDefinition:
  CustomProperties:
    - UserRole:
        PropertyType: Data

# ✅ ĐÚNG - New syntax
ComponentDefinitions:
  ComponentName:
    CustomProperties:
      UserRole:
        PropertyKind: Input
```

---

**Agent phải tuân thủ những component rules này TUYỆT ĐỐI khi tạo Power App YAML code.**