---
name: scalene-sync
description: Sync Claude Code session data to your Scalene dashboard
arguments:
  - name: scope
    description: "What to sync: 'all' for full history, 'session' for current session only (default: all)"
    required: false
    default: "all"
---

Sync Claude Code session metadata to the user's Scalene dashboard.

## Instructions

1. The sync script is at `${CLAUDE_PLUGIN_ROOT}/bin/scalene-sync.py`
2. The API URL is in the `SCALENE_API_URL` environment variable
3. The bearer token is in the `SCALENE_TOKEN` environment variable

If `$scope` is "all" or not provided:
- Run: `python3 ${CLAUDE_PLUGIN_ROOT}/bin/scalene-sync.py --api-url "$SCALENE_API_URL" --token "$SCALENE_TOKEN"`
- This imports all historical sessions from `~/.claude/projects/`
- Tell the user it may take a few minutes for large histories

If `$scope` is "session":
- Run: `python3 ${CLAUDE_PLUGIN_ROOT}/bin/scalene-sync.py --api-url "$SCALENE_API_URL" --token "$SCALENE_TOKEN" --session-only $CLAUDE_SESSION_ID`
- This syncs only the current session

After the sync completes, tell the user their dashboard is ready at `$SCALENE_API_URL/me` (strip the `/u/...` path component — just use the base domain).

## Privacy

Only metadata is exported — token counts, timestamps, model IDs, tool names. Never prompt text, file contents, or tool arguments. The script runs entirely on the user's machine.
