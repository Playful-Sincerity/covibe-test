---
id: dashboard-renderer
title: Build dashboard renderer from JSON
status: in-progress
assigned_to: wisdom-air
created_by: orchestrator
created_at: 2026-04-10 06:30
updated_at: 2026-04-10 06:29
completed_at: null
branch: wisdom-air/dashboard-renderer
appetite: small
workflow_type: implementation
folder: src
---

# Session Brief: Dashboard Renderer

**Dependencies:** None — can start immediately (uses interface contract, not actual collector output)
**Can run parallel with:** dashboard-collector
**Feeds into:** Final integration pipes collector | renderer

## Context
We're building a CoVibe status dashboard. This job builds the rendering half — a Python script that takes JSON from stdin and prints a formatted terminal dashboard.

## Task
Create `src/renderer.py` that:

1. Reads JSON from stdin (the collector's output format)
2. Renders a clean, readable terminal dashboard:

```
╔══════════════════════════════════════╗
║  CoVibe Dashboard — covibe-test      ║
╠══════════════════════════════════════╣

SESSIONS (2 active)
  wisdom-happy   active   Working on dashboard-renderer   2 min ago
  wisdom-air     active   Working on dashboard-collector   5 min ago

JOB BOARD
  Ready:       (none)
  In Progress: dashboard-collector (wisdom-air), dashboard-renderer (wisdom-happy)
  Review:      (none)
  Done:        (none)

RECENT MESSAGES
  [06:30] wisdom-happy: Posted 2 jobs to the board — dashboard-collector and dashboard-renderer
  [06:35] wisdom-air: Claimed dashboard-collector, starting work

╚══════════════════════════════════════╝
```

3. Use relative timestamps ("2 min ago", "1 hour ago") for last_updated
4. Color output with ANSI codes (green=active, yellow=in-progress, dim=done)

Requirements:
- Python 3 stdlib only
- Read from stdin, print to stdout
- Handle empty sections gracefully (no sessions, no jobs, etc.)
- Width adapts to terminal (use `shutil.get_terminal_size()`)

## Acceptance Criteria
- `echo '<test json>' | python3 src/renderer.py` produces readable output
- All sections render: header, sessions, job board, messages
- Colors work in terminal
- Graceful with empty/missing fields

## Notes
Use this test JSON for development:
```json
{"project":"covibe-test","participants":["wisdom-happy","wisdom-air"],"sessions":[{"user":"wisdom-happy","status":"active","current_job":"dashboard-renderer","last_updated":"2026-04-10 06:30","current_task":"Building the renderer"}],"jobs":[{"id":"dashboard-collector","title":"Build collector","status":"in-progress","assigned_to":"wisdom-air","dependencies":[]},{"id":"dashboard-renderer","title":"Build renderer","status":"in-progress","assigned_to":"wisdom-happy","dependencies":[]}],"messages":[{"from":"wisdom-happy","timestamp":"2026-04-10 06:30","body":"Jobs posted to the board"}]}
```
