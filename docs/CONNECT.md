# Connect Corsoul to an agent

This page covers the public free package. Use stdio for one local client or one loopback HTTP server
when several local clients need the same PGLite store.

## Recommended for Codex: the plugin

```text
codex plugin marketplace add CorGGai/corsoul-codex-plugin
codex plugin add corsoul-codex@corsoul-ai
```

Start a new Codex task and choose **Connect Corsoul and verify that Codex can recall memory**. The
plugin connects all Codex tasks to one shared `http://127.0.0.1:3848/mcp` owner; it does not create a
separate PGLite process for every task. See the [plugin README](../plugins/corsoul-codex/README.md)
for startup, privacy, managed-start baseline, and migration details.

For the recommended persistent owner, use the plugin's platform helper after reviewing it and
consenting to global npm installation and startup changes:

```text
# Windows PowerShell
powershell -ExecutionPolicy Bypass -File plugins/corsoul-codex/scripts/install-corsoul-pm2.ps1

# Linux or macOS
sh plugins/corsoul-codex/scripts/install-corsoul-pm2.sh
```

The helpers pin the reviewed runtime, supervise only `corsoul-mcp`, run `pm2 save`, and never start
`corsoul-activation-runner`. On Linux or macOS, finish the one-time `pm2 startup` command printed by
PM2. Crash restart and reboot restart are separate; the latter requires both the saved process list
and the platform startup hook.

Plugin-only mode is the default and does not edit project files. To make recall/capture guidance
persistent on every task, choose **Show the optional AGENTS.md memory setup. Preview it before
editing.** The dedicated skill offers project, global, or no-change modes and requires a complete
preview plus explicit confirmation before modifying the effective Codex instruction file.

## Connector for isolated clients

```text
npx -y --package=corsoul@0.1.21 corsoul connect codex --scope=myapp:assistant:v1
```

Run the command from the project that should receive the host contract. Codex writes or merges
`AGENTS.md` in the current directory; Claude Code and Cursor also default to project-level rules.
Start with `--dry-run`, or use `--no-contract` when you only want MCP configuration.

Supported targets are `codex`, `claude-code`, `cursor`, `cline`, `claude-desktop`, and `openclaw`.
Useful options:

- `--scope=<id>` — stable memory namespace written into the host contract.
- `--dry-run` — preview without writing.
- `--print` — print configuration for manual use.
- `--no-contract` — do not write host instructions.
- `--global` — use a global location where that target supports one; it does not make every target's
  contract global.
- `--data-dir=<path>` — give this connection its own PGLite store.

The connector backs up supported configuration before merging. Current 0.1.x releases intentionally
use the MCP server key `cortex` for backward compatibility even though the command and tool names are
Corsoul-branded. Do not add a second `corsoul` entry beside it; two stdio servers pointed at the same
PGLite directory can conflict.

For Codex Desktop, the repository plugin above is safer than this stdio connector because multiple
active tasks may launch multiple copies of a plugin-provided or manually configured stdio server.

## Claude Code plugin

```text
/plugin marketplace add CorGGai/corsoul-plugin
/plugin install corsoul-memory@corsoul
```

The plugin provides the free MCP server and a tool-driven memory skill. It does not guarantee that
every turn is captured; the model still chooses when to call tools.

## Manual Codex stdio configuration (single process only)

Edit `~/.codex/config.toml` (Windows: `%USERPROFILE%\.codex\config.toml`):

```toml
[mcp_servers.cortex]
command = "npx"
args = ["-y", "--package=corsoul", "corsoul"]
```

On Windows, use `command = "npx.cmd"` if `npx` is not resolved by the host. Restart Codex after
editing. The primary tools appear as `corsoul_*`; legacy `cortex_*` tools are registered only when
`CORSOUL_COMPAT_TOOLS=1` is explicitly set. Do not use this shape for concurrent Codex tasks that
share one PGLite directory.

## One shared local store: loopback HTTP

For a temporary foreground owner, run exactly one server process:

```text
npx -y --package=corsoul@0.1.21 corsoul --transport=http --host=127.0.0.1 --port=3848
```

Then write the Codex entry with the engine's own writer; no `mcp-remote` bridge is required:

```text
npx -y --package=corsoul@0.1.21 corsoul connect http --url=http://127.0.0.1:3848/mcp --config=~/.codex/config.toml --label=codex --scope=<your scope>
```

This writes `[mcp_servers.cortex]` with `url` and `http_headers` (the channel label and your scope);
re-running it keeps anything you added by hand.

The free HTTP server has no authentication. It is for same-machine loopback use only. Do not bind it
to a LAN address, open it in a firewall, tunnel it publicly, or set an unsafe override for routine
use.

## Licensed remote endpoint

If a Corsoul operator gives you a licensed endpoint and token, use HTTPS and keep the token in an
environment variable rather than command arguments or a committed file:

```toml
[mcp_servers.corsoul_cloud]
url = "https://brain.example/mcp"
bearer_token_env_var = "CORSOUL_MCP_TOKEN"
```

The endpoint, certificate, token, and permitted scopes must come from the operator. A `scope_id` by
itself is not access control.

## Verify

```text
npx -y --package=corsoul@0.1.21 corsoul doctor
```

After restarting the client, remember and recall one fact using the same scope. This round trip—not
`doctor` alone—confirms an external Postgres connection. If multiple local clients share one store,
confirm that only the loopback HTTP process owns the PGLite directory.

## Common failures

| Symptom | Check |
|---|---|
| Client starts two memory servers | Remove the duplicate entry; keep the connector-managed `cortex` key |
| `npx` is not found on Windows | Use `npx.cmd` or install `corsoul` globally and use `command = "corsoul"` |
| Recall fails after provider setup | Run `corsoul doctor`; restore the endpoint or follow the [existing-installation keyword-only steps](QUICKSTART.md#2-choose-recall-mode) |
| MCP starts but memory calls fail with `DATABASE_URL` | Check the Postgres/Docker process, published port, credentials, network, pgvector availability, and schema/extension permissions; then run a remember/recall round trip |
| Memories appear missing | Confirm every call uses the same stable scope and the same data directory |
| Port 3848 is unavailable | Stop the old local server or select another loopback port and update every client |
