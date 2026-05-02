---
name: regroup
description: Mid-task web-search re-evaluator that fires when stuck. Manual trigger /regroup, or phrases like "search for more", "look this up", "find more options", "out of ideas", "nothing is working", "stuck on this". Self-trigger after 3+ consecutive failed fix attempts on the same problem, an unrecognized error or tool not in training, exhausted options in the current plan, an edge case the plan didn't cover, or conflicting approaches with no clear winner. Do NOT fire on bare mentions of google docs, drive, sheets, maps, or analytics - those are products, not search intent.
---

# /regroup — Mid-Task Re-Evaluation via Web Search

This skill is NOT a standalone troubleshooter. It is a **re-evaluation injection** that fires mid-task when Claude is stuck or when the user signals it. It searches the web, merges the new data into the current working context, and produces an updated options list before continuing.

---

## When to invoke

Invoke this skill when any of these match:

- The user types `/regroup` anywhere in a message, or in a standing instruction.
- The user says something equivalent to giving up on the current path: "search for more", "look this up", "google this", "find more options", "get more data", "stuck on this", "out of ideas", "nothing is working", "try something else".
- Three or more consecutive fix attempts have failed on the same underlying problem.
- An error string, tool, or behavior shows up that you don't recognize from training and can't reason out from context.
- Every option already on the working plan has been tried; the list is empty.
- An edge case appears that the original plan didn't account for.
- Two viable approaches conflict and there's no further reasoning available to pick between them.

Do **not** invoke when:

- The user mentions "google docs", "google drive", "google sheets", "google maps", or "google analytics" — those are product names, not a request to search the web.
- It's the first failure of a fix attempt — give your own reasoning at least 2 attempts before reaching for `/regroup`.
- The user is mid-explanation and hasn't asked for help yet.

---

## Self-trigger announcement

When self-triggering, announce it before searching — never silently change direction. Use this line:

> "I'm hitting a wall on [specific blocker]. Running /regroup to pull in more options before continuing."

Replace `[specific blocker]` with the concrete thing that's failing (e.g. "the Ollama port refusing connections from the container", not "this issue").

---

## Workflow

### Step 1 — Capture current state

Before searching, snapshot:
- **Goal**: What is the overall task trying to accomplish?
- **Blocker**: What specifically is failing or unknown right now?
- **Options tried**: What has already been attempted and failed?
- **Options remaining**: What was still on the list before /regroup fired?

This state carries forward into the re-evaluation output.

### Step 2 — Build search queries

Construct 2–4 queries covering both angles:

**Blocker queries** (specific):
- Exact error string + tool/service name + environment
- e.g. `ECONNREFUSED 11434 Ollama Docker Windows`

**Goal context queries** (broader):
- What are the known working approaches for this type of task?
- e.g. `Ollama Docker container networking Windows host access alternatives`

Keep queries 4–7 words. Run them in parallel where possible.

### Step 3 — Fetch and extract

For the top 2–3 results per query, use web fetch to read full pages where snippets are insufficient. Extract:
- Confirmed fixes or workarounds for the blocker
- Alternative approaches to the goal that bypass the blocker entirely
- Version-specific notes or known bugs relevant to the environment
- Community-confirmed dead ends (saves retrying things that won't work)

### Step 4 — Update the options list

Produce a structured re-evaluation in this format:

---

**/regroup Re-Evaluation**

**Searching for:** [blocker] + [goal context]

---

**Issues flagged:**
- [Issue A found by search] — [What it means for the current approach, with source]
- [Issue B] — [Impact]

**Options (re-evaluated):**

1. **[Option]** HIGH confidence — [Why search data supports this as strongest path]
   - What to do: [concrete steps or command]
   - Source: [link]

2. **[Option]** MEDIUM confidence — [Viable but has a known complication]
   - What to do: [steps]
   - Source: [link]

3. **[Option]** LOW confidence — [Search data suggests this is unlikely to work, but worth knowing]
   - What to do: [steps]

*Options already on the list before /regroup fired keep their position. Confidence ratings are updated based on new data. New options found by search are added at the appropriate confidence level.*

---

**Recommended next step:** [Highest confidence option — specific action]

Proceed with this? Or pick a different option above.

---

### Step 5 — Wait for confirmation

Do not proceed until the user confirms. Accept:
- "Yes" / "go" / "do it" → proceed with recommended option
- A number → proceed with that option
- A modification → adjust and confirm before proceeding
- "No" / "skip" → present the next best option

---

## Self-Trigger Detection Rules

Monitor for these signals during any multi-step task:

| Signal | Threshold | Action |
|---|---|---|
| Same error repeating | 3rd occurrence | Self-trigger /regroup |
| Fix attempt fails | 3rd consecutive failure | Self-trigger /regroup |
| Unrecognized error/tool | First occurrence | Self-trigger /regroup |
| All listed options exhausted | 0 remaining | Self-trigger /regroup |
| Edge case not in original plan | First occurrence | Self-trigger /regroup |

When self-triggering, **announce it** before running — never silently change direction.

---

## Domain Search Hints

### Code errors
- Lead with the exact exception class or error code
- Include package name + version if visible in the trace
- Search both the error AND "alternative approach to [goal]"

### Docker / containers
- Include exit code — `137` (OOM), `139` (segfault), `1` (generic) mean different things
- Search the image name + error, not just the error alone
- Check for known image-specific issues: `[image-name] [error] github issues`

### Windows / PowerShell
- Always note admin vs non-admin context in the query
- Separate WSL2 searches from native Windows — they have different fix paths
- Include PowerShell version if relevant (`$PSVersionTable.PSVersion`)

### Home lab / server / networking
- Include the specific service name (Ollama, Caddy, Nginx, Tailscale, etc.)
- Port and protocol matter — include them
- For NUC/hardware issues: include the hardware model

---

## Anti-Patterns

- **Never self-trigger silently** — always announce before searching
- **Never replace the entire options list** — flag dead ends, then add new options
- **Never proceed after re-evaluation without confirmation**
- **Never search once and stop** — if first results are thin, run a second query with adjusted terms
- **Never reproduce 15+ words verbatim from any source**
- **Do not trigger on first failure** — give Claude's own reasoning 2 attempts before reaching for /regroup
