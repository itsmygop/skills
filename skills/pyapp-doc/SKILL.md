---
name: pyapp
description: >
  Build standalone Python application binaries using PyApp (Rust-based wrapper).
  Use when users want to package a Python project into a single executable,
  configure PyApp build options (Python version, distribution, installation, execution mode),
  create offline/embedded distributions, manage PyApp runtime commands (update, restore, cache),
  or any task involving distributing Python apps as standalone binaries via PyApp.
---

# PyApp

## Overview

PyApp is a Rust-based wrapper that bootstraps Python applications at runtime, producing standalone binaries. It handles Python distribution provisioning, virtual environment creation, dependency installation, and application execution automatically.

## Prerequisites

- Rust toolchain with Cargo installed
- Python project with standard packaging (setup.py / pyproject.toml / wheel)

## Workflow

Building a PyApp binary follows these steps:

1. Configure the project (set environment variables)
2. Build with Cargo (`cargo build --release`)
3. Rename and distribute the binary

### Step 1: Configure the Project

Set environment variables before running `cargo build`. At minimum, define the project identity and execution mode.

**Minimal configuration:**

```bash
export PYAPP_PROJECT_NAME="myapp"
export PYAPP_PROJECT_VERSION="1.0.0"
export PYAPP_EXEC_SPEC="myapp.cli:main"
```

**Or embed a wheel directly:**

```bash
export PYAPP_PROJECT_PATH="/path/to/myapp-1.0.0-py3-none-any.whl"
```

#### Execution Modes

Choose one (mutually exclusive):

| Variable | Behavior | Example |
|---|---|---|
| `PYAPP_EXEC_SPEC` | Call an entry point object | `myapp.cli:main` |
| `PYAPP_EXEC_MODULE` | Run `python -m <module>` | `myapp` |
| `PYAPP_EXEC_SCRIPT` | Embed and run a .py script | `/path/to/script.py` |
| `PYAPP_EXEC_CODE` | Run arbitrary Python code | `print('hello')` |
| `PYAPP_EXEC_NOTEBOOK` | Embed and run a .ipynb notebook | `/path/to/nb.ipynb` |

#### Common Options

```bash
# Python version (default: latest stable CPython)
export PYAPP_PYTHON_VERSION=3.12

# Use UV instead of pip (faster)
export PYAPP_UV_ENABLED=true

# Embed distribution for offline use
export PYAPP_DISTRIBUTION_EMBED=true

# GUI app (uses pythonw.exe on Windows)
export PYAPP_IS_GUI=true

# Optional dependency groups
export PYAPP_PROJECT_FEATURES="feature1,feature2"
```

For the full configuration reference, see [references/config-reference.md](references/config-reference.md).

### Step 2: Build

```bash
cargo build --release
```

The binary is output to `target/release/pyapp` (or `pyapp.exe` on Windows).

### Step 3: Distribute

```bash
# Linux/macOS
mv target/release/pyapp myapp && chmod +x myapp

# Windows
mv target/release/pyapp.exe myapp.exe
```

### Step 4: Run

```bash
./myapp [args...]
```

On first run, PyApp automatically:
1. Downloads/extracts the Python distribution (or uses embedded)
2. Creates a virtual environment (unless full isolation)
3. Installs the project and dependencies
4. Executes the application

Subsequent runs skip installation and execute directly.

## Runtime Management Commands

The built binary includes `self` subcommands for runtime management:

| Command | Action |
|---|---|
| `<EXE> self update` | Update project to latest version |
| `<EXE> self restore` | Remove and reinstall |
| `<EXE> self remove` | Wipe installation |
| `<EXE> self python` | Invoke Python interpreter |
| `<EXE> self python-path` | Print Python path |
| `<EXE> self pip` | Invoke pip |
| `<EXE> self metadata` | Display metadata |
| `<EXE> self cache [dist\|pip\|uv]` | Manage cache |

For details see [references/runtime-commands.md](references/runtime-commands.md).

## Common Recipes

**Offline distribution (fully embedded):**

```bash
export PYAPP_PROJECT_PATH="./dist/myapp-1.0.0-py3-none-any.whl"
export PYAPP_DISTRIBUTION_EMBED=true
export PYAPP_PYTHON_VERSION=3.12
cargo build --release
```

**From requirements.txt:**

```bash
export PYAPP_PROJECT_DEPENDENCY_FILE="./requirements.txt"
export PYAPP_EXEC_MODULE="myapp"
cargo build --release
```

**Custom remote distribution:**

```bash
export PYAPP_PROJECT_NAME="myapp"
export PYAPP_PROJECT_VERSION="1.0.0"
export PYAPP_DISTRIBUTION_SOURCE="https://example.com/python.tar.gz"
export PYAPP_DISTRIBUTION_EMBED=true
cargo build --release
```

## References

- **[references/config-reference.md](references/config-reference.md)** - Complete environment variable reference for project, distribution, installation, and CLI configuration
- **[references/runtime-commands.md](references/runtime-commands.md)** - Detailed runtime `self` commands and behavior flow
