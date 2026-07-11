# ArcWatch

A native macOS desktop app that tracks live world-event rotations for the game **ARC Raiders** — showing which timed events are active on each map, how long they have left, and what's rotating in next.

Built with **Rust + Tauri**. This started life as a single-file Python terminal app (a Rich TUI) and is being rebuilt as a real windowed desktop app — partly as a genuinely useful tool, partly as a Rust learning project.

> 🚧 **Status: in progress.** The Tauri shell is scaffolded and running. Next up: porting the event-fetching and scheduling logic into Rust. See the [Roadmap](#roadmap).

<!-- TODO: add a screenshot / GIF here once the UI is built — this is the first thing a reader looks at.
     Record with Kap (https://getkap.co) and drop it in assets/.
![ArcWatch screenshot](assets/screenshot.png)
-->

## Why this exists

ARC Raiders runs timed world events that rotate per map (up to two active at once per map). Knowing what's live and what's coming up is genuinely useful in-game, but checking a website mid-session is clumsy. ArcWatch is a small always-available desktop dashboard for it.

I also wanted a focused project to **learn Rust**, so the design deliberately puts the real work in Rust rather than taking the easy all-JavaScript route (see [Architecture](#architecture)).

## Tech stack

- **[Rust](https://www.rust-lang.org/)** — application logic (data fetching, scheduling, time math) and the native shell.
- **[Tauri 2](https://v2.tauri.app/)** — desktop framework. Unlike Electron, Tauri uses the OS's built-in webview instead of bundling Chromium, so the app is a few MB instead of ~150 MB.
- **TypeScript** — the view layer (DOM rendering + per-second countdown ticks).
- **WKWebView** — the macOS system webview that renders the UI.

Data comes from the community [metaforge.app](https://metaforge.app) ARC Raiders events API.

## Architecture

The logic lives in **Rust**, exposed to the UI as Tauri commands. The webview is a thin view layer that calls into Rust and renders the result.

```
┌───────────────────────────────────────────────┐
│  WebView (WKWebView) — the VIEW                │
│  • invoke("get_schedule")                      │
│  • render table + panels as DOM/CSS            │
│  • setInterval(render, 1000) for countdowns    │
└───────────────▲────────────────────────────────┘
                │ invoke(...)  ⇅  returns JSON
┌───────────────┴────────────────────────────────┐
│  RUST core — the LOGIC + native shell          │
│  • fetch events (reqwest)                       │
│  • group by map, pick active / upcoming         │
│  • compute countdowns + "starting soon" list    │
│  • window / tray / always-on-top                │
└─────────────────────────────────────────────────┘
```

Fetching in Rust (rather than the browser's `fetch`) also avoids CORS restrictions on the upstream API.

## Getting started

**Prerequisites:** [Node.js](https://nodejs.org/) and the [Rust toolchain](https://www.rust-lang.org/tools/install).

```bash
# install frontend deps
npm install

# run the app in dev mode (hot-reloads the UI, rebuilds Rust on change)
npm run tauri dev

# produce a distributable .app + .dmg (in src-tauri/target/release/bundle/)
npm run tauri build
```

> **Note on unsigned builds:** a locally built `.app` is unsigned, so macOS Gatekeeper will warn on first launch. Right-click → Open, or run `xattr -dr com.apple.quarantine ArcWatch.app`.

## Roadmap

- [x] Scaffold Tauri + TypeScript shell
- [ ] Rust `get_schedule` command: fetch the events API via `reqwest`
- [ ] Rust: group by map, select active (≤2) and upcoming events, compute "starting soon (≤15m)" list
- [ ] Web UI: per-map table with progress bars and live countdowns
- [ ] "Starting Soon" panel with highlight
- [ ] Widget mode (always-on-top, borderless window)
- [ ] Menu-bar tray icon + launch-at-login
- [ ] App icon + packaged release

## Credits

Event data from [metaforge.app](https://metaforge.app). Not affiliated with or endorsed by Embark Studios / ARC Raiders.

## License

[MIT](LICENSE) © Curtis Ovard
