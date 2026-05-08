# はじめに — Hermes Agent 設定解説書

## 本書の目的

本資料は、Hermes Agent (https://github.com/NousResearch/hermes-agent) の全設定項目を網羅的に解説した技術書です。
最新ソースコードと公式ドキュメントの双方向検証に基づき、設定の優先順位・既定値・省略時挙動・相互依存関係を詳細に文書化しています。

Hermes Agent は、LLM をフロントエンドとした自律型 AI エージェントプラットフォームです。CLI, Gateway, メッセンジャープラットフォーム (Telegram, Discord, Slack, WhatsApp 等) からアクセス可能で、web 検索, ブラウジング, コード実行, スキル, cron 等豊富な機能を備えます。

設定は約 230 を超えるサブプロパティを持ち、CLI, 環境変数, config.yaml, ソース内の既定値の 4 層で構成されます。本資料はこれらの理解を助けるために作成されました。

## 対象読者

- **システム管理者**: Hermes Agent のセットアップ・デプロイ
- **運用者**: daily usage, troubleshooting, customization
- **開発者**: インテグレーション, プラグイン, フック
- **初心者**: 初めて Hermes Agent 触れる方

## 本書の利点

1. **網羅性**: 230 を超える設定項目を網羅
2. **正確性**: ソースコードベースの事実 + ドキュメントからの記載
3. **実践的**: 設定例, トラブルシューティング, よくある誤解
4. **検索しやすい**: 設定索引, 環境変数索引

## 読み方

1. **はじめての方**: 本書の目的 → docs/01_research_summary.md → docs/02_configuration_map.md への順に参照
2. **特定の設定を探している場合**: docs/14_reference_tables.md の索引から検索
3. **設定の優先順位を知りたい場合**: docs/03_setting_precedence.md 参照
4. **トラブルシューティング**: docs/13_troubleshooting.md 参照

## 対象バージョン

| 項目 | 値 |
|--|--|
| Hermes Agent | v0.12.0 (unreleased) |
| コミット | `7d66d30` (2026-05-08) |
| config schema version | 23 |
| GitHub | https://github.com/NousResearch/hermes-agent |
| 公式ドキュメント | https://hermes-agent.nousresearch.com/docs |

## 本書の構成

| 章 | ファイル | 内容 |
|---|--|-----|
| 0 | `docs/00_overview.md` | 本書の目的・構成・読み方 |
| 1 | `docs/01_research_summary.md` | 調査サマリー・ソース列記 |
| 2 | `docs/02_configuration_map.md` | 設定体系の全体図 |
| 3 | `docs/03_setting_precedence.md` | 設定の優先順位 |
| 4 | `docs/04_model_and_providers.md` | model / providers / custom_providers |
| 5 | `docs/05_auxiliary_models.md` | auxiliary モデル（全10タスク） |
| 6 | `docs/06_web_browser_tools.md` | web / browser 設定 |
| 7 | `docs/07_terminal_tools_approvals.md` | terminal / approvals / code_execution |
| 8 | `docs/08_gateway_channels.md` | gateway / Discord / Telegram / Slack / WhatsApp |
| 9 | `docs/09_memory_sessions_curator.md` | memory / sessions / curator / delegation |
| 10 | `docs/10_display_personality_voice.md` | display / personality / skin / voice / TTS / STT |
| 11 | `docs/11_cron_hooks_automation.md` | cron / kanban / hooks |
| 12 | `docs/12_security_operations.md` | security / privacy / checkpoints |
| 13 | `docs/13_troubleshooting.md` | トラブルシューティング |
| 14 | `docs/14_reference_tables.md` | 設定項目リファレンス表 |
| 15 | `docs/15_appendix_source_notes.md` | 付録（ソースコード根拠、ドキュメント差分） |

## 免責任事項

- 本書はコミット `7d66d30` 時点の情報に基づきます。
- 将来のバージョンでは変更されている可能性があります。
- 本資料の記載に起因する直接・間接的な損害について一切責任を負いません。
- 本資料は情報提供のみを目的とし、公式ドキュメントの代替ではありません。

## 調査範囲

### ソースコードの調査

| ファイル | 役割 | 記載行数 |
|--|--|--|
| `hermes_cli/config.py` | DEFAULT_CONFIG, loading, migration | ~5200 |
| `hermes_cli/env_loader.py` | .env file loading | ~175 |
| `hermes_constants.py` | environment variables | **175行**
| `hermes_cli/setup_wizard.py` | setp wizard | --- |
| `hermes_cli/commands.py` | CLI slash commands | --- |
| `agent/auxiliary_client.py` | auxiliary model resolution | ~182515 |
| `agent/model_metadata.py` | built-in providers | --- |
| `tools/web_tools.py` | web_search, web_extract | --- |
| `tools/browser_tool.py` | browser tool | --- |
| `tools/terminal_tool.py` | terminal backend | --- |
| `tools/approval.py` | approval logic | --- |
| `tools/tts_tool.py` | TTS | --- |
| `tools/transcription_tools.py` | STT | --- |

### 公式ドキュメントの調査

| カテゴリ | ファイル数 | 重要なファイル |
|--|--|-- |
| developer-guide | 15 | architecture.md, gateway-internals.md, prompt-assembly.md |
| getting-started | 5 | installation.md, quickstart.md |
| guides | 14 | automation-templates.md, tips.md, local-ollama-setup.md |
| reference | 10 | environment-variables.md, model-catalog.md, cli-commands.md |
| user-guide | 62 | configuration.md, configuring-models.md, sessions.md |
| integrations | 2 | providers.md |
| messaging | 13 | telegram.md, discord.md, slack.md, whatsapp.md |
| **合計** | **121** | |

### 設定体系の概要

Hermes Agent の設定は以下の 4 層で構成されます。

```
 Layer 4: source defaults (DEFAULT_CONFIG)
    ↓ deep_merge
 Layer 3: config.yaml (~/.hermes/config.yaml)
    ↓ ${VAR} expansion
 Layer 2: environment variables (OS + .env)
    ↓ CLI overrides
 Layer 1: CLI arguments (hermes --provider, /model, etc.)
```

各層の詳細は `docs/02_configuration_map.md` と `docs/03_setting_precedence.md` を参照してください。

## 環境

| 項目 | 値 |
|--|--|
| OS | Linux |
| Python | --- |
| Hermes Agent source | `source/hermes-agent/` |
| 作業ディレクトリ | `hermes-agent-config-guide/` |
