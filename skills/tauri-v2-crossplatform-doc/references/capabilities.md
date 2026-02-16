# Tauri V2 Capabilities & Permissions

## Table of Contents
- [Overview](#overview)
- [Capability Files](#capability-files)
- [CapabilityBuilder API](#capabilitybuilder-api)
- [Platform-Specific Capabilities](#platform-specific-capabilities)
- [Common Permissions](#common-permissions)

---

## Overview

Tauri V2 uses a capability-based security model. Capabilities define what permissions a window/webview has access to.

**Location:** `src-tauri/capabilities/*.json`

---

## Capability Files

### Basic Capability

```json
{
  "$schema": "https://schemas.tauri.app/capability.schema.json",
  "identifier": "main-capability",
  "description": "Main window permissions",
  "local": true,
  "windows": ["main"],
  "permissions": [
    "core:default",
    "fs:default",
    "path:default",
    "event:default",
    "window:default"
  ]
}
```

### Scoped Permission

```json
{
  "identifier": "fs-read-limited",
  "description": "Limited file system read access",
  "local": true,
  "windows": ["main"],
  "permissions": [
    {
      "identifier": "fs:allow-read",
      "allow": [
        { "path": "$APP/*" },
        { "path": "$RESOURCE/*" }
      ],
      "deny": [
        { "path": "$HOME/.ssh/*" }
      ]
    }
  ]
}
```

### Remote URL Capability

```json
{
  "identifier": "remote-api",
  "description": "Allow remote API access",
  "remote": {
    "urls": ["https://api.example.com/*"]
  },
  "permissions": [
    "http:default"
  ]
}
```

---

## CapabilityBuilder API

Runtime capability creation (requires `dynamic-acl` feature):

```rust
use tauri::CapabilityBuilder;
use serde::Serialize;

#[derive(Serialize)]
struct FileScope {
    path: String,
}

// Basic capability
let cap = CapabilityBuilder::new("runtime-cap")
    .window("main")
    .permission("fs:default")
    .permission("path:default")
    .build();

// Scoped permission
let cap = CapabilityBuilder::new("scoped-fs")
    .window("editor")
    .permission_scoped(
        "fs:read",
        vec![FileScope { path: "/data/".into() }],    // allowed
        vec![FileScope { path: "/secret/".into() }]   // denied
    )
    .build();

// Platform-specific
let cap = CapabilityBuilder::new("linux-only")
    .window("main")
    .platform(Target::Linux)
    .permission("shell:default")
    .build();

// Add to app
app.add_capability(cap)?;
```

---

## Platform-Specific Capabilities

```json
{
  "identifier": "desktop-only",
  "description": "Desktop platform permissions",
  "local": true,
  "windows": ["main"],
  "platforms": ["linux", "windows", "macOS"],
  "permissions": [
    "shell:allow-open",
    "dialog:default"
  ]
}
```

Supported platforms: `"linux"`, `"windows"`, `"macOS"`, `"android"`, `"iOS"`

---

## Common Permissions

### Core Permissions

| Permission | Description |
|------------|-------------|
| `core:default` | Core Tauri functionality |
| `event:default` | Event system |
| `window:default` | Window management |
| `webview:default` | Webview control |
| `path:default` | Path resolution |
| `resources:default` | Resource access |
| `image:default` | Image processing |
| `menu:default` | Menu system (desktop) |
| `tray:default` | System tray (desktop) |

### File System

| Permission | Description |
|------------|-------------|
| `fs:default` | Default FS operations |
| `fs:allow-read` | Read files |
| `fs:allow-write` | Write files |
| `fs:allow-exists` | Check file existence |
| `fs:allow-mkdir` | Create directories |
| `fs:allow-remove` | Delete files |
| `fs:allow-rename` | Rename files |
| `fs:allow-copy-file` | Copy files |
| `fs:allow-home-read` | Read home directory |
| `fs:allow-home-write` | Write to home directory |

### Dialog

| Permission | Description |
|------------|-------------|
| `dialog:default` | All dialog operations |
| `dialog:allow-open` | Open file dialog |
| `dialog:allow-save` | Save file dialog |
| `dialog:allow-message` | Message dialog |
| `dialog:allow-ask` | Ask dialog |
| `dialog:allow-confirm` | Confirm dialog |

### Shell

| Permission | Description |
|------------|-------------|
| `shell:default` | Default shell operations |
| `shell:allow-open` | Open URLs/files with default app |
| `shell:allow-execute` | Execute commands |
| `shell:allow-spawn` | Spawn child processes |

### HTTP

| Permission | Description |
|------------|-------------|
| `http:default` | HTTP client operations |
| `http:allow-fetch` | Fetch requests |

### Notification

| Permission | Description |
|------------|-------------|
| `notification:default` | Notification operations |
| `notification:allow-show` | Show notifications |
| `notification:allow-request-permission` | Request permission |

### Clipboard

| Permission | Description |
|------------|-------------|
| `clipboard-manager:default` | Clipboard operations |
| `clipboard-manager:allow-read` | Read clipboard |
| `clipboard-manager:allow-write` | Write clipboard |

### Global Shortcut

| Permission | Description |
|------------|-------------|
| `global-shortcut:default` | Global shortcut operations |
| `global-shortcut:allow-register` | Register shortcuts |
| `global-shortcut:allow-unregister` | Unregister shortcuts |

---

## Path Variables

Use in scope definitions:

| Variable | Description |
|----------|-------------|
| `$APP` | App data directory |
| `$APPCONFIG` | App config directory |
| `$APPDATA` | App data directory |
| `$APPLOCALDATA` | App local data directory |
| `$APPCACHE` | App cache directory |
| `$APPLOG` | App log directory |
| `$RESOURCE` | Resource directory |
| `$HOME` | User home directory |
| `$DESKTOP` | Desktop directory |
| `$DOCUMENT` | Documents directory |
| `$DOWNLOAD` | Downloads directory |
| `$PICTURE` | Pictures directory |
| `$VIDEO` | Videos directory |
| `$AUDIO` | Audio directory |
| `$TEMP` | Temp directory |
