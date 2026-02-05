# Tauri V2 Configuration Reference

## Table of Contents
- [Complete tauri.conf.json Structure](#complete-tauriconfjson-structure)
- [Build Configuration](#build-configuration)
- [App Configuration](#app-configuration)
- [Bundle Configuration](#bundle-configuration)
- [Plugins Configuration](#plugins-configuration)

---

## Complete tauri.conf.json Structure

```json
{
  "productName": "my-app",
  "version": "0.1.0",
  "identifier": "com.example.myapp",
  "build": {
    "beforeBuildCommand": "npm run build",
    "beforeDevCommand": "npm run dev",
    "devUrl": "http://localhost:3000",
    "frontendDist": "../dist"
  },
  "app": {
    "security": {
      "csp": "default-src 'self'; script-src 'self'",
      "dangerousDisableAssetCspModification": false
    },
    "windows": [
      {
        "label": "main",
        "title": "My App",
        "width": 800,
        "height": 600,
        "minWidth": 400,
        "minHeight": 300,
        "resizable": true,
        "fullscreen": false,
        "center": true,
        "decorations": true,
        "transparent": false,
        "visible": true
      }
    ],
    "trayIcon": {
      "iconPath": "icons/tray.png",
      "iconAsTemplate": true
    }
  },
  "bundle": {
    "active": true,
    "targets": "all",
    "icon": [
      "icons/32x32.png",
      "icons/128x128.png",
      "icons/icon.icns",
      "icons/icon.ico"
    ],
    "windows": { /* See windows.md */ },
    "macOS": { /* See macos.md */ },
    "linux": { /* See linux.md */ }
  },
  "plugins": {}
}
```

---

## Build Configuration

```json
{
  "build": {
    "beforeBuildCommand": "npm run build",
    "beforeDevCommand": "npm run dev",
    "devUrl": "http://localhost:3000",
    "frontendDist": "../dist"
  }
}
```

| Field | Description |
|-------|-------------|
| `beforeBuildCommand` | Command to run before `tauri build` |
| `beforeDevCommand` | Command to run before `tauri dev` |
| `devUrl` | Dev server URL for hot reload |
| `frontendDist` | Path to built frontend assets |

---

## App Configuration

### Window Configuration

```json
{
  "app": {
    "windows": [
      {
        "label": "main",
        "title": "Window Title",
        "width": 800,
        "height": 600,
        "minWidth": 400,
        "minHeight": 300,
        "maxWidth": 1920,
        "maxHeight": 1080,
        "x": 100,
        "y": 100,
        "center": true,
        "resizable": true,
        "fullscreen": false,
        "decorations": true,
        "transparent": false,
        "alwaysOnTop": false,
        "visible": true,
        "focus": true,
        "skipTaskbar": false,
        "fileDropEnabled": true,
        "url": "index.html"
      }
    ]
  }
}
```

### Security Configuration

```json
{
  "app": {
    "security": {
      "csp": "default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'",
      "dangerousDisableAssetCspModification": false,
      "freezePrototype": true
    }
  }
}
```

### Tray Configuration

```json
{
  "app": {
    "trayIcon": {
      "iconPath": "icons/tray.png",
      "iconAsTemplate": true,
      "menuOnLeftClick": false
    }
  }
}
```

---

## Bundle Configuration

### Targets

```json
{
  "bundle": {
    "targets": "all"
  }
}
```

Available targets:
- `"all"` - All supported formats for current platform
- `["nsis", "msi"]` - Specific formats

| Platform | Formats |
|----------|---------|
| Windows | `nsis`, `msi` |
| macOS | `app`, `dmg` |
| Linux | `deb`, `rpm`, `appimage` |

### Icons

```json
{
  "bundle": {
    "icon": [
      "icons/32x32.png",
      "icons/128x128.png",
      "icons/128x128@2x.png",
      "icons/icon.icns",
      "icons/icon.ico"
    ]
  }
}
```

### Resources (Bundle Extra Files)

```json
{
  "bundle": {
    "resources": [
      "assets/*",
      "data/config.json"
    ]
  }
}
```

### External Binaries

```json
{
  "bundle": {
    "externalBin": [
      "binaries/ffmpeg"
    ]
  }
}
```

---

## Plugins Configuration

```json
{
  "plugins": {
    "fs": {
      "scope": ["$APP/*", "$RESOURCE/*"]
    },
    "shell": {
      "open": true,
      "scope": [
        { "name": "explorer", "cmd": "explorer" }
      ]
    },
    "http": {
      "scope": ["https://api.example.com/*"]
    }
  }
}
```

---

## Cargo.toml Features

```toml
[dependencies]
tauri = { version = "2.9.5", features = [
    "tray-icon",           # System tray support
    "protocol-asset",      # Asset protocol
    "dynamic-acl",         # Dynamic ACL (default)
    "linux-libxdo",        # Linux clipboard support
    "macos-proxy",         # macOS proxy (requires 14+)
    "specta",              # Specta integration
    "devtools",            # Enable devtools in release
] }
```
