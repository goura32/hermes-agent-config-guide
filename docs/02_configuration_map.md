# 設定体系の全体図

## Hermes Agent の設定システム

Hermes Agent の設定システムは **4層の優先順位** で構成される。

```
┌─────────────────────────────────────────────┐
│  Layer 4: ソース内デフォルト                    │
│  (hermes_cli/config.py: DEFAULT_CONFIG)     │
│  どの層でも指定されない場合に適用                         │
├─────────────────────────────────────────────┤
│  Layer 3: 設定ファイル                         │
│  ~/.hermes/config.yaml (全設定)                  │
│  ~/.hermes/.env      (クレデンシャルのみ)         │
├─────────────────────────────────────────────┤
│  Layer 2: 環境変数                            │
│  各設定項目に対応する HERMES_* 環境変数             │
│  .env ファイル経由またはシェル環境                 │
├─────────────────────────────────────────────┤
│  Layer 1: CLI引数                             │
│  hermes --provider, hermes model   /  等         │
│  プロセス有効 (終了で消失)                            │
└─────────────────────────────────────────────┘
```

## 設定ファイルの役割

### config.yaml (~/.hermes/config.yaml)

**全設定**を管理する yamlファイル。以下を含む:

- model / providers / custom_providers
- auxiliary (vision, web_extract, compression等)
- terminal, web, browser
- display, dashboard, curator
- memory, sessions, cron, delegation
- approvals, hooks, security
- compression, prompt_caching, openrouter
- bedrock, tts, stt, voice
- slack, discord, telegram, whatsapp
- kanban, code_execution, logging, skills

### .env (~/.hermes/.env)

**クレデンシャルのみ**を管理する dotenvファイル。以下を含む:

- OPENROUTER_API_KEY, ANTHROPIC_API_KEY, GOOGLE_API_KEY
- OPENAI_API_KEY, XAI_API_KEY etc.
- Discord, Telegram, Slack の BOT_TOKEN
- SearXNG, Firecrawl, Tavily 等の API keys

## 環境変数の分類

| カテゴリ | 例 | 説明 |
|--|-|-|
| **HERMES_* 環境変数** | HERMES_HOME, HERMES_MODEL | Hermes固有設定 |
| **プロバイダー固有** | OPENROUTER_API_KEY, ANTHROPIC_API_KEY | 各種APIキー |
| **プラットフォーム固有** | DISCORD_TOKEN, TELEGRAM_TOKEN | メッセンジャー |
| **ツール固有** | SEARXNG_URL, BROWSERBASE_API_KEY | 特定ツール |
| **システム共通** | PYTHONPATH, PATH | OS環境 |

## スラッシュコマンドとの関係

スラッシュコマンドは config.yaml の値を一時的に上書きする。例えば:

- `/model google/gemini-3-flash-preview` → model.default を上書き
- `/provider openrouter` → model.provider を上書き
- `/yolo` → approvals.mode を "off" に
- `/compact` → compression をトリガー

これらの変更はプロセス単位で有効。次回以降は元の設定が適用される。

## runtime defaults の仕組み

1. `load_config()` が呼ばれると `DEFAULT_CONFIG` が読み込まれる
2. User config.yaml が読み込まれ、`_deep_merge()` によりマージ
3. マージ結果が `_expand_env_vars()` で展開
4. `migrate_config()` でバージョンマイグレーション
5. `_normalize_*()` で正規化
6. `_inject_profile_env_vars()` でプロファイル固有の env 変数を追加
7. `_inject_platform_plugin_env_vars()` でプラットフォーム固有の env 変数を追加

## 関連ファイル一覧

```
hermes_cli/
├── config.py          # DEFAULT_CONFIG, load_config, migrate_config
├── env_loader.py      # .env ファイル読み込み
├── commands.py        # CLI スラッシュコマンド
├── setup_wizard.py    # セットアップウィザード

hermes_constants.py    # HERMES_HOME 管理等

agent/
├── auxiliary_client.py # auxiliary モデル解決
├── models.py          # モデルアダプター
├── model_metadata.py  # 組み込みプロバイダー

tools/
├── web_tools.py       # web backend 解決
├── web_providers/     # SearXNG, Brave Free, DDG
├── browser_tool.py    # browser 設定
├── browser_providers/ # Browserbase, browser-use, Firecrawl
├── terminal_tool.py   # terminal backend
├── delegation_tool.py # subagent タイムアウト
└── cronjob_tools.py   # cron scheduler

cron/
├── jobs.py            # cron job 定義
└── scheduler.py       # cron スケジューラー

gateway/
├── run.py             # gateway ランナー
└── platforms/         # Telegram, Discord, Slack, WhatsApp

hermes_state.py        # セッションDB
model_tools.py         # tool discovery
toolsets.py            # toolset 定義
```
