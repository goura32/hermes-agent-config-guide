# Hermes Agent 設定解説書

> Hermes Agent (https://github.com/NousResearch/hermes-agent) の全設定項目を網羅的に解説した商用出版レベルの技術書。

## 対象バージョン

| 項目 | 値 |
|---|---|
| **Hermes Agent** | v0.12.0 (unreleased, main branch) |
| **コミット** | [`7d66d30`](https://github.com/NousResearch/hermes-agent/commit/7d66d30) (2026-05-08) |
| **config schema version** | 23 |

## 本書の目的

本書は Hermes Agent の設定システムを完全に理解し、正しく運用するために作成された技術書です。

- **最新ソースコードの双方向検証**: 公式ドキュメントだけでなく、`hermes_cli/config.py` (DEFAULT_CONFIG, ~5200行) と `hermes_cli/env_loader.py` を直接検証
- **設定の優先順位体系**: config.yaml → .env → 環境変数 → CLI引数の4層を詳細に文書化
- **設定項目の網羅**: 230 を超えるサブプロパティを全カバー
- **実用的な設定例**: OpenRouter, ローカルOllama, CloudBrowser, 全メッセンジャー連携など

## 章一覧

| 章 | ファイル | 内容 |
|---|---|---|
| 0 | `docs/00_overview.md` | この文書の目次 |
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

## 使い方

1. はじめての場合は `docs/00_overview.md` から読み始める
2. 特定の設定を知る場合は `docs/14_reference_tables.md` の索引から探す
3. 優先順位を知る場合は `docs/03_setting_precedence.md` を参照
4. プロバイター/モデルの設定は `docs/04_model_and_providers.md` を参照

**注**: ソースコード (`source/hermes-agent/`) は調査用にローカルへクローンしているが、本書のリポジトリには `.gitignore` で除外されている。正確な最新情報は 公式リポジトリ を参照のこと。

## 構成

```
hermes-agent-config-guide/
  README.md              # このファイル
  .gitignore             # source/ の除外等
  docs/                  # 全16章の本文
    00_overview.md
    01_research_summary.md
    02_configuration_map.md
    03_setting_precedence.md
    ...
    15_appendix_source_notes.md
  research/              # 調査メモ
  reviews/               # レビュー結果（round_01, round_02, final）
  source/                # 調査用ソース（除外）
  scripts/               # 調査スクリプト
  assets/                # 添付ファイル
```

## 制限事項

- **対象バージョン**: この文書はコミット `7d66d30` 時点の情報に基づきます。将来のバージョンでは変更されている可能性があります。
- **未確認事項**: 以下の項目はソースコードでの直接確認がとれませんでした:
  - auto-detect の正確な provider 優先順（`agent/model_metadata.py` で確認が必要）
  - browser.engine の完全な値リスト
  - terminal.backend の完全な値リスト
  - auxiliary.timeout が実際に API に適用される箇所
  - HERMES_* timeout 環境変数の参照箇所
- これらの項目は本書で「未確認」と注明しています。

## 使用方法

このリポジトリをローカル Clone して、Markdown リーダーで読むことを推奨します。

```bash
git clone https://github.com/YOUR-USERNAME/hermes-agent-config-guide.git
cd hermes-agent-config-guide
# docs/*.md をお好きなエディターで開く
```

## 注意事項

- 本書の情報は調査時点でのソースコードに基づくものであり、将来のバージョンでは変更されている可能性があります。
- 本書の内容に起因する直接的・間接的な損害について一切責任を負いません。
- 本書は公式ドキュメントの代替ではありません。公式ドキュメントも併せて参照してください: https://hermes-agent.nousresearch.com/docs

## ライセンス

このリポジトリの内容は [MIT License](LICENSE) に従います。
