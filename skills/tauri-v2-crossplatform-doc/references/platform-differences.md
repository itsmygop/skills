# Tauri V2 Platform Differences Reference

## Table of Contents
- [WebView Engine Comparison](#webview-engine-comparison)
- [Bundle Format Comparison](#bundle-format-comparison)
- [API Availability Matrix](#api-availability-matrix)
- [Conditional Compilation Guide](#conditional-compilation-guide)
- [Platform-Specific Considerations](#platform-specific-considerations)

---

## WebView Engine Comparison

| Platform | Engine | Base | Pre-installed | Notes |
|----------|--------|------|---------------|-------|
| **Windows** | WebView2 | Chromium | Win10 1803+ | Requires runtime installation on older systems |
| **macOS** | WKWebView | WebKit | Always | System component |
| **Linux** | WebKitGTK 4.1 | WebKit | No | Must install via package manager |

### WebView Property Support

| Property | Windows | macOS | Linux |
|----------|---------|-------|-------|
| `acceptFirstMouse` | ✗ | ✓ | ✗ |
| `allowLinkPreview` | ✗ | ✓ (also iOS) | ✗ |
| `backgroundColor` | ✓ (alpha ignored Win8+) | ✗ | ✓ |
| `backgroundThrottling` | ✗ | ✓ (14.0+) | ✗ |
| `dataDirectory` | ✓ | ✗ | ✓ |
| `dataStoreIdentifier` | ✗ | ✓ (14.0+) | ✗ |
| `devtools` | ✓ | ✓ (private API) | ✓ |
| `dragDropEnabled` | ✓ (disable for HTML5 D&D) | ✓ | ✓ |
| `incognito` | ✓ | ✓ | ✓ |
| `scrollbarStyle: fluentOverlay` | ✓ (WebView2 125+) | ✗ | ✗ |

---

## Bundle Format Comparison

| Format | Platform | Extension | Notes |
|--------|----------|-----------|-------|
| **NSIS** | Windows | `.exe` | Modern, cross-compilable |
| **WiX/MSI** | Windows | `.msi` | Traditional Windows Installer |
| **.app** | macOS | `.app` | Application bundle |
| **DMG** | macOS | `.dmg` | Disk image for distribution |
| **deb** | Linux | `.deb` | Debian/Ubuntu |
| **rpm** | Linux | `.rpm` | Fedora/RHEL/openSUSE |
| **AppImage** | Linux | `.AppImage` | Portable, no install |

### Code Signing Requirements

| Platform | Required For | Tool |
|----------|--------------|------|
| **Windows** | SmartScreen bypass, trust | signtool, Azure Code Signing |
| **macOS** | Gatekeeper, distribution | codesign, notarytool |
| **Linux** | Not required | N/A |

---

## API Availability Matrix

### Window APIs

| API | Windows | macOS | Linux |
|-----|---------|-------|-------|
| `title_bar_style()` | ✗ | ✓ | ✗ |
| `traffic_light_position()` | ✗ | ✓ | ✗ |
| `hidden_title()` | ✗ | ✓ | ✗ |
| `tabbing_identifier()` | ✗ | ✓ | ✗ |
| `drag_and_drop()` | ✓ | ✓ | ✓ |
| `parent(HWND)` | ✓ | ✗ | ✗ |
| `owner(HWND)` | ✓ | ✗ | ✗ |
| `parent(*mut c_void)` | ✗ | ✓ | ✗ |
| `transient_for(GtkWindow)` | ✗ | ✗ | ✓ |

### System Features

| Feature | Windows | macOS | Linux |
|---------|---------|-------|-------|
| System Tray | ✓ | ✓ | ✓ (requires libayatana-appindicator) |
| Native Menu | ✓ | ✓ | ✓ |
| Global Shortcuts | ✓ | ✓ | ✓ |
| Notifications | ✓ | ✓ | ✓ |
| Clipboard | ✓ | ✓ | ✓ (requires libxdo feature) |

---

## Conditional Compilation Guide

### Platform Detection

```rust
// Single platform
#[cfg(target_os = "windows")]
#[cfg(target_os = "macos")]
#[cfg(target_os = "linux")]

// Desktop (all desktop platforms)
#[cfg(desktop)]

// Windows (shorter form)
#[cfg(windows)]

// Unix-like (Linux + BSD variants)
#[cfg(any(
    target_os = "linux",
    target_os = "dragonfly",
    target_os = "freebsd",
    target_os = "netbsd",
    target_os = "openbsd"
))]

// Not a specific platform
#[cfg(not(target_os = "windows"))]
```

### Build Mode Detection

```rust
// Development mode (tauri dev)
#[cfg(dev)]

// Production mode (tauri build)
#[cfg(not(dev))]

// Debug build (tauri dev OR tauri build --debug)
#[cfg(debug_assertions)]

// Release build
#[cfg(not(debug_assertions))]

// Runtime check
fn main() {
    if cfg!(dev) {
        println!("Development mode");
    }

    let is_dev: bool = tauri::is_dev();
}
```

### Feature Detection

```rust
// Tray icon feature
#[cfg(all(desktop, feature = "tray-icon"))]

// Wry feature (WebView)
#[cfg(feature = "wry")]
```

### Full Example

```rust
use tauri::{WebviewWindowBuilder, WebviewUrl};

pub fn create_window(app: &tauri::AppHandle) -> tauri::Result<()> {
    let mut builder = WebviewWindowBuilder::new(
        app,
        "main",
        WebviewUrl::App("index.html".into())
    )
    .title("My App")
    .inner_size(800.0, 600.0);

    // macOS: transparent titlebar with custom traffic light position
    #[cfg(target_os = "macos")]
    {
        use tauri::{TitleBarStyle, LogicalPosition};
        builder = builder
            .title_bar_style(TitleBarStyle::Transparent)
            .hidden_title(true)
            .traffic_light_position(LogicalPosition::new(20.0, 20.0));
    }

    // Windows: disable drag-drop for HTML5 compatibility
    #[cfg(windows)]
    {
        builder = builder.drag_and_drop(false);
    }

    // Linux: set transient parent if available
    #[cfg(target_os = "linux")]
    {
        // Linux-specific setup
    }

    builder.build()?;
    Ok(())
}
```

---

## Platform-Specific Considerations

### Windows

| Consideration | Details |
|---------------|---------|
| WebView2 Runtime | Required; configure installation mode |
| Admin Rights | `perMachine` install requires elevation |
| Code Signing | Recommended for SmartScreen |
| Drag & Drop | Must disable for HTML5 drag-drop |
| Min OS Version | Windows 7+ (with WebView2) |

### macOS

| Consideration | Details |
|---------------|---------|
| Code Signing | Required for distribution |
| Notarization | Required for distribution outside App Store |
| Entitlements | Required for sandboxed features |
| Universal Binary | Build for both `aarch64` and `x86_64` |
| Min OS Version | macOS 10.13+ |
| Traffic Lights | Custom positioning with `Overlay` title bar |

### Linux

| Consideration | Details |
|---------------|---------|
| System Dependencies | Must install webkit2gtk-4.1, etc. |
| Clipboard | Requires `linux-libxdo` feature |
| Tray Icon | Requires libayatana-appindicator |
| Multiple Formats | deb, rpm, AppImage for distribution |
| GTK | Uses GTK for windowing |
| Flatpak | Supported via webkit2gtk-4.1 |

---

## Quick Reference: What Works Where

```
Feature                    Windows    macOS    Linux
─────────────────────────────────────────────────────
WebView2/WKWebView/GTK       ✓          ✓        ✓
System Tray                  ✓          ✓        ✓*
Native Menus                 ✓          ✓        ✓
Transparent Titlebar         ✗          ✓        ✗
Traffic Light Position       ✗          ✓        ✗
Window Tabbing               ✗          ✓        ✗
HWND Parent Window           ✓          ✗        ✗
GTK Transient Window         ✗          ✗        ✓
Fluent Scrollbars            ✓**        ✗        ✗
Background Throttling        ✗          ✓***     ✗
Code Signing Required        No         Yes      No

* Requires libayatana-appindicator
** Requires WebView2 125+
*** Requires macOS 14.0+
```
