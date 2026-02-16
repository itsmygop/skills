# PyApp Runtime Commands

The built PyApp binary includes `self` subcommands for managing the installation at runtime.

## Commands Reference

### self update

Update the project to the latest available version within the current distribution.

```bash
<EXE> self update
```

**Note:** Unavailable if `PYAPP_SKIP_INSTALL=true` unless `PYAPP_ALLOW_UPDATES=true` is also set.

---

### self restore

Remove the current installation and reinstall from scratch.

```bash
<EXE> self restore
```

Useful when the installation becomes corrupted or needs a fresh start.

---

### self remove

Completely wipe the current installation.

```bash
<EXE> self remove
```

---

### self python

Directly invoke the Python interpreter installed by PyApp.

```bash
<EXE> self python
<EXE> self python -c "import sys; print(sys.version)"
<EXE> self python script.py
```

---

### self python-path

Output the filesystem path to the Python interpreter.

```bash
<EXE> self python-path
# Example output: /home/user/.local/share/pyapp/myapp/python/bin/python3
```

Useful for scripting or debugging.

---

### self pip

Invoke pip for the installed environment.

```bash
<EXE> self pip list
<EXE> self pip install <package>
```

---

### self metadata

Display customized metadata output (configured via `PYAPP_METADATA_TEMPLATE`).

```bash
<EXE> self metadata
```

---

### self cache

Manage cached assets (distribution, pip, or uv).

```bash
# Show cache location
<EXE> self cache dist
<EXE> self cache pip
<EXE> self cache uv

# Remove cached assets (with subcommand)
<EXE> self cache dist remove
```

---

## Runtime Behavior Flow

On execution, PyApp follows this decision tree:

```
Is application installed?
├─ No → Is distribution cached?
│       ├─ No → Is distribution embedded?
│       │       ├─ No → Download from source
│       │       └─ Yes → Extract from binary
│       └─ Yes → Continue
│
│       Full isolation enabled?
│       ├─ Yes → Unpack distribution directly
│       └─ No → Create virtual environment
│               (UV enabled? → Use UV : Use venv)
│
│       Is project embedded?
│       ├─ Yes → Install from embedded data
│       └─ No → Dependency file provided?
│               ├─ Yes → Install from dependency file
│               └─ No → Install single project from PyPI
│
└─ Yes → Continue

Management command invoked? (e.g., self update)
├─ Yes → Execute management command
└─ No → Execute project
```

**Key points:**
- First run provisions Python and installs dependencies
- Subsequent runs execute immediately (no network)
- Embedded distributions enable fully offline binaries
- Management commands allow runtime maintenance
