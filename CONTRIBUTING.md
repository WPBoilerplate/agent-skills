# Contributing to Agent Skills

We welcome contributions! This project is a great opportunity to share your WordPress expertise with the community.

## Why Contribute Here?

**You don't need to be a coding wizard.** Unlike typical open source projects, Agent Skills is primarily about capturing *knowledge* and *best practices* in a structured format. If you understand WordPress deeply—whether that's block development, performance optimization, plugin security, or any other domain—you can make a meaningful contribution.

Most skills are written in Markdown. The "code" is mostly procedural checklists, decision trees, and reference documentation.

## Ways to Contribute

### 1. Improve Existing Skills

- **Fix outdated information** — WordPress evolves quickly. If you spot something that's changed, open a PR.
- **Add missing edge cases** — Did you hit a gotcha that isn't documented? Add it to the "Failure modes" section.
- **Clarify procedures** — If a step confused you, it'll confuse others. Make it clearer.
- **Expand references** — Add deeper documentation on specific topics.

### 2. Create New Skills

Have expertise in a WordPress area we don't cover yet? Consider adding a new skill.

Before starting:
1. Check [existing skills](skills/) to avoid overlap
2. Review the [Authoring Guide](docs/authoring-guide.md) for structure requirements
3. Open an issue to discuss scope (optional but recommended for larger skills)

Scaffold a new skill:

```bash
node shared/scripts/scaffold-skill.mjs <skill-name> "<description>"
```

### 3. Add Evaluation Scenarios

Every skill needs test scenarios under `eval/scenarios/`. These are simple markdown files describing:
- A realistic prompt/task
- What the AI should do
- How to verify it worked

### 4. Report Issues

Found a skill giving bad advice? Open an issue with:
- Which skill
- What went wrong
- What the correct behavior should be

## Skill Structure

```
skills/<skill-name>/
├── SKILL.md              # Main instructions (short, procedural)
├── references/           # Deep-dive docs on specific topics
│   └── *.md
└── scripts/              # Deterministic helpers (optional)
    └── *.mjs
```

### SKILL.md Requirements

Every `SKILL.md` needs YAML frontmatter with `name`, `description`, and `compatibility`, plus these sections:

1. **When to use** — Conditions that trigger this skill
2. **Inputs required** — What the AI needs to gather first
3. **Procedure** — Step-by-step checklist
4. **Verification** — How to confirm it worked
5. **Failure modes** — Common problems and fixes
6. **Escalation** — When to ask for human help

## Guidelines

- Target WordPress 6.9+ and PHP 7.2.24+
- Avoid legacy patterns (Classic themes, pre-Gutenberg APIs)
- Keep `SKILL.md` short — push depth into `references/`
- Add at least one eval scenario for new skills
- Run `node eval/harness/run.mjs` before submitting

## Submitting Changes

1. Fork the repo
2. Create a branch
3. Make your changes
4. Run `node eval/harness/run.mjs`
5. Open a pull request
