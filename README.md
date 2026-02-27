# Custom Skills

A collection of OpenCode and Claude Code compatible agent skills for common development tasks.

## Skills

| Skill | Description |
|-------|-------------|
| [codespell-fix](./skills/codespell-fix/) | Run codespell to detect and fix spelling mistakes across the codebase |
| [write-good-fix](./skills/write-good-fix/) | Detect and fix weak English prose (passive voice, weasel words, adverbs) |
| [python-modernize](./skills/python-modernize/) | Migrate Python projects to uv + ruff + pyproject.toml |
| [copyright-update](./skills/copyright-update/) | Update stale copyright years to the current year |

## Compatibility

These skills are compatible with both:

- **OpenCode** - Place in `~/.config/opencode/skills/` or `.opencode/skills/`
- **Claude Code** - Place in `~/.claude/skills/` or within a plugin's `skills/` directory

Each skill includes trigger phrases in the description for automatic activation.

## Installation

### For OpenCode

**Global installation (available in all projects):**
```bash
mkdir -p ~/.config/opencode/skills
git clone https://github.com/albedo-c/custom-skills.git ~/.config/opencode/skills/custom-skills
```

**Per-project installation:**
```bash
mkdir -p .opencode/skills
cp -r /path/to/custom-skills/skills/* .opencode/skills/
```

### For Claude Code

**Global installation:**
```bash
mkdir -p ~/.claude/skills
git clone https://github.com/albedo-c/custom-skills.git ~/.claude/skills/custom-skills
```

**As a plugin:**
```bash
# Copy to your plugin's skills directory
cp -r skills/ /path/to/your-plugin/skills/
```

## Required Tools

Each skill requires specific tools to be installed:

| Skill | Tool | Install |
|-------|------|---------|
| codespell-fix | `codespell` | `pip install codespell` or `uvx codespell` |
| write-good-fix | `write-good` | `npm install -g write-good` or `npm install -g write-good --prefix ~/.local` |
| python-modernize | `uv`, `ruff` | `uv` (already installed), `uv tool install ruff` |
| copyright-update | (none) | Built-in tools only |

Verify installations:
```bash
codespell --version    # >= 2.0
write-good --version  # >= 1.0
uv --version          # >= 0.10
ruff --version        # >= 0.15
```

## Usage

### OpenCode

When you describe a task matching a skill, OpenCode will automatically load and use it. You can also explicitly invoke:

```
Apply the codespell-fix skill to find and fix spelling errors
```

### Claude Code

Skills are automatically discovered when you describe a task. Trigger phrases are included in each skill's description:

- "fix spelling", "check for typos", "spell check"
- "fix grammar", "improve writing", "fix passive voice"
- "modernize Python", "migrate to uv", "set up ruff"
- "update copyright", "fix copyright years"

## License

MIT License - See individual skill files for details.

## Contributing

Feel free to open issues or PRs to improve these skills.
