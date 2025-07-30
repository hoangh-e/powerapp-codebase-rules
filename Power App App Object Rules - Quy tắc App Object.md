# POWER APP APP OBJECT RULES - QUY TẮC APP OBJECT

> **CRITICAL**: Quy tắc chi tiết về App Object structure, App properties, và Global variables/functions trong Power App.

---

## 📋 MỤC LỤC
1. [App Object Overview](#1-app-object-overview)
2. [App Folder Structure](#2-app-folder-structure)
3. [App Properties](#3-app-properties)
4. [Global Variables & Functions](#4-global-variables--functions)
5. [App Integration Rules](#5-app-integration-rules)

**NEW Global State Management:**
- [Global UI State Management](#45-global-ui-state-management)
- [Global Error and Loading States](#46-global-error-and-loading-states)

---

## 1. APP OBJECT OVERVIEW

### 1.1 App Object Concept
**CRITICAL**: App object là main application object trong Power Apps với specific properties điều khiển overall app behavior và appearance. Vì App source code không thể view trực tiếp, chúng ta organize App properties trong separate files trong `App` folder structure.

### 1.2 App vs Screen vs Component Hierarchy
```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│     APP     │───▶│   SCREENS   │───▶│ COMPONENTS  │
│             │    │             │    │             │
│ • Variables │    │ • Use App   │    │ • Receive   │
│ • Functions │    │   globals   │    │   data via  │
│ • Constants │    │ • Pass data │    │   properties│
│ • Styles    │    │   to comps  │    │ • Cannot    │
│             │    │             │    │   access    │
└─────────────┘    └─────────────┘    │   App       │
       ▲                   ▲          │   directly  │
       │                   │          └─────────────┘
       │                   │
    Isolated           Can Access
   from Components     App Globals
```

---

## 2. APP FOLDER STRUCTURE

### 2.1 Required App Folder Structure
**BẮT BUỘC**: Tạo App folder với individual files cho mỗi App property:

```
src/App/
├── BackEnabled
├── ConfirmExit
├── ConfirmExitMessage
├── Formulas
├── MinScreenHeight
├── MinScreenWidth
├── OnError
├── OnMessage
├── OnStart
├── SizeBreakpoints
├── StartScreen
└── Theme
```

### 2.2 App Property File Format
**CRITICAL**: Mỗi App property file chứa CHỈ formula/value cho property đó:

#### File: `src/App/Formulas`
```
//Global styles
Styles = {
  Title: {
    Font: Font.'Segoe UI',
    Size: 24,
    Weight: FontWeight.Semibold,
    Color: ColorValue("#0078D4"),
    Alignment: Align.Center
  },
  Button: {
    Font: Font.'Segoe UI',
    Size: 14,
    Weight: FontWeight.Normal,
    Color: ColorValue("#FFFFFF"),
    Fill: ColorValue("#0078D4")
  }
};

//Font sizes for consistency
FontSizes = {
  Base: 12,
  Small: 10,
  Medium: 14,
  Large: 16,
  XLarge: 18,
  XXLarge: 20
};

//Permission checking function
CheckPermission(UserPerm:Number, RequiredPerm:Number):Boolean = (
  Mod(Int(UserPerm / RequiredPerm), 2) = 1
);

//Current user object
UserInfo = {
  Name: "Demo User",
  Role: "Employee",
  Permission: 15,
  Department: "IT"
};
```

#### File: `src/App/OnStart`
```
Set(varAppVersion, "1.0.0");
Set(varCurrentUser, UserInfo);
Set(varAppInitialized, true)
```

#### File: `src/App/StartScreen`
```
DashboardScreen
```

---

## 3. APP PROPERTIES

### 3.1 Navigation Properties
```yaml
# BackEnabled - Controls back button behavior
BackEnabled: =true

# StartScreen - Initial screen when app starts
StartScreen: ="LoginPage"
```

### 3.2 Exit Confirmation Properties
```yaml
# ConfirmExit - Whether to show exit confirmation
ConfirmExit: =true

# ConfirmExitMessage - Custom exit confirmation message
ConfirmExitMessage: ="Bạn có chắc chắn muốn thoát ứng dụng?"
```

### 3.3 App Lifecycle Properties
```yaml
# OnStart - Code executed when app starts
OnStart: |
  =Set(varCurrentUser, UserInfo);
  Set(varAppInitialized, true)

# OnError - Error handling code
OnError: |
  =Notify("Đã xảy ra lỗi: " & FirstError.Message, NotificationType.Error)

# OnMessage - Message handling code
OnMessage: |
  =Switch(Message.MessageType, "Notification", Notify(Message.Text))
```

### 3.4 Screen Size Properties
```yaml
# MinScreenHeight - Minimum screen height
MinScreenHeight: =600

# MinScreenWidth - Minimum screen width  
MinScreenWidth: =800

# SizeBreakpoints - Responsive breakpoints
SizeBreakpoints: =Table(
  {Name: "Small", MinWidth: 0},
  {Name: "Medium", MinWidth: 768},
  {Name: "Large", MinWidth: 1024}
)
```

### 3.5 Theme Property
```yaml
# Theme - App theme settings
Theme: =Theme.Default
```

---

## 4. GLOBAL VARIABLES & FUNCTIONS

### 4.1 Formulas Organization - BẮT BUỘC
**ORGANIZE** global formulas theo logical groups:

```
//=== STYLING ===
Styles = {...};
FontSizes = {...};
Colors = {...};

//=== BUSINESS LOGIC ===
CheckPermission(...);
ValidateInput(...);
FormatCurrency(...);

//=== USER DATA ===
UserInfo = {...};
UserRoles = {...};
UserPermissions = {...};

//=== COMPUTED VALUES ===
UserHasAdminPerm = CheckPermission(...);
UserCanEdit = CheckPermission(...);
```

### 4.2 Font Size Management - CRITICAL
**CRITICAL**: Tất cả font sizes PHẢI được define globally trong `src/App/Formulas`:

```
//Font size hierarchy
FontSizes = {
  Base: 12,           // Base font size for normal content
  Small: 10,          // FontSizes.Base - 2 for captions, hints
  Medium: 14,         // FontSizes.Base + 2 for labels, inputs  
  Large: 16,          // FontSizes.Base + 4 for section headers
  XLarge: 18,         // FontSizes.Base + 6 for page titles
  XXLarge: 20         // FontSizes.Base + 8 for main headers
};

//Typography scale for consistency
Typography = {
  Caption: {Size: FontSizes.Small, Weight: FontWeight.Normal},
  Body: {Size: FontSizes.Base, Weight: FontWeight.Normal},
  Label: {Size: FontSizes.Medium, Weight: FontWeight.Semibold},
  Heading3: {Size: FontSizes.Large, Weight: FontWeight.Bold},
  Heading2: {Size: FontSizes.XLarge, Weight: FontWeight.Bold},
  Heading1: {Size: FontSizes.XXLarge, Weight: FontWeight.Bold}
};
```

### 4.3 Global Function Rules
**CRITICAL**: App/Formulas KHÔNG hỗ trợ complex function definitions với multiple statements:

```
# ✅ ĐÚNG - Simple expressions only
CheckPermission(UserPerm:Number, RequiredPerm:Number):Boolean = (
  Mod(Int(UserPerm / RequiredPerm), 2) = 1
);

ValidateEmail(Email:Text):Boolean = (
  And(Not(IsBlank(Email)), Contains(Email, "@"))
);

# ❌ SAI - Complex function definitions không được hỗ trợ
NavigateToScreen(screenName:Text, param:Text):Boolean = (
  If(screenName = "Dashboard", Set(varScreen, "Dashboard"); true, false)
);
```

### 4.4 Global Variable Naming - CRITICAL
**CRITICAL**: Tránh naming global variables giống như external data source names:

```
# ✅ ĐÚNG - Use different names để tránh conflicts
UserRoles = {
  Admin: "Administrator", 
  Employee: "Employee"
};

UserPermissions = {
  View: 1,
  Create: 2,
  Edit: 4,
  Delete: 8
};

AppSettings = {
  Version: "1.0.0",
  Environment: "Production"
};

# ❌ SAI - Conflicts với potential data source names
VaiTro = {...};  // Có thể conflict với external data source
Quyen = {...};   // Có thể conflict với external data source
```

### 4.5 Global UI State Management - CRITICAL
**CRITICAL**: Define global UI states và loading patterns trong App/Formulas:

#### 4.5.1 Loading States Definition
```
//=== LOADING STATES ===
LoadingStates = {
  Button: {
    Text: "Đang tải...",
    DisabledText: "Vui lòng đợi...",
    LoadingIcon: Icon.Loading
  },
  Screen: {
    Message: "Đang tải dữ liệu...",
    Overlay: RGBA(255, 255, 255, 0.8),
    SpinnerColor: RGBA(0, 120, 212, 1)
  },
  Form: {
    Submitting: "Đang lưu...",
    Validating: "Đang kiểm tra...",
    Processing: "Đang xử lý..."
  }
};

//Usage trong screens:
//Text: =If(varIsLoading, LoadingStates.Button.Text, "Submit")
//Fill: =LoadingStates.Screen.Overlay
```

#### 4.5.2 Error Messages Definition
```
//=== ERROR MESSAGES ===
ErrorMessages = {
  Network: "Lỗi kết nối mạng. Vui lòng kiểm tra kết nối và thử lại.",
  Validation: "Vui lòng kiểm tra thông tin nhập và thử lại.",
  Permission: "Bạn không có quyền thực hiện thao tác này.",
  NotFound: "Không tìm thấy dữ liệu yêu cầu.",
  Timeout: "Yêu cầu đã hết thời gian chờ. Vui lòng thử lại.",
  Duplicate: "Dữ liệu đã tồn tại trong hệ thống.",
  Required: "Vui lòng điền đầy đủ thông tin bắt buộc.",
  InvalidFormat: "Định dạng dữ liệu không hợp lệ.",
  General: "Có lỗi xảy ra. Vui lòng thử lại hoặc liên hệ quản trị viên."
};

//Usage trong screens:
//Text: =ErrorMessages.Network  
//Notify(ErrorMessages.Validation, NotificationType.Error)
```

#### 4.5.3 Success Messages Definition
```
//=== SUCCESS MESSAGES ===
SuccessMessages = {
  Created: "Đã tạo mới thành công!",
  Updated: "Đã cập nhật thành công!",
  Deleted: "Đã xóa thành công!",
  Saved: "Đã lưu thành công!",
  Uploaded: "Đã tải lên thành công!",
  Sent: "Đã gửi thành công!",
  Approved: "Đã phê duyệt thành công!",
  Completed: "Đã hoàn thành thành công!"
};

//Usage trong screens:
//Notify(SuccessMessages.Created, NotificationType.Success)
```

### 4.6 Global Error and Loading States - CRITICAL
**CRITICAL**: Centralized error và loading state management:

#### 4.6.1 Global Loading Pattern
```
//=== GLOBAL LOADING MANAGEMENT ===
GlobalLoadingManager = {
  SetLoading: (screenName: Text, isLoading: Boolean) => {
    UpdateContext({
      ["varLoading_" & screenName]: isLoading
    })
  },
  IsLoading: (screenName: Text) => {
    ["varLoading_" & screenName]
  }
};

//Usage:
//GlobalLoadingManager.SetLoading("UserManagement", true)
//If(GlobalLoadingManager.IsLoading("UserManagement"), "Loading...", "Ready")
```

#### 4.6.2 Global Error State Pattern
```
//=== GLOBAL ERROR MANAGEMENT ===
GlobalErrorManager = {
  SetError: (errorType: Text, message: Text) => {
    Set(varGlobalError, {
      Type: errorType,
      Message: message,
      Timestamp: Now(),
      Screen: App.ActiveScreen.Name
    })
  },
  ClearError: () => {
    Set(varGlobalError, Blank())
  },
  HasError: () => {
    Not(IsBlank(varGlobalError))
  }
};

//Usage trong OnError:
//GlobalErrorManager.SetError("Network", ErrorMessages.Network)
```

#### 4.6.3 Modal State Management
```
//=== MODAL STATE MANAGEMENT ===
ModalStates = {
  UserForm: "varShowUserForm",
  DeleteConfirm: "varShowDeleteConfirm", 
  DetailView: "varShowDetailView",
  Settings: "varShowSettings"
};

//Modal helper functions
ShowModal = (modalName: Text) => {
  Set(modalName, true)
};

HideModal = (modalName: Text) => {
  Set(modalName, false)
};

//Usage:
//ShowModal(ModalStates.UserForm)
//HideModal(ModalStates.DeleteConfirm)
```

---

## 5. APP INTEGRATION RULES

### 5.1 Screen Development Workflow - MANDATORY App Check
**CRITICAL**: Trước khi modify bất kỳ screen nào, LUÔN check `src/App/` folder trước:

**WORKFLOW cho Screen Development:**
1. **FIRST**: Check `src/App/Formulas` cho available global variables và functions
2. **SECOND**: Check `src/App/OnStart` cho initialization context
3. **THIRD**: Use existing App resources trước khi tạo new local variables
4. **FOURTH**: Implement screen logic using App globals when appropriate

```yaml
# ✅ ĐÚNG - Check App first, use existing globals
# In screen OnVisible - use existing App globals
OnVisible: |
  =Set(varScreenTitle, "Dashboard");
  Set(varUserCanEdit, CheckPermission(UserInfo.Permission, UserPermissions.Edit));
  Set(varCurrentUserRole, UserInfo.Role)

# In control properties - reference App styles và variables  
Size: =FontSizes.Medium
Fill: =Styles.Button.Fill
Color: =Styles.Title.Color
Visible: =UserHasAdminPerm

# ❌ SAI - Creating duplicate variables without checking App
OnVisible: |
  =Set(varUserInfo, {Name: "John", Role: "Admin"});  # UserInfo already exists in App
  Set(varFontSize, 14);  # FontSizes already exists in App
```

### 5.2 App-Component Isolation Rules - CRITICAL RESTRICTION
**CRITICAL**: App global variables và functions CHỈ accessible trong App và Screens. Custom Components KHÔNG THỂ access App globals trực tiếp.

**ISOLATION RULES:**
- **App Variables**: `UserInfo`, `Styles`, `FontSizes`, `UserPermissions` → NOT accessible in Components
- **App Functions**: `CheckPermission()`, `ValidateEmail()`, etc. → NOT accessible in Components  
- **App Constants**: `UserHasAdminPerm`, `UserCanEdit`, etc. → NOT accessible in Components

```yaml
# ❌ SAI - Component trying to access App globals directly
ComponentDefinitions:
  MyComponent:
    CustomProperties:
      UserRole:
        PropertyKind: Input
        DataType: Text
        Default: =UserInfo.Role  # ❌ ERROR - Cannot access App.UserInfo in Component
    Children:
      - MyButton:
          Control: Classic/Button
          Properties:
            Visible: =UserHasAdminPerm  # ❌ ERROR - Cannot access App variable in Component
            Size: =FontSizes.Medium     # ❌ ERROR - Cannot access App FontSizes in Component

# ✅ ĐÚNG - Pass App data to Component via custom properties
# In Screen:
- MyComponentInstance:
    Control: CanvasComponent
    ComponentName: MyComponent
    Properties:
      UserRole: =UserInfo.Role              # Pass App data as property
      HasPermission: =UserHasAdminPerm      # Pass App computed value as property
      ButtonStyle: =Styles.Button           # Pass App style object as property
      FontSize: =FontSizes.Medium           # Pass App font size as property
```

### 5.3 Component Data Requirements
**ALWAYS** pass required App data to Components explicitly:

```yaml
# ✅ ĐÚNG - Explicit data passing pattern
- NavigationComponent:
    Control: CanvasComponent
    ComponentName: NavigationComponent
    Properties:
      CurrentUser: =UserInfo                     # Pass App user data
      UserPermissions: ={                        # Pass App permission checks
        CanCreate: UserHasCreatePerm,
        CanEdit: UserHasEditPerm,
        CanDelete: UserHasDeletePerm
      }
      AppStyles: =Styles                         # Pass App styling
      FontSizes: =FontSizes                      # Pass App font sizes
```

### 5.4 OnStart Optimization
**OPTIMIZE** OnStart cho fast app loading:

```yaml
# ✅ ĐÚNG - Lightweight initialization
OnStart: |
  =Set(varAppVersion, "1.0.0");
  Set(varCurrentUser, UserInfo);
  Set(varAppInitialized, true)

# ❌ SAI - Heavy data loading trong OnStart
OnStart: |
  =Set(varAllUsers, AllUsersDataSource);
  Set(varAllDepartments, DepartmentsDataSource);
  Set(varAllProjects, ProjectsDataSource)
```

### 5.5 Error Handling
**IMPLEMENT** comprehensive error handling:

```yaml
# File: src/App/OnError
Switch(FirstError.Kind,
  ErrorKind.Network, Notify("Lỗi kết nối mạng. Vui lòng kiểm tra kết nối.", NotificationType.Error),
  ErrorKind.Sync, Notify("Lỗi đồng bộ dữ liệu. Vui lòng thử lại.", NotificationType.Warning),
  ErrorKind.Validation, Notify("Dữ liệu không hợp lệ: " & FirstError.Message, NotificationType.Error),
  Notify("Đã xảy ra lỗi: " & FirstError.Message, NotificationType.Error)
);
Set(varLastError, {
  Message: FirstError.Message,
  Kind: FirstError.Kind,
  Timestamp: Now()
})
```

---

## 🚨 APP OBJECT CRITICAL ERRORS

### App Formulas Function Definition Error
**ERROR**: "Function definition statements are not supported in this context"

```yaml
# ❌ SAI - Complex function definitions trong App/Formulas
NavigateToScreen(screenName:Text, param:Text):Boolean = (
  If(screenName = "Dashboard", Set(varScreen, "Dashboard"); true, false)
);

# ✅ ĐÚNG - Simple expressions only
CheckPermission(UserPerm:Number, RequiredPerm:Number):Boolean = (
  Mod(Int(UserPerm / RequiredPerm), 2) = 1
);
```

### App Variable Name Conflicts
**ERROR**: "ConnectedDataSource already contains a definition"

```yaml
# ❌ SAI - Conflicts với external data source names
VaiTro = {...};  // Conflicts với potential SharePoint table
Quyen = {...};   // Conflicts với potential SharePoint table

# ✅ ĐÚNG - Use different names
UserRoles = {...};
UserPermissions = {...};
```

### Component Access App Globals Error
**ERROR**: App globals not accessible trong Components

```yaml
# ❌ SAI - Component accessing App globals directly
Properties:
  Size: =FontSizes.Medium      # ERROR trong Component
  Fill: =Styles.Button.Fill    # ERROR trong Component

# ✅ ĐÚNG - Pass via custom properties
# Screen passes to Component:
Properties:
  ComponentFontSize: =FontSizes.Medium
  ComponentStyle: =Styles.Button
```

---

## 🚨 APP PROPERTY RULES

### App Property Rules
**CRITICAL**: Tuân thủ những rules này khi working với App properties:

#### Global Variables và Functions trong Formulas
- **ALWAYS** define global variables trong `Formulas` property
- **ALWAYS** define reusable functions trong `Formulas` property
- **ALWAYS** define global styles và constants trong `Formulas` property
- **NEVER** define variables trong individual screen OnVisible events nếu chúng được sử dụng globally

#### OnStart Property Rules
- **ALWAYS** use `OnStart` cho app initialization logic
- **ALWAYS** use `OnStart` cho user authentication setup
- **NEVER** put heavy data loading trong `OnStart` - use lazy loading instead

#### StartScreen Property Rules
- **ALWAYS** specify initial screen name as plain text
- **NEVER** use formulas trong `StartScreen` property
- **ALWAYS** ensure specified screen exists trong app

#### Global Function Naming Conventions
```yaml
# ✅ ĐÚNG - Descriptive function names
CheckPermission(UserPerm:Number, RequiredPerm:Number):Boolean
ValidateEmail(Email:Text):Boolean
FormatCurrency(Amount:Number):Text

# ❌ SAI - Generic hoặc unclear names
Check(a:Number, b:Number):Boolean
Validate(z:Text):Boolean
Format(y:Number):Text
```

---

**Agent phải tuân thủ những App Object rules này TUYỆT ĐỐI khi tạo Power App YAML code.**