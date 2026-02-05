# Tauri V2 核心 API 参考

## 命令系统 (Commands)

### 基础命令
```rust
#[tauri::command]
fn greet(name: &str) -> String {
    format!("Hello, {}!", name)
}

// 异步命令
#[tauri::command]
async fn async_command() -> Result<String, String> {
    Ok("done".to_string())
}

// 注册命令
tauri::Builder::default()
    .invoke_handler(tauri::generate_handler![greet, async_command])
```

### 带状态的命令
```rust
use tauri::State;

struct AppState {
    counter: Mutex<i32>,
}

#[tauri::command]
fn increment(state: State<AppState>) -> i32 {
    let mut counter = state.counter.lock().unwrap();
    *counter += 1;
    *counter
}
```

## 状态管理 (State)

```rust
use std::{collections::HashMap, sync::Mutex};
use tauri::State;

struct Storage {
    store: Mutex<HashMap<u64, String>>,
}

struct DbConnection {
    db: Mutex<Option<Connection>>,
}

#[tauri::command]
fn storage_insert(key: u64, value: String, storage: State<Storage>) {
    storage.store.lock().unwrap().insert(key, value);
}

tauri::Builder::default()
    .manage(Storage { store: Default::default() })
    .manage(DbConnection { db: Default::default() })
```

## 事件系统 (Events)

### Rust 端
```rust
use tauri::{Manager, Listener, Emitter};

// 监听事件
webview.listen("my-event", |event| {
    println!("received: {:?}", event.payload());
});

// 只监听一次
webview.once("init-event", |event| { ... });

// 取消监听
webview.unlisten(event_id);

// 发射事件
app.emit("backend-event", payload)?;
app.emit_to(target, "specific-event", payload)?;
```

## 窗口 API

### 获取器
| 方法 | 返回类型 |
|------|---------|
| `scale_factor()` | `f64` |
| `inner_size()` / `outer_size()` | `PhysicalSize<u32>` |
| `inner_position()` / `outer_position()` | `PhysicalPosition<i32>` |
| `is_fullscreen()` / `is_maximized()` / `is_minimized()` | `bool` |
| `is_visible()` / `is_focused()` / `is_decorated()` | `bool` |
| `title()` | `String` |
| `theme()` | `Theme` |
| `current_monitor()` | `Option<Monitor>` |

### 设置器
| 方法 | 参数 |
|------|------|
| `set_title()` | `&str` |
| `set_size()` / `set_min_size()` / `set_max_size()` | `Size` |
| `set_position()` | `Position` |
| `maximize()` / `minimize()` / `set_fullscreen()` | - / - / `bool` |
| `show()` / `hide()` / `set_focus()` | - |
| `set_decorations()` / `set_resizable()` | `bool` |
| `set_always_on_top()` | `bool` |
| `center()` | - |
| `start_dragging()` | - |

### 创建新窗口
```rust
use tauri::WebviewWindowBuilder;

WebviewWindowBuilder::new(app, "second", tauri::WebviewUrl::App("index.html".into()))
    .title("Second Window")
    .inner_size(800.0, 600.0)
    .build()?;
```

## 插件系统

```rust
use tauri::plugin::{Builder as PluginBuilder, TauriPlugin};

#[tauri::command]
async fn plugin_command() -> Result<(), String> { Ok(()) }

pub fn init<R: Runtime>() -> TauriPlugin<R> {
    PluginBuilder::new("my-plugin")
        .setup(|app, api| { Ok(()) })
        .on_event(|app, event| {
            match event {
                RunEvent::Ready => println!("ready"),
                RunEvent::WindowEvent { label, event, .. } => { ... }
                _ => ()
            }
        })
        .invoke_handler(tauri::generate_handler![plugin_command])
        .build()
}
```

## Capabilities 权限系统

```rust
use tauri::CapabilityBuilder;

CapabilityBuilder::new("main-capability")
    .window("main")
    .permission("path:default")
    .permission("event:default")
    .permission_scoped(
        "fs:read",
        vec!["/data/"],      // 允许
        vec!["/sensitive/"]  // 拒绝
    )
    .platform(Target::Windows)
    .build()
```

### Capability JSON
```json
{
  "identifier": "fs-read-home",
  "description": "Allow file access",
  "local": true,
  "windows": ["main"],
  "permissions": ["fs:allow-home-read"],
  "platforms": ["linux", "windows"]
}
```

## 生命周期事件

```rust
tauri::Builder::default()
    .build(tauri::generate_context!())?
    .run(|app, event| {
        match event {
            RunEvent::Ready => { }
            RunEvent::WindowEvent { label, event, .. } => { }
            RunEvent::ExitRequested { api, .. } => {
                // api.prevent_exit();
            }
            _ => {}
        }
    });
```
