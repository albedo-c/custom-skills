---
name: codespell-fix
description: Run codespell to detect and fix common spelling mistakes across the entire codebase, then commit the changes on a new branch from master
license: MIT
metadata:
  tool: codespell
  version: ">=2.0"
  workflow: git-branch-and-commit
---

## What I do

- Run `codespell` to generate a list of spelling suggestions (no auto-write)
- Read and review every suggestion to confirm it is a genuine misspelling (not a technical term, variable name, or domain-specific word)
- For each valid fix: read the affected line in the file, understand the context, and manually apply the correction using file editing tools
- Create a new git branch from `master` named `fix/codespell-YYYYMMDD` (using today's date)
- Stage changed files, review the diff, and commit with a clear message

## When to use me

Use this skill when you want to sweep the codebase for spelling mistakes in:

- Comments and docstrings
- String literals and error messages
- Documentation files (`.md`, `.mdx`, `.rst`, `.txt`)
- Variable names and identifiers that contain English words

## Step-by-step workflow

### 1. Verify codespell is available

```bash
codespell --version
```

If not installed:

```bash
pip install codespell
# or, if uv is available (preferred):
uvx codespell --version
```

### 2. Run a dry-run preview (no changes yet)

```bash
codespell \
  --quiet-level 3 \
  --skip=".git,*.lock,*.min.js,*.min.css,node_modules,__pycache__,.venv,venv,dist,build,*.svg,*.png,*.jpg,*.jpeg,*.gif,*.ico,*.woff,*.woff2,*.ttf,*.eot" \
  .
```

Flags explained:

- `--quiet-level 3`: suppress binary file warnings, only show actual misspellings
- `--skip`: skip binary files, generated files, and dependency directories

### 3. Review the output carefully

For each suggestion, verify:

- Is it actually a misspelling? (not a technical abbreviation, acronym, or intentional term)
- Is the suggested correction correct in context?

### 4. Create a new git branch from master

```bash
git fetch origin
git checkout master
git pull origin master
git checkout -b fix/codespell-$(date +%Y%m%d)
```

If the default branch is `main` instead of `master`:

```bash
git checkout main && git pull && git checkout -b fix/codespell-$(date +%Y%m%d)
```

### 5. Apply fixes manually

For each spelling suggestion from codespell:

1. **Read the file** at the line containing the misspelling:
   - Use the Read tool to load the file
   - Find the exact line and word codespell flagged

2. **Verify the fix is correct**:
   - Confirm the word is genuinely misspelled (not a technical term, variable name, or domain-specific word)
   - Check if the suggested correction makes sense in context

3. **Apply the fix manually**:
   - Use the Edit tool to replace only the misspelled word
   - Do NOT use `--write-changes` — apply fixes one by one using file editing

Example workflow for each file:

```bash
# Run codespell to see suggestions (output shows file:line:misspelling -> suggestion)
codespell --quiet-level 3 --skip="...,*.svg,*.png" .
```

Then for each suggestion, read the file, locate the line, and edit:

- Old: `"This fuinction processes data"`
- New: `"This function processes data"`

### 6. Review the diff

```bash
git diff
```

Carefully inspect each change. If any fix looks wrong (corrected a technical term, variable name, etc.).
Revert that specific change

### 7. Stage and commit

```bash
git add -A
git commit -m "fix: typo/spelling correction/whatever is the most appropriate commit message"
```

## Rules and guardrails

- **Avoid** changing variable names, function names, class names, or any identifier that appears in code logic — only fix human-readable prose (comments, docstrings, strings intended for display, documentation), change the variable name only if it is a typo and make sure those changes are reflected in all the places where the variable is used.
- Do not mix spelling fixes with other changes in the same commit
- If more than 10 files are changed, split into multiple commits by file type (docs first, then source)
- Do not mention you used codespell anywhere
