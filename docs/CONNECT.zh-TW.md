# 把 Corsoul 接到 Agent

本頁只說明公開免費版 `corsoul`。單一客戶端優先使用 stdio；多個本機客戶端需要共用
同一份 PGLite 時，才使用一個 loopback HTTP server 作為唯一資料庫擁有者。

## Codex 推薦：安裝專用外掛

```text
codex plugin marketplace add CorGGai/corsoul-codex-plugin
codex plugin add corsoul-codex@corsoul-ai
```

安裝後開一個新的 Codex 任務，選擇 **Connect Corsoul and verify that Codex can recall memory**。
此外掛讓所有 Codex 任務連到同一個 `http://127.0.0.1:3848/mcp` owner，不會讓每個任務各自
開啟一個 PGLite process。啟動、隱私、受審核啟動基線與舊設定遷移請見
[外掛 README](../plugins/corsoul-codex/README.md)。

建議使用外掛內附的 PM2 安裝腳本建立常駐 owner；執行前應先審閱，並明確同意安裝全局 npm
套件及修改開機啟動設定：

```text
# Windows PowerShell
powershell -ExecutionPolicy Bypass -File plugins/corsoul-codex/scripts/install-corsoul-pm2.ps1

# Linux 或 macOS
sh plugins/corsoul-codex/scripts/install-corsoul-pm2.sh
```

腳本固定使用已審核的 Corsoul 版本，只守護 `corsoul-mcp`，執行 `pm2 save`，而且不會啟動
`corsoul-activation-runner`。Linux／macOS 還需按 PM2 畫面指示完成一次 `pm2 startup`。程序崩潰
自動拉起與重開機後恢復是兩件事；後者必須同時完成 `pm2 save` 和作業系統 startup hook。

預設的 Plugin-only 模式不會修改專案文件。若希望每個任務都載入 recall/capture 規則，請選
**Show the optional AGENTS.md memory setup. Preview it before editing.** 專用 skill 會提供專案、
全局或不變更三種選擇，先顯示完整 block 與 diff，取得明確確認後才修改實際生效的 Codex
指令文件。

## 隔離式單一 client：CLI 連接器

```text
npx -y corsoul connect codex --scope=myapp:assistant:v1
```

支援 `codex`、`claude-code`、`cursor`、`cline`、`claude-desktop` 與 `openclaw`。

- `--scope=<id>`：固定、假名化的記憶命名空間。
- `--dry-run`：只預覽，不寫檔。
- `--print`：印出設定供手動使用。
- `--no-contract`：不寫入 host instructions。
- `--data-dir=<path>`：讓該連線使用獨立的 PGLite 資料夾。

0.1.x 的 connector 為了向後兼容，仍使用 MCP server key `cortex`。不要再手動新增第二個
`corsoul` entry；兩個 stdio server 同時開同一個 PGLite 資料夾可能衝突。

Codex Desktop 可以同時保留多個任務，因此建議使用上面的 repository plugin，而不是讓
每個任務各啟一個 stdio server。

## Claude Code 外掛

```text
/plugin marketplace add CorGGai/corsoul-plugin
/plugin install corsoul-memory@corsoul
```

外掛提供免費 MCP server 與工具驅動的記憶 skill，但不保證每一輪都一定捕獲；模型仍會
自行決定是否呼叫工具。

## 手動設定 Codex stdio（只限單一 process）

編輯 `~/.codex/config.toml`；Windows 路徑是 `%USERPROFILE%\.codex\config.toml`：

```toml
[mcp_servers.cortex]
command = "npx"
args = ["-y", "--package=corsoul", "corsoul"]
```

Windows 若找不到 `npx`，改用 `command = "npx.cmd"`。儲存後重啟 Codex。多個 Codex 任務
若共用同一個 PGLite 資料夾，請勿使用此設定。

## 多個本機 client 共用記憶

若只需臨時前台服務，只啟動一個 loopback server：

```text
npx -y corsoul --transport=http --host=127.0.0.1 --port=3848
```

目前 Codex 可原生連 Streamable HTTP，不需要 `mcp-remote`：

```toml
[mcp_servers.cortex]
url = "http://127.0.0.1:3848/mcp"
```

免費 HTTP server **沒有驗證**，只可留在同機 loopback。不要綁 LAN 位址、開防火牆、
公開 tunnel，或把 unsafe override 當作日常配置。

## 授權遠端端點

若 operator 提供授權端點與 token，遠端一律使用 HTTPS，並從環境變數讀 token：

```toml
[mcp_servers.corsoul_cloud]
url = "https://brain.example/mcp"
bearer_token_env_var = "CORSOUL_MCP_TOKEN"
```

端點、憑證、token 與 scope 權限必須由 operator 提供。`scope_id` 自身不是存取控制。

## 驗證

```text
npx -y corsoul doctor
```

重啟 client 後，用同一 scope 做一次 remember → recall，再開新 session 召回一次。若多個
本機 client 共用資料，確認只有那一個 loopback HTTP process 開啟 PGLite。

英文完整版本：[CONNECT.md](CONNECT.md)。
