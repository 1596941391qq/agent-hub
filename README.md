# hub — message & manage a crew of CLI agents in tmux

A tiny, dependency‑free Bash CLI for running a *crew* of terminal coding agents
(e.g. [Claude Code](https://claude.com/claude-code), or any REPL) — one agent per
project, each in its own `tmux` session — and letting **you _and the agents
themselves_** check status and pass messages between sessions.

No desktop app, no MCP server, no daemon. Just `tmux` + one shell script — a
deliberately lightweight alternative to heavier multi‑agent orchestrators.

## Model
- One agent per `tmux` session named `cc-<name>` (e.g. `cc-frontend`, `cc-backend`, `cc-docs`).
- `hub` discovers every `cc-*` session and drives it via `tmux send-keys` / `capture-pane`.

## Requirements
`tmux`, `bash`. `git` optional (only for repo status in `hub ls`).

## Install
```bash
git clone https://github.com/<you>/agent-hub.git
chmod +x agent-hub/hub.sh
ln -s "$(pwd)/agent-hub/hub.sh" ~/.local/bin/hub   # ensure ~/.local/bin is on PATH
```

## Usage
| command | what it does |
|---|---|
| `hub ls` | list every `cc-*` session: git status (dirty count, ahead/behind) + remote + path |
| `hub peek <cc> [n]` | show the last N lines of an agent's screen (default 40) |
| `hub say <cc> "msg"` | send a message to an agent (wrapped as an agent‑to‑agent note) |
| `hub ask <cc> "msg"` | same, but ask the recipient to reply via `hub say <you> "…"` |
| `hub all "msg"` | broadcast to every agent except yourself |

`<cc>` accepts the full session name or any unique fragment (e.g. `front`, `back`).

## Agent‑to‑agent messages
Every message is prefixed with a short preamble that (1) names **sender → recipient**,
(2) flags it as **agent‑to‑agent** (be concise & machine‑readable, skip human‑facing
prose), and (3) for `ask`, tells the recipient the exact command to reply with. So the
agents coordinate **directly** — you dispatch from one place and can target a specific
agent, and they answer each other by running `hub say` in the other direction (no
screen‑scraping).

> The preamble text lives in the `_wrap()` function in `hub.sh` (Chinese by default).
> Edit it to your taste/language.

## Who is "sender"
Run `hub` from inside an agent's tmux session → it sends **as that agent**. Run it from
a plain shell → it sends as a central operator. Identity is auto‑detected from `$TMUX`.

## Caveats
- Delivery = append to the recipient's input box + Enter. If the recipient has
  half‑typed text, it will merge — send when they're idle (`hub peek` first).
- Newlines in a message are flattened to spaces (so it submits as one line). For long
  content/code, write a file and `hub say` the path.
- `hub all` hits every `cc-*` session — mind the noise.

## License
MIT — see [LICENSE](LICENSE).

---

### 中文简介
`hub` 是个**极简、零依赖**的 Bash 小工具:把多个跑在 `tmux` 里的 CLI agent(比如
Claude Code,每个项目一个、会话名 `cc-<名字>`)统一**查看状态 + 互相发消息**——你在中
控发号、`@` 指定 agent,agent 之间也能直接 `hub say` 来回对接。命令见上表(`ls`/`peek`/
`say`/`ask`/`all`)。发消息会自动包一层"**agent 间通讯**"提示词(标明收发方、要求简洁机器
可读、`ask` 附回信命令)。比 golutra 那类重型编排器轻得多,只要 `tmux` + 一个脚本。
