# Claude Code Configuration

## cmux

This environment runs inside [cmux](https://cmux.com/) — a native macOS terminal for multi-agent workflows.

Claude is allowed to use the `cmux` CLI to control the terminal layout, including:
- Creating and managing panes (`new-pane`, `new-split`, `close-surface`)
- Opening embedded browser panes (`--type browser`)
- Sending text/keys to other panes (`send`, `send-key`)
- Reading pane content (`read-screen`, `capture-pane`)
- Browser automation (`cmux browser` subcommands)
- Notifications (`notify`, `trigger-flash`)
- Workspace management (`new-workspace`, `select-workspace`, `rename-workspace`)
