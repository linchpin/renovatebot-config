<table width="100%">
	<tr>
		<td align="left" width="70%">
			<strong>Linchpin Renovate Config</strong><br />
			The shared Renovate (Mend) presets used across the WordPress projects Linchpin builds and maintains. One source of truth for dependency grouping, commit prefixes and package routing.
		</td>
		<td align="center" width="30%">
			<a href="https://docs.renovatebot.com/config-presets/"><img src="https://img.shields.io/badge/Renovate-presets-1A1F6C?logo=renovate&logoColor=fff" alt="Renovate presets" /></a>
			<a href="https://github.com/linchpin/renovatebot-config"><img src="https://img.shields.io/badge/Maintained%3F-yes-green.svg" alt="Maintained: yes" /></a>
		</td>
	</tr>
	<tr>
		<td>
			A <strong><a href="https://linchpin.com">Linchpin</a></strong> preset · <em>Actively maintained</em>
		</td>
		<td align="center" width="30%">
			<img src="https://assets.linchpin.com/linchpin-logo-primary.svg" width="100" alt="Linchpin" />
		</td>
	</tr>
</table>

## Available presets

This repository hosts two presets. Extend **one** of them — `automerge` already pulls in
everything from `default`, so extending both is redundant.

| Preset | Extend with | File | Use when |
| --- | --- | --- | --- |
| Default | `github>linchpin/renovatebot-config` | [`default.json`](default.json) | Every update waits for a human to merge it |
| Automerge | `github>linchpin/renovatebot-config:automerge` | [`automerge.json`](automerge.json) | Non-major updates should merge themselves |

`automerge.json` extends `github>linchpin/renovatebot-config` and changes **only** the automerge
behaviour. Grouping, commit prefixes, labels and package routing are defined once in
`default.json` and inherited, so the two presets can never drift apart.

### Usage

A project's own `renovate.json` extends the preset and adds only what is specific to that
project — credentials, ignored paths, and local overrides:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": [
    "config:recommended",
    "github>linchpin/renovatebot-config:automerge"
  ],
  "ignoreTests": true,
  "rebaseWhen": "conflicted",
  "rangeStrategy": "bump",
  "hostRules": [
    {
      "hostType": "packagist",
      "matchHost": "packagist.linchpin.com",
      "authType": "Basic",
      "username": "{{ secrets.SATISPRESS_USER }}",
      "password": "{{ secrets.SATISPRESS_PASSWORD }}"
    }
  ]
}
```

> [!IMPORTANT]
> For the `automerge` preset to actually merge anything, the consuming repository must allow it:
> **Repo → Settings → General → "Allow auto-merge"**. Without that setting Renovate opens the
> PRs and they sit there waiting.

The former `github>linchpin/renovatebot-automerge-config` repository is **deprecated** and now
forwards to `github>linchpin/renovatebot-config:automerge`. Point new and existing projects at
the preset above.

## Commit prefixes

Every prefix these presets emit is a valid [`@linchpinagency/commitlint-config`](https://github.com/linchpin/commitlint-config)
type and scope, and maps to a changelog section in
[`@linchpinagency/release-please-config`](https://github.com/linchpin/release-please-config):

| Prefix | Covers | Changelog section |
| --- | --- | --- |
| `update(wp-plugin): [.org]` | Public plugins via `wp-packages` or `wpackagist` | Changes to Existing Features 💅 |
| `update(wp-plugin): [packagist]` | Plugins from `packagist.linchpin.com` | Changes to Existing Features 💅 |
| `update(wp-theme): [.org]` | Public themes via `wp-packages` or `wpackagist` | Changes to Existing Features 💅 |
| `update(wp-theme): [packagist]` | Themes from `packagist.linchpin.com` | Changes to Existing Features 💅 |
| `build(npm)` | npm packages inside custom themes and plugins | *hidden* |
| `build(composer)` | Composer packages inside custom themes and plugins | *hidden* |
| `build(deps)` | Project-level build and platform dependencies | *hidden* |

`wp-plugin` and `wp-theme` are **scopes**, not types. The type says what happened — `update` for a
bump — and the scope says what it happened to. A package's registry travels in the bracketed tag,
which is what the scope slot used to carry.

release-please groups changelog sections strictly by commit type — `changelog-sections[].type` is the
only key its schema offers, there is no scope key — so WordPress updates share the section `update`
maps to rather than getting one of their own. The scope renders as the bold prefix on each bullet,
which is what keeps plugin and theme lines apart inside it, and the tag stays visible in the text:

```markdown
### Changes to Existing Features 💅

* **wp-plugin:** [.org] Update Plugins wordpress.org
* **wp-plugin:** [packagist] Update Plugins packagist.linchpin.com
* **wp-theme:** [.org] Update Themes wordpress.org
```

`commitBody` carries the same prefix as its rule's header. release-please parses those body lines
into their own changelog bullets, so a body labelled with a different type scatters one grouped
update across two sections.

Changing a prefix here means changing the type list in `commitlint-config` and the section list in
`release-please-config` — the latter's test suite asserts the two lists are identical, so a
mismatch fails CI rather than silently dropping commits from a changelog.

## Goals of this configuration file

### General Config
- Group all pull requests into a `maintenance/MM-YYYY` named branch

### Project Build Config

#### npm
- Group all [minor, patch, bump] **build** related `npm` packages into a single grouped pull request
- Create individual [major] **build** related `npm` packages into single pull requests

#### composer 
- Group all [minor, patch, bump] **build** related `composer` packages into a single grouped pull request
- Create individual [major] **build** related `composer` packages into single pull requests

### wordpress.org, wp-packages, wpackagist, packagist.linchpin.com maintenance config

> Public wordpress.org plugins and themes may be sourced from either
> [wp-packages](https://wp-packages.org) (`wp-plugin/*`, `wp-theme/*`) or the older
> [wpackagist](https://wpackagist.org) (`wpackagist-plugin/*`, `wpackagist-theme/*`).
> Both vendor prefixes are routed identically, so a project can migrate between them
> without changing its Renovate config.

#### WordPress Plugins
- Group all [minor, patch, bump] **plugin** updates from `wp-packages` or `wpackagist` (wordpress.org) and `packagist.linchpin.com` into a single pull request
- Create individual pull requests for any [major] **plugin** updates from `wp-packages` or `wpackagist` (wordpress.org) and `packagist.linchpin.com` reviewed by a human before merging

#### WordPress Themes
- Group all [minor, patch, bump] **theme** updates from `wp-packages` or `wpackagist` (wordpress.org) into a single pull request, tagged `[.org]`
- Group all [minor, patch, bump] **theme** updates from `packagist.linchpin.com` into a single pull request, tagged `[packagist]`
- Create individual pull requests for any [major] **theme** updates from either source, reviewed by a human before merging

Themes are split by source so each group can carry an honest `[.org]` or `[packagist]` tag. Note that
`linchpin/**` matches Linchpin themes as well as plugins, so the plugin rules exclude
`!linchpin/**theme` — without that exclusion the plugin rules sit later in `packageRules` and would
win, routing Linchpin themes into the plugin group.

### Linchpin Built Custom WordPress Plugins

#### npm
- Group all [minor, patch, bump] **plugin** related `npm` packages into a single grouped pull request
- Create individual [major] **plugin** related `npm` packages into a single group pull request

#### composer
- Group all [minor, patch, bump] **plugin** related `composer` packages into a single grouped pull request
- Create individual [major] **plugin** related `composer` packages into a single group pull request

### Linchpin Built Custom WordPress Themes 
#### npm
- Group all [minor, patch, bump] **theme** related `npm` packages into a single grouped pull request
- Create individual [major] **theme** related `npm` packages into a single group pull request

#### composer
- Group all [minor, patch, bump] **theme** related `composer` packages into a single grouped pull request
- Create individual [major] **theme** related `composer` packages into a single group pull request

## Changing a preset

Both presets are consumed live off `main` by every project that extends them, so a merge here
reaches all of them on Renovate's next run. There is no version pinning.

- Keep grouping, prefixes and routing in `default.json`. `automerge.json` should only ever
  contain automerge behaviour.
- Renovate resolves `github>owner/repo` to `default.json` and `github>owner/repo:name` to
  `name.json` at the repo root, so renaming a file changes the public preset name — and breaks
  every project still extending the old one.

Every pull request runs
[`validate-renovate-presets.yml`](.github/workflows/validate-renovate-presets.yml), which checks:

| Check | Catches |
| --- | --- |
| `jq empty` on each file | Trailing commas and malformed JSON. Renovate's own validator parses JSON5 and accepts these |
| `renovate-config-validator --no-global` | Unknown options, wrong types, invalid `matchPackageNames` patterns |
| Preset resolution via the GitHub API | A moved, renamed or typo'd `extends` reference. The validator does not fetch remote presets, so these otherwise fail only at runtime |
| `--strict` migration report | Deprecated options Renovate still auto-migrates. **Non-blocking** — reported, not enforced |

There are currently two pending migrations in `default.json`, both auto-migrated by Renovate at
runtime: `lockFileMaintenance.rebaseStalePrs` → `rebaseWhen`, and `baseBranches` →
`baseBranchPatterns`.

![Linchpin an award winning digital agency building immersive, high performing web experiences](https://assets.linchpin.com/github/linchpin-github-repo-banner.jpg)
