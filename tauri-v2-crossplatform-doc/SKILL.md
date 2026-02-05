---
name: tauri-v2-crossplatform-doc
description: |
  Tauri V2 desktop application development reference and guide. Use when:
  (1) Building cross-platform desktop apps with Tauri V2
  (2) Configuring tauri.conf.json, Cargo.toml, or capabilities
  (3) Writing Rust backend code (commands, state, events, plugins)
  (4) Setting up platform-specific features (Windows/macOS/Linux)
  (5) Configuring installers (NSIS, WiX, DMG, deb, rpm, AppImage)
  (6) Setting up CI/CD pipelines with GitHub Actions
  (7) Code signing and notarization
  (8) Using official Tauri plugins
  (9) Understanding WebView differences across platforms
  (10) Conditional compilation with #[cfg] attributes
  Trigger keywords: tauri, tauri v2, tauri 2, desktop app, cross-platform, webview, webkit, webview2
---

# Tauri V2 Development Guide

Quick reference for Tauri V2.9.5 desktop application development.

## Quick Start

```rust
// main.rs
fn main() {
    tauri::Builder::default()
        .setup(|app| {
            // Initialize app
            Ok(())
        })
        .manage(AppState::default())
        .invoke_handler(tauri::generate_handler![my_command])
        .plugin(tauri_plugin_fs::init())
        .run(tauri::generate_context!())
        .expect("error running app");
}

#[tauri::command]
fn my_command(state: tauri::State<AppState>) -> String {
    "Hello from Tauri!".into()
}
```

## Reference Navigation

| Topic | Reference File | When to Use |
|-------|----------------|-------------|
| **Core API** | [core-api.md](references/core-api.md) | Builder, commands, state, events, windows, plugins |
| **Configuration** | [config.md](references/config.md) | tauri.conf.json structure, Cargo features |
| **Capabilities** | [capabilities.md](references/capabilities.md) | Permissions, security, capability files |
| **Windows** | [windows.md](references/windows.md) | NSIS/WiX, WebView2, code signing |
| **macOS** | [macos.md](references/macos.md) | DMG, entitlements, notarization, traffic lights |
| **Linux** | [linux.md](references/linux.md) | System deps, deb/rpm/AppImage |
| **CI/CD** | [ci-cd.md](references/ci-cd.md) | GitHub Actions, cross-platform builds |
| **Plugins** | [plugins.md](references/plugins.md) | Official plugins (fs, dialog, shell, http, etc.) |
| **Platform Diff** | [platform-differences.md](references/platform-differences.md) | WebView engines, API availability matrix |

## Essential Patterns

### Command with State

```rust
use std::sync::Mutex;
use tauri::State;

struct AppState { counter: Mutex<i32> }

#[tauri::command]
fn increment(state: State<AppState>) -> i32 {
    let mut c = state.counter.lock().unwrap();
    *c += 1;
    *c
}
```

### Event Communication

```rust
// Rust: emit to frontend
app.emit("backend-event", payload)?;

// Rust: listen from frontend
app.listen("frontend-event", |event| {
    println!("{:?}", event.payload());
});
```

### Platform-Specific Code

```rust
#[cfg(target_os = "macos")]
{
    builder = builder.title_bar_style(TitleBarStyle::Transparent);
}

#[cfg(windows)]
{
    builder = builder.drag_and_drop(false);
}

#[cfg(target_os = "linux")]
{
    // Linux-specific setup
}
```

## Platform Quick Reference

| | Windows | macOS | Linux |
|-|---------|-------|-------|
| **WebView** | WebView2 (Chromium) | WKWebView | WebKitGTK 4.1 |
| **Installers** | NSIS (.exe), WiX (.msi) | .app, .dmg | .deb, .rpm, .AppImage |
| **Signing** | Optional (recommended) | Required | Not needed |
| **Min OS** | Win7+ (w/ WebView2) | macOS 10.13+ | Varies |

## Common Tasks

### Create New Window

```rust
use tauri::{WebviewWindowBuilder, WebviewUrl};

WebviewWindowBuilder::new(app, "new-window", WebviewUrl::App("page.html".into()))
    .title("New Window")
    .inner_size(800.0, 600.0)
    .build()?;
```

### Add Plugin

```toml
# Cargo.toml
[dependencies]
tauri-plugin-fs = "2"
```

```rust
// main.rs
tauri::Builder::default()
    .plugin(tauri_plugin_fs::init())
```

```json
// capabilities/main.json
{ "permissions": ["fs:default"] }
```

### Capability File

```json
{
  "$schema": "https://schemas.tauri.app/capability.schema.json",
  "identifier": "main",
  "windows": ["main"],
  "permissions": [
    "core:default",
    "fs:default",
    "dialog:default",
    "shell:allow-open"
  ]
}
```

## Build Commands

```bash
# Development
npm run tauri dev

# Production build
npm run tauri build

# Specific target
npm run tauri build -- --target x86_64-apple-darwin

# Cross-compile Windows from Linux/macOS
npm run tauri build -- --runner cargo-xwin --target x86_64-pc-windows-msvc
```

## Troubleshooting

| Issue | Solution |
|-------|----------|
| WebView2 not found (Windows) | Configure `webviewInstallMode` in config |
| Signing failed (macOS) | Check `APPLE_SIGNING_IDENTITY` env var |
| Missing deps (Linux) | Install `libwebkit2gtk-4.1-dev` |
| HTML5 drag-drop broken (Windows) | Set `dragDropEnabled: false` |
| Clipboard not working (Linux) | Enable `linux-libxdo` feature |
| Tray not showing (Linux) | Install `libayatana-appindicator3-dev` |
