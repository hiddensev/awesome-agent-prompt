# Usage

## Single prompt

1. Start `tmux`
2. Open `claude` in the project root. (codex also works by simply replacing model name in prompt)
3. Copy and send `two-agent-metaprompt.md`
4. Communicate with **A0 only** (your original pane), A1 agent will be controlled by A0.

## Split prompts

1. Run `two-agent-metaprompt-infra-template.md` once to create reusable `agents_template/`
2. For each new run, open `claude` in the project root
3. Copy and send `two-agent-metaprompt-session-bootstrap.md`
4. Communicate with **A0 only** (your original pane), A1 agent will be controlled by A0.
