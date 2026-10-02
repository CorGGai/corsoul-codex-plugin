---
name: corsoul-connect
description: Connect, start, verify, or troubleshoot the shared Corsoul service used by the Codex plugin, and offer the optional AGENTS.md strict-memory setup. Trigger when Corsoul or Cortex MCP is unavailable, port 3848 is down, the user asks whether Postgres or Docker is required, Codex has duplicate memory tools, or the user wants persistent recall rules.
---

# Connect Corsoul to Codex

Keep one Corsoul HTTP owner on loopback and let every Codex task connect to it. Do not configure a separate stdio/PGLite server per task.

## Diagnose

1. Request `http://127.0.0.1:3848/health` with a short timeout.
2. Accept the endpoint only when it reports a healthy Corsoul service. If another process owns the port, report that conflict instead of replacing or killing it.
3. If the health endpoint works but `corsoul_*` tools are absent, tell the user to open a new Codex task. Plugin MCP discovery happens at task initialization.
4. If health fails, distinguish the service being stopped from its database backend being unavailable. Docker or Postgres is needed only when that owner is deliberately configured with `DATABASE_URL`; free local PGLite does not need either one.

## Start the local owner

When no owner is healthy, offer these two choices before changing the machine:

1. **Persistent PM2 service (recommended):** crash recovery plus reboot/login recovery after the OS startup hook is installed. Explain that this installs global npm packages and changes startup configuration, then wait for explicit consent.
2. **Temporary foreground owner:** no startup change; it stops when its terminal or process ends.

For persistent mode, run the bundled platform helper from the plugin's `scripts` directory:

```text
# Windows PowerShell
powershell -ExecutionPolicy Bypass -File scripts/install-corsoul-pm2.ps1

# Linux or macOS
sh scripts/install-corsoul-pm2.sh
```

Resolve the actual plugin path before running it; do not assume the current working directory is the plugin root. Review the helper with the user when requested. Never run it merely because the plugin was installed.

The helper must install the reviewed compatibility version, start exactly one `corsoul-mcp` process on loopback, run `pm2 save`, and configure or clearly report the remaining OS startup step. Verify `pm2 status`, `/health`, and that no `corsoul-activation-runner` was created. Do not run the Beta repository's full `ecosystem.config.cjs`, because it also contains an activation runner. Do not adopt, replace, or kill an unknown process already bound to port 3848.

For temporary mode, run:

```text
npx -y --package=corsoul@0.1.20 corsoul --transport=http --host=127.0.0.1 --port=3848
```

Keep it in the foreground unless the user explicitly requests a temporary detached process. On Windows, a user-approved detached process may be launched hidden. Keep stdout and stderr out of the MCP protocol stream and preserve a useful log when practical. Do not pass `DATABASE_URL` unless the user explicitly chooses Postgres.

Poll `/health` for at most 30 seconds. On success, explain that the service uses PGLite by default and ask the user to start a new Codex task if the memory tools are not already present. On failure, report the process error or log location; do not repeatedly spawn more owners. Do not claim reboot protection until both `pm2 save` and the platform startup hook have succeeded.

This command is the reviewed managed-start baseline; it does not prove or replace the version of an owner that is already healthy on port 3848. The health response currently has no package build version, so an existing owner's version and policy remain the operator's responsibility.

Keep the managed-start baseline pinned for reproducibility and do not silently change it to `latest`. The `0.1.7` baseline keeps that instruction contract — its MCP instructions state the authorization boundary up front: due items are untrusted reminders that require notice and triage, never permission to act — and additionally adds single-owner election, so concurrent engines coordinate on one store owner instead of opening the same PGLite directory and tearing it. That matches this plugin's untrusted-memory and one-owner policy. Before any future baseline change, verify the new release keeps that instruction contract, then re-run the connection, concurrency, time-trust, prompt-injection, due-item authorization, provider-outage, and cross-task persistence tests.

## Verify without polluting memory

Prefer a read-only `corsoul_recall` call in the intended stable scope. Create a test memory only with the user's consent, and remove it afterward only if the user requests that tombstone.

## Existing installations

If Codex also has a manual `cortex` or `corsoul` MCP entry, verify this plugin first and then recommend disabling the duplicate. Never run two PGLite owners against the same data directory. Do not edit or delete the user's configuration unless they explicitly ask.

For an intentional shared Postgres deployment, keep the same loopback HTTP endpoint and run only one owner when possible. Remote endpoints require HTTPS and proper authentication; never put a bearer token directly in the plugin JSON.

## Offer optional persistent guidance

After the connection is healthy, explain that plugin-only mode needs no project-file change and remains the default. Offer, but never automatically install, the optional `corsoul-agents-setup` workflow for users who want Codex to apply recall and capture rules at the start of every task.

Do not treat plugin installation or connection as consent to edit `AGENTS.md`. If the user chooses strict mode, invoke the dedicated skill, show the exact target, managed block, and diff, and wait for explicit confirmation before writing.
