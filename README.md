# hub

**English** | [简体中文](./README.zh-CN.md)

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](./LICENSE)
![Shell](https://img.shields.io/badge/bash-%2B%20tmux-green.svg)

> A tiny, dependency‑free `tmux` CLI to run a **crew of CLI coding agents** (e.g. [Claude Code](https://claude.com/claude-code)) — one agent per project, each in its own session — and let **you _and the agents themselves_** check status and pass messages between sessions.

No desktop app, no MCP server, no daemon — just `tmux` + one shell script. A deliberately lightweight alternative to heavier multi‑agent orchestrators.

## Features

- 🗂️ **One agent per tmux session** named `cc-<name>` — `hub` auto‑discovers them all.
- 👀 **Status at a glance** — `hub ls` shows each agent's git state, remote, and path.
- 💬 **Agent‑to‑agent messaging** — dispatch from one central shell, or let agents message each other directly.
- 🧠 **Structured message framing** — every message is wrapped as an explicit agent‑to‑agent note: concise, machine‑readable, with reply instructions.
- 🪶 **Tiny & portable** — a single Bash file; only needs `tmux` (plus `git` for status).

## Requirements

- `tmux`, `bash`
- `git` *(optional — only for repo status in `hub ls`)*

## Install

```bash
git clone https://github.com/liang-senbei/agent-hub.git
chmod +x agent-hub/hub.sh
ln -s "$(pwd)/agent-hub/hub.sh" ~/.local/bin/hub   # ensure ~/.local/bin is on PATH
```

## Usage

| Command | Description |
|---|---|
| `hub ls` | List every `cc-*` session: git status (dirty / ahead‑behind), remote, path |
| `hub peek <cc> [n]` | Show the last *n* lines of an agent's screen (default `40`) |
| `hub say <cc> "msg"` | Send a message to an agent |
| `hub ask <cc> "msg"` | Same, but ask the recipient to reply via `hub say` |
| `hub all "msg"` | Broadcast to every agent except yourself |

`<cc>` accepts the full session name (`cc-frontend`) or any unique fragment (`front`).

```bash
# from a central shell
hub ls
hub ask backend "Need the schema of the leads table — fields + types."
# the backend agent replies, in its own session, with:
hub say frontend "leads: id, account_email TEXT, accepted INT(0/1), ..."
```

## How it works

### Session model
Each agent runs in its own `tmux` session named `cc-<name>`. `hub` discovers all `cc-*`
sessions and drives them via `tmux send-keys` (push a message) and `capture-pane` (read the screen).

### Agent‑to‑agent messages
Every message is prefixed with a short preamble that:

1. names **sender → recipient**,
2. flags it as **agent‑to‑agent** (be concise & machine‑readable, skip human‑facing prose),
3. for `ask`, gives the recipient the exact command to reply with.

So agents coordinate **directly** — you dispatch from one place (and can target a specific
agent), and they answer each other by running `hub say` in the other direction. No screen‑scraping.

> The preamble text lives in the `_wrap()` function in `hub.sh` (Chinese by default).
> Edit it for your language/style.

### Who is "sender"?
Run `hub` from inside an agent's tmux session → it sends **as that agent**. Run it from a
plain shell → it sends as a central operator. Identity is auto‑detected from `$TMUX`.

## Caveats

- Delivery appends to the recipient's input box + Enter. If the recipient has half‑typed
  text, it will merge — send when they're idle (`hub peek` first).
- Newlines in a message are flattened to spaces (so it submits as one line). For long
  content/code, write a file and `hub say` the path.
- `hub all` hits every `cc-*` session — mind the noise.

## License

[MIT](./LICENSE)
