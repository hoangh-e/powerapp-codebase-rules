# POWER APP BEST PRACTICES - THỰC HÀNH TỐT NHẤT

> **CRITICAL**: Best practices và optimization guidelines cho Power App development để đảm bảo performance, maintainability, và user experience.

---

## 📋 MỤC LỤC
1. [Performance Optimization](#1-performance-optimization)
2. [Code Organization](#2-code-organization)
3. [UI/UX Best Practices](#3-uiux-best-practices)
4. [Data Management](#4-data-management)
5. [Error Handling](#5-error-handling)
6. [Testing & Debugging](#6-testing--debugging)

---

## 1. PERFORMANCE OPTIMIZATION

### 1.1 Relative Calculations Best Practices
**CRITICAL**: Giảm thiểu fixed numbers trong favor của relative calculations:

```yaml
# ✅ EXCELLENT - Fully relative
X: =Parent.X + Parent.Width * 0.05
Y: =HeaderControl.Y + HeaderControl.Height * 1.2
Width: =Parent.Width * 0.9
Height: =Parent.Height / 3 - SidePanel.Height / 4

# ✅ GOOD - Mostly relative với minimal fixed numbers
X: =Parent.X + 5
Y: =HeaderControl.Y + HeaderControl.Height + 2
Width: =Parent.Width - 10
Height: =Parent.Height / 3

# ⚠️ ACCEPTABLE - Some fixed numbers cho standard spacing
X: =Parent.X + 20
Width: =Parent.Width - 40
Size: =14

# ❌ POOR - Too many fixed numbers
X: =Parent.X + 50
Y: =100
Width: =Parent.Width - 80
Height: =200
```

### 1.2 Calculation Preferences (theo thứ tự)
1. **Percentage/Ratio calculations**: `Width: =Parent.Width * 0.8`
2. **Relative to other controls**: `Y: =Control1.Y + Control1.Height`
3. **Mathematical relationships**: `Height: =Self.Width / 2`
4. **Small fixed adjustments**: `X: =Parent.X + 5`
5. **Standard fixed values**: `Size: =14` (fonts, borders)

### 1.3 OnStart Optimization
**CRITICAL**: Keep OnStart lightweight cho fast app loading:

```yaml
# ✅ ĐÚNG - Lightweight initialization
OnStart: |
  =Set(varAppVersion, "1.0.0");
  Set(varCurrentUser, {Name: "Demo User", Role: "Employee"});
  Set(varAppInitialized, true)

# ❌ SAI - Heavy operations trong OnStart
OnStart: |
  =Set(varAllData, LargeDataSource);
  Set(varProcessedData, ForAll(LargeDataSource, ComplexCalculation));
  Set(varCachedResults, ExpensiveOperation())
```

### 1.4 Lazy Loading Pattern
**IMPLEMENT** lazy loading cho better performance:

```yaml
# ✅ ĐÚNG - Load data khi cần
OnVisible: |
  =If(IsBlank(varScreenData), 
    Set(varIsLoading, true);
    Set(varScreenData, Filter(DataSource, Condition));
    Set(varIsLoading, false)
  )

# ❌ SAI - Load tất cả data upfront
OnStart: |
  =Set(varAllScreensData, {
    Screen1: DataSource1,
    Screen2: DataSource2,
    Screen3: DataSource3
  })
```

---

## 2. CODE ORGANIZATION

### 2.1 Property Organization
**ORGANIZE** properties theo thứ tự này:

```yaml
Properties:
  # 1. Position (X, Y)
  X: =Parent.X + 20
  Y: =Parent.Y + 10
  
  # 2. Size (Width, Height)
  Width: =Parent.Width - 40
  Height: =40
  
  # 3. Visual (Fill, Color, BorderColor)
  Fill: =RGBA(255, 255, 255, 1)
  Color: =RGBA(32, 54, 71, 1)
  BorderColor: =RGBA(230, 230, 230, 1)
  
  # 4. Behavior (OnSelect, OnChange)
  OnSelect: |
    =Navigate(NextScreen)
  OnChange: |
    =Set(varInputValue, Self.Text)
  
  # 5. Content (Text, Items)
  Text: ="Click me"
  Items: =MyCollection
  
  # 6. Other properties
  FontWeight: =FontWeight.Semibold
  Visible: =varShowControl
```

### 2.2 Naming Conventions Best Practices
```yaml
# ✅ ĐÚNG - Descriptive, hierarchical names
- 'Header.UserInfo.Avatar':
    Control: Image
- 'LoginForm.UsernameInput':
    Control: Classic/TextInput
- 'Dashboard.StatsCard.Title':
    Control: Label
- 'Navigation.MenuButton':
    Control: Classic/Button

# ✅ ĐÚNG - Component naming
ComponentDefinitions:
  HeaderComponent:
  NavigationComponent:
  StatsCardComponent:
  LoginFormComponent:

# ✅ ĐÚNG - Variable naming
varCurrentUser        # Current user data
varIsLoading         # Loading state
varSelectedItem      # Selected item
colMenuItems         # Collection of menu items
colFilteredData      # Filtered data collection
```

### 2.3 Collection vs Variable Naming
**CRITICAL**: Sử dụng naming conventions đúng:

```yaml
# ✅ ĐÚNG - Collection naming với 'col' prefix
ClearCollect(colAllUsers, ...)
ClearCollect(colMenuItems, ...)
ClearCollect(colWorkflowData, ...)
ClearCollect(colFilteredResults, ...)

# ✅ ĐÚNG - Variable naming với 'var' prefix cho simple variables
Set(varCurrentUser, ...)
Set(varIsLoading, ...)
Set(varSelectedItem, ...)
Set(varErrorMessage, ...)

# ❌ SAI - Confusion giữa collections và variables
Set(AllUsers, Table(...))           # Nên là ClearCollect(colAllUsers, ...)
ClearCollect(CurrentUser, ...)      # Nên là Set(varCurrentUser, ...)
```

### 2.4 File Structure Organization
```
src/
├── App/
│   ├── Formulas              # Global variables, functions, styles
│   ├── OnStart               # App initialization
│   ├── StartScreen           # Initial screen
│   └── OnError               # Error handling
├── Screens/
│   ├── DashboardScreen.yaml
│   ├── LoginScreen.yaml
│   └── SettingsScreen.yaml
└── Components/
    ├── HeaderComponent.yaml
    ├── NavigationComponent.yaml
    └── StatsCardComponent.yaml
```

---

## 3. UI/UX BEST PRACTICES

### 3.1 Responsive Design Principles
**IMPLEMENT** responsive design từ đầu:

```yaml
# ✅ ĐÚNG - Responsive sizing
Width: =If(Parent.Width < 600, 
  Parent.Width * 0.95,           # Mobile: take most width
  Min(Parent.Width * 0.7, 800)   # Desktop: limited width
)

Height: =If(Parent.Height < 400,
  Parent.Height * 0.8,           # Small screen: take most height
  Max(Parent.Height * 0.6, 300)  # Large screen: minimum height
)

# ✅ ĐÚNG - Responsive font sizes
Size: =If(Parent.Width < 600, FontSizes.Small, FontSizes.Medium)

# ✅ ĐÚNG - Responsive layout
X: =If(Parent.Width < 768,
  (Parent.Width - Self.Width) / 2,  # Center on mobile
  Parent.Width * 0.1                # Left margin on desktop
)
```

### 3.2 Accessibility Best Practices
**ENSURE** accessibility cho all users:

```yaml
# ✅ ĐÚNG - Proper contrast ratios
Color: =RGBA(32, 54, 71, 1)      # Dark text on light background
Fill: =RGBA(255, 255, 255, 1)    # Light background

# ✅ ĐÚNG - Focus indicators
BorderColor: =If(Self.Focus, 
  RGBA(0, 120, 212, 1),          # Blue focus border
  RGBA(200, 200, 200, 1)         # Gray normal border
)

# ✅ ĐÚNG - Meaningful text
Text: ="Submit Application"       # Clear action
HintText: ="Enter your email address"  # Helpful placeholder

# ✅ ĐÚNG - Touch-friendly sizing
Height: =Max(44, Parent.Height * 0.08)  # Minimum 44px for touch targets
```

### 3.3 Loading States
**PROVIDE** clear loading feedback:

```yaml
# ✅ ĐÚNG - Loading state management
OnSelect: |
  =Set(varIsLoading, true);
  // Perform operation
  Set(varIsLoading, false)

# Loading indicator
Visible: =varIsLoading

# Disabled state during loading
DisplayMode: =If(varIsLoading, DisplayMode.Disabled, DisplayMode.Edit)

# Loading text
Text: =If(varIsLoading, "Loading...", "Submit")
```

### 3.4 Color Consistency
**MAINTAIN** consistent color scheme:

```yaml
# Define trong App/Formulas
AppColors = {
  Primary: RGBA(0, 120, 212, 1),
  Secondary: RGBA(107, 114, 126, 1),
  Success: RGBA(16, 124, 16, 1),
  Warning: RGBA(255, 193, 7, 1),
  Danger: RGBA(220, 53, 69, 1),
  Light: RGBA(248, 249, 250, 1),
  Dark: RGBA(33, 37, 41, 1)
};

# Sử dụng trong screens
Fill: =AppColors.Primary
Color: =AppColors.Light
BorderColor: =AppColors.Secondary
```

---

## 4. DATA MANAGEMENT

### 4.1 Data Loading Strategies
**IMPLEMENT** efficient data loading:

```yaml
# ✅ ĐÚNG - Filter at source
Items: =Filter(LargeDataSource, 
  And(
    Status = "Active",
    CreatedDate >= DateAdd(Today(), -30, Days)
  )
)

# ✅ ĐÚNG - Pagination for large datasets
Items: =FirstN(FilteredData, 50)

# ✅ ĐÚNG - Cache frequently used data
OnVisible: |
  =If(IsBlank(varCachedUsers),
    Set(varCachedUsers, Users);
  )

# ❌ SAI - Load all data unnecessarily
Items: =AllDataSource  # Could be huge
```

### 4.2 State Management
**ORGANIZE** state management effectively:

```yaml
# ✅ ĐÚNG - Global state trong App/Formulas
CurrentUser = {
  Name: "Demo User",
  Role: "Employee",
  Permissions: 15
};

# ✅ ĐÚNG - Screen-level state
OnVisible: |
  =Set(varScreenState, {
    IsLoading: false,
    ErrorMessage: "",
    SelectedItems: [],
    FilterText: ""
  })

# ✅ ĐÚNG - Component-level state
OnSelect: |
  =UpdateContext({
    localExpanded: !localExpanded,
    localHighlighted: true
  })
```

### 4.3 Data Validation
**IMPLEMENT** comprehensive validation:

```yaml
# ✅ ĐÚNG - Input validation
OnChange: |
  =Set(varValidationErrors, {
    Email: If(And(Not(IsBlank(Self.Text)), Not(IsMatch(Self.Text, Email))), 
      "Invalid email format", ""),
    Required: If(IsBlank(Self.Text), "This field is required", "")
  })

# ✅ ĐÚNG - Form validation before submit
OnSelect: |
  =If(And(
    Not(IsBlank(EmailInput.Text)),
    Not(IsBlank(NameInput.Text)),
    IsMatch(EmailInput.Text, Email)
  ),
    // Submit form
    SubmitForm(),
    // Show validation errors
    Notify("Please correct the errors", NotificationType.Warning)
  )
```

---

## 5. ERROR HANDLING

### 5.1 Comprehensive Error Handling
**IMPLEMENT** trong App/OnError:

```yaml
# File: src/App/OnError
Switch(FirstError.Kind,
  ErrorKind.Network, 
    Notify("Lỗi kết nối mạng. Vui lòng kiểm tra kết nối.", NotificationType.Error);
    Set(varNetworkError, true),
    
  ErrorKind.Sync, 
    Notify("Lỗi đồng bộ dữ liệu. Vui lòng thử lại.", NotificationType.Warning);
    Set(varSyncError, true),
    
  ErrorKind.Validation, 
    Notify("Dữ liệu không hợp lệ: " & FirstError.Message, NotificationType.Error);
    Set(varValidationError, FirstError.Message),
    
  // Default error
  Notify("Đã xảy ra lỗi: " & FirstError.Message, NotificationType.Error);
  Set(varLastError, {
    Message: FirstError.Message,
    Kind: FirstError.Kind,
    Timestamp: Now(),
    Screen: App.ActiveScreen.Name
  })
)
```

### 5.2 User-Friendly Error Messages
**PROVIDE** clear, actionable error messages:

```yaml
# ✅ ĐÚNG - Clear error messages
Text: =If(varHasError,
  "Unable to load data. Please check your connection and try again.",
  "Data loaded successfully"
)

# ✅ ĐÚNG - Error recovery options
OnSelect: |
  =If(varHasError,
    Set(varHasError, false); RefreshData(),
    ProceedWithNormalFlow()
  )

# ❌ SAI - Technical error messages
Text: ="Error 500: Internal server error occurred"
```

### 5.3 Fallback UI States
**PROVIDE** fallback states cho better UX:

```yaml
# ✅ ĐÚNG - Empty state
Visible: =And(CountRows(DataSource) = 0, Not(varIsLoading))
Text: ="No data available. Try refreshing or check back later."

# ✅ ĐÚNG - Error state
Visible: =varHasError
Text: ="Something went wrong. Please try again."

# ✅ ĐÚNG - Loading state
Visible: =varIsLoading
Text: ="Loading your data..."
```

---

## 6. TESTING & DEBUGGING

### 6.1 Demo Data for Development
**PROVIDE** demo data cho better preview experience:

```yaml
# ✅ ĐÚNG - Fallback demo data trong OnVisible
OnVisible: |
  =If(IsBlank(varUserData),
    Set(varUserData, {
      Name: "Demo User",
      Email: "demo@company.com",
      Role: "Employee",
      Department: "IT"
    })
  );
  If(CountRows(colMenuItems) = 0,
    ClearCollect(colMenuItems,
      {Id: 1, Name: "Dashboard", Icon: "Home"},
      {Id: 2, Name: "Reports", Icon: "BarChart"},
      {Id: 3, Name: "Settings", Icon: "Settings"}
    )
  )
```

### 6.2 Debug Variables
**USE** debug variables cho development:

```yaml
# ✅ ĐÚNG - Debug information
Text: =If(varDebugMode,
  "Debug: " & varCurrentUser.Name & " | Screen: " & App.ActiveScreen.Name,
  "Welcome " & varCurrentUser.Name
)

# ✅ ĐÚNG - Performance monitoring
OnVisible: |
  =Set(varPageLoadStart, Now());
  // Load data
  Set(varPageLoadEnd, Now());
  Set(varLoadTime, DateDiff(varPageLoadStart, varPageLoadEnd, Milliseconds))
```

### 6.3 Testing Different States
**TEST** various application states:

```yaml
# ✅ ĐÚNG - Test loading state
OnSelect: |
  =Set(varIsLoading, true);
  // Simulate delay for testing
  Set(varTestTimer, Now() + Time(0,0,2));
  Set(varIsLoading, false)

# ✅ ĐÚNG - Test error state
OnSelect: |
  =Set(varHasError, true);
  Set(varErrorMessage, "Test error message for UI testing")

# ✅ ĐÚNG - Test empty state
OnSelect: |
  =Clear(colTestData);
  Set(varShowEmptyState, true)
```

---

## 🚨 PERFORMANCE TIPS

### Avoid Common Performance Pitfalls

1. **Minimize Fixed Numbers**: Use relative calculations whenever possible
2. **Lazy Load Data**: Don't load everything at startup
3. **Cache Frequently Used Data**: Store commonly accessed data in variables
4. **Use Efficient Filters**: Filter at the data source level
5. **Optimize Formulas**: Avoid complex calculations in frequently triggered events
6. **Implement Pagination**: For large datasets, show data in chunks
7. **Use Appropriate Data Types**: Choose the right data types for your needs
8. **Monitor Memory Usage**: Be aware of large collections and images

### Code Quality Checklist

- [ ] All properties use relative positioning/sizing
- [ ] Consistent naming conventions used throughout
- [ ] Error handling implemented at app and screen levels
- [ ] Loading states provided for all async operations
- [ ] Demo data available for development/preview
- [ ] Components are reusable and self-contained
- [ ] Global variables properly organized in App/Formulas
- [ ] Font sizes use global FontSizes object
- [ ] Colors use consistent RGBA format
- [ ] Collections use 'col' prefix, variables use 'var' prefix

---

**Agent phải tuân thủ những best practices này để tạo high-quality, maintainable Power App code.**