# Repository Overview

This document gives a short tour of the cosmic-text repository: what each
top-level folder contains, and how to build and run the project.

## `src/`

The library itself. `src/lib.rs` is the crate root and documents the typical
usage flow: create a `FontSystem`, build a `Buffer` with some `Metrics`, set
text and attributes on it, then inspect the resulting layout runs or draw them
with a `SwashCache`. The rest of the module is organized by concern:

- `attrs.rs` — text attributes such as color, font family, weight, style, and
  stretch, plus the `AttrsList` that maps spans of text to attributes.
- `buffer.rs` / `buffer_line.rs` — the `Buffer` type (a collection of lines)
  and `BufferLine` (a single shaped/laid-out line), including caching of
  shape and layout results per line.
- `bidi_para.rs` — a paragraph iterator that follows `unicode-bidi` paragraph
  boundary rules.
- `cached.rs` — a small generic cache helper (`Cached<T>`) used to track
  whether a derived value such as a shape or layout is stale.
- `cursor.rs` — the `Cursor` type describing a caret position (line, byte
  index, and run affinity).
- `glyph_cache.rs` — cache keys and flags (e.g. fake italic, disable hinting,
  pixel font) used to identify rasterized glyphs.
- `layout.rs` — the data structures produced by layout: laid-out glyphs and
  lines with their positions.
- `line_ending.rs` — the `LineEnding` enum (LF, CRLF, CR, LFCR, none) used
  when splitting and rejoining text.
- `math.rs` — small floating point helpers (`floorf`, `roundf`, `truncf`)
  with `std` and `no_std`/`libm` implementations.
- `render.rs` — the `Renderer` trait used to draw rectangles and glyphs when
  rendering a buffer or editor.
- `shape.rs` — text shaping: turning a line of text plus attributes into
  shaped glyph runs, including font fallback and line wrapping logic.
- `shape_run_cache.rs` — caching of shaping results keyed by text and
  attributes, to avoid re-shaping unchanged runs.
- `swash.rs` — the optional `SwashCache`, which rasterizes glyphs into images
  or pixels using the `swash` crate.
- `font/` — font discovery and matching. `font/mod.rs` and `font/system.rs`
  implement `FontSystem` and font matching against `fontdb`; `font/fallback/`
  contains per-platform (macOS, Windows, Unix, other) fallback font lists.
- `edit/` — text editing on top of a `Buffer`. `edit/mod.rs` defines the
  `Edit` trait and editing `Action`s; `edit/editor.rs` implements the default
  `Editor`; `edit/vi.rs` (feature `vi`) adds Vi-style modal editing via
  `modit`; `edit/syntect.rs` (feature `syntect`) adds syntax highlighting via
  `syntect`.

## `examples/`

Runnable example applications, each its own Cargo package and workspace
member:

- `editor/` — a windowed (`winit`) plain-text editor example.
- `editor-test/` — a non-interactive, automated test harness that simulates
  typing to validate editing/layout correctness (used by `test.sh`).
- `multiview/` — a windowed example showing multiple text views/buffers at
  once.
- `rich-text/` — a windowed example demonstrating rich text attributes
  (color, weight, style) within a single buffer.
- `terminal/` — a simple command-line example that shapes and prints text
  without opening a window.

## `tests/`

Integration tests for the library, covering editor state after modifications,
ellipsis rendering, rich-text layout, shaping/rendering, text decorations,
variable font weights, and line-wrap stability/fallback behavior. The
`tests/common/` folder holds shared test helpers, and `tests/images/` holds
reference images used by rendering tests.

## `benches/`

Criterion benchmarks: `layout.rs` benchmarks the layout pass, and
`text_shaping_benchmarks.rs` benchmarks text shaping.

## `sample/`

Plain-text sample files in various scripts and configurations (Arabic,
Bengali, Hebrew, Thai, Han, emoji, ligatures, tabs, mixed line endings, etc.)
used to exercise shaping, fallback, and layout during manual testing and
examples.

## `fonts/`

Bundled font files (Fira Mono, Inter, Inter Variable, Noto Sans, Noto Sans
Arabic, Noto Sans Hebrew) and their licenses, used by the examples and tests.

## `screenshots/`

Screenshot images referenced from README.md showing the library rendering
right-to-left and complex-script text.

## Top-level files

- `Cargo.toml` — the library's package manifest, including the optional
  `swash`, `syntect`, `vi`, `no_std`, and `std` feature flags described
  below, and the `[workspace]` that includes the `examples/*` packages.
- `CHANGELOG.md` — release history.
- `LICENSE-APACHE` / `LICENSE-MIT` — dual license texts.
- `deny.toml` — configuration for `cargo deny` dependency checks.
- `ci.sh` — the CI build/test matrix: checks formatting, builds and lints
  with various feature combinations (default, `no_std`, `std`, `std+swash`,
  `std+syntect`, `std+vi`, all features), and runs `cargo test --all-features`.
- `test.sh` — runs `ci.sh`, builds docs and all examples in release mode, then
  runs the `terminal` and `editor-test` example binaries.
- `editor.sh`, `editor-test.sh`, `multiview.sh`, `rich-text.sh`,
  `terminal.sh` — convenience scripts that `cargo run --release` the
  corresponding example with useful `RUST_LOG` settings.
- `redoxer.sh` — builds and runs an editor example under Redox OS via
  `redoxer`.

## Building and running

The project is a standard Cargo workspace; no other toolchain is required.

Build the library and all examples:

```sh
cargo build --release --all
```

Run the library's tests (the default feature set omits some optional
dependencies, so most development is done with `--all-features`):

```sh
cargo test --all-features
```

Run an individual example directly:

```sh
cargo run --release --package terminal
cargo run --release --package editor -- path/to/file.txt
```

or use the matching convenience script, e.g. `./terminal.sh` or
`./editor.sh path/to/file.txt`, which also set a useful `RUST_LOG` value.

To reproduce the full CI matrix locally (formatting check, feature-combination
builds and lints, and tests), run:

```sh
./ci.sh
```

`./test.sh` runs `ci.sh` plus a full documentation build, builds all examples,
and runs the `terminal` and `editor-test` binaries — this is closest to what
CI runs end-to-end.
