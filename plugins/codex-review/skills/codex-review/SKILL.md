---
name: codex-review
description: Iterative adversarial review of a spec, plan, design doc, or the current code diff using the Codex CLI. Keeps one Codex session across rounds, focuses on high/medium-severity issues, and confirms proposed changes with the user before applying them unless asked to auto-resolve. Use when asked for a codex review, second opinion, or adversarial review.
---

Argument: a file path (spec, plan, any doc), or `diff` for the working tree. All Codex calls are read-only; Codex never edits anything. Work in a fresh per-run directory so concurrent reviews never collide: run `mktemp -d "${TMPDIR:-/tmp}/codex-review.XXXXXX"` once, and use the printed path literally as `<dir>` in every later command (shell variables don't persist between calls). Never reuse another run's directory or thread id.

**Mode.** Default is **confirm**: never edit the doc/code until the user approves that round's decisions. Use **auto** only if the user explicitly asks ("auto", "fix everything", "resolve all without asking"), at the start or mid-review. In auto, apply your own recommendations and still show the table each round.

**Bar.** The goal is to fix what matters, not to exhaust Codex. Adversarial reviewers never run dry; each round digs into rarer edge cases. Only high/medium issues drive changes and further rounds. Low issues are listed once and skipped unless the user picks them.

## Model

Model and effort live in `~/.codex/review.config.toml` (a Codex profile). Round 1 passes `-p review`. `resume` ignores profiles and silently falls back to the user's default model, so every resume must pass the same values explicitly: read them once with
`sed -n 's/^model = "\(.*\)"/\1/p; s/^model_reasoning_effort = "\(.*\)"/\1/p' ~/.codex/review.config.toml` (prints model, then effort)
and add `-m <model> -c model_reasoning_effort=<effort>` to each resume, with the printed values written in literally (shell variables don't persist between calls). If the user names a different model for this review, use that instead of the profile.

If `~/.codex/review.config.toml` doesn't exist, drop `-p review`, `-m` and `-c` everywhere so every round uses the Codex defaults, and tell the user once.

## Round 1 — open the thread

Doc:
```bash
cat <path> | codex exec -p review -s read-only --skip-git-repo-check --json -o <dir>/r1.md \
  "Adversarially review this document. Find issues that matter: gaps, contradictions, unstated assumptions, edge cases likely to occur in practice, scope risks. For each issue give: number, severity (high/medium/low), the concrete scenario where it fails, and the smallest fix. Severity: high = breaks in normal use, or any security/data-loss/corruption risk; medium = realistic failure under plausible conditions, or a gap that blocks implementation; low = rare or contrived edge case, hardening, style, nice-to-have. List high and medium in full, most important first. Put low items last under 'Minor', one line each, max 5. Do not propose abstractions, configuration, or defensive code for unlikely scenarios. Do not restate the document. If there are no high or medium issues, end your reply with the line: NO_OPEN_ISSUES" > <dir>/events.jsonl
```
Diff: same command, but feed `git diff HEAD` (plus untracked files) on stdin and say "review these code changes" instead of "this document".

If the user gave context (prototype vs production, stakes, what's out of scope), prepend it to the prompt as `Context: ...`.

Capture the thread id: `grep -o '"thread_id":"[^"]*"' <dir>/events.jsonl | head -1`. Keep it for every later round. Read `r1.md`.

## Triage (every round)

Verify each issue against the actual doc/code. Treat Codex's severity as input, not a verdict, and re-rate it yourself. Recommend one of:
- **fix**: real and worth it. Describe the fix in a few words and prefer the smallest one.
- **skip**: one-line reason (wrong, already covered at line X, out of scope, or not worth it because it's rare and low-impact, or the fix adds more complexity than the risk it removes).
- **defer**: real, but belongs in a later iteration; say why.

Keep the fix proportional to likelihood × impact. Default low items to skip unless the fix is a one-liner. Never recommend a fix just to make Codex stop.

Then:
- **Confirm:** show the table (#, severity, issue, recommendation, fix or reason) and **stop**. The user replies `ok` to take all recommendations, overrides by number (`fix 4, skip 2`), or says `auto`. Apply only what they approve. When the user gives a reason for a skip, pass it to Codex.
- **Auto:** apply your recommendations, show the table, continue.

## Round N — resume the same thread

```bash
{ echo "Revised document below. Round <N>. Decisions on your previous issues:"; echo "<table: #, decision, reason>"; echo; echo "----- DOCUMENT -----"; cat <path>; } | \
  codex exec resume <thread_id> -m <model> -c model_reasoning_effort=<effort> --skip-git-repo-check -o <dir>/r<N>.md \
  "Re-review. Raise only (a) NEW high/medium issues, including any introduced by the revisions, or (b) previously raised issues you still consider unresolved after my stated reasoning. Skips are deliberate decisions: drop them unless you can show a concrete high-severity scenario. Do not repeat or add Minor items. Same format. If no high or medium issues remain, reply: NO_OPEN_ISSUES"
```
Diff: same, with the current diff in place of the document.

## Stop when

- Codex replies `NO_OPEN_ISSUES`, or
- every issue in the latest round is skipped/deferred with a reason and Codex raised nothing new, or
- the user says stop, or
- round 5 is reached: stop and tell the user what is still contested.

Finish with a short summary: rounds run, issues fixed, skipped (with reasons), deferred, Minor items not acted on, and what changed.
