# Usage

1. Start `tmux`
2. Edit `A0_AGENT` and `A1_AGENT` at the top of `two-agent-metaprompt.md`
3. `A0_AGENT` is the left-side supervisor you manually open; `A1_AGENT` is the right-side worker that A0 launches in tmux
4. Open your A0 runtime in the project root (for example `claude` or `codex`)
5. Copy and send this prompt
6. Communicate with **A0 only** (your original pane), A1 agent will be controlled by A0.
