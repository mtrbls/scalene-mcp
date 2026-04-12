---
name: scalene
description: Scalene AI coding scorecard
arguments:
  - name: action
    description: "setup = configure credentials, sync = import all history, sync session = current session only, status = show dashboard link"
    required: true
---

## Instructions

The sync script is at `${CLAUDE_PLUGIN_ROOT}/bin/scalene-sync.py`. Credentials are in `$SCALENE_API_URL` and `$SCALENE_TOKEN` environment variables.

### /scalene setup

Do NOT show menus, pickers, or multi-step wizards. Do this automatically:

1. Check if already configured:
   ```bash
   echo "URL=${SCALENE_API_URL:-}" && echo "TOKEN=${SCALENE_TOKEN:-}"
   ```
   If both are set, say "Already configured. Dashboard: <url>" and stop.

2. Start the device auth flow. Run:
   ```bash
   curl -s -X POST https://getscalene.com/api/cli/auth
   ```
   This returns JSON: `{"code": "ABC123", "url": "https://getscalene.com/cli/confirm?code=ABC123"}`

3. Tell the user: "Opening your browser to confirm..." and run:
   ```bash
   open "<url from step 2>"
   ```

4. Poll until confirmed (every 2 seconds, max 60 attempts):
   ```bash
   curl -s "https://getscalene.com/api/cli/poll?code=<code>"
   ```
   Response will be `{"status": "pending"}` until the user confirms in the browser, then `{"status": "confirmed", "api_url": "...", "token": "..."}`.

5. Once confirmed, write the credentials to the shell profile:
   ```bash
   echo 'export SCALENE_API_URL=<api_url>' >> ~/.zshrc && echo 'export SCALENE_TOKEN=<token>' >> ~/.zshrc && export SCALENE_API_URL=<api_url> && export SCALENE_TOKEN=<token>
   ```

6. Say "Connected!" and immediately run `/scalene sync` to import history.

### /scalene sync

Run: `python3 ${CLAUDE_PLUGIN_ROOT}/bin/scalene-sync.py --api-url "$SCALENE_API_URL" --token "$SCALENE_TOKEN"`

Imports all historical sessions from `~/.claude/projects/`. May take a few minutes for large histories.

### /scalene sync session

Run: `python3 ${CLAUDE_PLUGIN_ROOT}/bin/scalene-sync.py --api-url "$SCALENE_API_URL" --token "$SCALENE_TOKEN" --session-only $CLAUDE_SESSION_ID`

Syncs only the current session.

### /scalene status

Print the user's dashboard URL (the base domain from `$SCALENE_API_URL` + `/me`). No sync.

After any sync, tell the user their dashboard is updated.

## Privacy

Only metadata is exported — token counts, timestamps, model IDs, tool names. Never prompt text, file contents, or tool arguments.
