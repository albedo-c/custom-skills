---
name: python-modernize
description: Modernize a Python project by migrating to uv, pyproject.toml (PEP 621), and ruff for linting and formatting — replacing legacy setup.py, requirements.txt, black, flake8, and isort
license: MIT
metadata:
  tools: "uv, ruff"
  workflow: migration
  standard: "PEP 517, PEP 518, PEP 621"
---

## What I do

- Audit the project's current Python tooling setup
- Migrate packaging metadata to `pyproject.toml` (PEP 621 `[project]` table)
- Set up `uv` as the package and environment manager
- Configure `ruff` as the single tool for linting and formatting (replaces `black`, `flake8`, `isort`, `pyupgrade`)
- Add a `.python-version` file to pin the Python version
- Update CI configs and pre-commit hooks to use the new tools
- Run the formatter and linter to bring the codebase to a clean state

## When to use me

Use this skill when the project has any of:

- `setup.py` or `setup.cfg` (legacy packaging)
- `requirements.txt` without a `pyproject.toml`
- `Pipfile` / `Pipfile.lock` (pipenv)
- Separate `black`, `flake8`, `isort`, `pyupgrade` configurations
- No virtual environment tooling specified

## Step-by-step workflow

### 1. Audit the current state

Check for all legacy files and existing configs:

```bash
ls -la setup.py setup.cfg requirements*.txt Pipfile pyproject.toml .flake8 .black tox.ini 2>/dev/null
```

Read each file to understand:

- Project name, version, description, author, license
- All dependencies (runtime and dev/test)
- Any existing linter/formatter configuration
- Python version requirements

### 2. Verify uv is available

```bash
uv --version
```

If not installed:

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
# then reload shell: source ~/.bashrc or source ~/.zshrc
```

### 3. Create or migrate `pyproject.toml`

If `pyproject.toml` does not exist, create it with PEP 621 structure:

```toml
[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[project]
name = "your-project-name"
version = "0.1.0"
description = "A short description of the project"
readme = "README.md"
license = { text = "MIT" }
authors = [
    { name = "Author Name", email = "author@example.com" },
]
requires-python = ">=(project_python_version)"
dependencies = [
    # migrate from requirements.txt here
]

[project.optional-dependencies]
dev = [
    "pytest>=8.0",
    "pytest-cov>=5.0",
]

[project.urls]
Homepage = "https://github.com/org/repo"
Repository = "https://github.com/org/repo"
```

If `pyproject.toml` exists but uses `[tool.poetry]`, migrate the `[tool.poetry.dependencies]` section to `[project.dependencies]` format:

- Poetry format: `requests = "^2.28"` → PEP 621: `"requests>=2.28"`
- Poetry dev deps → `[project.optional-dependencies]` dev group

### 4. Configure ruff in `pyproject.toml`

Add the `[tool.ruff]` section — this replaces `black`, `flake8`, `isort`, and `pyupgrade`:

```toml
[tool.ruff]
target-version = "py3XX"  # match requires-python
line-length = 88           # same default as black

[tool.ruff.lint]
select = [
    "E",   # pycodestyle errors
    "W",   # pycodestyle warnings
    "F",   # pyflakes
    "I",   # isort
    "B",   # flake8-bugbear
    "C4",  # flake8-comprehensions
    "UP",  # pyupgrade — modernize Python syntax
    "N",   # pep8-naming
    "SIM", # flake8-simplify
    "TCH", # flake8-type-checking
    "RUF", # ruff-specific rules
]
ignore = [
    "E501",  # line too long — handled by formatter
    "B008",  # do not perform function calls in default arguments
]

[tool.ruff.lint.isort]
known-first-party = ["your_package_name"]

[tool.ruff.format]
# Matches black defaults
quote-style = "double"
indent-style = "space"
skip-magic-trailing-comma = false
line-ending = "auto"
```

### 5. Initialize uv and lock dependencies

```bash
# Initialize uv in the project (creates .venv and uv.lock)
uv sync

# If migrating from requirements.txt:
uv add $(cat requirements.txt | grep -v '^#' | grep -v '^$' | tr '\n' ' ')

# If migrating from Pipfile:
# Manually map Pipfile [packages] to uv add commands

# Add dev dependencies:
uv add --dev pytest pytest-cov ruff
```

### 6. Add a `.python-version` file

```bash
# Pin to the minimum Python version the project supports
echo "3.XX" > .python-version
```

This is read by `uv`, `pyenv`, and many CI systems automatically.

### 7. Run ruff to fix and format the codebase

```bash
# Fix all auto-fixable lint issues (includes isort, pyupgrade, etc.)
ruff check --fix .

# Format all Python files (replaces black)
ruff format .
```

Review any remaining lint errors that could not be auto-fixed:

```bash
ruff check .
```

Fix remaining issues manually, file by file.

### 8. Remove legacy config files

After confirming ruff and uv work correctly, remove obsolete files:

```bash
# Only remove after confirming everything works
rm -f setup.py           # if fully migrated to pyproject.toml
rm -f setup.cfg          # if fully migrated to pyproject.toml
rm -f Pipfile Pipfile.lock
rm -f .flake8            # replaced by [tool.ruff] in pyproject.toml
rm -f .isort.cfg         # replaced by [tool.ruff.lint.isort]
rm -f pyproject-black.toml  # replaced by [tool.ruff.format]
```

**Caution:** Do not remove `requirements.txt`.

### 9. Update CI configuration

**GitHub Actions example** — replace old workflow steps:

```yaml
# Before (legacy):
- run: pip install -r requirements.txt
- run: black --check .
- run: flake8 .
- run: isort --check .

# After (modern):
- uses: astral-sh/setup-uv@v4
  with:
    enable-cache: true
- run: uv sync --frozen
- run: ruff check .
- run: ruff format --check .
- run: uv run pytest
```

### 10. Update or create `.pre-commit-config.yaml`

```yaml
repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.15.4
    hooks:
      - id: ruff
        args: [--fix]
      - id: ruff-format
```

### 11. Commit the modernization

````bash
git add -A
git commit -m "chore: modernize Python tooling to uv + ruff
```

## Rules and guardrails

- Always read `pyproject.toml`, `setup.py`, `setup.cfg`, and all `requirements*.txt` fully before migrating — missing a dependency will break the project
- Do not remove `requirements.txt` if it is referenced in `Dockerfile`, CI, or deployment scripts without updating those references first
- The `ruff format` output is intentionally identical to `black` output — no manual formatting changes needed
- If the project uses `tox`, update `tox.ini` to call `uv run` instead of direct `python` or `pip` commands
- Test the project after migration: `uv run pytest` must pass before committing
- If `setup.py` has custom build logic (not just metadata), that logic may need to be preserved in a `hatch_build.py` or similar — do not delete it blindly
