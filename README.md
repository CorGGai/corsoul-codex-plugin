# Corsoul

**Local-first long-term memory for AI agents.**

Corsoul gives an agent durable memory across sessions. The public `corsoul` package is a useful,
MIT-licensed **L0/L1 objective-memory client**: raw events, structured facts, prospective intents,
and a settable agent core. The licensed system adds L2/L3 associative learning, consolidation, and
bounded personality capabilities.

- Website: [corsoul.com](https://corsoul.com)
- npm: [`corsoul`](https://www.npmjs.com/package/corsoul)
- Codex plugin: [Corsoul Memory for Codex](plugins/corsoul-codex)
- Claude Code plugin: [CorGGai/corsoul-plugin](https://github.com/CorGGai/corsoul-plugin)

## Start free

Requires Node.js 22 or newer.

For Codex, install the repository plugin instead of creating one stdio/PGLite process per task:

```text
codex plugin marketplace add CorGGai/corsoul-codex-plugin
codex plugin add corsoul-codex@corsoul-ai
```

Then start a new task and choose **Connect Corsoul and verify that Codex can recall memory**. The
plugin connects every task to one loopback HTTP owner and supplies safe scope, capture, privacy, and
prospective-memory rules. When the owner is not running, the plugin offers an explicit-consent PM2
setup that supervises only `corsoul-mcp`, saves the process list, and configures or explains the OS
startup hook. See the [plugin README](plugins/corsoul-codex/README.md).

The default plugin-only mode does not edit the repository. Users who want persistent rules on every
task can choose **Show the optional AGENTS.md memory setup. Preview it before editing.** The plugin
will show the target file, managed block, and diff, then wait for confirmation.

For other clients or an intentionally isolated single process, the CLI connector remains available:

```text
npx -y --package=corsoul@0.1.21 corsoul setup
npx -y --package=corsoul@0.1.21 corsoul connect cursor --scope=myapp:assistant:v1
npx -y --package=corsoul@0.1.21 corsoul doctor
```

`setup` lets you choose local Ollama, OpenAI, another OpenAI-compatible embedding endpoint, or
keyword-only recall. `connect` backs up and merges the selected client's MCP configuration. Restart
the client after connecting. Codex users should prefer the plugin because Codex can keep multiple
tasks active concurrently.

In current 0.1.x releases, selecting keyword-only does not erase provider values saved by an earlier
setup. Follow the [switching instructions](docs/QUICKSTART.md#2-choose-recall-mode) when changing an
existing installation.

The free MCP server exposes ten tools:

`corsoul_remember` · `corsoul_recall` · `corsoul_forget` · `corsoul_intend` · `corsoul_due` ·
`corsoul_resolve_intent` · `corsoul_set_core` · `corsoul_get_core` · `corsoul_peek` · `corsoul_copy`

The public Node SDK is equally small:

```ts
import { scope } from "corsoul"

const memory = scope("myapp:assistant:user-42")
await memory.remember("Prefers Traditional Chinese", { type: "note" })
const result = await memory.recall("Which language does the user prefer?")
```

## Privacy boundary

Corsoul is **local-first**, not automatically “data never leaves the device.”

- Memory is stored in local PGLite by default, or in a reachable Postgres with pgvector that you
  control.
- Keyword-only recall and local Ollama can run without sending memory text to a cloud provider.
- When you configure cloud embeddings, text submitted for embedding is sent to that provider.
- Licensed consolidation can send a scoped working set to the configured Corsoul brain for
  processing while the authoritative store remains local.

Do not store passwords, API keys, private keys, access tokens, or unredacted regulated data in
agent memory. See [Privacy and security](docs/PRIVACY-AND-SECURITY.md).

## Product surfaces

| Surface | Distribution | What it contains |
|---|---|---|
| `corsoul` | Public npm, MIT | Free local L0/L1 memory, MCP, CLI, and Node SDK |
| `@corsoul/client` | Delivered to licensed customers | Cloud-capable client and licensed consolidation transport; no engine algorithms |
| Corsoul brain | Private beta / licensed deployment | L2/L3 engine, consolidation, and personality capabilities |

`@corsoul/client` is not a public npm install, and the public `corsoul` package is not the direct
paid-to-free downgrade target. Paid-plan names and capabilities are beta labels; fixed public
pricing has not been announced.

## Documentation

- [Five-minute quickstart](docs/QUICKSTART.md)
- [Connect an agent](docs/CONNECT.md) · [繁體中文](docs/CONNECT.zh-TW.md)
- [Free Node SDK](docs/SDK.md)
- [Using Corsoul safely](docs/USING-CORSOUL-AGENT.en.md) · [繁體中文](docs/USING-CORSOUL-AGENT.md)
- [Tiers and availability](docs/TIERS.md)
- [Privacy and security](docs/PRIVACY-AND-SECURITY.md)
- [Public-repository boundary](docs/PUBLICATION-BOUNDARY.md)

## Repository scope

This repository is the public documentation and issue-tracking entry point for Corsoul. It does not
contain the proprietary engine, licensed-client artifacts, operator runbooks, internal prompts,
private topology, or credentials. See [PUBLICATION-BOUNDARY.md](docs/PUBLICATION-BOUNDARY.md) before
contributing.

Report security issues privately as described in [SECURITY.md](SECURITY.md), not in a public issue.

## 中文摘要

Corsoul 是 AI Agent 的本機優先長期記憶。公開免費版提供可正式使用的 L0/L1 客觀記憶、
前瞻意圖、MCP、CLI 與 Node SDK；L2/L3 關聯學習、睡眠整合與人格能力屬授權系統。
「本機優先」表示持久資料預設留在本機或自有 Postgres，不代表使用雲端 embeddings 或
授權整合時資料絕不離機。

## License

The public documentation in this repository and the public `corsoul` package are MIT licensed.
Corsoul's licensed client and cognitive engine are proprietary and are not included here.
