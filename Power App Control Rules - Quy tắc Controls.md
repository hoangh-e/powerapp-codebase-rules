# POWER APP CONTROL RULES - QUY TẮC CONTROLS

> **CRITICAL**: Quy tắc chi tiết về các controls được phép và cách sử dụng chúng trong Power App YAML.

---

## 📋 MỤC LỤC
1. [Allowed Controls](#1-allowed-controls)
2. [Control Properties Restrictions](#2-control-properties-restrictions)
3. [Control Z-Index & Rendering Order](#3-control-z-index--rendering-order)
4. [Control Variants](#4-control-variants)

---

## 1. ALLOWED CONTROLS

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
```yaml
Control: BarChart
Control: LineChart
Control: PieChart
Control: Legend
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

---

## 2. CONTROL PROPERTIES RESTRICTIONS

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

---

**Agent phải tuân thủ những quy tắc controls này TUYỆT ĐỐI khi tạo Power App YAML code.**