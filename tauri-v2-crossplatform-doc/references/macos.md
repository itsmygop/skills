# Tauri V2 macOS Platform Reference

## Table of Contents
- [DMG Configuration](#dmg-configuration)
- [Code Signing](#code-signing)
- [Notarization](#notarization)
- [Entitlements](#entitlements)
- [macOS-Specific APIs](#macos-specific-apis)
- [Universal Binary](#universal-binary)

---

## DMG Configuration

```json
{
  "bundle": {
    "macOS": {
      "dmg": {
        "appPosition": { "x": 180, "y": 170 },
        "applicationFolderPosition": { "x": 480, "y": 170 },
        "windowSize": { "width": 660, "height": 400 },
        "windowPosition": { "x": 100, "y": 100 },
        "background": "./assets/dmg-background.png"
      },
      "minimumSystemVersion": "10.13",
      "hardenedRuntime": true,
      "signingIdentity": "Developer ID Application",
      "entitlements": "./entitlements.plist",
      "frameworks": ["CoreLocation", "AVFoundation"],
      "infoPlist": "./Info.plist"
    }
  }
}
```

---

## Code Signing

### Environment Variables

```bash
export APPLE_SIGNING_IDENTITY="Developer ID Application: Your Name (TEAMID)"
export APPLE_CERTIFICATE="base64-encoded-certificate"
export APPLE_CERTIFICATE_PASSWORD="certificate-password"
```

### GitHub Actions

```yaml
jobs:
  build-macos:
    strategy:
      matrix:
        include:
          - args: '--target aarch64-apple-darwin'
            arch: 'silicon'
          - args: '--target x86_64-apple-darwin'
            arch: 'intel'
    runs-on: macos-latest
    env:
      APPLE_ID: ${{ secrets.APPLE_ID }}
      APPLE_PASSWORD: ${{ secrets.APPLE_PASSWORD }}
    steps:
      - name: Import Apple Developer Certificate
        env:
          APPLE_CERTIFICATE: ${{ secrets.APPLE_CERTIFICATE }}
          APPLE_CERTIFICATE_PASSWORD: ${{ secrets.APPLE_CERTIFICATE_PASSWORD }}
          KEYCHAIN_PASSWORD: ${{ secrets.KEYCHAIN_PASSWORD }}
        run: |
          echo $APPLE_CERTIFICATE | base64 --decode > certificate.p12
          security create-keychain -p "$KEYCHAIN_PASSWORD" build.keychain
          security default-keychain -s build.keychain
          security unlock-keychain -p "$KEYCHAIN_PASSWORD" build.keychain
          security set-keychain-settings -t 3600 -u build.keychain
          security import certificate.p12 -k build.keychain \
            -P "$APPLE_CERTIFICATE_PASSWORD" -T /usr/bin/codesign
          security set-key-partition-list -S apple-tool:,apple:,codesign: \
            -s -k "$KEYCHAIN_PASSWORD" build.keychain

      - name: Verify Certificate
        run: |
          CERT_INFO=$(security find-identity -v -p codesigning build.keychain | grep "Apple Development")
          CERT_ID=$(echo "$CERT_INFO" | awk -F'"' '{print $2}')
          echo "CERT_ID=$CERT_ID" >> $GITHUB_ENV

      - uses: tauri-apps/tauri-action@v0
        env:
          APPLE_SIGNING_IDENTITY: ${{ env.CERT_ID }}
        with:
          args: ${{ matrix.args }}
```

---

## Notarization

Requires Apple ID with App-Specific Password.

```bash
export APPLE_ID="your@email.com"
export APPLE_PASSWORD="app-specific-password"
export APPLE_TEAM_ID="TEAMID"
```

---

## Entitlements

Create `entitlements.plist`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
  "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <!-- App Sandbox (required for Mac App Store) -->
    <key>com.apple.security.app-sandbox</key>
    <true/>

    <!-- Network access -->
    <key>com.apple.security.network.client</key>
    <true/>
    <key>com.apple.security.network.server</key>
    <true/>

    <!-- File access -->
    <key>com.apple.security.files.user-selected.read-write</key>
    <true/>
    <key>com.apple.security.files.downloads.read-write</key>
    <true/>

    <!-- Camera/Microphone -->
    <key>com.apple.security.device.camera</key>
    <true/>
    <key>com.apple.security.device.microphone</key>
    <true/>

    <!-- Hardened Runtime -->
    <key>com.apple.security.cs.allow-jit</key>
    <true/>
    <key>com.apple.security.cs.allow-unsigned-executable-memory</key>
    <true/>
</dict>
</plist>
```

Reference in config:

```json
{
  "bundle": {
    "macOS": {
      "entitlements": "./entitlements.plist",
      "hardenedRuntime": true
    }
  }
}
```

---

## macOS-Specific APIs

### Conditional Compilation

```rust
#[cfg(target_os = "macos")]
fn macos_only() {
    // macOS-specific code
}
```

### Title Bar Styles

```rust
use tauri::TitleBarStyle;

WebviewWindowBuilder::new(app, "main", url)
    .title_bar_style(TitleBarStyle::Transparent)  // Transparent, Overlay, or Visible
    .hidden_title(true)
    .build()?;
```

Available styles:
- `TitleBarStyle::Visible` - Default macOS title bar
- `TitleBarStyle::Transparent` - Transparent title bar
- `TitleBarStyle::Overlay` - Content extends under title bar

### Traffic Light Position

```rust
use tauri::{LogicalPosition, Position};

#[cfg(target_os = "macos")]
WebviewWindowBuilder::new(app, "main", url)
    .title_bar_style(TitleBarStyle::Overlay)
    .traffic_light_position(LogicalPosition::new(20.0, 20.0))
    .build()?;
```

### Tabbing Identifier

```rust
#[cfg(target_os = "macos")]
WebviewWindowBuilder::new(app, "main", url)
    .tabbing_identifier("main-group")  // Group windows in tabs
    .build()?;
```

### Parent Window

```rust
#[cfg(target_os = "macos")]
impl WindowBuilder {
    fn parent(self, parent: *mut std::ffi::c_void) -> Self;
}
```

### Native NSWindow Access

```rust
#[cfg(target_os = "macos")]
{
    use cocoa::appkit::{NSColor, NSWindow};
    use cocoa::base::{id, nil};

    let ns_window = window.ns_window().unwrap() as id;
    unsafe {
        let bg_color = NSColor::colorWithRed_green_blue_alpha_(
            nil,
            50.0 / 255.0,
            158.0 / 255.0,
            163.5 / 255.0,
            1.0,
        );
        ns_window.setBackgroundColor_(bg_color);
    }
}
```

### WebView Properties (macOS-specific)

| Property | Description |
|----------|-------------|
| `acceptFirstMouse` | Click inactive webview also clicks through |
| `allowLinkPreview` | Long-press link preview (default: true) |
| `dataStoreIdentifier` | Custom data store (macOS >= 14) |
| `backgroundThrottling` | Background throttling policy (macOS >= 14) |

---

## Universal Binary

Build for both Intel and Apple Silicon:

```bash
# Apple Silicon
cargo tauri build --target aarch64-apple-darwin

# Intel
cargo tauri build --target x86_64-apple-darwin

# Or in CI, build both
```

### GitHub Actions Matrix

```yaml
strategy:
  matrix:
    include:
      - args: '--target aarch64-apple-darwin'
        arch: 'silicon'
      - args: '--target x86_64-apple-darwin'
        arch: 'intel'
```

---

## Complete macOS Bundle Config

```json
{
  "bundle": {
    "macOS": {
      "bundleName": "My App",
      "bundleVersion": "1.0.0",
      "minimumSystemVersion": "10.13",
      "hardenedRuntime": true,
      "signingIdentity": "Developer ID Application",
      "providerShortName": "TEAMID",
      "entitlements": "./entitlements.plist",
      "exceptionDomain": null,
      "infoPlist": "./Info.plist",
      "frameworks": [],
      "dmg": {
        "appPosition": { "x": 180, "y": 170 },
        "applicationFolderPosition": { "x": 480, "y": 170 },
        "windowSize": { "width": 660, "height": 400 },
        "background": "./assets/dmg-background.png"
      }
    }
  }
}
```
