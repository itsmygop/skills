# PyApp Configuration Reference

All configuration is done via environment variables at build time.

## Table of Contents

- [Project Configuration](#project-configuration)
- [Execution Mode](#execution-mode)
- [Distribution Configuration](#distribution-configuration)
- [Installation Configuration](#installation-configuration)
- [CLI Configuration](#cli-configuration)

---

## Project Configuration

| Variable | Description | Example |
|---|---|---|
| `PYAPP_PROJECT_NAME` | Project name (PEP 508 compliant) | `my-package` |
| `PYAPP_PROJECT_VERSION` | Project version | `1.0.0` |
| `PYAPP_PROJECT_PATH` | Path to wheel (.whl) or sdist (.tar.gz) to embed | `/path/to/pkg.whl` |
| `PYAPP_PROJECT_DEPENDENCY_FILE` | Path to requirements.txt or requirements.in | `/path/to/requirements.txt` |
| `PYAPP_PROJECT_FEATURES` | Optional dependency groups (comma-separated) | `feature1,feature2` |

**Notes:**
- If `PYAPP_PROJECT_PATH` is set, `NAME` and `VERSION` are auto-derived from package metadata
- `PYAPP_PROJECT_NAME` is normalized according to PEP 503

---

## Execution Mode

Set exactly one of these (mutually exclusive):

| Variable | Behavior | Example |
|---|---|---|
| `PYAPP_EXEC_SPEC` | Call entry point `pkg.mod:func` | `myapp.cli:main` |
| `PYAPP_EXEC_MODULE` | Run `python -m <module>` | `myapp` |
| `PYAPP_EXEC_SCRIPT` | Embed and run a Python script file | `/path/to/script.py` |
| `PYAPP_EXEC_CODE` | Execute arbitrary Python code | `print('Hello')` |
| `PYAPP_EXEC_NOTEBOOK` | Embed and run a Jupyter notebook | `/path/to/notebook.ipynb` |

**GUI Applications:**

| Variable | Description |
|---|---|
| `PYAPP_IS_GUI` | Set `true` to use `pythonw.exe` on Windows (no console) |

---

## Distribution Configuration

### Python Version

| Variable | Description | Default |
|---|---|---|
| `PYAPP_PYTHON_VERSION` | Python version to use | Latest stable CPython |

Example: `export PYAPP_PYTHON_VERSION=3.12`

### Distribution Source

| Variable | Description |
|---|---|
| `PYAPP_DISTRIBUTION_SOURCE` | URL to custom Python distribution archive |
| `PYAPP_DISTRIBUTION_PATH` | Local path to distribution archive (auto-enables embed) |
| `PYAPP_DISTRIBUTION_EMBED` | Set `true` to embed distribution in binary |

**Embedding prevents runtime downloads - ideal for offline distribution.**

### Variants

| Variable | Values | Description |
|---|---|---|
| `PYAPP_DISTRIBUTION_VARIANT_CPU` | `v1`, `v2`, `v3` (default), `v4` | CPU variant for Linux |
| `PYAPP_DISTRIBUTION_VARIANT_GIL` | `freethreaded` | Enable free-threaded Python (no GIL) |

### Isolation

| Variable | Description |
|---|---|
| `PYAPP_FULL_ISOLATION` | Set `true` to use full distribution copy instead of venv |

---

## Installation Configuration

### Package Installer

| Variable | Description |
|---|---|
| `PYAPP_UV_ENABLED` | Set `true` to use UV instead of pip (faster) |
| `PYAPP_PIP_EXTERNAL` | Set `true` to use standalone pip instead of bundled |
| `PYAPP_PIP_EXTRA_ARGS` | Additional pip install arguments |
| `PYAPP_UPGRADE_VIRTUALENV` | Set `true` to use virtualenv package instead of venv |

Example:
```bash
export PYAPP_UV_ENABLED=true
export PYAPP_PIP_EXTRA_ARGS="--no-cache-dir --extra-index-url https://pypi.org/simple"
```

### Skip/Allow Installation

| Variable | Description |
|---|---|
| `PYAPP_SKIP_INSTALL` | Set `true` to skip project installation (for pre-built distributions) |
| `PYAPP_ALLOW_UPDATES` | Set `true` to enable `update` command when install is skipped |

### Custom Install Directory

| Variable | Description |
|---|---|
| `PYAPP_INSTALL_DIR_<PROJECT>` | Custom install path (replace `<PROJECT>` with uppercased project name) |

Example: `export PYAPP_INSTALL_DIR_MYAPP=~/custom/path`

Default locations vary by OS:
- Linux: `~/.local/share/pyapp`
- macOS: `~/Library/Application Support/pyapp`
- Windows: `%LOCALAPPDATA%\pyapp`

---

## CLI Configuration

### Metadata Template

| Variable | Description |
|---|---|
| `PYAPP_METADATA_TEMPLATE` | Template string for `self metadata` output |

### Starship Integration

Example configuration for Starship prompt:

```toml
[custom.myapp]
command = "myapp self metadata"
when = true
# Windows: shell = ["cmd", "/C"]
# Other: shell = ["sh", "--norc"]
```
