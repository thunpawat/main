# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

This is `github/docs`, the source for docs.github.com. It contains two very different kinds of things that need different rules:

1. **Application code** (`src/`) — a Node.js/TypeScript Express + Next.js server that renders the docs site.
2. **Content** (`content/`, `data/`) — the actual English-language Markdown documentation, plus YAML data files (variables, reusables, feature versioning).

`docs` (this repo, public) and `docs-internal` (private) are synced frequently; content changes flow between them. The public repo only accepts contributions to `.md` files in `content/` and select `data/` sections (e.g. reusables) — infrastructure, workflows, and site-building code are not open to external modification here.

Dual license: CC-BY-4.0 for `assets/`, `content/`, `data/`; MIT for code (see `LICENSE` / `LICENSE-CODE`).

## Commands

```shell
npm ci                 # install deps (do this once per pulled branch)
npm run build           # build Next.js app — required before most tests, and after content/code changes
npm start / npm run dev # run the server at http://localhost:4000 (dev uses nodemon + ENABLED_LANGUAGES=en)
npm run tsc              # typecheck (no emit)
npm run lint              # eslint '**/*.{ts,tsx}' (add -- --fix to autofix)
npm run prettier            # format; npm run prettier-check to check only
npm run lint-content -- --paths <file-paths>  # lint Markdown content
```

### Tests (vitest)

**Do not run `npm test` with no path** — the full suite needs per-suite env vars and will produce false failures. **Run `npm run build` first** — many suites depend on Next.js build artifacts.

```shell
npm test -- src/<suite-name>/tests/          # e.g. src/search/tests/, src/versions/tests/
npm test -- src/search/tests/ai-search-proxy.ts   # single file
npm test -- --silent=false                    # include console.log output
```

Suites needing special env vars have dedicated scripts instead of raw `npm test`:

```shell
npm run test:article-api
npm run test:fixtures
npm run test:landings
npm run test:languages   # requires Elasticsearch running
npm run test:search      # requires Elasticsearch running
```

Content linter tests can scope to changed files only:

```shell
DIFF_FILES="content/foo.md content/bar.md" npm test -- src/content-linter/tests/
```

Playwright (rendering/e2e — build first, changes outside the test aren't picked up otherwise):

```shell
npm run build && npm run playwright-test -- playwright-rendering
npm run playwright-test -- playwright-rendering --ui   # keeps localhost:4000 open for debugging
```

Not every `src/foo/tests/*.ts` runs in CI automatically — suites must be manually added to `.github/workflows/test.yml`'s matrix, and to branch-protection required checks once merged.

## Architecture

### `src/` — "subject folder" pattern

Code is organized by subject/capability, not by role (no global `lib/`, `components/`, `middleware/` at top level). Each subject folder (e.g. `src/search/`, `src/versions/`, `src/redirects/`) is self-contained with its own `README.md`, and typically its own `lib/`, `middleware/`, `pages/`, `components/`, `tests/`. Use the *most specific* subject folder for new code; only put things in `src/frame/` (cross-cutting server/rendering spine) or `src/workflows/` (process scripts, not the running app) when there's truly no better home. Check a subject's `README.md` before working in it — several call out cross-subject dependencies explicitly.

Key subjects: `frame` (Express app bootstrap, middleware pipeline, page rendering, shared layout components), `content-render` (Markdown/Liquid → HTML pipeline), `content-linter` (custom markdownlint rules for `content/`), `search`, `versions` (GHES/GHEC version logic), `redirects`, `languages` (translations), `rest`/`graphql`/`webhooks` (generated API reference docs, synced from other GitHub repos), `article-api`, `data-directory`, `early-access`, `frontend` bits under `frame/components`.

`src/frame/lib/app.ts` (`createApp()`) builds the Express app; `src/frame/middleware/index.ts` orchestrates the full middleware pipeline (context building → page lookup → rendering); `src/frame/lib/page.ts`'s `Page` class represents a content page; `src/frame/lib/frontmatter.ts` is the AJV schema that validates every page's frontmatter.

Imports use the `@/` alias for absolute imports rooted at `src/` (e.g. `import getRedirect from '@/redirects/lib/get-redirect'`) — configured via `tsconfig.json` `paths`. All new code is TypeScript, not JavaScript.

Logging: use `createLogger` from `@/observability/logger` (module-scope instance, structured second-arg context) instead of `console.log` in server code; `console.log` is fine in one-off scripts.

### `content/` — the Markdown docs

Every article is Markdown with required YAML frontmatter (`title`, `versions`, plus optional `intro`, `permissions`, `layout`, `children` on index pages, etc. — full reference in `content/README.md`). Filenames are kebab-case derived from `title`; a test enforces this unless `allowTitleToDifferFromFilename: true` is set.

- **Versioning**: `versions` frontmatter controls which product/GHES versions a page applies to (`fpt`, `ghes`, etc., see `src/versions/lib/all-versions.ts`). Content can additionally use Liquid conditionals (`{% ifversion ... %}`) inline.
- **`index.md` files** define a product/category/map-topic's nav via required `children` frontmatter — a page not listed in some ancestor's `children` 404s even if the file exists.
- **Liquid templating** (`data` tag, reusables, versioning) is layered on top of Markdown; see `contributing/liquid-helpers.md`.
- Product/feature names must use Liquid variables from `data/variables/*.yml` (e.g. `{% data variables.product.prodname_copilot %}`), never hardcoded text — except in frontmatter `title` and `content/site-policy/`.
- Internal links to other docs pages must use `[AUTOTITLE](/path/to/article)`, never a hardcoded link title and never `{% link %}`.
- Reusable prose blocks live under `data/reusables/<topic>/<name>.md` and are referenced as `{% data reusables.topic.name %}`.
- RAI application/platform cards (`contentType: rai`) have an enforced section structure (linter rule GHD064) and may only reference reusables from `data/reusables/rai/`; template at `content/contributing/writing-for-github-docs/templates.md`.

### `data/`

Parsed and exposed to pages as `site.data`. Subdirectories: `variables` (short reusable strings), `reusables` (long reusable Markdown blocks), `features` (feature-based versioning), `glossaries`, `graphql` (schema synced from `github/github`), `webhooks` (payload JSON), `ui.yml` (localized layout strings). All YAML/Markdown here is translated by default.

## Content style conventions

(Condensed from `.github/instructions/style-guide-summary.instructions.md`; full guide at `content/contributing/style-guide-and-content-model/style-guide.md`.)

- Sentence case headers, starting at `##` (H2); never skip levels; text must exist between a header and its first subheader.
- Bulleted lists use `*`, not `-`. Numbered lists for procedures only.
- Active voice; avoid ambiguous modals ("may", "might", "should") when an action is required; say "people"/"users", not "customers"; avoid jargon like "repo", "PR", "i.e.", "e.g.".
- Parenthetical dashes: em dash, no surrounding spaces (`text—like this—text`), not spaced en/em dashes or hyphens.
- Code blocks: fence with a language tag, ~60 char lines, ALL CAPS placeholders (explained in prose), no `$` prompts, comment out sample output.
- Alerts (`> [!NOTE]` etc.) used sparingly, never consecutive, one per section max.
- Tables: every cell filled (`None`/`Not applicable`, not `N/A`), text left-aligned, icon-only columns centered.
- Word choice table (use → avoid): terminal→shell, sign in→log in, sign up→signup, email→e-mail, press→hit/tap, type (UI)→enter (UI), enter (CLI)→type (CLI), repository→repo, administrator→admin.

## Working conventions

- Never commit directly to `main`; work on a branch and open a draft PR.
- Avoid PRs over ~300 lines changed; offer to split larger changes.
- No git force-push, no git rebase (per repo instructions).
- Windows contributors: use `\r?\n` in regexes (not bare `\n`), prefer `path.posix`/the `slash` package over raw `path.join` for URL construction, and prefer Node scripts over Bash where possible.
- Node version is pinned via `.nvmrc`/`engines` in `package.json` (`^24 || ^26`).
