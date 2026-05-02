# regroup

> **Google It!** — when you're stuck, regroup searches the web, returns with new options, and lets you continue smarter.

A mid-task web-search re-evaluator for Claude Code that fires when you're stuck.

## What it does

`regroup` is not a standalone troubleshooter — it's a re-evaluation injection. When Claude has looped on the same fix three times, hit an unrecognized error, or run out of ideas, `regroup` searches the web, merges the new data into the working context, and produces an updated options list with confidence ratings. You confirm before anything proceeds.

## When it triggers

- Manual: you type `/regroup` anywhere in a message
- You say something like "look this up", "find more options", "stuck on this", "nothing is working", "out of ideas", or "try something else"
- Three or more consecutive fix attempts have failed on the same problem
- An unrecognized error string, tool, or behavior appears
- Every option in the current plan has been tried
- An edge case the plan didn't cover comes up

## Example output

```
/regroup Re-Evaluation
Searching for: ECONNREFUSED 11434 from Docker container + Ollama host networking on Windows

Issues flagged:
- Docker Desktop on Windows can't reach 127.0.0.1 from inside containers — needs host.docker.internal (Stack Overflow, GitHub Ollama #1431)
- Ollama default bind 127.0.0.1 blocks all container traffic, including host.docker.internal (Ollama docs)

Options (re-evaluated):
1. Set OLLAMA_HOST=0.0.0.0 then use host.docker.internal:11434 from container — HIGH
2. Run Ollama itself in a sidecar container on the same Docker network — MEDIUM
3. Switch the container to host network mode — LOW (Windows Docker Desktop doesn't support it)

Recommended next step: option 1 — restart Ollama with OLLAMA_HOST=0.0.0.0 and retry from the container.
```

## Install

```
/plugin marketplace add McMiller81/regroup
/plugin install regroup
```

Then run `/reload-plugins` (or restart Claude Code) to activate. Requires Claude Code with plugin support.

## Manual trigger

Type `/regroup` in any message and the skill fires immediately, regardless of context.

## License

MIT — see LICENSE.

## Origin

Drafted as a personal `/googleit` skill, refined and renamed to `regroup` for general use. Uses the Claude Code WebSearch tool (Brave-backed) — not Google search specifically.
