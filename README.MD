# Odoo 19 — Windows Development Environment Setup Guide

A reusable, step-by-step guide for setting up an Odoo 19 source-based development environment on Windows. Keep this for future machines or fresh environment rebuilds.

> Odoo 19 requires **Python 3.10+** and **PostgreSQL 13+**. Dependencies are listed in the source repo's `requirements.txt`.

---

## Table of Contents

1. [Folder Structure](#1-folder-structure)
2. [Prerequisites](#2-prerequisites)
3. [Setup Steps](#3-setup-steps)
4. [VS Code Configuration](#4-vs-code-configuration)
5. [Daily Usage](#5-daily-usage)
6. [Troubleshooting](#6-troubleshooting)
7. [Rebuilding the Environment](#7-rebuilding-the-environment)
8. [Quick Reference Checklist](#8-quick-reference-checklist)
9. [Essential Commands](#9-essential-commands)

---

## 1. Folder Structure

```text
C:\Users\<USERNAME>\projects\
│
├── .venv\
│
├── odoo\
│   ├── odoo-bin
│   ├── odoo\
│   ├── addons\
│   └── requirements.txt
│
├── custom_addons\
│   ├── module_1\
│   ├── module_2\
│   └── ...
│
├── odoo19.conf
└── odoo19.code-workspace
```

**Example (this machine):** `C:\Users\Computia.ME\projects\`

---

## 2. Prerequisites

Install the following before starting:

| Tool | Notes |
|---|---|
| **Python 3.10+** | Odoo 19's minimum supported version |
| **PostgreSQL 13+** | Odoo 19's minimum supported version |
| **Git** | For cloning the Odoo source |
| **VS Code** | Recommended editor |
| **Node.js / npm** | Only needed for Odoo's RTL support |
| **Visual Studio Build Tools** | Only if a Python dependency requires compilation |

---

## 3. Setup Steps

### 3.1 Verify Python

```powershell
python --version
# or
py --version
```

Target version: `Python 3.10.x`

```powershell
pip --version
```

### 3.2 Create the Projects Folder

```powershell
mkdir C:\Users\<USERNAME>\projects
cd C:\Users\<USERNAME>\projects
```

### 3.3 Clone Odoo 19

```powershell
git clone --branch 19.0 --single-branch https://github.com/odoo/odoo.git
```

This creates `projects\odoo`. The `19.0` branch is Odoo's recommended branch for this version.

### 3.4 Create the Custom Addons Folder

```powershell
mkdir custom_addons
```

Resulting structure:

```text
projects
├── odoo
└── custom_addons
```

### 3.5 Create the Virtual Environment

```powershell
cd C:\Users\<USERNAME>\projects
py -3.10 -m venv .venv
```

### 3.6 Allow PowerShell Script Execution

```powershell
Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned
```

> Administrator PowerShell is **not** required for `CurrentUser` scope.

### 3.7 Activate the Virtual Environment

```powershell
.\.venv\Scripts\Activate.ps1
```

You should see the `(.venv)` prefix in your prompt:

```text
(.venv) PS C:\Users\<USERNAME>\projects>
```

### 3.8 Confirm the Correct Python Is Active

```powershell
where.exe python
python --version
python -c "import sys; print(sys.executable)"
```

Expected result for all of the above: `C:\Users\<USERNAME>\projects\.venv\Scripts\python.exe`

> ⭐ **Rule of thumb:** always use `python` and `python -m pip` after activation — never a bare system-level `pip` — to avoid polluting your global Python install.

### 3.9 Install Odoo Dependencies

```powershell
cd C:\Users\<USERNAME>\projects\odoo
python -m pip install setuptools wheel
python -m pip install -r requirements.txt
```

This single step resolves most `ModuleNotFoundError` issues (e.g. `No module named 'babel'`).

> ❌ **Avoid** installing packages one at a time (`pip install babel`, `pip install psycopg2`, …) unless you're actively troubleshooting a specific failure. `requirements.txt` should cover everything.

### 3.10 Set Up PostgreSQL

- Install PostgreSQL **13+**
- Create a dedicated Odoo database user (do **not** use the `postgres` superuser)
- Grant that user permission to create databases

Example credentials:

```text
Username: odoo
Password: your_password
```

### 3.11 Create `odoo19.conf`

Location: `C:\Users\<USERNAME>\projects\odoo19.conf`

```ini
[options]

admin_passwd = admin

db_host = localhost
db_port = 5432
db_user = odoo
db_password = your_password

addons_path = C:\Users\<USERNAME>\projects\odoo\addons,C:\Users\<USERNAME>\projects\custom_addons

http_port = 8069
```

> ⚠️ Replace `your_password` with your actual PostgreSQL password — don't reuse the example literally.

### 3.12 Test Odoo from the Command Line

With `.venv` active, from the `projects` folder:

```powershell
python .\odoo\odoo-bin -c .\odoo19.conf
```

If successful, Odoo will start and listen at:

```text
http://localhost:8069
```

---

## 4. VS Code Configuration

### 4.1 Workspace File — `odoo19.code-workspace`

```json
{
    "folders": [
        { "path": "odoo" },
        { "path": "custom_addons" }
    ],

    "settings": {
        "python.defaultInterpreterPath": "C:\\Users\\Computia.ME\\projects\\.venv\\Scripts\\python.exe",
        "xml.symbols.enabled": false,
        "python.languageServer": "Default",
        "python.autoComplete.extraPaths": [
            "C:\\Users\\Computia.ME\\projects\\odoo"
        ]
    },

    "launch": {
        "version": "0.2.0",
        "configurations": [
            {
                "name": "odoo",
                "type": "debugpy",
                "request": "launch",
                "stopOnEntry": false,
                "python": "C:\\Users\\Computia.ME\\projects\\.venv\\Scripts\\python.exe",
                "program": "C:\\Users\\Computia.ME\\projects\\odoo\\odoo-bin",
                "console": "internalConsole",
                "args": [
                    "-c",
                    "C:\\Users\\Computia.ME\\projects\\odoo19.conf",
                    "--dev=xml"
                ]
            },
            {
                "name": "odoo (upgrade)",
                "type": "debugpy",
                "request": "launch",
                "stopOnEntry": false,
                "python": "C:\\Users\\Computia.ME\\projects\\.venv\\Scripts\\python.exe",
                "program": "C:\\Users\\Computia.ME\\projects\\odoo\\odoo-bin",
                "console": "internalConsole",
                "args": [
                    "-c",
                    "C:\\Users\\Computia.ME\\projects\\odoo19.conf",
                    "--dev=xml",
                    "-u",
                    "${input:modulesToUpgrade}"
                ]
            }
        ],
        "inputs": [
            {
                "id": "modulesToUpgrade",
                "type": "promptString",
                "description": "Enter the modules to upgrade:",
                "default": "your_module"
            }
        ]
    }
}
```

### 4.2 Select the Interpreter

`Ctrl+Shift+P` → **Python: Select Interpreter** → choose:

```text
C:\Users\<USERNAME>\projects\.venv\Scripts\python.exe
```

---

## 5. Daily Usage

### 5.1 Starting Odoo

**Option A — PowerShell**

```powershell
cd C:\Users\<USERNAME>\projects
.\.venv\Scripts\Activate.ps1
python .\odoo\odoo-bin -c .\odoo19.conf
```

**Option B — VS Code**

Go to **Run and Debug** → select **odoo** → press `F5`.

### 5.2 Creating a New Custom Module

```text
custom_addons
└── school_management_training
    ├── __init__.py
    ├── __manifest__.py
    │
    ├── models
    │   ├── __init__.py
    │   ├── student.py
    │   ├── teacher.py
    │   └── course.py
    │
    ├── views
    │   ├── student_views.xml
    │   ├── teacher_views.xml
    │   └── course_views.xml
    │
    ├── security
    │   ├── ir.model.access.csv
    │   └── security.xml
    │
    └── data
```

Because `custom_addons` is already in `addons_path`, Odoo will detect the module automatically.

### 5.3 Upgrading a Custom Module

**PowerShell**

```powershell
python .\odoo\odoo-bin -c .\odoo19.conf -u school_management_training
```

**VS Code** — Run and Debug → **odoo (upgrade)** → enter the module name when prompted.

---

## 6. Troubleshooting

### `ModuleNotFoundError` (e.g. `No module named 'babel'`)

1. Confirm the venv Python is active:
   ```powershell
   where.exe python
   ```
   The first result must be `...\projects\.venv\Scripts\python.exe`.
2. Check installed packages:
   ```powershell
   python -m pip list
   ```
3. If dependencies are missing:
   ```powershell
   cd C:\Users\<USERNAME>\projects\odoo
   python -m pip install -r requirements.txt
   ```
   Do **not** immediately delete and recreate `.venv` — reinstalling dependencies usually fixes it.

### PowerShell: "running scripts is disabled"

```powershell
Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned
```

Close and reopen PowerShell, then reactivate:

```powershell
.\.venv\Scripts\Activate.ps1
```

### `python` Points to the Wrong Interpreter

```powershell
where.exe python
```

If `.venv`'s Python isn't listed first, activate it:

```powershell
.\.venv\Scripts\Activate.ps1
```

### Odoo Can't Find a Custom Module

1. Verify `addons_path` in `odoo19.conf` includes your `custom_addons` folder.
2. Restart Odoo.
3. In the UI: **Apps → Update Apps List**, then search for the module.

### XML Changes Aren't Appearing

Run Odoo with the `--dev=xml` flag (already included in the VS Code launch config).

---

## 7. Rebuilding the Environment

If `.venv` becomes corrupted, you can rebuild it without touching Odoo source or custom modules.

```powershell
cd C:\Users\<USERNAME>\projects
deactivate
```

Delete the `.venv` folder, then:

```powershell
py -3.10 -m venv .venv
.\.venv\Scripts\Activate.ps1
cd .\odoo
python -m pip install setuptools wheel
python -m pip install -r requirements.txt
```

Your `odoo`, `custom_addons`, and `odoo19.conf` remain untouched.

---

## 8. Quick Reference Checklist

Use this order every time you set up a fresh PC:

- [ ] Install Python 3.10+
- [ ] Install PostgreSQL 13+
- [ ] Install Git
- [ ] Install VS Code
- [ ] Create `projects` folder
- [ ] Clone Odoo 19 source
- [ ] Create `custom_addons`
- [ ] Create `.venv`
- [ ] Activate `.venv`
- [ ] Verify `where.exe python`
- [ ] Verify `python --version`
- [ ] Install `setuptools` + `wheel`
- [ ] Install `requirements.txt`
- [ ] Configure PostgreSQL user
- [ ] Create `odoo19.conf`
- [ ] Set `addons_path`
- [ ] Run Odoo from PowerShell
- [ ] Confirm `localhost:8069` loads
- [ ] Create/open VS Code workspace
- [ ] Select `.venv` interpreter
- [ ] Configure F5 debugging
- [ ] Create a custom module
- [ ] Update Apps List
- [ ] Start development

---

## 9. Essential Commands

The five commands to memorize for this setup:

```powershell
cd C:\Users\Computia.ME\projects
```

```powershell
.\.venv\Scripts\Activate.ps1
```

```powershell
where.exe python
```

```powershell
cd .\odoo
python -m pip install -r requirements.txt
```

```powershell
python odoo-bin -c ..\odoo19.conf
```

**Key principle to remember:**

> `.venv` → Python environment → Odoo requirements → PostgreSQL → `odoo19.conf` → VS Code

---

### Note
Created by: Abdulla Alebreeq