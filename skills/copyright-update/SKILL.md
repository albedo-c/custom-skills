---
name: copyright-update
description: Scan the entire codebase for copyright year notices and update stale years to the current year, preserving the original start year in ranges
license: MIT
metadata:
  workflow: git-branch-and-commit
  current-year: "2026"
---

## What I do

- Scan every tracked source file for copyright year notices using regex patterns
- Preserve the original start year and update only the trailing year to the current year
- Handle all common copyright formats across all file types
- Update `LICENSE` files separately (they often have their own format)
- Create a git branch, apply all updates, and commit

## When to use me

Use this skill at the start of a new year, or when auditing a codebase that has stale copyright notices. Common situations:

- Files still showing `Copyright 2023` or `Copyright 2020-2024`
- LICENSE file has an outdated year
- New contributor files with no copyright notice (optionally add one)
- CI checks requiring up-to-date copyright headers

## Year update rules

The current year is **2026**.

| Found in file                     | Updated to  |
| --------------------------------- | ----------- |
| `2023` (single year)              | `2023-2026` |
| `2021-2024` (range, end is stale) | `2021-2026` |
| `2025-2026` (already current)     | no change   |
| `2026` (already current single)   | no change   |
| `2025` (single, one year behind)  | `2025-2026` |

**Rule:** preserve the original start year. Only update the end year (or add `-CURRENTYEAR` to a single stale year). Never change a year that is already current.

## Step-by-step workflow

### 1. Determine the current year

```bash
date +%Y
```

Use this value as `CURRENT_YEAR` throughout. As of today it is **2026**.

### 2. Find all files with copyright notices

```bash
# Search for all copyright patterns across the codebase
grep -rn --include="*.py" --include="*.js" --include="*.ts" \
  --include="*.jsx" --include="*.tsx" --include="*.java" \
  --include="*.c" --include="*.cpp" --include="*.h" \
  --include="*.go" --include="*.rs" --include="*.rb" \
  --include="*.sh" --include="*.md" --include="*.rst" \
  --include="*.txt" --include="*.toml" --include="*.yaml" \
  --include="*.yml" --include="*.json" --include="*.xml" \
  --include="*.html" --include="*.css" --include="*.scss" \
  -iE "(copyright|©|\(c\))" . \
  --exclude-dir=".git" \
  --exclude-dir="node_modules" \
  --exclude-dir=".venv" \
  --exclude-dir="venv" \
  --exclude-dir="dist" \
  --exclude-dir="build" \
  --exclude-dir="__pycache__"
```

Also check the `LICENSE` file specifically:

```bash
grep -n -iE "(copyright|©|\(c\))" LICENSE LICENSE.md LICENSE.txt 2>/dev/null
```

### 3. Identify stale copyright patterns

Common patterns to look for (all case-insensitive):

```
# Single year (stale if year < CURRENT_YEAR):
Copyright (c) 2023
Copyright © 2022
# Copyright 2021
// Copyright 2020

# Year ranges (stale if end year < CURRENT_YEAR):
Copyright (c) 2018-2023
Copyright © 2020-2024
# Copyright 2019-2025
/* Copyright 2021-2024 */

# Already current (do NOT change):
Copyright (c) 2026
Copyright (c) 2025-2026
Copyright © 2023-2026
```

### 4. Create a new git branch

```bash
git fetch origin
git checkout master 2>/dev/null || git checkout main
git pull
git checkout -b fix/copyright-$(date +%Y%m%d)
```

### 5. Apply updates file by file

For each file containing a stale copyright year, read the file and apply targeted edits.

**Pattern: single stale year → add current year as range end**

Regex to match: `(Copyright\s+(?:\(c\)|©)?\s*)(\d{4})(\s)`

- If captured year `$2` < 2026: replace with `$1$2-2026$3`
- If captured year `$2` == 2026: leave unchanged

**Pattern: range with stale end year → update end year only**

Regex to match: `(Copyright\s+(?:\(c\)|©)?\s*\d{4}-)(\d{4})`

- If captured end year `$2` < 2026: replace with `$12026`
- If captured end year `$2` == 2026: leave unchanged

**Using sed for bulk updates (always review the result):**

```bash
# Update stale single years (e.g. 2020–2025) to ranges ending in 2026
# Review before committing!
grep -rln --include="*.py" -E "Copyright.*\b(202[0-5]|201[0-9]|200[0-9])\b" . | while read f; do
  # Single year to range: "Copyright 2023" -> "Copyright 2023-2026"
  sed -i -E 's/(Copyright[^0-9]*)([0-9]{4})([^0-9-])/\1\2-2026\3/g' "$f"
  echo "Updated: $f"
done
```

**However**, prefer using file editing tools instead of `sed` — read each file, find the specific copyright line, and edit it precisely. This avoids accidentally matching year numbers that are not copyright years (e.g. version numbers, dates in code).

### 6. Handle `LICENSE` file

The LICENSE file often has a standalone year line like:

```
MIT License

Copyright (c) 2021 Author Name
```

Read the LICENSE file, find the copyright line, and update:

- `Copyright (c) 2021` → `Copyright (c) 2021-2026`
- `Copyright (c) 2021-2024` → `Copyright (c) 2021-2026`

### 7. Handle file headers with SPDX or structured copyright

Some projects use SPDX headers:

```
# SPDX-FileCopyrightText: 2023 Author Name
# SPDX-License-Identifier: MIT
```

Update the year in `SPDX-FileCopyrightText` lines using the same rules:

- `2023` → `2023-2026`
- `2021-2024` → `2021-2026`

### 8. Review all changes

```bash
git diff
```

Verify:

- No version numbers were accidentally changed (e.g. `v2023.1` should NOT become `v2023-2026.1`)
- No data/timestamps in code were changed
- Only copyright year fields were updated

If any incorrect changes were made, revert those specific files:

```bash
git checkout -- path/to/file
```

Then manually edit only the copyright line.

### 9. Commit the changes

```bash
git add -A
git commit -m "chore: update copyright years to 2026

Updated stale copyright year notices across the codebase.
Preserved original start years; updated end years to 2026.
No functional changes."
```

## Rules and guardrails

- **Preserve the original start year** — always. `Copyright 2018 Acme Corp` becomes `Copyright 2018-2026 Acme Corp`, never `Copyright 2026 Acme Corp`
- Do NOT change years that appear in contexts other than copyright notices: version numbers (`v2024.1`), dates in comments (`# Added 2023-05`), date literals in code (`"2024-01-01"`), or changelog entries
- When in doubt about whether a year is part of a copyright notice or something else, read the surrounding context carefully before editing
- Do not add copyright headers to files that have none — that is a policy decision for the project maintainers, not an automated task
- If a file uses a different copyright year intentionally (e.g. a vendored file with a third-party copyright), leave it unchanged
- Commit only copyright changes in this branch — do not mix with other work
