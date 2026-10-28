# bable-flint

[한국어](README.md)

A Mac app that runs the local coding model **Flint** fast and connects the coding apps you already use to it.
It installs, starts and stops a tuned server on top of [rec](https://github.com/recursive-oss/rec) 0.1.1. It connects Claude Code, Codex CLI, Codex in the ChatGPT app, the Claude app and opencode with one button each, and disconnecting puts them back the way they were.

Measured: generation is faster than rec's original runtime, 107.2 → 129.7 tok/s (+21%) at 32K context and 74.0 → 91.5 tok/s (+24%) at 64K.
Measured on: M4 Max (40-core GPU), 48 GB memory, macOS 27.2, rec 0.1.1, flint-int4.

## Requirements

- An Apple Silicon Mac with an **M4 Pro, M4 Max, M5 Pro or M5 Max**
- **48 GB** of memory or more
- **32 GB** free disk space or more (21 GB model + runtime + conversation cache). 10 GB if the model is already downloaded.
- macOS 15 or later
- [rec](https://github.com/recursive-oss/rec) **0.1.1**, installed first. Any other version stops the install.
- `python3` (included with the Xcode Command Line Tools)
- Only for Codex in the ChatGPT app: Python 3.14 or later (for example Homebrew's `python3`)

If your Mac falls short, the app shows what is missing and downloads or changes nothing.

## Install

There is no prebuilt app download. Build the app from this repository. You need Swift 6 (Xcode 16 or later).

```sh
cd app
./build.sh
open build/bable-flint.app
```

You can move `build/bable-flint.app` to your Applications folder.

## First run

The app walks you through it:

1. It checks that this Mac meets the requirements.
2. It checks that rec is installed.
3. **Download** fetches the Flint model (about 21 GB).
4. **Install** sets up the tuned runtime. rec's original runtime stays as it is.

When it is done, press **Start server**.

## Everyday use

- Left panel: server state, speed of the last request (tok/s), memory, conversation cache and connected apps, with **Start server**, **Restart** and **Stop server** below.
- Tabs on the right: **Options**, **Apps**, **Manage**.
- Top right: the **한/EN** language switch and zoom (−, 100%, +, or ⌘+ ⌘− ⌘0).
- Closing the window keeps the server running. The Dock icon hides and the flame icon stays in the menu bar.
- Menu bar icon: state and speed, start, stop and restart the server, open the window, copy the address, quit.
- Quitting while the server runs asks: **Quit, Keep Server On**, **Quit and Stop Server** or **Cancel**.
- The server only accepts requests from this Mac (127.0.0.1).

## Options

These are the values the server starts with. Changes take effect when the server restarts.

- **Mode**: Fast (the tuned runtime) or Standard (rec's original runtime, about 20% slower)
- **Context**, **output limit**, **sampling**
- **Workers**: generation workers, helper workers, read chunk
- **SSD conversation cache**: folder and maximum size. When you continue a conversation, the server does not read the earlier part again, even after a restart.
- **Reset to defaults** resets the values to where they started.

### Auto-tune

**Auto-tune** restarts the server for each candidate value, measures the speed, and proposes the fastest workers and read chunk for this Mac.

- It takes 5 to 20 minutes. The server is unavailable meanwhile.
- It only proposes a change that is more than 3% faster than what you have now. Values that change the answer to the same question are dropped.
- You can apply the result right away or only put it into Options. Stopping it restarts the server with the previous values.

## Connecting apps

In the **Apps** tab, each app shows what connecting will change. Press connect to go ahead. Disconnecting restores the original settings and moves the files Flint made to the Trash.

- **Claude Code**: your usual `claude` stays as it is; a separate `claude-flint` command is added. Skills, MCP servers and history are kept apart from your usual claude.
- **Codex CLI**: your usual `codex` stays as it is; a separate `codex-flint` command is added. Web search, sub-agents and image input do not work.
- **Codex in the ChatGPT app**: adds **Flint (local)** at the bottom of the model list. It puts one line in `~/.codex/config.toml`, and a relay running on this Mac sends only Flint requests to Flint. Restart the ChatGPT app to see it. While connected, Codex's other requests also pass through this relay.
- **Claude app**: switches the app to its third-party (3P) mode to use **Flint (local)**. Quit the Claude app with ⌘Q before connecting or disconnecting. While connected, your claude.ai sign-in and chats are hidden; they come back when you disconnect.
- **opencode**: adds one provider to its settings. Pick Flint in `/models`, or start it with `opencode -m flint/flint`.

Any other app can be set up by hand. The same tab shows the address (OpenAI-compatible `http://127.0.0.1:11500/v1`, Anthropic-compatible `http://127.0.0.1:11500`), the model name `flint` and the key. You can show or hide the key, make a new one, and test the connection. A new key blocks the old one at once and updates the connected apps.

## Manage

- **Start at login**: starts the server and the menu bar icon when you log in.
- **Check** runs a check of rec and the installed runtime. If something is still wrong, **Reinstall**.
- **Log**: the server log, live, with an errors-only filter.

## Uninstall

Use the remove button in **Manage**. It stops the server, restores every connected app, then moves these to the Trash:

- rec and the model
- the conversation cache
- the tuned runtime
- the app's settings and connection files
- the app itself

Until you empty the Trash, you can put them back.

---

## For developers

### Build and test

```sh
cd app
swift test          # FlintCore tests
./build.sh          # build/bable-flint.app (ad-hoc signed)

cd ..
pnpm test           # = python3 -E -s -m unittest tests.test_chat tests.test_responses tests.test_install tests.test_codex_relay
```

- `tests.test_codex_relay` runs only with Python 3.14 or later on macOS and is skipped elsewhere.
- Tests that need rec's runtime run on the Mac:
  `~/.local/share/rec-tuned/rt/python/bin/python3.12 tests/test_server_defaults.py` (the same for `test_disk_cache_pieces.py`)
- Install and run without the app: `python3 install.py --check` (check only), `python3 install.py`, then `rec stop; rec-serve-tuned`.

### Environment variables

The app passes these when it starts the server. rec's original runtime ignores them.

| Variable | When unset | Meaning |
|---|---|---|
| `REC_DISK_CACHE` | `~/.cache/rec-kv` | Conversation cache folder; `0` turns it off |
| `REC_DISK_CACHE_GB` | 32 | Maximum cache size (GiB); the oldest entries go first |
| `REC_DISK_CACHE_MIN` | 8192 | Prompts shorter than this are not saved |
| `REC_EXP_WORKERS` | chip default (`76,64,64` on M4 Max and M5 Max) | Generation, MLP and head workers |
| `REC_EXP_CHUNK` | chip default | Prompt read chunk |
| `REC_TEMPERATURE`, `REC_TOP_P`, `REC_TOP_K`, `REC_PRESENCE_PENALTY` | rec's defaults (the app passes 1.0, 0.95, 20, 0) | Sampling used when a request leaves it out; a value the request sends wins. Out of range stops the server at start |
| `REC_OUTPUT_LIMIT` | none | Maximum tokens in one answer |
| `REC_METRICS` | none | Appends token counts, time, speed and memory per request to this JSONL file. No conversation text |

### Files

- `app/`: bable-flint.app. `Sources/FlintCore` (server, options, connections and tuning logic, tested in `Tests/`), `Sources/Flint` (the windows), `build.sh`
- `install.py`: copies rec's runtime, lays `overlay/` on top, and creates `~/.local/share/rec-tuned` and `~/.local/bin/rec-serve-tuned`
- `overlay/rec_mlx/`: the changed rec files
- `relay/flint_codex_relay.py`: the relay for Codex in the ChatGPT app
- `tests/`: Python tests

### Files changed from rec

Seven rec 0.1.1 files are changed and two are added (`disk_cache.py`, `responses.py`). Each changed file starts with a `Modified from recursive-oss/rec 0.1.1` line. The list and what changed are in [`NOTICE`](NOTICE).

## License

Apache-2.0 (`LICENSE`, `NOTICE`). rec is Apache-2.0 too. The bundled Pretendard font is under the SIL Open Font License 1.1.
