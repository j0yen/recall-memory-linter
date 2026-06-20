# recall-memory-linter

A deterministic linter that walks a Markdown memory store and reports the three ways it rots: stale entries, duplicates, and broken wiki-links.

## Why it exists

A flat pile of `MEMORY.md`-style notes degrades silently. Nothing tells you an entry is months stale, that two notes say the same thing, or that a `[[link]]` points at a file that no longer exists. The damage is invisible until a future session reads the corpus and is misled by it.

The fix that pays off first isn't smarter retrieval — it's a clean foundation to retrieve against. Before embeddings or ranking earn their keep, the on-disk store has to be healthy, and "healthy" has to be something you can check on every run rather than trust. `recall-lint` is that check: it audits the corpus and reports what's wrong. It does not retrieve, rank, or rewrite anything — it only tells you the truth about the files.

## Install

```sh
git clone --depth 1 https://github.com/j0yen/recall-memory-linter.git
cd recall-memory-linter
./install.sh
```

`install.sh` runs `cargo install --path . --locked` and drops the `recall-lint` binary in `~/.cargo/bin/`. Requires `cargo` / `rustc` 1.85+ and `git`. To build without installing:

```sh
cargo build --release   # → target/release/recall-lint
```

## Quickstart

Point it at a memory root. An empty or clean store exits `0`:

```sh
$ recall-lint ./memory
$ echo $?
0
```

A store with problems reports them and exits `1`:

```sh
$ recall-lint ./memory
broken_link: ./memory/plan.md:12 → [[archived-spec]]
duplicate: ./memory/a.md, ./memory/copy-of-a.md
stale: ./memory/old-note.md (last_seen=2026-04-01, age=79d)
```

For tooling, ask for JSON — a stable schema with a top-level `findings` array, sorted deterministically so runs on identical inputs diff cleanly:

```sh
$ recall-lint ./memory --format json
{"findings":[{"kind":"broken_link","paths":["./memory/plan.md"],"target":"archived-spec","line":12}, ...]}
```

Options:

| Flag | Default | Effect |
| --- | --- | --- |
| `--format <text\|json>` | `text` | human-readable, grouped by kind, or stable JSON |
| `--stale-days <N>` | `30` | flag entries whose newest timestamp is older than `N` days |
| `--ignore <glob>` | — | root-relative glob to skip; repeatable |

## What it checks

Every `*.md` file under the root is one memory entry (non-Markdown files and dot-directories are skipped). Three findings:

- **stale** — the newest of `last_recalled_at`, `created_at` (RFC3339, from optional YAML frontmatter), or file mtime is older than `--stale-days`.
- **duplicate** — two or more entries share a byte-identical body after frontmatter is stripped and trimmed; one finding lists all paths.
- **broken_link** — a `[[target]]` that no `*.md` file resolves, reported with source path and 1-indexed line.

Exit codes are part of the contract: `0` clean, `1` findings present, `2` invocation error (bad path, unreadable file, malformed args).

## Where it fits

Part of the **recall** family — the agentic-memory stack. `recall-lint` is the hygiene gate; the other pieces handle storage, retrieval, and operations on top of a corpus it has vouched for.

## Status

V1, and deliberately narrow: it audits the on-disk corpus and nothing more. Retrieval, ranking, and write-back are later phases that stand on this clean foundation. Each acceptance criterion has a matching test under `tests/acceptance_ac<n>.rs`.

## Provenance

Scaffolded from a PRD through the [`autobuilder`](https://github.com/j0yen/autobuilder) pipeline. Originally a subdirectory of the [`wintermute`](https://github.com/j0yen/wintermute) monorepo; this is a standalone snapshot for easier consumption.

## License

Licensed under either of Apache-2.0 ([LICENSE-APACHE](LICENSE-APACHE)) or MIT ([LICENSE-MIT](LICENSE-MIT)), at your option.
