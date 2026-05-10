# Agent Skills for WPBoilerplate

**Teach AI coding assistants how to build plugins the WPBoilerplate way.**

Agent Skills are portable bundles of instructions, checklists, and scripts that help AI assistants (Claude, Copilot, Codex, Cursor, etc.) understand the WPBoilerplate plugin architecture, avoid common mistakes, and follow the established patterns.

## Why Agent Skills?

AI coding assistants are powerful, but they often:
- Register hooks directly in constructors instead of through the Loader
- Define constants outside `define_constants()`, causing PHP notices
- Hardcode asset version strings instead of reading `*.asset.php` manifests
- Put classes in the wrong namespace or directory, breaking PSR-4 autoloading
- Edit `build/` directly instead of `src/`

Agent Skills solve this by giving AI assistants **expert-level WPBoilerplate knowledge** in a format they can actually use.

## Available Skills

| Skill | What it teaches |
|---|---|
| **wp-plugin-development** | Hooks via the Loader singleton, PSR-4 namespace layout, asset manifests, admin pages, lifecycle hooks, and the `@wordpress/scripts` build pipeline |
| **wp-packages-strategy** | Prioritize `@wordpress/*` packages over external deps; detect React conflicts; webpack aliasing; `validate-packages.mjs` script |

## Quick Start

### Install globally for Claude Code

```bash
# Clone agent-skills
git clone https://github.com/WPBoilerplate/agent-skills.git
cd agent-skills

# Build the distribution
node shared/scripts/skillpack-build.mjs --clean

# Install all skills globally (available across all projects)
node shared/scripts/skillpack-install.mjs --global

# Or install a specific skill only
node shared/scripts/skillpack-install.mjs --global --skills=wp-plugin-development
```

This installs skills to `~/.claude/skills/` where Claude Code will automatically discover them.

### Install into your plugin repo

```bash
# Clone agent-skills
git clone https://github.com/WPBoilerplate/agent-skills.git
cd agent-skills

# Build the distribution
node shared/scripts/skillpack-build.mjs --clean

# Install into your WPBoilerplate plugin
node shared/scripts/skillpack-install.mjs --dest=../your-plugin --targets=codex,vscode,claude,cursor
```

This copies skills into:
- `.codex/skills/` for OpenAI Codex
- `.github/skills/` for VS Code / GitHub Copilot
- `.claude/skills/` for Claude Code (project-level)
- `.cursor/skills/` for Cursor (project-level)

### Install globally for Cursor

```bash
node shared/scripts/skillpack-install.mjs --targets=cursor-global
```

This installs skills to `~/.cursor/skills/` where Cursor will discover them.

### Available options

```bash
# List available skills
node shared/scripts/skillpack-install.mjs --list

# Dry run (preview without installing)
node shared/scripts/skillpack-install.mjs --global --dry-run

# Install specific skills to a project
node shared/scripts/skillpack-install.mjs --dest=../my-plugin --targets=claude,cursor --skills=wp-plugin-development
```

### Manual installation

Copy any skill folder from `skills/` into your project's instructions directory for your AI assistant:

- Claude Code: `.claude/skills/`
- Cursor: `.cursor/skills/`
- VS Code / Copilot: `.github/skills/`
- OpenAI Codex: `.codex/skills/`

## How It Works

Each skill contains:

```
skills/<skill-name>/
├── SKILL.md              # Main instructions (when to use, procedure, verification)
├── references/           # Deep-dive docs on specific topics
│   └── *.md
└── scripts/              # Deterministic helpers (detection, validation)
    └── *.mjs
```

When you ask your AI assistant to work on a WPBoilerplate plugin, it reads these skills and follows the documented procedures rather than guessing.

## Compatibility

- **WordPress 6.9+** (PHP 7.4+ per WPBoilerplate `composer.json`)
- Targets the `main` branch of [WPBoilerplate/wordpress-plugin-boilerplate](https://github.com/WPBoilerplate/wordpress-plugin-boilerplate)
- Works with any AI assistant that supports project-level instructions

## Contributing

We welcome contributions! See [CONTRIBUTING.md](CONTRIBUTING.md) for details.

```bash
# Scaffold a new skill
node shared/scripts/scaffold-skill.mjs <skill-name> "<description>"

# Run the skill test suite
node skills/<skill-name>/scripts/test-skill.mjs
```

## Documentation

- [Authoring Guide](docs/authoring-guide.md) — How to create and improve skills
- [Principles](docs/principles.md) — Design philosophy
- [Packaging](docs/packaging.md) — Build and distribution
- [Compatibility Policy](docs/compatibility-policy.md) — Version targeting
- [AI Authorship](docs/ai-authorship.md) — How AI tools were used

## License

GPL-2.0-or-later
