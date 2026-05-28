# context-distill — Session Context Distillation for Claude Code

**[日本語版 README はこちら](README.ja.md)**

Distill session context into structured YAML snapshots for AI-to-AI knowledge transfer across Claude Code sessions.

## What It Does

When you finish a work session, `/context-distill` captures what happened — key decisions, failures & recovery, reusable patterns — and saves it as a structured YAML file that future AI sessions can parse and reuse.

Think of it as **session memory that AI agents can actually search and learn from**, not just human-readable notes.

## Quick Start

```bash
claude plugin install context-distill
```

Then in Claude Code:
```
/context-distill
```

## How It Works

1. **Captures** what files were read/modified, what decisions were made and why
2. **Structures** everything into a YAML snapshot (not markdown prose)
3. **Persists** to `memory/snapshots/{date}_{topic}/snapshot.yaml`
4. **Indexes** in `memory/snapshots/index.jsonl` for future search

### Example Output

```yaml
key_decisions:
  - decision: "use event table instead of status column"
    why: "state transitions have business meaning (contract lifecycle)"
    evidence: "domain analysis during design review"
    reusable: true

failures_and_recovery:
  - failure: "migration script failed on existing data"
    recovery: "added default value for new column, re-ran migration"
    prevention: "always test migrations against production-like data"

reusable_patterns:
  - pattern: "ZIP file content replacement via full rewrite"
    snippet: |
      with zipfile.ZipFile(src) as zin, zipfile.ZipFile(dst, 'w') as zout:
          for item in zin.namelist():
              data = new_content if item == target else zin.read(item)
              zout.writestr(item, data)
    when: "modifying files inside ZIP-based formats (OOXML, JAR, etc.)"
```

## Key Principles

| Principle | Rule |
|-----------|------|
| **No fabrication** | Unknown values → `unknown`, never guess |
| **YAML only** | Machine-parseable, no markdown paragraphs |
| **PII awareness** | Sensitive sessions → anonymize paths and names |
| **Append-only** | Never overwrite existing snapshots |

## Handover vs Snapshot

| | handover.md | snapshot.yaml |
|---|---|---|
| Audience | Human + AI | AI only |
| Format | Markdown | YAML |
| Purpose | Session overview | Machine-parseable knowledge |
| Lifecycle | Overwritten each session | Append-only, all preserved |

Both can coexist. Handover is for humans catching up; snapshot is for AI agents learning from past sessions.

## Requirements

- Claude Code
- A `memory/` directory in your project (created automatically on first run)

## License

MIT
