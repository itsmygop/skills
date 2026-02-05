# Tauri V2 Windows Platform Reference

## Table of Contents
- [WebView2 Installation Modes](#webview2-installation-modes)
- [NSIS Installer](#nsis-installer)
- [WiX MSI Installer](#wix-msi-installer)
- [Code Signing](#code-signing)
- [Windows-Specific APIs](#windows-specific-apis)

---

## WebView2 Installation Modes

Windows requires WebView2 Runtime (Chromium-based). Configure installation mode:

```json
{
  "bundle": {
    "windows": {
      "webviewInstallMode": {
        "type": "downloadBootstrapper",
        "silent": true
      }
    }
  }
}
```

| Mode | type | Size Impact | Requires Internet | Notes |
|------|------|-------------|-------------------|-------|
| Download Bootstrapper | `"downloadBootstrapper"` | ~0 | Yes | **Default**, downloads at install |
| Embed Bootstrapper | `"embedBootstrapper"` | +1.8MB | Yes | Better Win7 MSI compat |
| Offline Installer | `"offlineInstaller"` | +127MB | No | Fully offline |
| Fixed Runtime | `"fixedRuntime"` | +180MB | No | Embed specific version |
| Skip | `"skip"` | 0 | No | User must have WebView2 |

### Fixed Runtime Example

```json
{
  "bundle": {
    "windows": {
      "webviewInstallMode": {
        "type": "fixedRuntime",
        "path": "C:\\path\\to\\webview2_runtime"
      }
    }
  }
}
```

---

## NSIS Installer

Modern installer, supports cross-compilation from Linux/macOS.

```json
{
  "bundle": {
    "windows": {
      "nsis": {
        "installMode": "currentUser",
        "languages": ["en-US", "zh-CN", "ja-JP"],
        "installerIcon": "./icons/installer.ico",
        "sidebarImage": "./icons/sidebar.bmp",
        "headerImage": "./icons/header.bmp",
        "startMenuFolder": "My App",
        "minimumWebview2Version": "100.0.0.0",
        "template": "./custom-installer.nsi",
        "displayLanguageSelector": true
      }
    }
  }
}
```

### Install Modes

| Mode | Registry | Install Location | Admin Required |
|------|----------|------------------|----------------|
| `currentUser` | HKCU | User directory | No (**Default**) |
| `perMachine` | HKLM | Program Files | Yes |
| `both` | HKLM/HKCU | User choice | Yes |

### Cross-Compile from Linux/macOS

```bash
npm run tauri build -- --runner cargo-xwin --target x86_64-pc-windows-msvc
```

---

## WiX MSI Installer

Traditional Windows Installer format.

```json
{
  "bundle": {
    "windows": {
      "wix": {
        "language": "en-US",
        "bannerPath": "./assets/banner.bmp",
        "dialogImagePath": "./assets/dialog.bmp",
        "enableElevatedUpdateTask": true,
        "fragmentPaths": ["./wix/fragment.wxs"],
        "componentRefs": ["ComponentId1"],
        "featureRefs": ["FeatureId1"],
        "upgradeCode": "550e8400-e29b-41d4-a716-446655440000",
        "fipsCompliant": false,
        "template": "./wix/template.wxs"
      }
    }
  }
}
```

---

## Code Signing

### Using Certificate Thumbprint

```json
{
  "bundle": {
    "windows": {
      "certificateThumbprint": "A1B1A2B2A3B3A4B4A5B5A6B6A7B7A8B8A9B9A0B0",
      "digestAlgorithm": "sha256",
      "timestampUrl": "http://timestamp.comodoca.com",
      "tsp": false
    }
  }
}
```

### Using Custom Sign Command

```json
{
  "bundle": {
    "windows": {
      "signCommand": "signtool sign /sha1 %CERT_THUMBPRINT% /fd SHA256 /tr http://timestamp.server.com /td SHA256 %1"
    }
  }
}
```

### Azure Code Signing

```json
{
  "bundle": {
    "windows": {
      "signCommand": "trusted-signing-cli -e https://wus2.codesigning.azure.net -a MyAccount -c MyProfile -d MyApp %1"
    }
  }
}
```

### GitHub Actions Signing

```yaml
- uses: tauri-apps/tauri-action@v0
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
    TAURI_SIGNING_PRIVATE_KEY: ${{ secrets.TAURI_SIGNING_PRIVATE_KEY }}
    TAURI_SIGNING_PRIVATE_KEY_PASSWORD: ${{ secrets.TAURI_SIGNING_PRIVATE_KEY_PASSWORD }}
```

---

## Windows-Specific APIs

### Conditional Compilation

```rust
#[cfg(windows)]
fn windows_only() {
    // Windows-specific code
}

#[cfg(target_os = "windows")]
fn also_windows_only() {
    // Same as above
}
```

### Parent Window (HWND)

```rust
use windows::Win32::Foundation::HWND;

#[cfg(windows)]
impl WindowBuilder {
    fn parent(self, parent: HWND) -> Self;
    fn owner(self, owner: HWND) -> Self;
}
```

### Drag and Drop

```rust
// Must disable Tauri drag-drop to use HTML5 drag-drop on Windows
WebviewWindowBuilder::new(app, "main", url)
    .drag_and_drop(false)  // Windows-specific
    .build()?;
```

### Window Configuration

```json
{
  "app": {
    "windows": [
      {
        "label": "main",
        "dragDropEnabled": false
      }
    ]
  }
}
```

### Scrollbar Style (WebView2 125+)

```rust
// Fluent UI overlay scrollbars (Windows only, WebView2 125+)
WebviewWindowBuilder::new(app, "main", url)
    .scrollbar_style(ScrollbarStyle::FluentOverlay)
    .build()?;
```

---

## Complete Windows Bundle Config

```json
{
  "bundle": {
    "windows": {
      "allowDowngrades": true,
      "certificateThumbprint": null,
      "digestAlgorithm": "sha256",
      "timestampUrl": "http://timestamp.comodoca.com",
      "tsp": false,
      "webviewInstallMode": {
        "type": "downloadBootstrapper",
        "silent": true
      },
      "nsis": {
        "installMode": "currentUser",
        "languages": ["en-US"],
        "displayLanguageSelector": false
      },
      "wix": null
    }
  }
}
```
