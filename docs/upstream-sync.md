# Upstream Sync

This document outlines the approach to keep skills up to date when WordPress and Gutenberg release new versions.

## Strategy

Skills in this repo target WordPress 6.9+ (see `docs/compatibility-policy.md`). When a new WordPress or Gutenberg release changes APIs, patterns, or best practices covered by a skill, the skill must be updated.

## Manual sync process

1. Monitor WordPress core and Gutenberg release notes.
2. Identify which skills are affected by API or best-practice changes.
3. Update the relevant `SKILL.md` and `references/*.md` files.
4. Update the `compatibility:` frontmatter if the minimum version changes.
5. Run `node eval/harness/run.mjs` to verify scenarios still pass.
6. Open a PR with a clear description of what changed and why.

## Automated sync (future)

A script `shared/scripts/update-upstream-indices.mjs` (planned) will fetch upstream data and update version indexes. CI can run this on a schedule and open PRs when changes are detected.

## Data sources

- WordPress core releases: https://wordpress.org/news/category/releases/
- Gutenberg releases: https://github.com/WordPress/gutenberg/releases
- WordPress developer docs: https://developer.wordpress.org/
