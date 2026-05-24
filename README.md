# recall-memory-linter

> Today's flat MEMORY.md store has no decay signal, no duplicate detection, and no link integrity check, so stale or contradictory notes silently mislead future sessions.

## Why

Today's flat MEMORY.md store has no decay signal, no duplicate detection, and no link integrity check, so stale or contradictory notes silently mislead future sessions. Before building retrieval, embeddings, or push-mode, prove the file layout is healthy: a deterministic linter that walks the Markdown memory store and reports stale, duplicate, and broken-link entries gives the user (and Claude) a trusted hygiene baseline to iterate retrieval against. This V1 slice does not retrieve, rank, or write memories; it only audits the on-disk corpus so later phases stand on a clean foundation.

## Build

```sh
cargo build --release
```

Produces `target/release/recall-lint`. Symlink into `~/.local/bin/` if you want it on `$PATH`.

## Usage

```sh
recall-lint --help
```

## Audience

Single user (the author) on one laptop running Claude Code, plus the Claude sessions that read/write ~/.claude/recall/. Linter is invoked manually from a shell or by a scheduled hook; output is consumed by humans reading the terminal and by tooling parsing JSON.

## Acceptance criteria

This project was scaffolded from a PRD via the `autobuilder` pipeline. The MUST-level acceptance criteria are:

- **AC1**: CLI binary `recall-lint` accepts a positional path to a memory root directory and exits 0 when the directory is empty, emitting an empty findings array in JSON mode.
- **AC2**: Walks the root recursively, treats every *.md file as a memory entry, and ignores non-Markdown files and dot-prefixed directories.
- **AC3**: Parses YAML frontmatter (optional, fenced by ---). Recognizes `last_recalled_at` and `created_at` as RFC3339 timestamps; flags entries whose newest of (last_recalled_at, created_at, file mtime) is older than --stale-days (default 30) as ...
- **AC4**: Detects duplicates: two entries with byte-identical body (post-frontmatter, trimmed) produce one finding kind=`duplicate` listing all paths in deterministic sorted order.
- **AC5**: Detects broken `[[wiki-links]]`: a link `[[target]]` resolves if any *.md file under the root has stem `target` or relative path `target.md`; otherwise emits finding kind=`broken_link` with source path, target, and 1-indexed line number.
- **AC6**: Output mode is selectable: `--format text` (default, human-readable, grouped by kind) or `--format json` (stable schema with top-level `findings` array; each finding has `kind`, `paths`, and kind-specific fields). JSON output is determin...
- **AC7**: Exit code reflects findings: 0 = no findings, 1 = at least one finding, 2 = invocation error (bad path, unreadable file, malformed args). Exit codes are stable across runs on identical inputs.

Each AC has a matching integration test under `tests/acceptance_ac<n>.rs`.

## Provenance

Built via the [`autobuilder`](https://github.com/j0yen/autobuilder) pipeline (PRD intake -> intent-card -> scaffold -> iterate-and-prove). Originally consolidated as a subdir of the [`wintermute`](https://github.com/j0yen/wintermute) monorepo; this standalone repo is a fresh-init snapshot for easier consumption and distribution.

## License

Licensed under either of:

- Apache License, Version 2.0 ([LICENSE-APACHE](LICENSE-APACHE))
- MIT license ([LICENSE-MIT](LICENSE-MIT))

at your option.
