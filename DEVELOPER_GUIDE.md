# DAX.OSI - Complete Developer Guide

**Version:** 1.0 Beta  
**Project Type:** Cross-Platform Desktop Application (Avalonia UI Framework)  
**Target Framework:** .NET 9.0  
**Languages:** C#, XAML (minimal)

---

## Table of Contents

1. [Project Overview](#project-overview)
2. [Architecture](#architecture)
3. [Project Structure](#project-structure)
4. [Core Systems](#core-systems)
5. [UI Framework & Controls](#ui-framework--controls)
6. [Key Components](#key-components)
7. [Default Applications](#default-applications)
8. [Plugin System](#plugin-system)
9. [Settings & User Management](#settings--user-management)
10. [Development Workflow](#development-workflow)
11. [Building & Testing](#building--testing)
12. [Troubleshooting](#troubleshooting)
13. [API Reference](#api-reference)

---

## Project Overview

**DAX.OSI** is a cross-platform desktop application that simulates a custom operating system with a graphical interface. It's built entirely in C# using the Avalonia UI framework, allowing it to run on Windows, Linux, and macOS.

### Key Features

- **Custom UI Framework**: Fully custom-built window system, buttons, text boxes, and other controls
- **User Management**: Multi-user system with profile management and authentication
- **Accent Theming**: Dynamic theme switching with multiple pre-built color schemes
- **Terminal System**: Built-in terminal emulator with command execution
- **Plugin Architecture**: Extensible plugin system for custom applications
- **Notification System**: Toast-style notification bubbles with auto-dismiss
- **File Management**: File explorer and save dialogs
- **Default Applications**: NotePad, Terminal, File Explorer

### Technology Stack

- **Framework**: Avalonia 11.3.9 (Cross-platform UI)
- **.NET Runtime**: .NET 9.0
- **Build System**: MSBuild (via .NET SDK)
- **Language**: C# 12.0+

---

## Architecture

### High-Level Design

```
┌─────────────────────────────────────────────────────────────┐
│                    DAX.OSI Application                      │
├─────────────────────────────────────────────────────────────┤
│  MainWindow (Avalonia Window)                               │
│  ├── Canvas (Root rendering surface)                        │
│  │   ├── BootScreen (Loading/startup)                       │
│  │   ├── LoginScreen (User authentication)                  │
│  │   └── DesktopScreen (Main application interface)         │
│  │       └── DosiWindowUI instances (windows)               │
│  │           ├── Terminal                                   │
│  │           ├── NotePad                                    │
│  │           ├── File Explorer                              │
│  │           └── Custom Plugins                             │
│  └── NotificationOverlay (Top-level notifications)          │
└─────────────────────────────────────────────────────────────┘
```

### Design Patterns

1. **Singleton Pattern**: Used for managers (UserManager, AccentManager, WindowManager, NotificationBubbleManager, PluginManager, TerminalManager)
2. **Observer Pattern**: Events for theme changes, window registration, user creation
3. **Factory Pattern**: Window creation and plugin instantiation
4. **MVC-like**: Separation between managers (logic) and UI controls (presentation)

---

## Project Structure

### Directory Layout

```
DAX.OSI - Master Source/
├── DAX.OSI/                          # Main application (WinExe)
│   ├── App.cs                        # Application initialization
│   ├── MainWindow.cs                 # Root window and canvas setup
│   ├── Program.cs                    # Entry point
│   ├── App.axaml / MainWindow.axaml  # XAML files (minimal use)
│   ├── DAX.OSI.csproj               # Project file
│   │
│   ├── DefaultApplications/          # Built-in applications
│   │   ├── NotePad.cs               # Text editor
│   │   └── DOSITerminal.cs          # Terminal emulator
│   │
│   ├── UI/                           # User interface screens
│   │   ├── BootScreen.cs            # Startup loading screen
│   │   ├── LoginScreen.cs           # User login interface
│   │   ├── DesktopScreen.cs         # Main desktop with taskbar
│   │   ├── FileExplorer.cs          # File browser
│   │   ├── SaveFileDialog.cs        # Save file dialog
│   │   ├── AccentChangerWindow.cs   # Theme selector
│   │   └── TestWindow.cs            # Testing utility
│   │
│   └── Users/                        # User data directory
│       └── [Username]/
│           └── Documents/            # User documents
│
└── DOSI.Core/                        # Core library (netstandard)
    ├── DOSI.Core.csproj             # Project file
    │
    ├── Accent/
    │   └── AccentManager.cs          # Theme management
    │
    ├── Animations/
    │   └── DOSILoadingAnimation.cs  # Loading animation logic
    │
    ├── Assets/
    │   └── Fonts/                    # Embedded fonts
    │       ├── NotoSans-Regular.ttf
    │       └── Inconsolata-Regular.ttf
    │
    ├── Controls/                     # Custom UI controls
    │   ├── DOSIWindowUI.cs          # Base window class
    │   ├── DOSIButton.cs            # Custom button
    │   ├── DOSITextBox.cs           # Single-line text input
    │   ├── DOSIRichTextBox.cs       # Multi-line text input
    │   ├── DOSIListBox.cs           # List control
    │   ├── DOSIScreen.cs            # Base screen class
    │   ├── DOSIUserProfileImage.cs  # User avatar
    │   └── DosiPluginWindowUI.cs    # Plugin window base
    │
    ├── Notifications/
    │   ├── DOSINotificationBubble.cs # Individual notification
    │   ├── NotificationBubbleManager.cs  # Notification system
    │   └── NotificationHelper.cs    # Notification utilities
    │
    ├── Plugins/
    │   ├── IDosiPlugin.cs           # Plugin interface
    │   └── PluginManager.cs         # Plugin loading/management
    │
    ├── Settings/
    │   └── SystemSettings.cs        # System configuration
    │
    ├── Terminal/
    │   ├── DOSITerminalControl.cs   # Terminal UI control
    │   └── TerminalManager.cs       # Command processing
    │
    ├── User/
    │   └── UserManager.cs           # User profile management
    │
    └── Windowing/
        └── WindowManager.cs         # Window lifecycle management
```

---

## Core Systems

### 1. Application Startup (App.cs)

The application initialization happens in `App.cs`:

```csharp
public override void Initialize()
{
    InitializeSystemOnStartup();
    // Initialize fonts, themes, styles
}

public override void OnFrameworkInitializationCompleted()
{
    // Create main window and display it
}

private void InitializeSystemOnStartup()
{
    // 1. Initialize system folder (DOSI directory)
    // 2. Create or load SystemSettings.json
    // 3. Initialize AccentManager with default theme
    // 4. Initialize UserManager
}
```

**Key Points:**
- System folder is created at: `AppContext.BaseDirectory/DOSI/`
- All user data, settings, and plugins are stored within this folder
- Font files are embedded in DOSI.Core and loaded via `avares://` protocol

### 2. Main Window & Canvas (MainWindow.cs)

The main window serves as the root container:

```csharp
public class MainWindow : Window
{
    private Canvas _canvas;              // Root rendering surface
    private Canvas _notificationOverlay; // Top-level notification area
    private BootScreen? _bootScreen;
    private LoginScreen? _loginScreen;
    private DesktopScreen? _desktopScreen;
}
```

**Key Responsibilities:**
- Creates a root Canvas for all UI elements
- Manages screen transitions (Boot → Login → Desktop)
- Handles keyboard shortcuts (Ctrl+T for terminal)
- Manages the notification overlay
- Applies accent theme changes with cross-fade animations

**Screen Transitions:**
```
Boot Screen → Login Screen → Desktop Screen
   (1-2 sec)  (user input)  (window management)
```

### 3. System Settings (SystemSettings.cs)

Global system configuration stored in `DOSI/SystemSettings.json`:

```csharp
public class SystemSettings
{
    public string DefaultAccent { get; set; } = "Blue";
    public bool Fullscreen { get; set; } = false;
    // Add more settings as needed
}
```

**Key Features:**
- Singleton pattern for global access
- JSON serialization for persistence
- Auto-creates default file if missing
- Accessed via: `SystemSettings.Instance` or `SystemSettings.SystemFolder`

### 4. User Management (UserManager.cs)

Multi-user system with profile management:

```csharp
public class UserProfile
{
    public string Username { get; set; }
    public string Password { get; set; }
    public string Accent { get; set; }
}

public class UserManager
{
    public UserProfile? CurrentUser { get; private set; }
    public bool IsAuthenticated { get; private set; }
    public void Login(string username, string password);
    public void CreateUser(string username, string password, string accent);
}
```

**Key Features:**
- File-based storage in `DOSI/Users/[username]/`
- Each user has profile JSON and a Documents folder
- Default user ("User") created on first run
- Events fired when users are created
- Passwords stored as plain text (for beta - consider hashing in production)

### 5. Accent/Theme System (AccentManager.cs)

Dynamic theme switching with accent colors:

```csharp
public class AccentTheme
{
    public string Name { get; set; }
    public Color PrimaryColor { get; set; }
    public Color SecondaryColor { get; set; }
    public Color AccentColor { get; set; }
    public Color TextColor { get; set; }
    public Color BackgroundColor { get; set; }
}

public class AccentManager
{
    public AccentTheme CurrentTheme { get; private set; }
    public IReadOnlyDictionary<string, AccentTheme> AvailableThemes { get; }
    public event EventHandler<AccentTheme>? ThemeChanged;
    public void SetTheme(string themeName);
}
```

**Available Themes:**
- Blue (default)
- Green
- Purple
- Orange
- Red
- (Easily extensible in `InitializeDefaultThemes()`)

### 6. Window Management (WindowManager.cs)

Manages all DosiWindowUI instances and z-ordering:

```csharp
public class WindowManager
{
    public DosiWindowUi? FocusedWindow { get; }
    public IReadOnlyCollection<DosiWindowUi> AllWindows { get; }
    public IReadOnlyList<DosiWindowUi> WindowsInZOrder { get; }
    public event EventHandler<WindowEventArgs>? WindowRegistered;
    public event EventHandler<WindowEventArgs>? WindowUnregistered;
    
    public void RegisterWindow(DosiWindowUi window);
    public void UnregisterWindow(DosiWindowUi window);
    public void BringToFront(DosiWindowUi window);
}
```

**Key Features:**
- Tracks all open windows
- Maintains z-order (front-to-back)
- Manages window focus
- Fires events for window lifecycle

### 7. Notification System (NotificationBubbleManager.cs)

Toast notifications with auto-dismiss:

```csharp
public class NotificationBubbleManager
{
    public static NotificationBubbleManager? Instance { get; }
    public void ShowNotification(string title, string message, int dismissTimeMs = 4000);
}

// Usage via NotificationHelper:
await NotificationHelper.Notify("File saved successfully");
```

**Key Features:**
- Displays notifications in bottom-right corner
- Auto-dismiss after 4 seconds (customizable)
- Stack multiple notifications
- Automatically repositions when canvas resizes
- Fully async-compatible

---

## UI Framework & Controls

### Custom Control Hierarchy

```
Control (Avalonia base)
├── DOSIScreen
│   ├── BootScreen
│   ├── LoginScreen
│   └── DesktopScreen
│
├── DOSIWindowUI (Custom window implementation)
│   ├── DosiTerminal
│   ├── NotePad
│   ├── FileExplorer
│   ├── SaveFileDialog
│   └── [Custom Plugin Windows]
│
├── DosiPluginWindowUI (Plugin-specific window)
│   └── [User-created plugin windows]
│
└── [Standard DOSI Controls]
    ├── DOSIButton
    ├── DOSITextBox
    ├── DOSIRichTextBox
    ├── DOSIListBox
    ├── DOSIUserProfileImage
    └── DOSINotificationBubble
```

### DOSIWindowUI - Custom Window Control

The most important custom control - a fully custom window implementation with title bar, buttons, and resizing.

```csharp
public class DosiWindowUi : Control
{
    // Window properties
    public string WindowTitle { get; set; }
    public double Width { get; set; }
    public double Height { get; set; }
    
    // Content area where child controls are added
    public Panel? ContentArea { get; protected set; }
    
    // Window state
    public bool IsMaximized { get; set; }
    public bool IsMinimized { get; set; }
    
    // Methods
    public void Close();
    public void Minimize();
    public void Maximize();
}
```

**Features:**
- Custom-drawn title bar with window controls
- Draggable title bar for moving windows
- Resizable borders
- Minimize/Maximize/Close buttons
- Double-click to maximize
- Right-click context menu support

**Example - Creating a Window:**

```csharp
var notepad = new NotePad();
notepad.Width = 700;
notepad.Height = 500;
// Add to desktop via WindowManager or directly to canvas
```

### Other Key Controls

#### DOSIButton
```csharp
var button = new DosiButton
{
    Text = "Click Me",
    Width = 90,
    Height = 36,
    Background = new SolidColorBrush(Color.FromRgb(76, 175, 80))
};
button.Click += (s, e) => { /* Handle click */ };
```

#### DOSITextBox (Single-line)
```csharp
var textBox = new DosiTextBox
{
    Placeholder = "Enter text...",
    BackgroundColor = new SolidColorBrush(Color.FromRgb(255, 255, 255)),
    TextColor = new SolidColorBrush(Color.FromRgb(0, 0, 0))
};
var text = textBox.Text;
```

#### DOSIRichTextBox (Multi-line)
```csharp
var richText = new DosiRichTextBox
{
    Placeholder = "Start typing...",
    HorizontalAlignment = HorizontalAlignment.Stretch,
    VerticalAlignment = VerticalAlignment.Stretch
};
richText.Text = "Multi-line content";
```

#### DOSIListBox
```csharp
var listBox = new DosiListBox();
listBox.Items.Add("Item 1");
listBox.Items.Add("Item 2");
var selected = listBox.SelectedItem;
```

### DOSIScreen - Base Screen Class

All full-screen overlays inherit from this:

```csharp
public abstract class DosiScreen : Control
{
    // Full-screen control with accent-aware styling
}
```

**Usage:**
- Extend for new full-screen displays
- Automatically handles accent theming
- Stretches to fill available space

---

## Key Components

### BootScreen (UI/BootScreen.cs)

Initial loading/splash screen displayed when app starts.

**Purpose:**
- Visual feedback during startup
- Displays loading animation
- Automatically transitions to LoginScreen

**Key Methods:**
- `TransitionToLoginScreen()` - Fade out and switch to login
- `ShowLoadingAnimation()` - Display animated loading indicator

### LoginScreen (UI/LoginScreen.cs)

User authentication interface.

**Features:**
- Displays all available users as clickable icons
- Username/Password input fields
- Password verification
- Create new user button
- User accent preference selection

**Key Properties:**
- Displays users from `DOSI/Users/` directory
- Shows user profile images
- Input validation
- Event system for login completion

### DesktopScreen (UI/DesktopScreen.cs)

Main desktop interface after successful login.

**Features:**
- Taskbar at top showing current time
- Application menu for launching apps
- Desktop background with accent theme
- Window management area

**Key Components:**
- **Taskbar**: Time display, app launcher
- **Application Menu**: List of available applications (default + plugins)
- **Desktop Area**: Where DosiWindowUI windows are rendered

### FileExplorer (UI/FileExplorer.cs)

File browser and navigation system.

**Features:**
- Navigate file system
- Display files and folders
- Double-click to open files
- Create new files/folders
- Delete files
- Copy/paste operations

**Key Points:**
- User-specific scope (starts in user's Documents)
- File type associations (e.g., .txt opens in NotePad)

### SaveFileDialog (UI/SaveFileDialog.cs)

Modal dialog for saving files.

**Features:**
- File name input
- Directory navigation
- File type filters
- Save/Cancel buttons

**Usage:**
```csharp
var dialog = new SaveFileDialog();
dialog.SetSaveCallback(filename => {
    // Handle save
    HideDialog();
});
dialog.SetCancelCallback(() => HideDialog());
ShowDialog(dialog);
```

### AccentChangerWindow (UI/AccentChangerWindow.cs)

Theme selector window for changing accent colors.

**Features:**
- Display all available themes
- Preview theme colors
- Select and apply theme
- Save selection to user profile

---

## Default Applications

### 1. NotePad (DefaultApplications/NotePad.cs)

Simple text editor application.

```csharp
public class NotePad : DosiWindowUi
{
    private DosiRichTextBox? _textContent;
    private DosiButton? _saveButton;
    private DosiButton? _clearButton;
    
    public string GetTextContent();
    public void SetTextContent(string content);
}
```

**Features:**
- Multi-line text editing
- Save to file (user's Documents)
- Clear text
- File dialog integration

**File Operations:**
- Saves to: `DOSI/Users/[username]/Documents/[filename].txt`
- Supports custom file names
- Shows save notifications

**Usage in Terminal:**
```
notepad          # Open empty NotePad
notepad file.txt # Open and edit file.txt
```

### 2. DOSITerminal (DefaultApplications/DOSITerminal.cs)

Command-line interface for executing commands.

```csharp
public class DosiTerminal : DosiWindowUi
{
    private DosiTerminalControl _terminalControl;
    private TerminalManager _terminalManager;
    
    public TerminalManager TerminalManager { get; }
    public void FocusTerminal();
}
```

**Features:**
- Command input/output display
- Built-in commands (help, clear, echo, etc.)
- User creation and management
- Theme switching
- Plugin information
- File operations

**Available Commands:**
- `help` - Show available commands
- `clear` - Clear terminal display
- `echo [text]` - Echo text
- `version` - Display version info
- `exit` / `shutdown` - Close terminal
- `accent [name]` - Change theme
- `adduser` - Create new user (interactive)
- `explorer` - Open file explorer
- `notepad` - Open notepad
- `plugins` - List loaded plugins
- `test` - Run test window

**Usage:**
```
Open from Desktop → Applications menu → Terminal
Or: Ctrl+T keyboard shortcut
```

### 3. File Explorer (UI/FileExplorer.cs)

File system browser.

**Features:**
- Browse directories
- View files
- Open files with default application
- Create/delete files and folders
- Navigate with breadcrumb trail

---

## Plugin System

### IDosiPlugin Interface

All plugins must implement this interface:

```csharp
public interface IDosiPlugin
{
    string PluginName { get; }
    string Version { get; }
    string Author { get; }
    string Description { get; }
    
    Type GetMainWindowType();  // Must return type inheriting from DOSIWindowUI
    
    void OnLoad();    // Called when plugin loads
    void OnUnload();  // Called when plugin unloads
}
```

### Creating a Plugin

**Step 1:** Create a new Class Library project targeting .NET 9.0

**Step 2:** Reference DOSI.Core

**Step 3:** Create a window class inheriting from `DosiPluginWindowUI`:

```csharp
using DOSI.Core.Controls;
using Avalonia.Controls;

public class MyPluginWindow : DosiPluginWindowUI
{
    public MyPluginWindow()
    {
        WindowTitle = "My Custom Plugin";
        Width = 500;
        Height = 400;
        
        // Add controls to ContentArea
        var button = new Button { Content = "Click Me" };
        ContentArea?.Children.Add(button);
    }
}
```

**Step 4:** Create plugin class implementing `IDosiPlugin`:

```csharp
using DOSI.Core.Plugins;

public class MyPlugin : IDosiPlugin
{
    public string PluginName => "My Plugin";
    public string Version => "1.0.0";
    public string Author => "Your Name";
    public string Description => "A sample plugin";
    
    public Type GetMainWindowType() => typeof(MyPluginWindow);
    
    public void OnLoad()
    {
        Console.WriteLine("[MyPlugin] Plugin loaded!");
    }
    
    public void OnUnload()
    {
        Console.WriteLine("[MyPlugin] Plugin unloaded!");
    }
}
```

**Step 5:** Build DLL and place in plugin directory:

```
DOSI/Users/[username]/Applications/MyPlugin.dll
```

### PluginManager

Handles plugin discovery and loading:

```csharp
var manager = PluginManager.Instance;

// Discover and load plugins for current user
var plugins = manager.DiscoverAndLoadPlugins();

// Get loaded plugins
var loadedPlugins = manager.LoadedPlugins;

// Create window instance
var windowType = plugin.GetMainWindowType();
var window = (DosiWindowUi)Activator.CreateInstance(windowType);
```

**Key Features:**
- Automatically discovers DLLs in `Users/[username]/Applications/` folder
- Loads plugins dynamically using reflection
- Creates user applications folder if missing
- Gracefully handles plugin load failures

---

## Settings & User Management

### System Settings (DOSI/SystemSettings.json)

```json
{
  "DefaultAccent": "Blue",
  "Fullscreen": false
}
```

**Location:** `DOSI/SystemSettings.json`  
**Auto-created:** On first application run  
**Access:** `SystemSettings.Instance`

### User Profiles (DOSI/Users/[username]/)

Directory structure for each user:

```
DOSI/Users/tyler/
├── tyler.json           # User profile (username, password, accent)
└── Documents/           # User documents folder
    ├── file1.txt
    ├── file2.txt
    └── ...
```

**User Profile JSON (DOSI/Users/tyler/tyler.json):**

```json
{
  "username": "tyler",
  "password": "password123",
  "accent": "Blue"
}
```

### User Operations

```csharp
var userManager = UserManager.Instance;

// Check if authenticated
if (userManager.IsAuthenticated)
{
    var user = userManager.CurrentUser;
    Console.WriteLine($"Logged in as: {user.Username}");
}

// Login
userManager.Login("tyler", "password123");

// Create new user
userManager.CreateUser("newuser", "password", "Green");

// Event for user creation
userManager.UserCreated += (s, user) => {
    Console.WriteLine($"User created: {user.Username}");
};
```

---

## Development Workflow

### Setting Up Development Environment

**Requirements:**
- .NET 9.0 SDK or later
- Visual Studio 2022, JetBrains Rider, or VS Code with C# extension
- Git (optional, for version control)

**Clone/Open Project:**
```bash
# Open the solution
dotnet open "DAX.OSI - Master Source.sln"

# Or build from command line
dotnet build
```

### Adding New Features

#### Adding a New Control

1. Create file in `DOSI.Core/Controls/DosiMyControl.cs`
2. Inherit from `Control`
3. Implement custom rendering and logic
4. Register styles in `App.cs` if needed
5. Use in your applications

#### Adding a New Terminal Command

1. Open `DOSI.Core/Terminal/TerminalManager.cs`
2. Add method like `private void CommandMyCommand(string[] args)`
3. Register in `RegisterDefaultCommands()`:
   ```csharp
   RegisterCommand("mycommand", CommandMyCommand);
   ```
4. Implement command logic

#### Adding a New Built-In Application

1. Create class in `DAX.OSI/DefaultApplications/MyApp.cs`
2. Inherit from `DosiWindowUi`
3. Implement UI in constructor
4. Add terminal command to launch it (or add to application menu)

#### Adding a New Screen

1. Create in `DAX.OSI/UI/MyScreen.cs`
2. Inherit from `DosiScreen`
3. Implement full-screen layout
4. Handle transitions in `MainWindow.cs`

### Code Organization Best Practices

1. **Keep Managers Lightweight**: Logic for each manager should be cohesive
2. **Use Events**: Avoid tight coupling between components
3. **Async/Await**: Use for long-running operations to prevent UI freezing
4. **Dispatcher for UI**: Update UI elements on `Dispatcher.UIThread`
5. **Resource Cleanup**: Implement `IDisposable` for resources
6. **Comments**: Document public APIs and complex logic

### Testing Approach

**Manual Testing Checklist:**
- [ ] Application starts without errors
- [ ] Boot screen displays and transitions
- [ ] Login screen shows users
- [ ] Can create new user
- [ ] Can log in with valid credentials
- [ ] Desktop displays after login
- [ ] Terminal opens and accepts commands
- [ ] NotePad opens and saves files
- [ ] File Explorer navigates correctly
- [ ] Theme switching works
- [ ] Multiple windows can open simultaneously
- [ ] Windows can be moved, resized, minimized, maximized
- [ ] Notifications display and dismiss correctly
- [ ] Plugins load correctly if present

---

## Building & Testing

### Build Commands

```bash
# Build Debug
dotnet build

# Build Release
dotnet build --configuration Release

# Run Debug
dotnet run

# Publish (creates self-contained executable)
dotnet publish -c Release -r win-x64  # For Windows 64-bit
dotnet publish -c Release -r linux-x64 # For Linux 64-bit
dotnet publish -c Release -r osx-x64   # For macOS 64-bit
```

### Output Directories

- **Debug Build**: `bin/Debug/net9.0/`
- **Release Build**: `bin/Release/net9.0/`
- **Published**: `bin/Release/net9.0/publish/`

### Running the Application

```bash
# From command line
cd DAX.OSI
dotnet run

# From IDE
- Click Run button
- Or press F5 (most IDEs)
```

### Debug Output

Console output is printed to the debug console in your IDE. Look for:
- `[PluginManager]` - Plugin loading information
- `[NotePad]` - NotePad file operations
- `[TerminalManager]` - Terminal command execution
- `[UserManager]` - User management operations

---

## Troubleshooting

### Common Issues

#### Issue: "System Folder: [path]" shows incorrect location
**Solution:** System folder is relative to `AppContext.BaseDirectory`. For development, it's usually the build output directory. Check the console output for the actual path.

#### Issue: NotePad won't save files
**Cause:** User not authenticated or Documents folder doesn't exist  
**Solution:** 
- Ensure you're logged in
- Check that `DOSI/Users/[username]/Documents/` exists (should auto-create)
- Check file permissions

#### Issue: Plugins don't load
**Cause:** DLL not in correct location or doesn't implement IDosiPlugin  
**Solution:**
- Verify plugin is in `DOSI/Users/[username]/Applications/`
- Ensure plugin implements `IDosiPlugin`
- Check console output for plugin loading errors
- Verify plugin targets .NET 9.0

#### Issue: UI elements not rendering correctly
**Cause:** Layout issues or control properties not set correctly  
**Solution:**
- Ensure controls have `HorizontalAlignment` and `VerticalAlignment` set appropriately
- Set `Width`/`Height` or use `double.NaN` for auto-sizing
- Check that parent controls have sufficient space
- Use Grid or StackPanel for proper layout management

#### Issue: Accent theme doesn't apply to new windows
**Cause:** Theme set before window created or not propagated  
**Solution:**
- Ensure `AccentManager.SetTheme()` is called before creating UI
- Listen to `ThemeChanged` event to update existing windows
- Recreate controls when theme changes

#### Issue: Terminal commands not working
**Cause:** Command not registered or arguments incorrect  
**Solution:**
- Check if command is registered in `RegisterDefaultCommands()`
- Verify command method signature: `void Command(string[] args)`
- Type `help` in terminal to see available commands

### Debug Tips

1. **Enable Console Output**: Set output to console in project settings
2. **Add Breakpoints**: Use IDE debugger to pause and inspect state
3. **Log Statements**: Add `Console.WriteLine()` for tracking execution
4. **Check File System**: Manually browse `DOSI/` folder to verify file creation
5. **Monitor Events**: Add event handlers to verify callbacks are firing
6. **Exception Handling**: Catch and log exceptions to see what's failing

---

## API Reference

### Key Classes & Methods

#### AccentManager
```csharp
AccentManager.Instance              // Get singleton
.CurrentTheme                       // Get current theme
.AvailableThemes                    // Get all themes
.SetTheme(string name)              // Change theme
.ThemeChanged += handler            // Listen for changes
```

#### UserManager
```csharp
UserManager.Instance                // Get singleton
.CurrentUser                        // Get logged-in user
.IsAuthenticated                    // Check if logged in
.Login(username, password)          // Authenticate user
.CreateUser(username, password, accent)  // Create new user
.UserCreated += handler             // Listen for new users
```

#### WindowManager
```csharp
WindowManager.Instance              // Get singleton
.AllWindows                         // Get all open windows
.FocusedWindow                      // Get focused window
.WindowsInZOrder                    // Get windows front-to-back
.RegisterWindow(window)             // Track new window
.UnregisterWindow(window)           // Remove tracked window
.BringToFront(window)               // Focus and raise window
```

#### NotificationBubbleManager
```csharp
NotificationBubbleManager.Instance  // Get singleton (auto-created)
.ShowNotification(title, message, dismissTime)  // Show notification

// Or use helper
NotificationHelper.Notify(message)  // Show quick notification
```

#### PluginManager
```csharp
PluginManager.Instance              // Get singleton
.LoadedPlugins                      // Get plugins
.DiscoverAndLoadPlugins()           // Scan and load plugins
```

#### SystemSettings
```csharp
SystemSettings.Instance             // Get singleton
SystemSettings.SystemFolder         // Get DOSI folder path
.DefaultAccent                      // Get default theme
.Fullscreen                         // Check fullscreen setting
.Save()                             // Persist to JSON
```

#### DosiWindowUi
```csharp
window.WindowTitle = "Title"        // Set title
window.Width = 500                  // Set width
window.Height = 400                 // Set height
window.ContentArea                  // Add controls here
window.Close()                      // Close window
window.Minimize()                   // Minimize window
window.Maximize()                   // Maximize window
```

#### TerminalManager
```csharp
terminal.DisplayWelcomeMessage()    // Show help
terminal.RegisterCommand(name, action)  // Add command
terminal.ExecuteCommand(cmdLine)    // Run command
```

### Common Imports

```csharp
using Avalonia;
using Avalonia.Controls;
using Avalonia.Layout;
using Avalonia.Media;
using DOSI.Core.Controls;
using DOSI.Core.Accent;
using DOSI.Core.User;
using DOSI.Core.Settings;
using DOSI.Core.Notifications;
using DOSI.Core.Windowing;
using DOSI.Core.Plugins;
using DOSI.Core.Terminal;
```

---

## Project Configuration Files

### DAX.OSI.csproj
```xml
<Project Sdk="Microsoft.NET.Sdk">
    <PropertyGroup>
        <OutputType>WinExe</OutputType>
        <TargetFramework>net9.0</TargetFramework>
        <Nullable>enable</Nullable>
        <AvaloniaUseCompiledBindingsByDefault>true</AvaloniaUseCompiledBindingsByDefault>
    </PropertyGroup>
    <!-- References DOSI.Core -->
</Project>
```

### DOSI.Core.csproj
```xml
<Project Sdk="Microsoft.NET.Sdk">
    <PropertyGroup>
        <TargetFramework>net9.0</TargetFramework>
        <ImplicitUsings>enable</ImplicitUsings>
        <Nullable>enable</Nullable>
    </PropertyGroup>
    <!-- Contains embedded fonts and core controls -->
</Project>
```

---

## Performance Considerations

1. **Notifications**: Auto-dismiss prevents memory leaks, limit to ~10 visible
2. **Windows**: WindowManager tracks all open windows, closing unused windows is important
3. **Plugins**: Loaded on-demand, cached after load - consider unload strategy
4. **File Operations**: Use async methods to prevent UI freezing
5. **Drawing**: Avalonia handles rendering optimization
6. **Memory**: Dispose resources in window `OnClosed()` events

---

## Future Roadmap (Post-Beta)

- [ ] Implement password hashing (consider bcrypt)
- [ ] Add file encryption for user data
- [ ] Implement copy/paste to system clipboard
- [ ] Add file drag-and-drop support
- [ ] Create plugin marketplace/repository
- [ ] Implement system sounds/music
- [ ] Add file compression utilities
- [ ] Implement user preferences/settings dialog
- [ ] Add window snapping/tiling
- [ ] Create application store

---

## Support & Contact

For issues, questions, or contributions:
- Check the troubleshooting section above
- Review example code in existing applications
- Examine console output for error messages
- Check the API Reference section

---

**Document Version:** 1.0  
**Last Updated:** December 4, 2025  
**Status:** Beta Ready

