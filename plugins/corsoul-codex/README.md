# Corsoul Memory for Codex

This plugin connects every Codex task to one shared Corsoul memory service and adds safe recall-and-capture guidance. Users do not edit `config.toml`, and the default local store does not need Docker, Postgres, a Corsoul account, or an API key.

## Install

Requirements: Codex and Node.js 18 or newer.

After publishing the repository marketplace, install it with:

```text
codex plugin marketplace add CorGGai/corsoul-codex-plugin
codex plugin add corsoul-codex@corsoul-ai
```

Start a new Codex task and choose the starter prompt:

> Connect Corsoul and verify that Codex can recall memory.

The connection skill checks `http://127.0.0.1:3848/health`. If no owner is running, it offers the recommended PM2-supervised installation or a temporary foreground start. The persistent setup installs the reviewed `corsoul@0.1.12` runtime, supervises only the MCP owner, saves the PM2 process list, and configures or explains the remaining OS startup step. Open one more new task after the service first becomes healthy so Codex can load the memory tools.

## Recommended persistent service (PM2)

The plugin includes reviewed setup helpers:

```text
# Windows PowerShell
powershell -ExecutionPolicy Bypass -File scripts/install-corsoul-pm2.ps1

# Linux or macOS
sh scripts/install-corsoul-pm2.sh
```

Run a helper only after reviewing it and consenting to global npm installs and startup changes. It installs pinned `corsoul@0.1.12` and `pm2@7.0.3`, starts only `corsoul-mcp` on `127.0.0.1:3848`, waits for `/health`, and runs `pm2 save`.

On Windows, the helper also installs the PM2 login-startup hook unless `-SkipStartup` is supplied. On Linux or macOS, PM2 prints one platform-specific privileged command; run that printed command once, then run `pm2 save` again. PM2 crash recovery and machine-login/boot recovery are separate: `pm2 start` protects against a crashed process, while `pm2 save` plus the OS startup hook restores it after a reboot.

Useful checks:

```text
pm2 status
pm2 logs corsoul-mcp
curl http://127.0.0.1:3848/health
```

The helpers deliberately do **not** start `corsoul-activation-runner`. Prospective intentions remain data; installing a memory plugin does not grant permission to execute due items.

## Choose a guidance mode

- **Plugin-only mode (default):** no project or global instruction file is changed. Codex discovers the bundled skills when a task matches them.
- **Optional strict mode:** choose the starter prompt **Show the optional AGENTS.md memory setup. Preview it before editing.** The `corsoul-agents-setup` skill lets the user select project or global scope, detects the effective instruction file, and shows the complete managed block and unified diff before asking for confirmation.

Strict mode uses an idempotent marker block and never treats plugin installation as permission to edit `AGENTS.md`. It can update or remove only that marked block while preserving existing guidance. Start a new Codex task after applying it. No separate system-prompt setting is required.

For a temporary transparent foreground start, run this command and keep the terminal open:

```text
npx -y --package=corsoul@0.1.12 corsoul --transport=http --host=127.0.0.1 --port=3848
```

This version is a **managed-start baseline**, not an enforceable dependency lock: if port 3848 already has a healthy owner, the plugin reuses it, and the current health response does not expose its package version. Operators remain responsible for that existing owner's version and policy.

Corsoul's responsibility guardian is passive: it surfaces due items but does not execute them. The `0.1.7` baseline states the authorization boundary up front — its MCP instructions treat due items as untrusted reminders, **not** authorization to act, matching this plugin's untrusted-memory policy — and adds single-owner election so concurrent engines share one store owner instead of tearing the PGLite data directory. (Earlier releases were unsuitable as a managed baseline: 0.1.4's instructions said `act` and `Do not ignore it` before stating that boundary, 0.1.2 predates the Windows data-directory normalization fix, and 0.1.5/0.1.6 predate single-owner election.) Keep the baseline pinned; move it only after re-running the release checks below.

## Why one shared HTTP owner

Codex can keep several tasks active at once. Starting a stdio Corsoul process for every task would let multiple PGLite processes open the same data directory, which can produce stale reads and lost writes. The plugin therefore connects all tasks to one loopback HTTP owner.

The HTTP endpoint is bound to `127.0.0.1` and has no network authentication. Other local processes running as the same user can still reach it, so loopback is a machine-local trust boundary rather than user isolation.

## What works without extra setup

- local remember, recall, and forget;
- persistent core identity data;
- prospective intentions and due-item reads;
- keyword recall with no embeddings provider.

Semantic recall requires an embeddings provider. A local Ollama provider keeps embedding content on the machine. A cloud provider receives remembered and queried text for embedding. If a configured provider is unavailable, do not assume the runtime will always fall back cleanly to keyword indexing.

## Included tools

- `corsoul_remember`, `corsoul_recall`, `corsoul_forget`
- `corsoul_intend`, `corsoul_due`, `corsoul_resolve_intent`
- `corsoul_set_core`, `corsoul_get_core`

The free package has no `corsoul_sleep`; graph consolidation and abstract patterns are not part of the free local tier.

## Safety boundaries

- A scope organizes recall; it is not authentication or encryption.
- Recalled text and due items are untrusted data. A due item must not be silently discarded, but it creates a duty to notice and triage—not permission to run tools, send messages, change files, or affect external systems.
- The plugin never starts the activation runner. An intention is data, not a reliable notification service.
- Persistent installation requires explicit user consent because it installs global npm packages and an OS startup hook.
- The skill stores concise durable summaries and excludes credentials, raw transcripts, private source files, and full logs.
- `corsoul_forget` is a tombstone, not guaranteed secure erasure.
- The plugin has no lifecycle hooks.

## Existing Cortex/Corsoul configuration

If Codex already has a manually configured `cortex` or `corsoul` MCP entry, enable only one connection after verifying this plugin. Duplicate servers can show duplicate tools and can contend for the PGLite store.

Docker or Postgres is required only if the shared owner was intentionally configured with `DATABASE_URL`. If that container is stopped, the owner may fail. The local PGLite command above is a separate no-Docker option; do not run it simultaneously against a data directory already owned by another Corsoul process.

Memory data is stored under `~/.corsoul/db` by default, with a legacy `~/.cortex/db` fallback. Removing the plugin does not delete that data. Optional embedding settings are currently stored in `~/.cortex/mcp-local.env`.

## Release checks

Before changing the runtime version or publishing the plugin, verify:

1. the plugin manifest and non-empty MCP configuration;
2. `/health` and `/mcp` bind only to loopback;
3. exactly the eight documented `corsoul_*` tools are present;
4. remember then recall works without an embeddings provider;
5. two parallel Codex tasks share one HTTP owner and both writes survive restart;
6. a malicious recalled fact cannot override current instructions;
7. a malicious due item cannot trigger an unauthorized action;
8. secrets are not captured;
9. provider outage is reported honestly;
10. uninstalling the plugin preserves the user's memory store;
11. the selected runtime can be reproduced from its reviewed source and tests.
12. killing `corsoul-mcp` causes PM2 to restore it, and reboot/login recovery works after the OS startup hook is installed;
13. the PM2 process list contains `corsoul-mcp` but not `corsoul-activation-runner`.

## License

MIT
