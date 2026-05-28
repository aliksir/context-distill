---
name: context-distill
description: Distill session context into structured YAML snapshots for AI-to-AI knowledge transfer. Captures decisions, failures, recovery, and reusable patterns. Trigger with "/context-distill", "コンテキスト抽出", "context distill", or "snapshot 生成".
---

# `/context-distill` — Session Context Distillation

Distill session context (files read, design decisions, failures & recovery, reusable patterns, metrics) into **structured YAML** and persist to `memory/snapshots/`. Built for AI-to-AI knowledge transfer across sessions.

## Trigger Words

- Explicit: `/context-distill`
- Implicit: "コンテキスト抽出", "context distill", "snapshot 生成", "save session context"
- Integration: runs alongside `/handover` (handover = human-readable, snapshot = machine-parseable)

## 4 Principles

### 1. No Fabrication

If a fact cannot be confirmed, write `unknown`. Never fill in creative guesses.
- ✅ `tool_calls_total: unknown`
- ❌ `tool_calls_total: 50  # probably around this`

### 2. YAML Structure (No Verbose Narratives)

snapshot.yaml is meant for **machine parsing by AI**. No markdown paragraphs.
- ✅ `decision: "use xlsx as SSOT, fix pptx to match" / why: "v3 inheritance mismatch"`
- ❌ `# About important decisions\nIn this session we decided...`

Each `decision` / `failure` / `pattern` = **1 YAML object, 1-3 lines max**.

### 3. PII Awareness (Confidentiality Levels)

| confidentiality | file paths | key_decisions | failures | reusable_patterns |
|---|---|---|---|---|
| **high** (sensitive) | anonymize (`${ANON:N}`) or omit | no customer/employee names | same | same, strip proper nouns |
| medium | relative paths recommended | full | full | full |
| low / public | full paths ok | full | full | full |

### 4. Append-Only (No Overwrites)

- Never overwrite an existing `snapshot.yaml` — append only
- For re-runs, create a new file with `-v2`, `-v3` suffix
- `index.jsonl` is also append-only

## Extraction Steps (6 Steps)

### Step 1: Get Real Time

Get the current time using your system's clock. AI time estimates are unreliable.

```bash
date '+%Y-%m-%d %H:%M'        # Linux/macOS
node -e "console.log(new Date().toLocaleString('sv').slice(0,16))"  # Cross-platform
```

Record `takeover_at` and `handover_at` as SSOT.

### Step 2: Read Access Log (if available)

If your project maintains an access log (e.g., `memory/access-log.jsonl`), filter entries for the current session. Extract important `files_read` / `files_modified`.

If no access log exists, use conversation context and handover notes as fallback.

### Step 3: Read Handover Notes (if available)

Check for `handover.md` or equivalent session notes. Use as supplementary source for:
- Session scale / category / carry-over items
- Completed tasks / changed files
- Metrics (intervention_count, review_cycles, etc.)

### Step 4: Context Reflection

Reflect on the current session across these dimensions:

| Dimension | YAML Field |
|-----------|-----------|
| "What was decided, and why?" | `key_decisions[]` |
| "What failed, and how was it recovered?" | `failures_and_recovery[]` |
| "What procedures/code are reusable next time?" | `reusable_patterns[]` |
| "Numeric metrics" | `metrics{}` |
| "What carries over to the next session?" | `carry_over[]` |

### Step 5: Write snapshot.yaml

Create `memory/snapshots/{YYYYMMDD}_{topic-slug}/snapshot.yaml`.

- Directory name: `{YYYYMMDD}_{topic-slug}`
  - `topic-slug`: kebab-case, 3-6 words, max 40 chars
  - No colons, slashes, or spaces (filesystem safety)
- If same name exists: append `-v2`, `-v3` (no overwrites, Principle 4)

Use the template below as a starting point. Fill all fields; use `unknown` for unknowns.

### Step 6: Append to index.jsonl

Append **one line** to `memory/snapshots/index.jsonl`:

```jsonl
{"date":"2026-05-16","topic":"my-feature-work","category":"development","keywords":["feature","api","auth"],"path":"20260516_my-feature-work/snapshot.yaml","scale":"squad"}
```

`keywords`: 5-8 terms for future semantic search.

## Scale-Based Detail Level

| Scale | Detail |
|-------|--------|
| **Recon** (questions/research) | Minimal: meta + files_read + 1-2 key_decisions. Empty arrays for unused sections |
| **Squad** (1-2 file changes) | Standard: all 9 sections. 0-2 failures/patterns |
| **Platoon** (3-5 files + design decisions) | Detailed: all 9 sections, 3-5 key_decisions, 2-3 patterns, full metrics |
| **Battalion** (6+ files) | Complete: all fields, review result links in references, fp_root_cause |

Don't mass-produce snapshots full of `unknown`. **If no fact exists, use empty array `[]`**.

## Failure Behavior

- Cannot gather required info → **abort, don't leave partial files**
- Directory creation fails → abort, report error
- index.jsonl append fails → snapshot.yaml is ok to keep, fix index next run

## snapshot.yaml Template

```yaml
meta:
  session_id: "YYYY-MM-DD_HHMMSS"
  date: "YYYY-MM-DD"
  takeover_at: "YYYY-MM-DD HH:MM"
  handover_at: "YYYY-MM-DD HH:MM"
  duration_min: 0
  topic: "<kebab-case-slug>"
  category: "development"        # development / consultation / business / decision / support / other
  scale: "squad"                 # recon / squad / platoon / company / battalion
  confidentiality: "low"         # high / medium / low / public
  evidence_level: "unknown"      # suspicion / static_check_passed / test_passed / root_cause_explained / integration_verified / production_validated

files_read:
  - { path: "src/main.ts", role: "base", note: "entry point analysis" }

files_modified:
  - { path: "src/feature.ts", action: "create" }

key_decisions:
  - decision: "short description of what was decided"
    why: "reason for this decision"
    evidence: "how this was validated"
    reusable: true

failures_and_recovery:
  - failure: "what went wrong"
    recovery: "how it was fixed"
    prevention: "how to prevent next time"

reusable_patterns:
  - pattern: "description of the reusable pattern"
    snippet: |
      code or steps here
    when: "when to apply this pattern"

metrics:
  intervention_count: 0
  review_cycles: 0
  tool_calls_total: unknown

carry_over:
  - { task: "remaining work", urgency: "medium" }

references:
  handover: "path/to/handover.md"
```

## Integration with `/handover`

| Aspect | handover.md | snapshot.yaml |
|--------|-------------|---------------|
| Audience | Human + AI | **AI only** |
| Format | Markdown narrative | YAML structured |
| Purpose | Session handoff overview | Machine-parse for similar case search & reuse |
| Update | Overwrite (latest session only) | Append-only (all sessions preserved) |

Both contain overlapping info, but serve different purposes. Handover is verbose human-readable; snapshot is concise machine-parseable.

## Philosophy

AI agents accumulate experience across sessions — but only if that experience is stored in a format other AI agents can efficiently parse. This skill creates that structured layer, complementing human-readable notes (handover, chat logs, lessons) with a machine-optimized YAML format for cross-session learning.
