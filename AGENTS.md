# AGENTS.md

Mattermost plugin (from `mattermost-plugin-starter-template`) that restyles the
chat UI into Telegram-style bubbles. It is **webapp-only** — there is no server
code, and the `go.mod` / `plugin.go` at the repo root are vestigial template
leftovers that `make` never builds.

## Build & verify

- Build bundle: `make dist` → `dist/ir.quera.mattermost-chat-ui-<version>.tar.gz`
  (version comes from `plugin.json`). `make clean` removes all artifacts.
- Style check: `make check-style` runs eslint only. The `golangci-lint`,
  `check-types`, and `test` Makefile branches are skipped: `plugin.json` has no
  `server` field, and `webapp/package.json` defines `check-types` and `test` as
  empty strings. **There are no tests.** Don't invent test/lint commands that
  don't exist.
- Release version lives in `plugin.json`; `webapp/package.json` mirrors it —
  bump both together.

## Committing

- Commit changes after making them (unless the user says otherwise). Keep
  commits focused on the task at hand.

## Go tooling is a trap

- The root `go.mod` declares the plugin module name
  (`github.com/fmotalleb/mattermost-chat-ui`, go 1.26).
- `go build ./...`, `go test ./...`, or `golangci-lint run ./...` from the repo
  root will walk into `webapp/node_modules/flatted/golang/...` and fail with
  "predeclared any requires go1.18". Do not run broad Go commands at root.
- The only real Go is under `build/` (its own `go.mod`, the starter-template
  tooling). Don't edit it except when syncing with the upstream template
  (`make sync`).

## Architecture

- `webapp/src/index.js` is the entrypoint: registers the plugin and polls every
  5s, computing `<body>` background luma and toggling the class
  `mm-chat-ui--light-theme` / `mm-chat-ui--dark-theme`.
- All styling lives in `webapp/src/style.scss` (Telegram-inspired light/dark
  palettes). Webpack injects it inline into `webapp/dist/main.js` via
  `style-loader` — there is no separate CSS file.
- Rules target Mattermost's internal DOM classes (`.post`, `.post__body`,
  `.col.col__controls`, ...), which change between Mattermost releases.
  Test styling against a real server; upstream UI breakage has caused fixes in
  past commits.
- `webapp/src/manifest.js` just re-exports `plugin.json`; keep it unchanged.
