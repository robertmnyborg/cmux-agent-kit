# cmux-agent-kit

Three small scripts for running many Claude Code sessions in [cmux](https://github.com/manaflow-ai/cmux): pair two tabs side by side, spin a side thread into a fresh session with a written brief, and open a two-agent split. Stock cmux, no fork. They use only the public `cmux` CLI and `~/.config/cmux/cmux.json` actions, so cmux updates do not break them.

Tested against cmux 0.64.25 (106) on macOS.

Why a kit: a long Claude Code session is the biggest cost driver. In one two-week measurement (via `npx ccusage`), a single 8-day session accounted for 48% of total spend. `cmux-spinoff` moves a side thread out of that session. The child starts from a 60-line brief instead of the full transcript.

Map and visualizer companion: [agent-overworld](https://github.com/robertmnyborg/agent-overworld) draws every cmux tab as a map and uses the parent-to-child edges that `cmux-spinoff` records. The kit works without it.

## Install

```sh
git clone https://github.com/robertmnyborg/cmux-agent-kit
cd cmux-agent-kit
mkdir -p ~/.local/bin && ln -sf "$PWD"/bin/* ~/.local/bin/     # needs ~/.local/bin on PATH
cp -R skills/* ~/.claude/skills/                                # optional: /spinoff and /tandem for Claude Code
```

Requirements: cmux, zsh, python3, `jq` and `fzf` (cmux-pair only), `claude` on PATH. `cmux-tandem` also needs the Antigravity CLI `agy` unless you override the left pane (below).

## cmux-pair

Shows two tabs side by side. cmux nests tabs inside panes, not panes inside tabs, so the closest stock shape is a space. `cmux-pair` creates a new space titled "A + B" in the same workspace group, A on the left, B on the right. `--unpair` moves both tabs back and closes the pair.

```sh
cmux-pair                       # fzf picker: Tab marks two tabs, Enter pairs them
cmux-pair surface:3 surface:7   # skip the picker
cmux-pair --unpair              # run inside the paired space
```

The picker needs a terminal, so bind it as a cmux action. Merge `examples/cmux.json` into `~/.config/cmux/cmux.json`. It binds Ctrl+Cmd+P to pair and Ctrl+Cmd+Shift+P to unpair. The action opens a throwaway tab (`target: newTabInCurrentPane`) and the script closes that tab when done. State lives in `~/.local/state/cmux-pair/<pair workspace UUID>` and holds the origin space.

## cmux-spinoff

Opens a child Claude session in a new tab in the same pane and working directory. The child reads a handoff brief you wrote, not the parent transcript. The parent-to-child edge appends to `~/.overworld/edges.jsonl` (override the directory with `OVERWORLD_DIR`).

```sh
cmux-spinoff "Rate limit audit" ~/.overworld/handoffs/2026-10-21-rate-limit-audit.md
cmux-spinoff "Rate limit audit" brief.md --focus    # also jump to the new tab
```

Runs only inside a cmux terminal (`$CMUX_SURFACE_ID` must be set). The bundled skill `skills/spinoff` makes Claude write the brief for you: say "/spinoff" and it writes Goal, Decided, Facts with sources, Files, Open questions, and First step, then calls the script. Rule baked into the skill: the parent does not continue the side thread afterwards, and no secrets go into the brief.

Each edge is one JSON line: `parent`, `child` (surface UUIDs), `workspace`, `title`, `brief`, `at`.

## cmux-tandem

A broad-context agent on the left, Claude Code on the right. The split rule: Claude Code for localized work with cheap tests (1-2 files), the other agent for repo-wide reading and open-ended research.

```sh
cmux-tandem                 # inside cmux: split the current pane right and start claude there
cmux-tandem ~/code/myapp    # outside cmux: new workspace "tandem-myapp", agy left, claude right
```

Requires Antigravity (`agy`) for the default left pane. Swap either side with env vars:

```sh
export CMUX_TANDEM_LEFT_CMD="gemini"                  # default: agy
export CMUX_TANDEM_RIGHT_CMD="claude --model opus"    # default: claude
```

Inside cmux the current pane is the left side, so `CMUX_TANDEM_LEFT_CMD` applies only when you launch from outside. The original harness ran `agy --dangerously-skip-permissions`; this kit does not. Add that flag to the env var yourself if you want it.

`skills/tandem` lets you type "/tandem" in Claude Code to run it.

## cmux behaviors worth knowing

- `close-surface` refuses the last tab in a space (`invalid_state`). Use `close-workspace`.
- Dragging a tab between spaces in the UI emits `surface.closed` then `surface.created`, not `surface.moved`. Only the CLI `move-surface` emits `surface.moved`.
- Config actions have no CLI trigger, so the keyboard path of `cmux-pair` needs a human keypress to test. Config schema: `web/data/cmux.schema.json` in the cmux repo. `target` is `currentTerminal` or `newTabInCurrentPane`.

## License

MIT
