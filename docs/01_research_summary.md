# 調査サマリー

## 調査対象

- **プロジェクト**: Hermes Agent (NousResearch)
- **URL**: https://github.com/NousResearch/hermes-agent
- **調査時点のコミット**: `7d66d30` (2026-05-08, main branch)
- **config schema version**: 23
- **設定ファイル**: `~/.hermes/config.yaml` (全設定), `~/.hermes/.env` (クレデンシャル)

## 調査したソースコード

### 主要設定ファイル

| ファイル | 目的 | 行数 (近似) |
|--|-|--|
| `hermes_cli/config.py` | DEFAULT_CONFIG定義, 読み込み/保存, マイグレーション, 環境変数展開 | ~5200行 |
| `hermes_cli/env_loader.py` | .envファイル読み込み (優先順位: user > project) | ~175行 |
| `hermes_constants.py` | HERMES_HOMEパス, プロファイル, イオ/WSL検出, env定数 | ~350行 |
| `run_agent.py` | AIAgentクラスの初期化, プロバイダー解決 | ~12000行 |
| `agent/models.py` | モデルアダプター, クレデンシャル解決 | --- |
| `agent/auxiliary_client.py` | auxiliaryモデル解決 (auto/main/custom) | ~182,000行 |
| `agent/model_metadata.py` | 組み込みプロバイダー定義, モデルカタログ | --- |
| `hermes_cli/commands.py` | CLIスラッシュコマンド定義 | ~72,000行 |
| `hermes_cli/setup_wizard.py` | セットアップウィザード | --- |

### トール関連ファイル

| ファイル | 内容 |
|--|-|
| `tools/web_tools.py` | web_search, web_extractのバックエンド解決 |
| `tools/browser_tool.py` | browser tool の provider 解決, CDP, agent-browser |
| `tools/terminal_tool.py` | terminal backend (local/docker/ssh/modal等) |
| `tools/approval.py` | approval mode (manual/smart/off) |
| `tools/tts_tool.py` | TTS プロバイダー (edge/elevenlabs/openai/xai等) |
| `tools/transcription_tools.py` | STT プロバイダー (local/groq/openai/mistral) |
| `tools/voice_mode.py` | voice mode (録音/TTS) |
| `tools/vision_tools.py` | vision model 解決 |
| `tools/cronjob_tools.py` | cron scheduler |
| `tools/delegate_tool.py` | delegation 設定 |
| `tools/kanban_tools.py` | kanban 設定 |
| `hermes_state.py` | セッションDB |
| `model_tools.py` | tool discovery, function call |
| `toolsets.py` | toolset定義 |

### プロバイダー関連ファイル

| ファイル | 内容 |
|--|-|
| `tools/web_providers/searxng.py` | SearXNG web backend |
| `tools/web_providers/brave_free.py` | Brave Free web backend |
| `tools/web_providers/ddgs.py` | DuckDuckGo web backend |
| `tools/browser_providers/base.py` | browser provider interface |
| `tools/browser_providers/browserbase.py` | Browserbase cloud browser |
| `tools/browser_providers/browser_use.py` | browser-use cloud browser |
| `tools/browser_providers/firecrawl.py` | Firecrawl cloud browser |

### ゲートウェイ/プラットフォーム

| ファイル | 内容 |
|--|-|
| `gateway/run.py` | ゲートウェイランナー |
| `gateway/platforms/telegram.py` | Telegram platform |
| `gateway/platforms/discord.py` | Discord platform |
| `gateway/platforms/slack.py` | Slack platform |
| `gateway/platforms/whatsapp.py` | WhatsApp platform |
| `gateway/platforms/base.py` | platform base class |

## ソースコード上の重要発見

### 1. config.py の キャッシュメカニズム

`load_config()` は `_LOAD_CONFIG_CACHE` にファイルの `(mtime_ns, size)` をキーにして結果をキャッシュする。ファイルが更新されると自動的に再読み込みされる（atomic_yaml_writeがインODEを更新するため）。

### 2. .env の 読み込み順序

```
1. システム環境 (OS環境変数, 最初から存在)
2. ~/.hermes/.env       (ユーザーの.env, override=True)
3. プロジェクト.env       (開発フォールバック, 最初の.envが存在しない場合はoverride)
```

### 3. クレデンシャル自動検出

名前が `_API_KEY`, `_TOKEN`, `_SECRET`, `_KEY` で終わる環境変数がクレデンシャルとして判定される。これらの値のみが非ASCII文字のサニタイジング対象となる。

### 4. custom_providers の 変遷

version 12 でのマイグレーション:
- 旧形式: `custom_providers` (list形式) → 新形式: `providers` (dict形式)
- custom_providers.name → display_name
- custom_providers.base_url → url (or api)
- custom_providers.api_mode → transport
- custom_providers.model → default_model

### 5. モデル解決の優先順位

```
1. CLI: /model <value> or --provider <value>
2. 環境変数: HERMES_MODEL, HERMES_INFERENCE_PROVIDER
3. config.yaml: model.default, model.provider
4. プレビュー/自動検出 (model_metadata.py)
```

### 6. auxiliary モデル解決

- `provider: auto` → 利用可能な最初のプロバイダーを自動選択
- `provider: main` → メインチャットモデルを使用
- `provider: custom` → 任意の名前付きプロバイダー
- どのauxiliaryタスクもフォールバック: `openrouter:google/gemini-3-flash-preview`

### 7. config schema version

現在の最新は 23。マイグレーションパス: 3→4→5→8→9→11→12→13→14→15→16→17→20→21→22→23。

## ドキュメントとの差異

### 未確認点（調査中に発見）

1. `HERMES_STREAM_READ_TIMEOUT`, `HERMES_STREAM_STALE_TIMEOUT` の実装位置は未確認
2. `HERMES_API_TIMEOUT`, `HERMES_API_CALL_STALE_TIMEOUT` の実装位置は未確認
3. `HERMES_AGENT_TIMEOUT`, `HERMES_AGENT_NOTIFY_INTERVAL` の実装位置は未確認

これらは `hermes_cli/config.py` の OPTIONAL_ENV_VARS には記載されていない。公式ドキュメントでの言及に依存する必要がある。

## 公式ドキュメント調査

### ドキュメント構造 (website/docs/)

```
developer-guide/   (15 files) - 内部アーキテクチャ
getting-started/   (5 files)  - インストール/クイックスタート
guides/            (14 files) - 運用チュートリアル
integrations/      (2 files)  - 統合
reference/         (10 files) - リファレンス
user-guide/        (62 files) - ユーザー向けガイド
messaging/         (13 files) - メッセージング統合
```

### 設定関連の主要ドキュメント

| ドキュメント | 説明 |
|--|-|
| `user-guide/configuration.md` | 設定ファイルの基本 |
| `user-guide/configuring-models.md` | モデル/プロバイダー設定 |
| `reference/environment-variables.md` | 全環境変数 |
| `reference/cli-commands.md` | CLIコマンド |
| `reference/slash-commands.md` | スラッシュコマンド |
| `user-guide/features/provider-routing.md` | プロバイダールーティング |
| `user-guide/features/fallback-providers.md` | フェールバックプロバイダー |
| `user-guide/features/web-search.md` | web_search/browsing |
| `user-guide/features/browser.md` | browser tool |
| `user-guide/features/cron.md` | cronジョブ |
| `user-guide/features/delegation.md` | サブエージェント |
| `user-guide/features/hooks.md` | スクリプトフックス |
| `user-guide/features/memory.md` | メモリ |
| `user-guide/features/memory-providers.md` | 外部メモリプロバイダー |
| `user-guide/features/kanban.md` | kanban |
| `user-guide/features/skills.md` | スキル |
| `user-guide/features/skins.md` | スキン |
| `user-guide/features/tools.md` | ツール |
| `user-guide/features/mcp.md` | MCP |
| `integrations/providers.md` | サポート一覧 |
| `user-guide/sessions.md` | セッション管理 |
