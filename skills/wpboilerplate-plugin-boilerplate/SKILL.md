---
name: wpboilerplate-plugin-boilerplate
description: "Use when working in plugins scaffolded from WPBoilerplate/wordpress-plugin-boilerplate (main branch): adding hooks via the Loader singleton, defining constants in Includes\\Main, adding admin pages under Admin\\Partials, enqueuing assets from build/*.asset.php manifests, and respecting the PSR-4 namespace map (Includes/Admin/Public)."
compatibility: "Targets WordPress 6.9+ (PHP 7.4+ per composer.json). Assumes @wordpress/scripts build pipeline and the namespaced PSR-4 layout from the main branch (not the legacy master branch)."
---

# WPBoilerplate Plugin Boilerplate

## When to use

Use this skill when working in a plugin scaffolded from
[WPBoilerplate/wordpress-plugin-boilerplate](https://github.com/WPBoilerplate/wordpress-plugin-boilerplate)
(`main` branch), for tasks such as:

- adding or moving WordPress hooks/actions/filters
- registering admin pages, menus, or submenu pages
- enqueuing admin or frontend assets (CSS/JS)
- defining new plugin constants
- adding activation, deactivation, or uninstall logic
- adding or modifying i18n / text-domain loading
- refactoring existing code into the namespaced PSR-4 layout
- adding new Gutenberg blocks or block styles
- scaffolding a renamed copy via `init-plugin.sh`

## Inputs required

- Plugin root path and slug (post-`init-plugin.sh` rename if applicable).
- Whether `init-plugin.sh` has been run (affects namespace prefix and text domain).
- Target WordPress + PHP versions (boilerplate requires PHP 7.4+).
- Whether `build/` artifacts are present (`npm run build` has been run).

## Procedure

### 0) Rename the boilerplate (new plugins only)

If `init-plugin.sh` has **not** yet been run, run it once from the plugin parent directory:

```bash
bash wordpress-plugin-boilerplate/init-plugin.sh
```

The script prompts for a **Title Case** plugin name (e.g. `My Awesome Plugin`) and derives
every identifier automatically:

| Variable | Example | Used for |
|---|---|---|
| `slug` | `my-awesome-plugin` | directory name, text domain, handle |
| `prefix` | `my_awesome_plugin` | function prefixes, option names |
| `define` | `MY_AWESOME_PLUGIN` | PHP constants |
| `namespace` / `class` | `My_Awesome_Plugin` | PSR-4 namespace prefix |

It then does `git mv` + `git grep`/`sed` replacements across every file in the repo.
**After running it**, all boilerplate identifiers (`WordPress_Plugin_Boilerplate`, etc.) are
replaced with your plugin's identifiers. Verify with:

```bash
grep -r "WordPress_Plugin_Boilerplate" . --include="*.php" | grep -v vendor
# should return no results
```

The script also optionally installs extra Composer packages (GitHub updater, etc.) and
removes dev tooling (PHPCS, PHPStan) if you decline them during the interactive prompts.

> **Never** run `init-plugin.sh` on a plugin that is already in production — it does
> in-place find-and-replace across all tracked files.

### 1) Respect the namespace + folder layout

- `Admin\*` classes → `admin/`
- `Includes\*` classes → `includes/`
- `Public\*` classes → `public/`
- New classes must match the PSR-4 map in `composer.json` (and the custom `Autoloader.php`).
- Run `composer dump-autoload` after adding a new class file.

See: `references/structure.md`

### 2) Follow the boot flow; put constants in the right file

- Only `WORDPRESS_PLUGIN_BOILERPLATE_PLUGIN_FILE` is defined in the bootstrap.
- All other constants belong in `includes/Main.php::define_constants()` using the private
  `define($name, $value)` guard. Never define constants elsewhere.
- Hook registration happens in `define_admin_hooks()` / `define_public_hooks()` via the Loader,
  never with direct `add_action`/`add_filter` calls inside class constructors.
- The `apply_filters('wordpress-plugin-boilerplate-load', true)` gate in `load_hooks()` is the
  supported kill switch for third-party integrations.

See: `references/boot-flow.md`

### 3) Add admin pages and enqueues correctly

- Create new screen classes under `admin/Partials/` with namespace `...\Admin\Partials`.
- Instantiate the class in `includes/Main.php::define_admin_hooks()`.
- Register every hook through `$this->loader->add_action(...)` — never directly.
- To add new admin assets, add entry points to `webpack.config.js` and enqueue via
  `Admin\Main` methods using the `*.asset.php` manifest.

See: `references/admin.md`

### 4) Security baseline (always)

Before shipping:

- Validate/sanitize input early; escape output late.
- Use nonces to prevent CSRF and capability checks for authorization.
- Avoid directly trusting `$_POST` / `$_GET`; use `wp_unslash()` and specific keys.
- Use `$wpdb->prepare()` for SQL; avoid building SQL with string concatenation.

See: `references/security.md`

### 5) Data storage, cron, migrations (if needed)

- Prefer options (`get_option` / `update_option`) for small config; custom tables only if the data volume or query patterns make options impractical.
- For custom tables, install via [`berlindb/core`](https://github.com/berlindb/core) as a Composer package — it provides query, schema, and row classes that wrap `$wpdb` safely and consistently.
- For cron tasks, ensure idempotency (safe to run twice) and provide a manual run path via WP-CLI or an admin action.
- For schema changes, write upgrade routines keyed on a stored schema version option; never assume the DB matches the current code.

See: `references/data-and-cron.md`

### 6) Add frontend code under public/

- All frontend classes go under `public/` with namespace `...\Public`.
- Instantiate in `includes/Main.php::define_public_hooks()` and register via the Loader.
- Source JS → `src/js/`, source SCSS → `src/scss/`. Never edit `build/` directly.
- Read asset versions and dependencies from `build/*.asset.php` — never hardcode.

See: `references/public.md`

### 7) Lifecycle: Activator, Deactivator, uninstall, i18n

- Activation setup (tables, options, roles) → `Includes\Activator::activate()`.
- Deactivation lightweight cleanup (cron, transients) → `Includes\Deactivator::deactivate()`.
- Data removal (delete_option, drop tables) → `uninstall.php` only.
- i18n is loaded on `init`, not `plugins_loaded` — do not move it.

See: `references/lifecycle.md`

### 8) Build assets through @wordpress/scripts

- Run `npm run build` (production) or `npm run start` (watch) before testing.
- Standard JS/SCSS entries, custom blocks (`src/blocks/**/block.json`), and block stylesheets
  (`src/scss/blocks/core/`) are all picked up automatically by `webpack.config.js`.
- Static assets (`src/media/`, `src/fonts/`) are copied to `build/` by CopyPlugin.

**To add a new JS or CSS file**, follow this 5-step workflow (full diffs in the reference):

1. **Create** the source file in `src/js/<name>.js` and/or `src/scss/<name>.scss`.
2. **Register** a new entry in `webpack.config.js` `entry:` object pointing at that source file.
3. **Load the manifest** — in the constructor of `Admin\Main` (backend) or `Public\Main`
   (frontend), add `include …build/js/<name>.asset.php` and/or `…build/css/<name>.asset.php`.
4. **Enqueue** — call `wp_enqueue_script` / `wp_enqueue_style` inside `enqueue_scripts()` /
   `enqueue_styles()` using the manifest arrays for dependencies and version. Never hardcode either.
5. **Build** — run `npm run build` and confirm `build/js/<name>.js` + `build/js/<name>.asset.php`
   (and/or their CSS equivalents) are present.

See: `references/build-system.md` → "Adding a new JS / CSS file — complete workflow"

## Verification

- `composer dump-autoload` completes with no errors.
- `npm run build` produces `*.asset.php` for every enqueued entry.
- Plugin activates with no PHP fatals or notices.
- Admin menu item appears at `/wp-admin/admin.php?page=<plugin-slug>`.
- Frontend assets enqueue on the correct pages (check with Query Monitor or browser devtools).
- Settings save and read correctly (capability check + nonce enforced).
- Uninstall removes all intended data — and nothing else.
- `WORDPRESS_PLUGIN_BOILERPLATE_PLUGIN_URL` holds a URL, not a version string (known source bug —
  the guard prevents a fatal; the first definition wins).
- PHPCS passes: `./vendor/bin/phpcs`. The project uses **WordPress-Extra + WordPress-Docs + PHPCompatibility** standards (`phpcs.xml.dist`). Exclusions allow short array syntax and relax some doc-comment rules. `vendor/`, `node_modules/`, and `build/` are excluded. Minimum PHP target is 7.4. Dev packages required: `squizlabs/php_codesniffer`, `wp-coding-standards/wpcs`, `phpcompatibility/php-compatibility`, `dealerdirect/phpcodesniffer-composer-installer`.
- PHPUnit / PHPCS repo lint passes (if present); JS build succeeds if the plugin ships assets.

## Failure modes / debugging

- **PHP fatal on every admin or frontend page load** — `build/*.asset.php` files are missing because `npm run build` has not been run. The constructors of `Admin\Main` and `Public\Main` `include` those manifests directly; a missing file is a PHP fatal, not a silent 404. Run `npm run build`.
- **Class not found** — namespace mismatch in file path, or `composer dump-autoload` not run after adding the file.
- **Constants double-defined PHP notice** — constant defined outside `define_constants()` without the guard; move it inside and use the private `define()` pattern.
- **Hooks not firing** — hook registered with direct `add_action` instead of through the Loader, or `apply_filters('wordpress-plugin-boilerplate-load', true)` was filtered to `false`.
- **Asset 404 after build** — check that `WORDPRESS_PLUGIN_BOILERPLATE_PLUGIN_URL` is correct (see known double-define bug in `boot-flow.md`).
- **Activation hook not firing** — hook registered in the wrong scope (not at main-file root), wrong main file path, or plugin is network-activated (use `register_activation_hook` at top level of the main plugin file, not inside a class method called later).
- **Activation / deactivation callback not running** — the bootstrap registers namespaced functions (`WordPress_Plugin_Boilerplate\wordpress_plugin_boilerplate_activate`), not class methods. The Autoloader is not active at that point; the class file is `require_once`d manually inside each function. Do not refactor these to instance methods.
- **Settings not saving** — settings not registered with `register_setting()`, wrong option group name, missing capability check, or nonce failure. Confirm all four are correct before debugging further.
- **Security regression** — nonce present but capability check missing (or vice-versa); or user input sanitized on save but not escaped on output. Always pair the two.
- **Text-domain not loading** — `do_load_textdomain()` moved to `plugins_loaded`; it must stay on `init`.
- **Vendor conflict / class not found in vendor** — run `composer install` then `composer dump-autoload`; Mozart may need to re-scope namespaces.

See: `references/debugging.md`

## Escalation

- Boilerplate `agents.md`: `https://github.com/WPBoilerplate/wordpress-plugin-boilerplate/blob/main/agents.md`
- Boilerplate README: `https://github.com/WPBoilerplate/wordpress-plugin-boilerplate/blob/main/README.md`
- WordPress Plugin Handbook: `https://developer.wordpress.org/plugins/`
