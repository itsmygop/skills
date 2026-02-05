# Tauri V2 Official Plugins Reference

## Table of Contents
- [Plugin Installation](#plugin-installation)
- [Core Plugins](#core-plugins)
- [Official Plugin List](#official-plugin-list)
- [Plugin Configuration](#plugin-configuration)

---

## Plugin Installation

### Cargo.toml

```toml
[dependencies]
tauri-plugin-fs = "2"
tauri-plugin-dialog = "2"
tauri-plugin-shell = "2"
tauri-plugin-http = "2"
tauri-plugin-notification = "2"
tauri-plugin-store = "2"
```

### Register in main.rs

```rust
fn main() {
    tauri::Builder::default()
        .plugin(tauri_plugin_fs::init())
        .plugin(tauri_plugin_dialog::init())
        .plugin(tauri_plugin_shell::init())
        .plugin(tauri_plugin_http::init())
        .plugin(tauri_plugin_notification::init())
        .plugin(tauri_plugin_store::init())
        .run(tauri::generate_context!())
        .expect("error running app");
}
```

### Frontend Package

```bash
npm install @tauri-apps/plugin-fs
npm install @tauri-apps/plugin-dialog
npm install @tauri-apps/plugin-shell
npm install @tauri-apps/plugin-http
npm install @tauri-apps/plugin-notification
npm install @tauri-apps/plugin-store
```

---

## Core Plugins

Built into Tauri core (no additional installation):

| Plugin | Description | Desktop | Mobile |
|--------|-------------|---------|--------|
| `path` | Path resolution | ✓ | ✓ |
| `event` | Event system | ✓ | ✓ |
| `window` | Window management | ✓ | ✓ |
| `webview` | Webview control | ✓ | ✓ |
| `resources` | Resource access | ✓ | ✓ |
| `image` | Image processing | ✓ | ✓ |
| `menu` | Menu system | ✓ | ✗ |
| `tray` | System tray | ✓ | ✗ |

---

## Official Plugin List

### File System (`tauri-plugin-fs`)

```rust
// Rust
tauri::Builder::default()
    .plugin(tauri_plugin_fs::init())
```

```typescript
// JavaScript
import { readTextFile, writeTextFile, readDir } from '@tauri-apps/plugin-fs';

const content = await readTextFile('path/to/file.txt');
await writeTextFile('path/to/file.txt', 'content');
const entries = await readDir('path/to/dir');
```

**Permissions:** `fs:default`, `fs:allow-read`, `fs:allow-write`

---

### Dialog (`tauri-plugin-dialog`)

```rust
tauri::Builder::default()
    .plugin(tauri_plugin_dialog::init())
```

```typescript
import { open, save, message, ask, confirm } from '@tauri-apps/plugin-dialog';

// File dialogs
const file = await open({ multiple: false, filters: [{ name: 'Text', extensions: ['txt'] }] });
const savePath = await save({ filters: [{ name: 'Text', extensions: ['txt'] }] });

// Message dialogs
await message('Hello!', { title: 'Info', kind: 'info' });
const answer = await ask('Continue?', { title: 'Confirm', kind: 'warning' });
const confirmed = await confirm('Are you sure?');
```

**Permissions:** `dialog:default`, `dialog:allow-open`, `dialog:allow-save`, `dialog:allow-message`

---

### Shell (`tauri-plugin-shell`)

```rust
tauri::Builder::default()
    .plugin(tauri_plugin_shell::init())
```

```typescript
import { Command, open } from '@tauri-apps/plugin-shell';

// Open URL/file with default app
await open('https://tauri.app');
await open('/path/to/file.pdf');

// Execute command
const output = await Command.create('echo', ['Hello']).execute();
console.log(output.stdout);

// Spawn process
const child = await Command.create('long-running-process').spawn();
child.on('close', (data) => console.log('Closed with', data.code));
```

**Permissions:** `shell:default`, `shell:allow-open`, `shell:allow-execute`, `shell:allow-spawn`

---

### HTTP (`tauri-plugin-http`)

```rust
tauri::Builder::default()
    .plugin(tauri_plugin_http::init())
```

```typescript
import { fetch } from '@tauri-apps/plugin-http';

const response = await fetch('https://api.example.com/data', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ key: 'value' })
});
const data = await response.json();
```

**Permissions:** `http:default`, `http:allow-fetch`

---

### Notification (`tauri-plugin-notification`)

```rust
tauri::Builder::default()
    .plugin(tauri_plugin_notification::init())
```

```typescript
import { sendNotification, requestPermission, isPermissionGranted } from '@tauri-apps/plugin-notification';

if (!(await isPermissionGranted())) {
    await requestPermission();
}

sendNotification({ title: 'Hello', body: 'World!' });
```

**Permissions:** `notification:default`, `notification:allow-show`

---

### Store (`tauri-plugin-store`)

Persistent key-value storage.

```rust
tauri::Builder::default()
    .plugin(tauri_plugin_store::Builder::new().build())
```

```typescript
import { Store } from '@tauri-apps/plugin-store';

const store = new Store('settings.json');
await store.set('theme', 'dark');
const theme = await store.get('theme');
await store.save();
```

---

### SQL (`tauri-plugin-sql`)

SQLite, MySQL, PostgreSQL support.

```rust
tauri::Builder::default()
    .plugin(tauri_plugin_sql::Builder::default().build())
```

```typescript
import Database from '@tauri-apps/plugin-sql';

const db = await Database.load('sqlite:app.db');
await db.execute('CREATE TABLE IF NOT EXISTS users (id INTEGER PRIMARY KEY, name TEXT)');
await db.execute('INSERT INTO users (name) VALUES (?)', ['Alice']);
const users = await db.select('SELECT * FROM users');
```

---

### Clipboard Manager (`tauri-plugin-clipboard-manager`)

```rust
tauri::Builder::default()
    .plugin(tauri_plugin_clipboard_manager::init())
```

```typescript
import { writeText, readText } from '@tauri-apps/plugin-clipboard-manager';

await writeText('Copied text');
const text = await readText();
```

**Permissions:** `clipboard-manager:allow-read`, `clipboard-manager:allow-write`

---

### Global Shortcut (`tauri-plugin-global-shortcut`)

```rust
tauri::Builder::default()
    .plugin(tauri_plugin_global_shortcut::Builder::new().build())
```

```typescript
import { register, unregister } from '@tauri-apps/plugin-global-shortcut';

await register('CommandOrControl+Shift+C', (event) => {
    console.log('Shortcut triggered!', event);
});

await unregister('CommandOrControl+Shift+C');
```

**Permissions:** `global-shortcut:allow-register`, `global-shortcut:allow-unregister`

---

### Updater (`tauri-plugin-updater`)

```rust
tauri::Builder::default()
    .plugin(tauri_plugin_updater::Builder::new().build())
```

```typescript
import { check } from '@tauri-apps/plugin-updater';
import { relaunch } from '@tauri-apps/plugin-process';

const update = await check();
if (update) {
    await update.downloadAndInstall();
    await relaunch();
}
```

---

### Process (`tauri-plugin-process`)

```rust
tauri::Builder::default()
    .plugin(tauri_plugin_process::init())
```

```typescript
import { exit, relaunch } from '@tauri-apps/plugin-process';

await exit(0);  // Exit app
await relaunch();  // Restart app
```

---

### OS (`tauri-plugin-os`)

```rust
tauri::Builder::default()
    .plugin(tauri_plugin_os::init())
```

```typescript
import { platform, arch, version, type, locale } from '@tauri-apps/plugin-os';

const os = await platform();  // 'linux', 'macos', 'windows'
const architecture = await arch();  // 'x86_64', 'aarch64'
const osVersion = await version();
const osType = await type();  // 'Linux', 'Darwin', 'Windows_NT'
const userLocale = await locale();
```

---

### Window State (`tauri-plugin-window-state`)

Persist and restore window position/size.

```rust
tauri::Builder::default()
    .plugin(tauri_plugin_window_state::Builder::default().build())
```

Window state is automatically saved and restored.

---

### Log (`tauri-plugin-log`)

```rust
use tauri_plugin_log::{Target, TargetKind};

tauri::Builder::default()
    .plugin(
        tauri_plugin_log::Builder::default()
            .targets([
                Target::new(TargetKind::Stdout),
                Target::new(TargetKind::LogDir { file_name: None }),
            ])
            .build()
    )
```

```typescript
import { trace, debug, info, warn, error } from '@tauri-apps/plugin-log';

info('Application started');
error('Something went wrong');
```

---

## Plugin Configuration

### Capability Example

```json
{
  "identifier": "main-capability",
  "windows": ["main"],
  "permissions": [
    "core:default",
    "fs:default",
    "dialog:default",
    "shell:allow-open",
    "http:default",
    "notification:default",
    "clipboard-manager:default",
    "global-shortcut:default"
  ]
}
```

### Scoped Plugin Permissions

```json
{
  "permissions": [
    {
      "identifier": "fs:allow-read",
      "allow": [{ "path": "$APP/*" }]
    },
    {
      "identifier": "http:allow-fetch",
      "allow": [{ "url": "https://api.example.com/*" }]
    },
    {
      "identifier": "shell:allow-execute",
      "allow": [{ "cmd": "git", "args": true }]
    }
  ]
}
```
