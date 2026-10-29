# Leptos Log Lens

A client-side analyzer for Nginx access logs. Drop a log file onto the page and get request statistics, an hourly histogram, the busiest endpoints and a sortable, filterable request table. Everything runs in the browser with Rust and WebAssembly: the file is never uploaded anywhere.

The example is small enough to read in one sitting but deals with a realistic amount of data: a 500,000-line (~70 MB) log loads and filters smoothly.

## What it shows

- **Derived state with `Memo`.** One `RwSignal` holds the parsed file; filtered entry indices, summary statistics, the histogram and the top endpoints are `Memo`s derived from it and from the filter signals. Changing a filter only recomputes what depends on it.
- **Sharing state with context.** Filters and the selected language are provided once (`provide_context`) and used anywhere (`expect_context`), without prop drilling.
- **Two-way binding** of form fields with `bind:value`.
- **Keyed lists with `<For>`.** Row keys include a dataset id, so loading a new file never reuses rows of the previous one.
- **Async work without freezing the UI.** The file is read in slices with `Blob.slice().arrayBuffer()` (`web-sys`, `wasm-bindgen-futures`); each slice is parsed while the next one is being read, with regular yields to the browser so the progress bar keeps moving.
- **Internationalization without a library.** English, German and Turkish texts and number formats in [`src/i18n.rs`](src/i18n.rs). A missing translation is a compile error; switching languages only updates text nodes.
- **Keeping WASM fast.** A single-pass byte-level parser (with [`memchr`](https://crates.io/crates/memchr)), interned strings so that entries are small and allocation-free, and display-only fields re-read from the file text for the visible rows only.
- **Native tests for browser code.** The parser, statistics and filters do not depend on the DOM, so `cargo test` runs them natively.

## Getting Started

See the [Examples README](../README.md) for setup and run instructions.

## Quick Start

Run `trunk serve --open` to run this example, then drop [`samples/access.log`](samples/access.log) onto the page.

Parsing is much slower in debug builds; use `trunk serve --release --open` when trying your own large log files.

## Log Format

Lines are expected in Nginx's `combined` format, optionally followed by the request time (`$request_time`):

```nginx
log_format timed '$remote_addr - $remote_user [$time_local] "$request" '
                 '$status $body_bytes_sent "$http_referer" "$http_user_agent" $request_time';
```

Lines that cannot be parsed are counted and skipped.

## Project Layout

| File | Purpose |
|---|---|
| `src/app.rs` | Root component: state, derived `Memo`s, page layout |
| `src/loader.rs` | Reads a `File` in slices and parses it while reading |
| `src/parser.rs` | Byte-level line parser, interning tables |
| `src/stats.rs` | Summary, hourly histogram, top endpoints |
| `src/filter.rs` | Filter criteria and the reactive filter state |
| `src/i18n.rs` | Languages, translations, number formats |
| `src/perf.rs` | `performance.mark`/`measure` helpers (visible in the browser profiler) |
| `src/components/` | File drop zone, filter bar, stat cards, charts, request table |

## Testing

```sh
cargo test
```
