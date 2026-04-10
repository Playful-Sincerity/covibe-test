---
id: dashboard-collector
title: Build .covibe/ data collector script
status: ready
assigned_to: null
created_by: orchestrator
created_at: 2026-04-10 06:30
updated_at: 2026-04-10 06:30
completed_at: null
branch: null
appetite: small
workflow_type: implementation
folder: src
---

# Session Brief: Dashboard Collector

**Dependencies:** None — can start immediately
**Can run parallel with:** dashboard-renderer
**Feeds into:** dashboard-renderer consumes the JSON this produces

## Context
We're building a CoVibe status dashboard. This job builds the data collection half — a Python script that reads all `.covibe/` files and outputs structured JSON to stdout.

## Task
Create `src/collector.py` that:

1. Takes a repo path as an argument (default: current directory)
2. Reads `.covibe/config.md` — extracts project name, participants
3. Reads all `.covibe/sessions/*.md` — extracts user, status, current_job, last_updated, current task
4. Reads all `.covibe/jobs/**/*.md` (recursive, handles phased structure) — extracts id, title, status, assigned_to, dependencies
5. Reads the 10 most recent `.covibe/messages/*.md` — extracts from, timestamp, body
6. Outputs a single JSON object to stdout:

```json
{
  "project": "name",
  "participants": ["user1", "user2"],
  "sessions": [
    {"user": "...", "status": "active", "current_job": "...", "last_updated": "...", "current_task": "..."}
  ],
  "jobs": [
    {"id": "...", "title": "...", "status": "ready", "assigned_to": null, "dependencies": []}
  ],
  "messages": [
    {"from": "...", "timestamp": "...", "body": "..."}
  ]
}
```

Requirements:
- Python 3 stdlib only (no pip installs)
- Parse YAML-like frontmatter with simple regex (not a YAML library)
- Handle missing files/directories gracefully
- Sort messages by filename (newest last)

## Acceptance Criteria
- `python3 src/collector.py /path/to/repo` outputs valid JSON
- JSON includes all active sessions, all jobs, recent messages
- Works on the covibe-test repo itself as test data

## Notes
Frontmatter is delimited by `---` lines. Extract key-value pairs with simple string splitting, not a full YAML parser.
