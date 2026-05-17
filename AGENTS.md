# AGENTS.md — tachiyomi-extensions-for-kuron

## Project Overview

Fork of Tachiyomi/Keiyoushi manga reader extensions. Auto-syncs from [Keiyoushi](https://github.com/keiyoushi/extensions-source) every 6 hours via CI. Being analyzed as R&D for **Kuron app** ([nhasixapp](https://github.com/shirokun20/nhasixapp)) which uses a **config-driven** source system instead of compiled Kotlin extensions.

## Build & Commands

| Command | Purpose |
|---------|---------|
| `./gradlew` | Build all extension APKs |
| `./gradlew spotlessApply` | Auto-format Kotlin/Groovy code |
| `./gradlew spotlessCheck` | Check formatting (CI gate) |
| `./gradlew clean` | Clean build artifacts |
| `./gradlew :src:en:mangadex:assembleDebug` | Build single extension |

**Work on one extension only**: Edit `settings.gradle.kts` — comment out `loadAllIndividualExtensions()` and uncomment `loadIndividualExtension("lang", "name")`. This avoids loading 1000+ modules.

**Requirements**: JDK 17, Android SDK 34 (compile), min SDK 21. Gradle JVM args: `-Xmx6144m`.

## Module Structure

```
src/<lang>/<source>/        # Individual extensions (21 language dirs)
lib/                        # Reusable utility libs (14 modules)
lib-multisrc/               # Multi-source theme frameworks (67 themes)
core/                       # Shared utilities (keiyoushi.utils)
source-config-templates/    # JSON config templates for Kuron (see below)
template/                   # Extension documentation templates
gradle/build-logic/         # Custom Gradle plugins
```

**Language codes**: `src/<lang>/` uses ISO 639-1 (e.g., `en`, `ja`, `id`, `pt`). Use `all` for multi-language sources.

**Gradle module names**: `:src:<lang>:<source>`, `:lib:<name>`, `:lib-multisrc:<theme>`.

## Extension Pattern (Kotlin)

Each extension in `src/<lang>/<source>/`:

- **`build.gradle`** — declares `extName`, `extClass` (relative path to Source class), `extVersionCode`, optionally `isNsfw`. Applies `kei.plugins.extension.legacy`.
- **Package**: `eu.kanade.tachiyomi.extension.<lang>.<source>`
- **Class**: implements `HttpSource` or `SourceFactory`
- **Key methods**: `fetchPopularManga`, `fetchSearchManga`, `getMangaDetails`, `getChapterList`, `getPageList`

### Multisrc Pattern

Themes in `lib-multisrc/` generate multiple extensions from one codebase:
- Uses `kei.plugins.multisrc` plugin
- Sets `baseVersionCode` in build.gradle.kts
- Dependencies use `api(project(":lib:..."))` (not `implementation`)
- Theme class lives in `eu.kanade.tachiyomi.extension.<theme>.<ThemeName>` package

### Lib Pattern

Reusable modules in `lib/`:
- Uses `kei.plugins.library` plugin
- Code in `keiyoushi.lib.<name>` package
- Extensions depend via `implementation(project(":lib:<name>"))`

## Code Style

- **Formatter**: ktlint via Spotless (`spotlessApply` / `spotlessCheck`)
- **Kotlin**: 4-space indent, 140 char max line, trailing commas, no star imports
- **No explicit `joinToString(", ")`** — default separator is already `", "`
- **Use `class` not `data class`** for DTOs (reduces bytecode)
- **Use `parseAs<T>()`** from `keiyoushi.utils` — not manual `json.decodeFromString`
- **Cache `Regex`, `SimpleDateFormat`** at class/file level — never in hot paths
- **Use `response.asJsoup()`** — not `Jsoup.parse(response.body.string())`
- **Use `element.absUrl("href")`** — not manual URL concatenation

## source-config-templates → Kuron Config System

This repo contains JSON config templates that map to **Kuron's config-driven source system** (not compiled Kotlin). The flow:

1. **Choose template**: `source-config-rest-template.json` (API) or `source-config-scraper-template.json` (HTML scraping)
2. **Installable source** (appears in Source Manager): add entry to `manifest-installable-entry-template.json`, upload config to `app/config/<source>-config.json` in Kuron repo
3. **Bundled/fallback source** (in APK): save config to `assets/configs/<source>-config.json`, register in `RemoteConfigService._bundledAssetPaths`

**Key config fields**:
- `source` — unique ID, must be consistent across app
- `baseUrl` — root URL
- `network.headers` — request headers (Referer must have trailing slash)
- `api.endpoints` / `scraper.urlPatterns` — endpoint definitions
- `list.fields` / `detail.fields` — JSONPath (REST) or CSS selectors (scraper)
- `searchForm` — UI form params (`query`, `select`, `tag`, `page` types)
- `features` — toggle `chapters`, `favorite`, `download`, `comments`, `related`

**Two search modes**: `searchConfig` (legacy query-string) or `searchForm` (modern dynamic form). Scraper adapter supports `raw:` prefix for raw queries.

**Output rule**: Always write generated config JSON files to `output-json-config/` at the repo root. Create the folder if it doesn't exist. Name files as `<source-id>-config.json`.

**Verify against real website**: Always inspect the actual website's HTML before writing scraper configs. Even sites that share a theme or template (e.g., Madara, Mangareader) often have custom class names, IDs, or structure variations. Use a browser to check real selectors — do not guess or blindly copy from existing configs.

## CI & Testing

- **PR check**: builds extensions in chunks of 80 modules, then runs [extensions-inspector](https://github.com/keiyoushi/extensions-inspector) for APK validation
- **No unit test framework** — extensions are tested manually or via the inspector
- **Pre-commit**: end-of-file-fixer + trailing-whitespace hooks

## Sparse Checkout

This repo is large. Use partial clone for development:
```bash
git clone --filter=blob:none --sparse <url>
cd tachiyomi-extensions/
git sparse-checkout set --cone --sparse-index
git sparse-checkout add common core gradle lib lib-multisrc utils
git sparse-checkout add src/<lang>/<source>
```

## Key References

- **CONTRIBUTING.md** — exhaustive extension development guide (patterns, utilities, networking, DTOs)
- **gradle/libs.versions.toml** — available dependency versions
- **gradle/kei.versions.toml** — custom plugin IDs
- **source-config-templates/README.md** — Kuron config workflow (Indonesian)
