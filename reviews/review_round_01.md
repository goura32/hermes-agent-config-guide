# レビュー ラウンド 01

## レビュー日時
- 2026-05-08

## 総評
全16章（00〜15）が作成された。構成と網羅性は良好。ただし、以下の重要な指摘がある。

---

### 指摘1: api / base_url / url のエイリアス扱いの誤り (docs/04, docs/05, docs/06 等)

**程度**: 重大
**該当**: docs/04_model_and_providers.md 行94-96, docs/05_auxiliary_models.md

**問題**: `api`, `base_url`, `url` の3つすべてが「必須」として記載されている。本来これらは同じ意味のエイリアスで、**いずれか1つ**で良い。すべてのフィールドが必須だと誤解を招く。

**修正**:
```
# 修正前
| api | string | はい | base_url (または url, api のエイリアス) |
| base_url | string | はい | base_url (または url, api のエイリアス) |
| url | string | はい | base_url (api, base_url のエイリアス) |

# 修正後
| url, api, base_url | string | はい（いずれか1つ） | base_url。3つのエイリアス、いずれか1つ指定で良い。 |
```

**根拠**: sourceコードの `_normalize_custom_provider_entry` で、`url`, `api`, `base_url` がすべて同じ値にマージされる確認済み。

---

### 指摘2: auto の選択アルゴリズムが推測ベース (docs/03_setting_precedence.md)

**程度**: 注意
**該当**: docs/03_setting_precedence.md 行215-222

**問題**: auto の選択アルゴリズムを「一般的には: 1.ANTHROPIC→2.OPENROUTER→3.GOOGLE→4.OPENAI」と記載しているが、これは推測であり、ソースコードで直接確認していない。

**修正**: 「推測」と明示するか、ソースコードで確認後修正が必要。現在 `agent/model_metadata.py` で実際の定義を確認していない。

**提案**: 以下に書き換える:
```
auto で選択アルゴリズム
ソースコード（agent/model_metadata.py）での正確な順序は未確認。
一般的に利用可能な最初のものが選択されると言われる。
```

---

### 指摘3: timeout 項目の多くが未確認 (docs/03_setting_precedence.md)

**程度**: 注意
**該当**: docs/03_setting_precedence.md 行224-250

**問題**: 多くの timeout 項目に「未確認」と記載されている。これらは OPTIONAL_ENV_VARS から直接取得した情報だが、実装での参照箇所を確認していない。

**修正**: 本書の付録で未確認事項として明記し、実際の動作を確認したユーザーに報告を促す。

---

### 指摘4: プロバイダー解決の流れが不完全 (docs/04_model_and_providers.md)

**程度**: 重大
**該当**: docs/04_model_and_providers.md 行161-204

**問題**: プロバイダー解決の流れで、"nous" が "NOUS_API_KEY or OAuthフロー" と記載されているが、実際には OAuthフローの詳細が不明。また "copilot" と "copilot-acp" が同じ COPILOT_API_TOKEN が必要とあるが、これも確認していない。

**修正**: 各プロバイダーの設定要件について、実装で確認できたもののみ記載し、未確認は明確に注明。

---

### 指摘5: browser.engine の値が不完全 (docs/06_web_browser_tools.md)

**程度**: 注意
**該当**: docs/06_web_browser_tools.md 行???

**問題**: browser.engine の値が "auto", "lightpanda", "chrome" のみ記載。sourceコードで確認した範囲ではもっと多くのエンジンがサポートされている可能性。

**修正**: `hermes_cli/config.py` から browser engine の定義を再確認。

---

### 指摘6: terminal.backend の選択肢の項目が不完全 (docs/07_terminal_tools_approvals.md)

**程度**: 注意
**該当**: docs/07_terminal_tools_approvals.md 行???

**問題**: docker_env, docker_volumes 等の設定が config.py の DEFAULT_CONFIG 内の terminal セクションの定義と完全に一致するか確認していない。

---

### 指摘7: 環境変数の記載が不完全 (docs/14_reference_tables.md)

**程度**: 大目
**該当**: docs/14_reference_tables.md 最終セクション

**問題**: 環境変数リストが OPTIONAL_ENV_VARS の全リストに基づくが、いくつかの変数が不足（例: FIRECRAWL_API_KEY, BROWSERBASE_API_KEY, GPT_API_KEY, MINIMAX_API_KEY, KIMI_API_KEY 等）。

**修正**: source_code_findings.md の OPTIONAL_ENV_VARS の全リストから再抽出。

---

### 指摘8: マイナーな誤字脱字

程度: 軽微
該当: 複数章

- "プロバイター" 表記が "プロバイダー" と混在している → 統一: "プロバイダー"
- "インタープット" → 正: "インタープト"（ただし文脈により要確認）
- "インターバル" → 正: "インターバル"（ただし文脈により要確認）
- "コミット" → 正: "コミット"（ただし文脈により要確認）
- "プロンプト" → 正: "プロンプト"
- "スキャニング" → 正: "スキャニング"
- "スキャン" → 正: "スキャン"
- "スクリプト" → 正: "スクリプト"
- "バックアップ" → 正: "バックアップ"
- "ダッシュボード" → 正: "ダッシュボード"

---

### 指摘9: docs/09_memory_sessions_curator.md の内容が浅い

**程度**: 注意
**該当**: docs/09_memory_sessions_curator.md

**問題**: memory, sessions, curator, delegation の4つのトピックを1章に収めているため、各トピックの説明が簡略化されすぎている。特に memory provider と external memory integration は公式ドキュメントに詳細があったはず。

**修正**: memory_provider 設定（obsidian, notion, 等）の詳細を公式ドキュメントから再追加。

---

### 指摘10: docs/11_cron_hooks_automation.md の内容が簡略化されすぎ

**程度**: 大目
**該当**: docs/11_cron_hooks_automation.md

**問題**: cron の CLIコマンドベースの設定（/cron schedule 等）の詳細が不足。config.yaml の cron section だけでなく、CLIコマンドでの cron ジョブ管理も記載すべき。

## レビュー結果

改稿が必要な指摘8件（指摘1-8が改稿対象、指摘9-10は内容不足のため改稿対象）。
指摘9, 10 は本文の再執筆が必要。
