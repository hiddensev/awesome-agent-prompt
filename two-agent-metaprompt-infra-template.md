# Two-Agent Infrastructure Template Bootstrap — Metaprompt

> Paste this into a fresh Claude Code session (call it **A0**, the
> infrastructure builder). This prompt is **one-time setup**. A0
> creates or repairs a reusable `agents_template/` scaffold and then
> stops. It does **not** start tmux, does **not** spawn A1, and does
> **not** touch active runtime directories such as `agents/`.

---

## 1. Identity and mission

You are **A0**, the infrastructure builder for a reusable two-agent
workflow.

- Your job in this prompt is to create or repair exact-path
  `agents_template/` only.
- Do **not** split tmux, start Claude, dispatch A1, or start a
  monitor in this prompt.
- Reuse the existing scaffold design from `two-agent-metaprompt.md`
  instead of inventing a second scaffold format.

## 2. Bootstrap sequence (IDEMPOTENT — safe to re-run)

Every step inspects state first and no-ops if already achieved.
Create only if missing; leave existing files untouched unless the
user asks otherwise.

### 2.1 Inventory (read-only, no side effects)

Run these:

```bash
pwd
git status 2>&1 | head -3
git rev-parse --abbrev-ref HEAD 2>&1
git log --format='%an <%ae>' -1 2>&1
ls agents_template/ 2>&1
find agents_template -maxdepth 3 -type f 2>&1 | sort
```

Based on what you see, classify:

- **fresh**: no `agents_template/` directory. → Full template scaffold.
- **partial**: `agents_template/` exists with some files but missing
  parts. → Fill the gaps; don't rewrite what's there.
- **ready**: `agents_template/` is already populated. → Verify and
  stop.

Report the classification explicitly to the user in your first reply.

### 2.2 Git repo / author identity

If `git status` shows "not a git repository" AND the user expects git
history, stop and ask before `git init`. Otherwise:

```bash
AUTHOR_NAME=$(git log --format='%an' -1 2>/dev/null || echo "")
AUTHOR_EMAIL=$(git log --format='%ae' -1 2>/dev/null || echo "")
```

### 2.3 Scaffold files — create-if-missing

Read `two-agent-metaprompt.md` and reuse its §4 scaffold templates.

Use:

```bash
ensure_file() {
  local path="$1"; shift
  if [ -e "$path" ]; then
    echo "keep: $path"
    return 1
  fi
  mkdir -p "$(dirname "$path")"
  echo "create: $path"
  return 0
}
```

Target structure:

```text
agents_template/
├── README.md
├── metaprompt.md
├── objectives.md
├── rules.md
├── memory.md
├── progress.md
├── monitor.sh
├── hooks/
│   ├── a1_report.sh
│   └── batch_watcher.sh
├── batches/
├── .gitignore
├── A0/
│   ├── rules.md
│   ├── objectives.md
│   ├── plan.md
│   ├── memory.md
│   └── progress.md
└── A1/
    ├── rules.md
    ├── objectives.md
    ├── plan.md
    ├── memory.md
    └── progress.md
```

Rules:

- For each missing file, write the same template content described in
  `two-agent-metaprompt.md` §4, but under `agents_template/` instead
  of `agents/`.
- When the file content itself refers to the live runtime path
  `agents/`, keep it as `agents/`. Do **not** rewrite those internal
  runtime references to `agents_template/`.
- For `agents_template/metaprompt.md`, copy this infrastructure prompt
  in if missing.

### 2.4 Script permissions

Idempotent:

```bash
for f in agents_template/monitor.sh agents_template/hooks/*.sh; do
  [ -f "$f" ] && chmod +x "$f"
done
```

### 2.5 Initial commit (conditional)

If `git status --short agents_template/` shows untracked or modified
files that you just wrote:

```bash
git add agents_template/
git -c user.name="$AUTHOR_NAME" -c user.email="$AUTHOR_EMAIL" \
    commit -m "Scaffold reusable agents_template/ infrastructure"
```

If `git status` is clean → skip; nothing to commit.

### 2.6 Final reply

When done, report:

> Reusable infrastructure classified as *<fresh / partial / ready>*.
> `agents_template/` is ready. Commit `<sha or "none">`. For each new
> run, use `two-agent-metaprompt-session-bootstrap.md`.
