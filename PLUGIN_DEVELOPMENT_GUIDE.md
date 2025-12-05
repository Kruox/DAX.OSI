# DAX.OSI Plugin Development Guide

**For External Plugin Developers**

This guide walks you through creating custom plugins for DAX.OSI on your local development machine. You will develop plugins independently using only the DOSI.Core library published as a NuGet package.

**Note:** You do not need access to the full DAX.OSI master source. All you need is DOSI.Core and the .NET 9.0 SDK.

---

## Quick Start (5 Minutes)

If you want to get a plugin running immediately:

```bash
# 1. Create workspace
mkdir ~/MyDAXPlugins
cd ~/MyDAXPlugins

# 2. Create project
dotnet new classlib -n MyPlugin -f net9.0
cd MyPlugin

# 3. Add packages
dotnet add package DOSI.Core
dotnet add package Avalonia --version 11.3.9

# 4. Build
dotnet build --configuration Release

# 5. Deploy to DAX.OSI Applications folder
cp bin/Release/net9.0/MyPlugin.dll ~/path/to/DOSI/Users/YourUsername/Applications/

# 6. Run DAX.OSI and check 'plugins' command in terminal
```

Then add your code to the generated files (see [Creating Your First Plugin](#creating-your-first-plugin)).

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Project Setup](#project-setup)
3. [Creating Your First Plugin](#creating-your-first-plugin)
4. [Plugin Structure](#plugin-structure)
5. [Testing Your Plugin](#testing-your-plugin)
6. [Common Patterns](#common-patterns)
7. [Debugging](#debugging)
8. [Packaging & Distribution](#packaging--distribution)
9. [Troubleshooting](#troubleshooting)
10. [Complete Examples](#complete-examples)

---

## Prerequisites

### Required Software

1. **.NET 9.0 SDK or Later**
   - Download from: https://dotnet.microsoft.com/download
   - Verify installation: `dotnet --version`

2. **IDE (Choose One)**
   - **Visual Studio 2022** (Community, Professional, or Enterprise)
     - Download: https://visualstudio.microsoft.com/vs/
     - Install ".NET desktop development" workload
   - **JetBrains Rider** 
     - Download: https://www.jetbrains.com/rider/
     - Full .NET support out of the box
   - **Visual Studio Code** + C# Extension
     - Download: https://code.visualstudio.com/
     - Install "C# Dev Kit" extension

3. **Git** (Optional but Recommended)
   - Download: https://git-scm.com/

### System Requirements

- **Windows**: Windows 10/11 with .NET 9.0 runtime
- **Linux**: Any modern distribution with .NET 9.0 runtime
- **macOS**: macOS 10.15+ with .NET 9.0 runtime
- **Disk Space**: 500MB+ for development tools
- **RAM**: 4GB minimum (8GB recommended)

### Verify Your Setup

Open terminal/PowerShell and run:

```bash
# Check .NET installation
dotnet --version
# Expected output: 9.0.x or higher

# Check if git is available (optional)
git --version
```

---

## Project Setup

### Step 1: Create Your Plugin Workspace

Choose a location on your machine to store your plugin projects:

```bash
# Create workspace directory
mkdir ~/MyDAXPlugins
cd ~/MyDAXPlugins
```

### Step 2: Install .NET 9.0 SDK

1. Download .NET 9.0 SDK from: https://dotnet.microsoft.com/download
2. Install and verify:
   ```bash
   dotnet --version
   # Should output: 9.0.x or higher
   ```

### Step 3: Choose and Install IDE

**Visual Studio 2022 (Recommended for Windows/Mac):**
- Download: https://visualstudio.microsoft.com/vs/
- Install ".NET desktop development" workload

**JetBrains Rider (Recommended for all platforms):**
- Download: https://www.jetbrains.com/rider/
- Full .NET support included

**Visual Studio Code (Lightweight Option):**
- Download: https://code.visualstudio.com/
- Install "C# Dev Kit" extension

### Step 4: Verify Your Setup

```bash
# Verify .NET is available
dotnet --version

# Verify you can create a new project
dotnet new classlib -n TestProject -f net9.0
dotnet build TestProject
# Should succeed without errors
```

Once this works, clean up and you're ready to create plugins:
```bash
rm -rf TestProject
```

---

## Creating Your First Plugin

### Project Structure Overview

```
~/MyDAXPlugins/
│
├── MyFirstPlugin/               # Your plugin project
│   ├── MyFirstPlugin.csproj
│   ├── MyFirstPlugin.cs         # Plugin main class (implements IDosiPlugin)
│   ├── MyFirstPluginWindow.cs   # Plugin UI window (inherits from DosiPluginWindowUI)
│   ├── obj/
│   └── bin/
│       └── Release/
│           └── MyFirstPlugin.dll  # Final output (deploy this)
│
└── MySecondPlugin/              # Additional plugins follow same structure
    ├── MySecondPlugin.csproj
    ├── MySecondPlugin.cs
    └── ...
```

### Step 1: Create a New Class Library Project

**Using Command Line (All Platforms):**

```bash
cd ~/MyDAXPlugins

# Create new class library
dotnet new classlib -n MyFirstPlugin -f net9.0

# Navigate into project
cd MyFirstPlugin
```

**Using Visual Studio 2022:**

1. Create new project → "Class Library"
2. Name: `MyFirstPlugin`
3. Location: `~/MyDAXPlugins/`
4. Framework: **.NET 9.0**
5. Click Create

**Using JetBrains Rider:**

1. File → New Project
2. Select "Class Library"
3. Name: `MyFirstPlugin`
4. Framework: **.NET 9.0**
5. Create

### Step 2: Add DOSI.Core NuGet Package Reference

**Command Line:**

```bash
cd MyFirstPlugin
dotnet add package DOSI.Core
```

**Visual Studio 2022:**

1. Right-click project → "Manage NuGet Packages"
2. Search: "DOSI.Core"
3. Select package and click "Install"

**JetBrains Rider:**

1. Right-click project → "Manage NuGet Packages"
2. Search: "DOSI.Core"
3. Click "Install"

### Step 3: Add Avalonia Package Reference

The DOSI.Core package includes Avalonia as a dependency, but you may need to explicitly add it:

```bash
dotnet add package Avalonia --version 11.3.9
```

### Step 4: Update Project File (Optional Optimization)

Edit `MyFirstPlugin.csproj`:

```xml
<Project Sdk="Microsoft.NET.Sdk">

    <PropertyGroup>
        <TargetFramework>net9.0</TargetFramework>
        <ImplicitUsings>enable</ImplicitUsings>
        <Nullable>enable</Nullable>
        <LangVersion>latest</LangVersion>
        <AssemblyName>MyFirstPlugin</AssemblyName>
    </PropertyGroup>

    <ItemGroup>
        <PackageReference Include="DOSI.Core" Version="1.0.0" />
        <PackageReference Include="Avalonia" Version="11.3.9" />
    </ItemGroup>

</Project>
```

---

## Creating a Plugin - Complete Example

### Step 1: Create the Plugin Main Class

Create file: `MyFirstPlugin.cs`

```csharp
using DOSI.Core.Plugins;

namespace MyFirstPlugin;

/// <summary>
/// Example plugin implementing IDosiPlugin interface
/// </summary>
public class MyFirstPlugin : IDosiPlugin
{
    public string PluginName => "My First Plugin";
    public string Version => "1.0.0";
    public string Author => "Your Name";
    public string Description => "A simple example plugin to learn DAX.OSI development";

    /// <summary>
    /// Return the main window type that will be instantiated
    /// </summary>
    public Type GetMainWindowType() => typeof(MyFirstPluginWindow);

    /// <summary>
    /// Called when plugin is loaded
    /// </summary>
    public void OnLoad()
    {
        Console.WriteLine($"[{PluginName}] Plugin loaded successfully!");
    }

    /// <summary>
    /// Called when plugin is unloaded
    /// </summary>
    public void OnUnload()
    {
        Console.WriteLine($"[{PluginName}] Plugin unloading...");
    }
}
```

### Step 2: Create the Plugin Window

Create file: `MyFirstPluginWindow.cs`

```csharp
using DOSI.Core.Controls;
using Avalonia.Controls;
using Avalonia.Layout;
using Avalonia.Media;

namespace MyFirstPlugin;

/// <summary>
/// Main window for the plugin
/// Inherits from DosiPluginWindowUI for standard window styling
/// </summary>
public class MyFirstPluginWindow : DosiPluginWindowUI
{
    private TextBlock? _helloText;
    private TextBlock? _versionText;
    private Button? _clickButton;
    private int _clickCount = 0;

    public MyFirstPluginWindow()
    {
        // Set window properties
        WindowTitle = "My First Plugin";
        Width = 400;
        Height = 300;

        // Create UI
        InitializeUI();
    }

    private void InitializeUI()
    {
        if (ContentArea == null)
            return;

        // Create main container
        var mainContainer = new StackPanel
        {
            Orientation = Orientation.Vertical,
            Spacing = 15,
            Padding = new Avalonia.Thickness(20),
            VerticalAlignment = VerticalAlignment.Center,
            HorizontalAlignment = HorizontalAlignment.Center
        };

        // Title
        _helloText = new TextBlock
        {
            Text = "Welcome to My First Plugin!",
            FontSize = 24,
            FontWeight = Avalonia.Media.FontWeight.Bold,
            Foreground = new SolidColorBrush(Color.FromRgb(0, 0, 0)),
            TextAlignment = Avalonia.Media.TextAlignment.Center
        };
        mainContainer.Children.Add(_helloText);

        // Version info
        _versionText = new TextBlock
        {
            Text = "Version 1.0.0",
            FontSize = 14,
            Foreground = new SolidColorBrush(Color.FromRgb(100, 100, 100)),
            TextAlignment = Avalonia.Media.TextAlignment.Center
        };
        mainContainer.Children.Add(_versionText);

        // Button
        _clickButton = new Button
        {
            Content = "Click Me (0 clicks)",
            Width = 150,
            Height = 40,
            Padding = new Avalonia.Thickness(10),
            HorizontalAlignment = HorizontalAlignment.Center
        };
        _clickButton.Click += (s, e) => HandleButtonClick();
        mainContainer.Children.Add(_clickButton);

        // Instructions
        var instructionsText = new TextBlock
        {
            Text = "Click the button to test interaction",
            FontSize = 12,
            Foreground = new SolidColorBrush(Color.FromRgb(150, 150, 150)),
            TextAlignment = Avalonia.Media.TextAlignment.Center
        };
        mainContainer.Children.Add(instructionsText);

        // Add to window
        ContentArea.Children.Add(mainContainer);
    }

    private void HandleButtonClick()
    {
        _clickCount++;
        if (_clickButton != null)
        {
            _clickButton.Content = $"Click Me ({_clickCount} clicks)";
        }
    }
}
```

### Step 3: Build the Plugin

```bash
cd MyFirstPlugin
dotnet build --configuration Release
```

Output will be in: `MyFirstPlugin/bin/Release/net9.0/MyFirstPlugin.dll`

### Step 4: Deploy the Plugin

1. **Determine the system folder path** (where DAX.OSI stores data):
   - Windows: `%APPDATA%/DOSI/` or `AppContext.BaseDirectory/DOSI/`
   - Linux/Mac: Similar path relative to app location

2. **Create Applications folder for user**:
   ```
   DOSI/Users/[username]/Applications/
   ```
   (This is created automatically when first user logs in)

3. **Copy plugin DLL**:
   ```bash
   # Windows
   copy "MyFirstPlugin\bin\Release\net9.0\MyFirstPlugin.dll" "Path/To/DOSI/Users/YourUsername/Applications/"
   
   # Linux/Mac
   cp MyFirstPlugin/bin/Release/net9.0/MyFirstPlugin.dll ~/path/to/DOSI/Users/YourUsername/Applications/
   ```

---

## Testing Your Plugin

### Manual Testing Steps

#### 1. **Build Your Plugin**

```bash
cd MyFirstPlugin
dotnet build --configuration Release
```

Output location: `MyFirstPlugin/bin/Release/net9.0/MyFirstPlugin.dll`

#### 2. **Install Plugin to DAX.OSI**

Your plugin needs to be placed in the DAX.OSI system folder where the application looks for plugins:

```
DOSI/Users/[username]/Applications/MyFirstPlugin.dll
```

**To find the correct path:**

1. Run DAX.OSI application
2. Log in with a user (or create one)
3. Open Terminal (Ctrl+T)
4. The console output shows paths like: `[PluginManager] Discovering plugins in: /path/to/DOSI/Users/User/Applications/`

**Deploy your plugin:**

```bash
# Windows
copy "MyFirstPlugin\bin\Release\net9.0\MyFirstPlugin.dll" "C:\path\to\DOSI\Users\YourUsername\Applications\"

# Linux/Mac
cp MyFirstPlugin/bin/Release/net9.0/MyFirstPlugin.dll ~/path/to/DOSI/Users/YourUsername/Applications/
```

#### 3. **Test Plugin Loading**

1. Launch DAX.OSI application
2. Log in (or create a new user if first time)
3. Open Terminal (Press Ctrl+T or click Terminal in desktop menu)
4. Type: `plugins`
5. You should see your plugin listed:
   ```
   My First Plugin v1.0.0 by Your Name
   A simple example plugin to learn DAX.OSI development
   ```

If your plugin doesn't appear:
- Check the console output for error messages
- Verify the DLL is in the correct `Applications` folder
- Check that your class implements `IDosiPlugin` correctly
- Ensure the window class inherits from `DosiPluginWindowUI`

#### 4. **Launch Your Plugin from Desktop**

If your plugin is discovered:
1. Look at the Desktop screen (after login)
2. Click "Applications" button/menu on taskbar
3. Your plugin should appear in the list
4. Click it to launch your plugin window

#### 5. **Check Console Output**

Look for console messages like:
```
[PluginManager] Discovering plugins in: .../Users/User/Applications/
[PluginManager] Loading: MyFirstPlugin.dll
[MyFirstPlugin] Plugin loaded successfully!
```

If you see errors, they'll be displayed here. Common errors:
- `Could not load file or assembly` - version mismatch
- `Type does not implement IDosiPlugin` - interface not implemented correctly
- `File not found` - DLL not in Applications folder

### Plugin Structure

A valid DAX.OSI plugin consists of exactly **two classes**:

**1. Plugin Main Class (implements IDosiPlugin):**
```csharp
public class MyPlugin : IDosiPlugin
{
    public string PluginName { get; }        // Display name
    public string Version { get; }           // Version number
    public string Author { get; }            // Author name
    public string Description { get; }       // Short description
    public Type GetMainWindowType();         // Return window class
    public void OnLoad();                    // Called on load
    public void OnUnload();                  // Called on unload
}
```

**2. Plugin Window Class (inherits from DosiPluginWindowUI):**
```csharp
public class MyPluginWindow : DosiPluginWindowUI
{
    // Your UI implementation
    public MyPluginWindow()
    {
        WindowTitle = "My Plugin";
        Width = 500;
        Height = 400;
        // Add controls to ContentArea
    }
}
```

---

## What You CAN and CANNOT Do

### ✅ WHAT YOU CAN DO (As an External Plugin Developer)

**Create Custom Plugins:**
- ✅ Create custom window-based applications
- ✅ Use all DOSI.Core components and controls
- ✅ Access accent/theme system
- ✅ Create and manage data files
- ✅ Use notification system
- ✅ Access window management API
- ✅ Build custom UIs with Avalonia controls
- ✅ Use all .NET 9.0 capabilities

**Distribute Your Plugins:**
- ✅ Share plugin DLLs independently
- ✅ Create your own plugin installers
- ✅ Monetize your plugins
- ✅ Open-source your plugins
- ✅ Create plugin repositories

### ❌ WHAT YOU CANNOT DO

**Modify Core DAX.OSI:**
- ❌ Cannot create built-in applications (those require master source)
- ❌ Cannot add terminal commands to system (plugin-level only)
- ❌ Cannot modify the Boot, Login, or Desktop screens
- ❌ Cannot create new accent themes (can use existing ones)
- ❌ Cannot modify user management system
- ❌ Cannot modify file system structure
- ❌ Cannot modify window manager functionality

**Access Internals:**
- ❌ Cannot access private members of DOSI.Core classes
- ❌ Cannot modify core system settings beyond what APIs allow
- ❌ Cannot hook into internal event systems not exposed in API

**Why These Limitations?**
- Ensures system stability and security
- Prevents plugins from breaking core functionality
- Protects user data integrity
- Maintains consistent UI/UX experience
- Allows DAX.OSI to be updated safely

---

## Common Patterns

### Pattern 1: Plugin with Data Storage

```csharp
public class MyDataPlugin : IDosiPlugin
{
    private string _dataFolder = Path.Combine(
        SystemSettings.SystemFolder, 
        "Users", 
        UserManager.Instance.CurrentUser?.Username ?? "User",
        "Applications",
        "MyDataPluginData"
    );

    public void OnLoad()
    {
        // Create data folder if needed
        Directory.CreateDirectory(_dataFolder);
    }

    public void SaveData(string filename, string content)
    {
        var filepath = Path.Combine(_dataFolder, filename);
        File.WriteAllText(filepath, content);
    }

    public string LoadData(string filename)
    {
        var filepath = Path.Combine(_dataFolder, filename);
        return File.Exists(filepath) ? File.ReadAllText(filepath) : string.Empty;
    }
}
```

### Pattern 2: Plugin with Notifications

```csharp
using DOSI.Core.Notifications;

public class MyNotificationPlugin : DosiPluginWindowUI
{
    private Button? _notifyButton;

    public MyNotificationPlugin()
    {
        _notifyButton = new Button { Content = "Show Notification" };
        _notifyButton.Click += (s, e) => ShowNotification();
    }

    private async void ShowNotification()
    {
        await NotificationHelper.Notify("This is a notification!");
    }
}
```

### Pattern 3: Plugin with Terminal Commands

Plugins can register custom terminal commands:

```csharp
public class MyTerminalPlugin : DosiPluginWindowUI
{
    private DosiTerminalControl? _terminal;

    public MyTerminalPlugin()
    {
        _terminal = new DosiTerminalControl();
        // Register custom commands
        RegisterCommand("mycommand", args => HandleMyCommand(args));
    }

    private void RegisterCommand(string name, Action<string[]> handler)
    {
        // Command registration logic
    }
}
```

### Pattern 4: Theme-Aware UI

```csharp
using DOSI.Core.Accent;

public class ThemedPlugin : DosiPluginWindowUI
{
    public ThemedPlugin()
    {
        var accentManager = AccentManager.Instance;
        var theme = accentManager.CurrentTheme;

        var background = new SolidColorBrush(theme.PrimaryColor);
        var textColor = new SolidColorBrush(theme.TextColor);

        // Use theme colors in UI
        // ...

        // Listen for theme changes
        accentManager.ThemeChanged += (s, newTheme) => UpdateTheme(newTheme);
    }

    private void UpdateTheme(AccentTheme theme)
    {
        // Update UI to match new theme
    }
}
```

---

## Debugging

### Enable Debug Logging

Add to your plugin:

```csharp
private const string LogPrefix = "[MyPlugin]";

private void Log(string message)
{
    Console.WriteLine($"{LogPrefix} {message}");
}

public void OnLoad()
{
    Log("Plugin initializing...");
    Log($"System folder: {SystemSettings.SystemFolder}");
    Log($"Current user: {UserManager.Instance.CurrentUser?.Username}");
}
```

### Breakpoints in IDE

**Visual Studio 2022:**
- Click left margin next to line number
- Red circle appears
- Program pauses when line is reached
- Use Debug menu to step through code

**JetBrains Rider:**
- Same as VS: click left margin to set breakpoint
- Use Shift+F9 to start debugging
- Step through with F10 (step over) or F11 (step into)

### Inspect Variables

Hover over variables while debugging to see their values. Use immediate window to evaluate expressions.

### Common Debug Scenarios

**Plugin not loading:**
```csharp
Log($"Plugin type: {this.GetType().Name}");
Log($"Main window type: {GetMainWindowType()}");
Log($"Window implements DosiWindowUi: {GetMainWindowType().IsAssignableTo(typeof(DosiWindowUi))}");
```

**Window not rendering:**
```csharp
public MyWindow()
{
    Log($"ContentArea is null: {ContentArea == null}");
    Log($"Window title set to: {WindowTitle}");
    Log($"Dimensions: {Width}x{Height}");
}
```

**Theme not applying:**
```csharp
public void OnThemeChanged(AccentTheme theme)
{
    Log($"Theme changed to: {theme.Name}");
    Log($"Primary color: {theme.PrimaryColor}");
}
```

---

## Packaging & Distribution

### Creating a Release Build

```bash
cd MyFirstPlugin

# Build release version
dotnet build --configuration Release

# Output: MyFirstPlugin/bin/Release/net9.0/MyFirstPlugin.dll
```

### Plugin Package Structure

```
MyFirstPlugin-1.0.0/
├── MyFirstPlugin.dll
├── README.md
│   ├── Plugin Name
│   ├── Version
│   ├── Author
│   ├── Description
│   └── Installation instructions
├── LICENSE
└── CHANGELOG.md
```

### Distribution Options

**Option 1: Direct Distribution**
- Zip the DLL and documentation
- Share via download link
- Users copy to `Users/[username]/Applications/`

**Option 2: Plugin Installer**
- Create batch/shell script that:
  1. Finds DOSI system folder
  2. Creates Applications folder
  3. Copies DLL
  4. Displays success message

**Example Installer (Windows batch):**
```batch
@echo off
REM MyFirstPlugin Installer

set DOSI_PATH=%APPDATA%\DOSI
set USERNAME=%USERNAME%
set INSTALL_PATH=%DOSI_PATH%\Users\%USERNAME%\Applications

if not exist "%INSTALL_PATH%" mkdir "%INSTALL_PATH%"

copy "MyFirstPlugin.dll" "%INSTALL_PATH%"

echo.
echo Plugin installed successfully!
echo Location: %INSTALL_PATH%
echo.
echo Run DAX.OSI and use the 'plugins' command to verify.
pause
```

---

## Troubleshooting

### Issue: "Plugin not found" when launching

**Cause:** DLL not in correct location or filename case mismatch

**Solution:**
```bash
# Verify file exists
ls "DOSI/Users/[username]/Applications/MyFirstPlugin.dll"

# Check DAX.OSI console output for exact path searched
```

### Issue: "Could not load file or assembly"

**Cause:** Dependency version mismatch or missing assembly

**Solution:**
- Ensure plugin targets .NET 9.0
- Verify DOSI.Core reference is correct
- Check all dependencies are compatible

### Issue: "Type does not implement IDosiPlugin"

**Cause:** Main class doesn't implement interface correctly

**Solution:**
```csharp
// Correct
public class MyPlugin : IDosiPlugin
{
    public string PluginName => "...";
    public string Version => "...";
    // etc.
}

// Also ensure no typos in interface name
```

### Issue: Window doesn't appear

**Cause:** ContentArea is null or window not registered

**Solution:**
```csharp
public MyWindow()
{
    WindowTitle = "My App";
    Width = 500;
    Height = 400;

    // Check ContentArea is initialized BEFORE adding controls
    if (ContentArea != null)
    {
        var button = new Button { Content = "Test" };
        ContentArea.Children.Add(button);
    }
}
```

### Issue: Crashes on startup

**Solution:** Check console output for stack trace:

```csharp
try
{
    // Your code
}
catch (Exception ex)
{
    Console.WriteLine($"[MyPlugin] Error: {ex.Message}");
    Console.WriteLine($"[MyPlugin] Stack: {ex.StackTrace}");
}
```

---

## Examples

### Example 1: Simple Counter Plugin

```csharp
// CounterPlugin.cs
using DOSI.Core.Plugins;
using DOSI.Core.Controls;
using Avalonia.Controls;
using Avalonia.Layout;
using Avalonia.Media;

namespace CounterPlugin;

public class CounterPlugin : IDosiPlugin
{
    public string PluginName => "Counter";
    public string Version => "1.0.0";
    public string Author => "Developer";
    public string Description => "A simple counter application";

    public Type GetMainWindowType() => typeof(CounterWindow);
    public void OnLoad() => Console.WriteLine("[Counter] Loaded!");
    public void OnUnload() => Console.WriteLine("[Counter] Unloading...");
}

public class CounterWindow : DosiPluginWindowUI
{
    private int _count = 0;
    private TextBlock? _countDisplay;

    public CounterWindow()
    {
        WindowTitle = "Counter";
        Width = 300;
        Height = 200;
        InitializeUI();
    }

    private void InitializeUI()
    {
        if (ContentArea == null) return;

        var stack = new StackPanel { Spacing = 20, Padding = new Avalonia.Thickness(20) };

        _countDisplay = new TextBlock 
        { 
            Text = "0", 
            FontSize = 48,
            TextAlignment = Avalonia.Media.TextAlignment.Center
        };
        stack.Children.Add(_countDisplay);

        var button = new Button { Content = "Increment" };
        button.Click += (s, e) => { _count++; _countDisplay.Text = _count.ToString(); };
        stack.Children.Add(button);

        ContentArea.Children.Add(stack);
    }
}
```

### Example 2: Color Picker Plugin

See the complete example in the repository at `Examples/ColorPickerPlugin/`

### Example 3: Todo List Application

See the complete example in the repository at `Examples/TodoListApp/`

---

## Next Steps

1. **Start Small**: Create a simple "Hello World" plugin first
2. **Explore Examples**: Look at NotePad and Terminal implementations
3. **Review DOSI.Core**: Understand available components
4. **Join Community**: Share your plugins with other developers
5. **Contribute**: Submit improvements to DAX.OSI core

---

## Additional Resources

- **DOSI.Core NuGet Package**: https://www.nuget.org/packages/DOSI.Core
- **Avalonia Documentation**: https://docs.avaloniaui.net/ (UI Framework)
- **.NET Documentation**: https://learn.microsoft.com/en-us/dotnet/ (C# & .NET)
- **DAX.OSI GitHub**: [Link to repository]
- **Plugin Examples**: Check the DOSI.Core NuGet package source code on GitHub
- **Community Support**: [Link to community forum or Discord]

---

## Support

For issues or questions:

1. Check **Troubleshooting** section above
2. Review console output for error messages
3. Examine similar plugins in the solution
4. Search Avalonia documentation for UI issues

---

**Document Version:** 1.0  
**Last Updated:** December 4, 2025  
**Status:** Ready for Beta Developers

---

## Quick Reference Checklist

- [ ] .NET 9.0 SDK installed
- [ ] IDE installed and configured
- [ ] Solution cloned/copied to local machine
- [ ] Can build main solution successfully
- [ ] Created new Class Library project
- [ ] Added reference to DOSI.Core
- [ ] Added Avalonia NuGet package
- [ ] Created IDosiPlugin implementation
- [ ] Created DosiPluginWindowUI window
- [ ] Built plugin to DLL
- [ ] Copied DLL to Applications folder
- [ ] Plugin loads without errors
- [ ] Plugin window displays correctly
- [ ] Ready to add custom features!

