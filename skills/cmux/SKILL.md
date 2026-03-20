---
name: cmux
description: Reference for cmux CLI — terminal multiplexer with browser automation. Use when managing terminal panes, browser panes, workspaces, or automating web interactions via cmux.
---

# cmux CLI Reference

Terminal multiplexer with built-in browser. IDs use ref format: `surface:<N>`, `pane:<N>`, `workspace:<N>`, `window:<N>`. Refs are returned by creation commands — track them for subsequent operations.

## Terminal panes

```bash
cmux new-pane --type terminal [--direction left|right|up|down]
cmux send "command" --surface surface:<N>
cmux send-key Enter --surface surface:<N>
cmux read-screen --surface surface:<N> [--scrollback --lines 100]
```

## Browser panes

```bash
cmux new-pane --type browser --url "https://example.com"
cmux browser goto <url> --surface surface:<N>
cmux browser wait --load-state complete --surface surface:<N>
cmux browser snapshot --surface surface:<N> --compact
```

Snapshot returns an accessibility tree with element refs (`e1`, `e2`...). Use refs to interact:

```bash
cmux browser fill <ref> "text"
cmux browser click <ref>
cmux browser type <ref> "text"
cmux browser press <key>
cmux browser get url|title --surface surface:<N>
cmux browser screenshot --surface surface:<N>
cmux browser back|forward|reload --surface surface:<N>
cmux browser tab new|list|switch|close --surface surface:<N>
```

Browser workflow: open pane -> wait for load -> snapshot -> interact via refs -> verify with snapshot/get.

Append `--snapshot-after` to any interaction command to auto-snapshot.

## Workspace & layout

```bash
cmux tree                                          # full layout hierarchy
cmux identify                                      # show caller's IDs
cmux list-workspaces | new-workspace [--cwd <path>] | select-workspace --workspace <ref>
cmux list-panes | focus-pane --pane <ref> | close-surface --surface <ref>
cmux new-split left|right|up|down --surface <ref>
cmux resize-pane --pane <ref> -L|-R|-U|-D [--amount <n>]
cmux list-windows | new-window | focus-window --window <ref>
```

## Full help

Run `cmux --help` for the complete command list.
