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
- Documentation files (`.md`, `.rst`, `.txt`)
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

If a word is a legitimate technical term that codespell flags incorrectly, note it for the `.codespellrc` ignore list.

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

4. **Handle false positives**:
   - If a word is legitimate but codespell flags it, add it to `.codespellrc` later
   - Skip fixing words that are actually variable names, function names, or identifiers

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

Carefully inspect each change. If any fix looks wrong (corrected a technical term, variable name, etc.):

- Revert that specific change

### 7. Stage and commit

```bash
git add -A
git commit -m "fix: correct spelling errors detected by codespell

Reviewed each suggestion and applied fixes manually.
No logic or functional changes — comments, strings, and docs only."

### 8. Create or update .codespellrc (if needed)

If you encountered false positives (legitimate words codespell flagged incorrectly):

```ini
[codespell]
skip = .git,*.lock,*.min.js,node_modules,__pycache__,.venv,venv,dist,build
quiet-level = 3
ignore-words-list = sometechterm,anotheracronym
```

Add any legitimate technical terms to `ignore-words-list`. Commit this file alongside the spelling fixes.

## Rules and guardrails

- **Never** change variable names, function names, class names, or any identifier that appears in code logic — only fix human-readable prose (comments, docstrings, strings intended for display, documentation)
- If a word appears in both code identifiers AND comments (e.g. a typo that became a variable name), skip fixing it in comments too — it would cause confusion
- Always commit `.codespellrc` alongside the fixes so false positives are recorded for future runs
- Do not mix spelling fixes with other changes in the same commit
- If more than 10 files are changed, split into multiple commits by file type (docs first, then source)
