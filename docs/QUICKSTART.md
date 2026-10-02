# Corsoul free — five-minute quickstart

This guide covers the public `corsoul` package only. It does not require a source checkout, Docker,
a license, or the proprietary engine.

## 1. Check the prerequisite

Install Node.js 22 or newer, then verify:

```text
node --version
npm --version
```

## 2. Choose recall mode

```text
npx -y corsoul setup
```

Choose one of:

1. Ollama on the same machine — local semantic recall.
2. OpenAI embeddings — memory text sent to OpenAI for embedding.
3. Another OpenAI-compatible embedding endpoint — handling depends on that provider.
4. Keyword-only — no embedding provider and no semantic matching.

An Anthropic API key alone does **not** provide embeddings. The current 0.1.x CLI keeps provider
settings in `~/.cortex/mcp-local.env` for backward compatibility (Windows:
`%USERPROFILE%\.cortex\mcp-local.env`). Environment variables supplied by the MCP client take
precedence.

In current 0.1.x releases, option 4 does not clear values saved by an earlier setup. To switch an
existing installation to keyword-only, first back up `mcp-local.env`, then remove its provider
entries (`LLM_BASE_URL`, `LLM_EMBED_BASE_URL`, `LLM_EMBED_MODEL`, `LLM_EMBED_DIM`, `LLM_API_KEY`,
`LLM_EMBED_API_KEY`, and `OPENAI_API_KEY`). Remove the same variables from the MCP client's
environment block, restart the client/server, and run `corsoul doctor`. This does not require deleting
the memory store.

Do not edit a repository `.env` file with real credentials. If you need environment-variable
examples, use [`config/corsoul.env.example`](../config/corsoul.env.example) and keep the real file
outside Git.

## 3. Connect an agent

Codex users should install the repository plugin and use its PM2 helper for one persistent shared
HTTP owner. This avoids launching one PGLite process per active Codex task and restores the owner
after crashes and, once the OS startup hook is installed, after reboot/login. See
[CONNECT.md](CONNECT.md#recommended-for-codex-the-plugin). The helper never starts the activation
runner.

Change to the project that should receive the agent instructions, then pick one stable scope for the
same agent or subject:

```text
npx -y corsoul connect codex --scope=myapp:assistant:v1
```

Other supported targets:

```text
npx -y corsoul connect claude-code --scope=myapp:assistant:v1
npx -y corsoul connect cursor --scope=myapp:assistant:v1
npx -y corsoul connect cline --scope=myapp:assistant:v1
npx -y corsoul connect claude-desktop --scope=myapp:assistant:v1
npx -y corsoul connect openclaw --scope=myapp:assistant:v1
```

Preview without writing:

```text
npx -y corsoul connect codex --scope=myapp:assistant:v1 --dry-run
```

The connector can write host instructions in the current project. For Codex, that is `AGENTS.md`;
Claude Code and Cursor also default to project-level rule files. Use `--no-contract` if you only want
the MCP configuration. Use `--global` only for clients that support a global rule/config location,
and inspect `--dry-run` output before either choice.

Restart the target client after connection. See [CONNECT.md](CONNECT.md) for manual configuration,
loopback HTTP, and Windows notes.

## 4. Verify

```text
npx -y corsoul doctor
```

`doctor` validates the configured store mode and embedding provider, but it does not open an external
`DATABASE_URL`. The remember/recall round trip below is the database connectivity check.

Then ask the connected agent to perform a round trip with the same scope:

1. `corsoul_remember`: store “I prefer releases on Thursday.”
2. `corsoul_recall`: query “When do I prefer releases?”
3. Start a new session and recall again.

The free tier returns `facts` plus a confidence value. `related` and `patterns` remain empty because
L2/L3 are licensed-engine capabilities, not free-tier features.

## Important operating limits

- **PGLite is single-process.** Do not let several stdio servers open the same data directory.
  Use separate data directories or one loopback HTTP server as the sole owner.
- **External Postgres needs pgvector.** The server must be reachable, the `vector` extension must be
  available, and the database role must be allowed to create/use the required schema and extension.
  If Postgres runs in Docker, confirm the container, published port, and volume are healthy. A stopped
  container can make memory calls fail even when `corsoul doctor` reports the configured external URL.
- **Free HTTP has no authentication.** Keep it on `127.0.0.1`; do not expose it through a firewall
  or bind it to a LAN address.
- **A configured embedding provider is a live dependency.** If it is unavailable, recall may fail
  and a newly stored event may not yet be quick-indexed. Run `corsoul doctor`, restore the provider,
  or follow the existing-installation keyword-only steps above.
- **Scope is a namespace, not authorization.** A stable scope prevents accidental fragmentation;
  only server-enforced credentials create a security boundary.

## Uninstalling

Removing the npm package does not remove memory data or client configuration.

- Remove the package with npm if you installed it globally.
- Remove the MCP entry and the memory contract from each connected client.
- Review and separately remove the configured data directory only when you intentionally want to
  erase the local store and have made any required backup.

The default data directory is `~/.corsoul/db` (Windows: `%USERPROFILE%\.corsoul\db`). For continuity,
current 0.1.x releases reuse a legacy `~/.cortex/db` when it exists and the new default does not. Run
`corsoul doctor` to confirm the actual store before removing or moving data.
