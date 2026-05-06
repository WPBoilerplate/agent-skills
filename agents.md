# AI Agent Instructions — WPBoilerplate Agent Skills

This file tells AI coding assistants how to work inside **this repository**. It is not a skill
itself — it is the maintenance guide for the people and agents who author, test, and ship skills.

## What this repo is

A collection of portable agent skills that teach AI assistants how to build WordPress plugins
using [WPBoilerplate/wordpress-plugin-boilerplate](https://github.com/WPBoilerplate/wordpress-plugin-boilerplate).

Skills are plain-text bundles (`SKILL.md` + `references/*.md` + `scripts/*.mjs`) that AI assistants
read at task time. The `shared/scripts/` tooling builds and distributes them to any AI tool.

## Repo layout

```
agent-skills/
├── skills/                          # One subdirectory per skill
│   └── <skill-name>/
│       ├── SKILL.md                 # Required — frontmatter + 6 required sections
│       ├── references/              # Deep-dive topic docs (*.md)
│       └── scripts/                 # Deterministic helpers (*.mjs)
├── shared/scripts/                  # Skillpack tooling
│   ├── skillpack-build.mjs          # Compiles skills/ → dist/
│   ├── skillpack-install.mjs        # Installs from dist/ to tool directories
│   └── scaffold-skill.mjs           # Scaffolds a new skill stub
├── eval/scenarios/                  # JSON evaluation scenarios (one per skill)
├── docs/                            # Authoring guides and policy docs
├── CONTRIBUTING.md
├── README.md                        # User-facing install guide
└── agents.md                        # ← you are here
```

## Rules for working in this repo

### Never edit build output

The `dist/` directory is generated. It is gitignored. Do not create or edit files inside it.
Always work in `skills/`, `shared/`, `eval/`, or `docs/`.

### One skill = one directory under skills/

Every skill directory must contain `SKILL.md`. Directories without it are ignored by the
skillpack tooling.

### SKILL.md frontmatter is required

Every `SKILL.md` must start with valid YAML frontmatter:

```yaml
---
name: <skill-name>           # must match the directory name exactly
description: "..."           # one-line trigger description used by AI assistants
compatibility: "..."         # WordPress + PHP version target
---
```

### SKILL.md must contain all 6 sections

In this order:

1. `## When to use`
2. `## Inputs required`
3. `## Procedure`
4. `## Verification`
5. `## Failure modes / debugging`
6. `## Escalation`

### Reference files must end with an upstream link

Every file in `references/` must include a line like:

```
- Upstream reference: https://github.com/WPBoilerplate/wordpress-plugin-boilerplate/blob/main/...
```

This keeps references traceable and makes upstream drift detectable.

### Scripts must be plain Node.js ES modules

All scripts in `scripts/` must use `.mjs` extension, import only `node:*` built-ins, and
exit 0 on success. No third-party dependencies.

## How to add a new skill

1. **Scaffold the skeleton:**
   ```bash
   node shared/scripts/scaffold-skill.mjs <skill-name> "<one-line description>"
   ```
   This creates `skills/<skill-name>/SKILL.md` and `eval/scenarios/<skill-name>.md`.

2. **Fill in `SKILL.md`** — complete all 6 required sections with accurate procedures.

3. **Add reference files** under `references/` for any topic too detailed for `SKILL.md`.
   Each file must end with an upstream reference link.

4. **Add a detector script** under `scripts/detect_<skill-name>.mjs` that confirms the skill
   applies to the current repo and exits with a JSON report. See
   `skills/wpboilerplate-plugin-boilerplate/scripts/detect_wpboilerplate.mjs` for the pattern.

5. **Add a test script** under `scripts/test-skill.mjs` that validates the skill structure and
   detector output. It must exit 0 when all tests pass, exit 1 on failure.

6. **Register the skill** in `docs/skill-set-v1.md`.

7. **Verify:**
   ```bash
   node skills/<skill-name>/scripts/test-skill.mjs
   ```

## How to build and test the distribution

```bash
# Build all skills into dist/
node shared/scripts/skillpack-build.mjs --clean

# Preview global install without making changes
node shared/scripts/skillpack-install.mjs --global --dry-run

# List available skills
node shared/scripts/skillpack-install.mjs --list
```

## How to install skills

```bash
# Globally for Claude Code
node shared/scripts/skillpack-install.mjs --global

# Globally for Cursor
node shared/scripts/skillpack-install.mjs --targets=cursor-global

# Into a WPBoilerplate plugin repo
node shared/scripts/skillpack-install.mjs --dest=../my-plugin --targets=claude,cursor
```

## Maintenance protocol

### When the WPBoilerplate boilerplate changes

1. Check [WPBoilerplate/wordpress-plugin-boilerplate](https://github.com/WPBoilerplate/wordpress-plugin-boilerplate)
   for changes to `includes/Main.php`, `admin/Main.php`, `public/Main.php`, `webpack.config.js`,
   `composer.json`, and `package.json`.
2. Update the affected reference files in `skills/wpboilerplate-plugin-boilerplate/references/`.
3. Update upstream reference links if file paths changed.
4. Run `node skills/wpboilerplate-plugin-boilerplate/scripts/test-skill.mjs` to confirm no regressions.
5. Bump the `compatibility:` frontmatter if the WordPress or PHP version target changed.

### When adding a dependency or changing repo structure

Update this `agents.md` file to reflect the change so future agents have accurate information.

### README.md sync

`README.md` is the user-facing install guide. Keep the **Available Skills** table in sync with
the actual directories under `skills/`. When a skill is added or removed, update the table.

## What NOT to do

- Do not call `add_action()` or `add_filter()` directly in any script (these are WordPress APIs,
  not relevant here).
- Do not commit `dist/` — it is gitignored and regenerated on demand.
- Do not create skill directories without a `SKILL.md` — the tooling ignores them silently.
- Do not hardcode absolute paths in scripts — use `process.cwd()` and `path.join()`.
- Do not add npm dependencies — all scripts must use only `node:*` built-ins.

## Reference links

- Boilerplate repo: https://github.com/WPBoilerplate/wordpress-plugin-boilerplate
- Boilerplate agents.md: https://github.com/WPBoilerplate/wordpress-plugin-boilerplate/blob/main/agents.md
- WordPress/agent-skills (upstream pattern): https://github.com/WordPress/agent-skills
- WordPress Plugin Handbook: https://developer.wordpress.org/plugins/
