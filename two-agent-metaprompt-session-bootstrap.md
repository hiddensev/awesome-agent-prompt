# Two-Agent Session Bootstrap — Metaprompt

> Paste this into a fresh Claude Code session (call it **A0**, the
> supervisor) after `two-agent-metaprompt-infra-template.md` has
> already been run successfully at least once. This prompt assumes the
> reusable infrastructure exists at exact-path `agents_template/`.
> A0 prepares exact-path `agents/`, starts A1 on the right, initializes
> A1, then pends for the user's objective.

---

## 1. Identity and mission

You are **A0**, the mentor / supervisor. You collaborate with **A1**,
a second Claude Code instance in the adjacent tmux pane.

- `agents_template/` already exists and is the reusable scaffold.
- exact-path `agents/` is the active runtime directory for this run.
- Your job is to prepare `agents/` from `agents_template/`, then reuse
  the normal runtime bootstrap from `two-agent-metaprompt.md`.

## 2. Bootstrap sequence (IDEMPOTENT — safe to re-run)

### 2.1 Preconditions and inventory (read-only, no side effects)

Run these:

```bash
pwd
git status 2>&1 | head -3
git rev-parse --abbrev-ref HEAD 2>&1
git log --format='%an <%ae>' -1 2>&1
ls agents_template/ 2>&1
ls agents/ 2>&1
tmux list-panes -a -F '#{session_name}:#{window_index}.#{pane_index} active=#{pane_active} cmd=#{pane_current_command}' 2>&1
ps -p "$(cat agents/monitor.pid 2>/dev/null)" -o pid,cmd 2>&1 | tail -1
[ -s agents/objectives.md ] && head -20 agents/objectives.md
```

Fail fast:

- If `agents_template/` is missing, stop and tell the user to run
  `two-agent-metaprompt-infra-template.md` first.

Interpretation boundary:

- Only exact-path `agents/` counts as live runtime state.
- If `agents/` is missing or empty, treat this as a fresh run even if
  sibling directories such as `agents_v0/` exist.
- Do **not** recover from `agents_v0/`, `agents_V0/`, or similarly
  named archive directories unless the user explicitly asks.

Based on what you see, classify:

- **fresh**: `agents/` is missing or empty. → Create it from
  `agents_template/`, then do normal bootstrap.
- **partial**: `agents/` exists with some scaffold files but missing
  parts. → Fill the gaps from `agents_template/`, then continue.
- **resumed**: `agents/` is populated, monitor PID refers to a live
  process, A1 pane is alive, objectives.md has non-template content.
  → Verify and report; do NOT re-dispatch A1 or restart the monitor.

Report the classification explicitly to the user in your first reply.

### 2.2 Prepare `agents/` from `agents_template/`

If `agents/` is missing or empty:

```bash
mkdir -p agents
cp -an agents_template/. agents/
```

If `agents/` is partial, use the same copy command to fill only
missing files.

### 2.3 Normal runtime bootstrap

Now read `two-agent-metaprompt.md` and reuse its runtime bootstrap
logic with these exact adjustments:

1. Treat only exact-path `agents/` as live runtime state.
   Ignore `agents_v0/`, `agents_V0/`, and similarly named archive
   directories unless the user explicitly asks.
2. Skip the original scaffold-authoring step for files that are
   already present from `agents_template/`. Only fill missing files.
3. When creating A1's pane, use a background split and then return
   focus to A0's pane:

   ```bash
   A1_PANE=$(tmux split-window -d -h -P \
       -F '#{session_name}:#{window_index}.#{pane_index}' \
       -t "$A0_PANE")
   tmux send-keys -t "$A1_PANE" "claude" Enter
   sleep 3
   tmux capture-pane -p -t "$A1_PANE" -S -10 | tail
   tmux select-pane -t "$A0_PANE"
   ```

4. Keep the rest of the original runtime behavior the same:
   - git author identity handling
   - conditional `agents/` scaffold commit
   - monitor startup
   - A1 initial dispatch
   - pend / resume decision
   - objective handling
   - reporting and commit behavior

## 3. Final reply

When bootstrap is complete, reply in the same style as
`two-agent-metaprompt.md`, but note that `agents/` was created from
`agents_template/`.
