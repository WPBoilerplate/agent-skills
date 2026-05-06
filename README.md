# Agent Skills for WordPress

**Teach AI coding assistants how to build WordPress the right way.**

Agent Skills are portable bundles of instructions, checklists, and scripts that help AI assistants (Claude, Copilot, Codex, Cursor, etc.) understand WordPress development patterns, avoid common mistakes, and follow best practices.

## Why Agent Skills?

AI coding assistants are powerful, but they often:
- Generate outdated WordPress patterns (pre-Gutenberg, pre-block themes)
- Miss critical security considerations in plugin development
- Skip proper block deprecations, causing "Invalid block" errors
- Ignore existing tooling in your repo

Agent Skills solve this by giving AI assistants **expert-level WordPress knowledge** in a format they can actually use.

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

When you ask your AI assistant to work on WordPress code, it reads these skills and follows the documented procedures rather than guessing.

## Compatibility

- **WordPress 6.9+** (PHP 7.2.24+)
- Works with any AI assistant that supports project-level instructions

## Quick Start

### Manual installation

Copy any skill folder from `skills/` into your project's instructions directory for your AI assistant:

- Claude Code: `.claude/skills/`
- Cursor: `.cursor/skills/`
- VS Code / Copilot: `.github/skills/`
- OpenAI Codex: `.codex/skills/`

### Build and install (automated)

```bash
# Build distribution packages
node shared/scripts/skillpack-build.mjs --clean

# Install into your project
node shared/scripts/skillpack-install.mjs --dest=../your-wp-project --targets=claude,cursor

# Install globally for Claude Code
node shared/scripts/skillpack-install.mjs --global
```

## Contributing

We welcome contributions! See [CONTRIBUTING.md](CONTRIBUTING.md) for details.

```bash
# Scaffold a new skill
node shared/scripts/scaffold-skill.mjs <skill-name> "<description>"

# Run evaluation harness
node eval/harness/run.mjs
```

## Documentation

- [Authoring Guide](docs/authoring-guide.md) — How to create and improve skills
- [Principles](docs/principles.md) — Design philosophy
- [Packaging](docs/packaging.md) — Build and distribution
- [Compatibility Policy](docs/compatibility-policy.md) — Version targeting
- [AI Authorship](docs/ai-authorship.md) — How AI tools were used

## License

GPL-2.0-or-later
