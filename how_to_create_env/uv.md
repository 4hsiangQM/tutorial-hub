# uv Workflow Tutorial

This tutorial explains two common ways to use `uv` for Python environments:

1. `uv venv`: create a simple virtual environment.
2. `uv init`: create a full Python project managed by `uv`.

Use this guide when you want to decide which workflow fits your situation.

---

## Outline

1. [Quick Decision Guide](#quick-decision-guide)
2. [Workflow 1: Simple Environment with `uv venv`](#workflow-1-simple-environment-with-uv-venv)
3. [Workflow 2: Full Project with `uv init`](#workflow-2-full-project-with-uv-init)
4. [Comparison: `uv venv` vs `uv init`](#comparison-uv-venv-vs-uv-init)
5. [Recommended Workflows](#recommended-workflows)
6. [Common Errors](#common-errors)
7. [Important Concepts](#important-concepts)
8. [Command Cheat Sheet](#command-cheat-sheet)

---

## Quick Decision Guide

Use `uv venv` when:

- You only need a virtual environment.
- You are running simple Python scripts.
- The project already has a `requirements.txt`.
- You do not need `pyproject.toml`.
- You want something similar to the traditional `venv + pip` workflow.

Use `uv init` when:

- You are starting a new Python project.
- You want to manage dependencies with `uv add`.
- You want a reproducible setup with `pyproject.toml` and `uv.lock`.
- You want to use `uv sync`.
- You want a cleaner long-term project structure.

Use `uv sync` when:

- You cloned or downloaded an existing project.
- The project already has a `pyproject.toml`.
- You want to create the same project environment locally.
- The project has a `uv.lock` file and you want to install the locked dependency versions.

Simple rule:

```text
uv venv = I only need a virtual environment
uv init = I want a complete Python project
uv sync = I already have a uv project and want to install its environment
```

---

## Workflow 1: Simple Environment with `uv venv`

`uv venv` creates only a virtual environment.

It is similar to:

```bash
python -m venv .venv
```

but faster.

### Step 1: Create a Virtual Environment

```powershell
uv venv
```

This creates:

```text
.venv/
```

### Step 2: Activate the Virtual Environment

Windows PowerShell:

```powershell
.venv\Scripts\activate
```

macOS / Linux:

```bash
source .venv/bin/activate
```

### Step 3: Install Packages

Install packages directly:

```powershell
uv pip install numpy matplotlib pandas
```

Install packages from `requirements.txt`:

```powershell
uv pip install -r requirements.txt
```

### Step 4: Check Installed Packages

```powershell
uv pip list
```

### Step 5: Export Installed Packages

```powershell
uv pip freeze > requirements.txt
```

### Step 6: Run Python Scripts

After activating the environment:

```powershell
python main.py
```

or:

```powershell
python your_script.py
```

### Key Points

- `uv venv` does not create `pyproject.toml`.
- `uv venv` does not require `uv init`.
- `uv venv` is similar to the traditional `venv + pip` workflow.
- `uv sync` is usually not used in this workflow.

---

## Workflow 2: Full Project with `uv init`

`uv init` creates a full Python project.

It can create project files such as:

```text
pyproject.toml
.python-version
README.md
main.py
```

### Step 1: Create a New Project

Create a new project folder:

```powershell
uv init my-project
cd my-project
```

Or initialize the current folder:

```powershell
uv init
```

### Step 2: Handle Parent Workspace Errors

Sometimes `uv` searches parent folders and finds an existing `pyproject.toml`.

You may see:

```text
Failed to discover parent workspace
```

In that case, use:

```powershell
uv init --no-workspace
```

This tells `uv` to ignore the parent workspace and initialize the current folder as an independent project.

### Step 3: Review the Files Created by `uv init`

After running:

```powershell
uv init
```

You may see a structure like this:

```text
my-project/
|-- .python-version
|-- README.md
|-- main.py
`-- pyproject.toml
```

### Step 4: Understand `pyproject.toml`

Example:

```toml
[project]
name = "my-project"
version = "0.1.0"
description = "Add your description here"
readme = "README.md"
requires-python = ">=3.12"
dependencies = []
```

The most important part is:

```toml
[project]
```

Without `[project]`, `uv sync` will not know how to manage the project.

### Step 5: Create and Sync the Environment

```powershell
uv sync
```

`uv sync` will:

- Create `.venv` if it does not exist.
- Read dependencies from `pyproject.toml`.
- Install dependencies into `.venv`.
- Create or update `uv.lock`.

### Step 6: Add Packages

Install one package:

```powershell
uv add numpy
```

Install multiple packages:

```powershell
uv add numpy matplotlib pandas
```

This command updates:

- `pyproject.toml`
- `uv.lock`
- `.venv`

Example result in `pyproject.toml`:

```toml
[project]
name = "my-project"
version = "0.1.0"
requires-python = ">=3.12"
dependencies = [
    "matplotlib",
    "numpy",
    "pandas",
]
```

### Step 7: Remove Packages

```powershell
uv remove numpy
```

This removes the package from:

- `pyproject.toml`
- `uv.lock`
- `.venv`

### Step 8: Run Python Scripts

Use `uv run`:

```powershell
uv run python main.py
```

or:

```powershell
uv run python your_script.py
```

The benefit of `uv run` is that you do not need to manually activate the virtual environment.

`uv` automatically uses the project's `.venv`.

---

## Comparison: `uv venv` vs `uv init`

| Feature | `uv venv` | `uv init` |
|---|---|---|
| Creates `.venv` | Yes | With `uv sync` |
| Creates `pyproject.toml` | No | Yes |
| Requires `pyproject.toml` | No | Yes |
| Can use `uv pip install` | Yes | Yes |
| Can use `uv add` | Not recommended | Yes |
| Can use `uv sync` | Usually no | Yes |
| Good for quick testing | Yes | Less ideal |
| Good for formal projects | Less ideal | Yes |
| Supports lock file workflow | No | Yes |

---

## Recommended Workflows

### Case 1: Running Someone Else's Repository

Use this workflow when the project already has a `requirements.txt`:

```powershell
uv venv
.venv\Scripts\activate
uv pip install -r requirements.txt
python main.py
```

This is simple and works well for existing projects, examples, or tutorials.

### Case 2: Running an Existing `uv` Project

Use this workflow when the project already has a `pyproject.toml`:

```powershell
cd existing-project
uv sync
uv run python main.py
```

`uv sync` reads the project's `pyproject.toml` and installs the required dependencies into `.venv`.

If the project also has a `uv.lock` file, `uv sync` uses it to recreate the locked dependency versions. This helps you get the same environment as the original project.

If the project does not have a `uv.lock` file, `uv sync` can still install dependencies from `pyproject.toml`, but the exact package versions may be resolved again.

### Case 3: Starting Your Own Python Project

Use this workflow when you want a clean and reproducible project:

```powershell
uv init
uv add numpy matplotlib pandas
uv sync
uv run python main.py
```

This is better for long-term development.

---

## Common Errors

### Error 1: No `project` Table Found

You may see:

```text
No project table found in pyproject.toml
```

This means `uv` found a `pyproject.toml`, but that file does not contain:

```toml
[project]
```

`uv sync` needs `[project]` to understand the project metadata and dependencies.

Possible solutions:

```powershell
uv init --no-workspace
```

or manually create a valid `pyproject.toml`.

### Error 2: Failed to Discover Parent Workspace

You may see:

```text
Failed to discover parent workspace
```

This happens when `uv` searches parent folders and finds a broken or incomplete `pyproject.toml`.

Solution:

```powershell
uv init --no-workspace
```

This tells `uv` to ignore the parent workspace.

---

## Important Concepts

### `uv venv`

Creates only a virtual environment. It does not create `pyproject.toml`.

Good for quick setup and existing projects.

### `uv init`

Creates a full Python project with `pyproject.toml`.

Good for new projects and long-term development.

### `uv sync`

Synchronizes the environment based on `pyproject.toml` and `uv.lock`.

Usually used with `uv init` projects.

### `uv add`

Installs a package and records it in `pyproject.toml`.

Usually used with `uv init` projects.

### `uv pip install`

Works like `pip install`.

Can be used with `uv venv`, but does not automatically manage `pyproject.toml`.

### `uv run`

Runs commands inside the project environment.

You do not need to manually activate `.venv`.

---

## Command Cheat Sheet

### `uv venv` Workflow

```powershell
uv venv
.venv\Scripts\activate
uv pip install numpy matplotlib pandas
python main.py
```

### `uv init` Workflow

```powershell
uv init
uv add numpy matplotlib pandas
uv sync
uv run python main.py
```

### Existing `uv` Project Workflow

```powershell
cd existing-project
uv sync
uv run python main.py
```

### Common Commands

| Command | Purpose |
|---|---|
| `uv venv` | Create a virtual environment |
| `uv init` | Create a full Python project |
| `uv sync` | Install dependencies from `pyproject.toml` / `uv.lock` |
| `uv add package-name` | Install a package and save it to `pyproject.toml` |
| `uv remove package-name` | Remove a package from the project |
| `uv pip install package-name` | Install packages like `pip` |
| `uv run python main.py` | Run a script inside the uv environment |

---

## Final Recommendation

For existing repositories or examples with `requirements.txt`:

```powershell
uv venv
.venv\Scripts\activate
uv pip install -r requirements.txt
```

For existing `uv` projects with `pyproject.toml`:

```powershell
cd existing-project
uv sync
uv run python main.py
```

For new projects:

```powershell
uv init
uv add numpy matplotlib pandas
uv sync
uv run python main.py
```
