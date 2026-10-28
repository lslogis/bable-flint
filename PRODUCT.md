# Product

<!-- impeccable:product-schema 1 -->

## Platform

web

(The shipped app is native macOS, SwiftUI, in `app/`. Design work is first drawn as an HTML prototype, then ported to SwiftUI. None of the schema's platform values fits a Mac app, so `web` here covers the prototype only.)

## Users

- Decided by the user (2026-10-28): Flint is also distributed to other people, not only used on the author's own Mac. First-time users need enough explanation to install and connect without help.
- Who they are (inferred from the repo, not confirmed): Apple Silicon Mac owners (M4 Max class, 48 GB or more) who run a local model with rec and want their coding apps (Claude Code, Codex, opencode, the Claude app, the ChatGPT app's Codex) to use it.

## Product Purpose

bable-flint.app (the Flint app) runs the local Flint model server (rec 0.1.1 with a tuned overlay), shows how fast and how heavy it is, and connects or disconnects coding apps to it. Every connection can be undone back to the app's original settings.

## Operating Context

- Decided by the user (2026-10-28): both ways of use matter. The window stays open beside other work to glance at status and speed, and it is also opened to change options, connect apps, or manage the install.
- The server keeps running when the window closes. Quitting Flint asks before undoing connections.
- The UI is Korean.

## Capabilities and Constraints

- Three areas today: 서버 (power, options, speed, memory, conversation cache), 앱 연결 (address, key, per-app connect and restore), 관리 (rec doctor, reinstall or remove the tuned runtime, undo all connections, logs).
- Requests are logged as numbers only, never conversation text. The API key is never shown unless the user asks.
- Destructive actions move things to the Trash, never delete.
- The window has to work small (a glance panel) and large (settings). The user asked for in-app zoom (⌘+ / ⌘− / ⌘0) and for no visible scrollbars (2026-10-28).
- Decided by the user (2026-10-28): zoom scales the whole window, not only what is inside it ("창크기를 확대해야지 확대축소를 앱내부에있는것만 하면 어떻하냐").

## Brand Commitments

- Name: Flint. Model name shown to apps: "Flint (local)".
- The user first picked two themes from their own style reference (2026-10-28): 01 OBSIDIAN GOLD (charcoal, champagne gold) and 02 IVORY INK (ivory, deep green).
- Decided by the user (2026-10-28), replacing the above: 05 PEARL INDIGO (pearl white, indigo) in the layered material ("05로 가보자"), light only ("다크모드 필요없어"). Values are in DESIGN work, not here.

## Evidence on Hand

- Measured speeds in README.md (decode A/B, worker results on M4 Max 40-core). No user testimonials, download counts or benchmarks beyond those; do not invent any.

## Product Principles

- State first: whether the server is up, how fast it is, and what is connected must be readable at a glance.
- Every change is reversible and says how to undo it.
- Plain Korean that says what happens, including what Flint will not touch.
