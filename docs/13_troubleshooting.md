# トroubleshooting

## 本章で扱うこと

1. 一般的なトラブルシューティング
2. よくある設定ミス
3. デバッグ方法
4. 環境変数の特定と確認
5. コミュニティリソース

## よくあるトラブルシューティング

### 1. モデルが選択されない

**症状**: エラー "No model configured" または 自動検出が失敗

**原因と対策**:

1. config.yaml の model.default と model.provider を確認
```bash
grep -A 5 "model:" ~/.hermes/config.yaml
```

2. .env ファイルの API キーを確認
```bash
grep API_KEY ~/.hermes/.env
```

3. プロバイターが利用可能か確認
```bash
OPENROUTER_API_KEY=<key> python -c "import requests; r = requests.get('https://openrouter.ai/api/v1/models'); print(r.status_code)"
```

### 2. web_search が動かない

**症状**: web_search が空の結果 または エラー

**原因と対策**:

1. web.search_backend の設定
```bash
grep -A 2 "web:" ~/.hermes/config.yaml
```

2. バックエンドのキーが設定済みか確認
```bash
grep "BRAVE_SEARCH_API_KEY" ~/.hermes/.env
```

3. SearXNGがインストール済みか
```bash
curl -s http://localhost:8080/search 2>&1 | head -5
```

### 3. terminal tool がタイムアウトする

**症状**: "Process timed out" または "Terminal output exceeded limit"

**原因と対策**:

1. `terminal.timeout` が十分か
```bash
grep -A 1 "terminal:" ~/.hermes/config.yaml
```

2. `tool_output.max_bytes` が大きすぎる
```bash
grep "max_bytes" ~/.hermes/config.yaml
```

### 4. browser tool が動かない

**症状**: "Browser not found" または "Chrome not detected"

**原因と対策**:

1. Chromiumがインストール済みか
```bash
which chromium
which google-chrome
```

2. CDPエンドポイントが使用可能か
```bash
curl http://localhost:9222/json/version
```

3. browser.engineを確認
```bash
grep -A 10 "browser:" ~/.hermes/config.yaml
```

### 5. gateway が起動しない

**症状**: "Gateway failed to start"

**原因と対策**:

1. TELEGRAM_TOKEN/DISCORD_TOKEN等が未設定
2. config.yamlのsyntaxエラー
3. ~/.hermes/.env のsyntax

```bash
hermes config check  # configの検証
hermes logs  # ログの確認
```

### 6. auxiliary モデルが動かない

**症状**: "Auxiliary model unavailable"

**原因と対策**:

1. auxiliary.vision.provider を明示的に設定
2. OPENROUTER_API_KEY が設定済みか
3. fallback 先を確認

## よくある設定ミス

###ミス1: config.yaml のインデント

```yaml
# 間違い
model:
default: "google/gemini-3-flash-preview"  # インデントなし

# 正しい
model:
  default: "google/gemini-3-flash-preview"  # インデントあり
```

###ミス2: .env のカンマ区切り

```yaml
# 間違い (カンマ区切り)
providers:
  custom1:
    api_key: "key1,key2"

# 正しい (複数行)
providers:
  custom1:
    api_key: "key1"
    api_key2: "key2"
```

###ミス3: base_url のプロトコル

```yaml
# 間違い
  custom1:
    api: "localhost:8000/v1"  # 欠落: https:// or http://

# 正しい
  custom1:
    api: "http://localhost:8000/v1"  # プロトコルを含む
```

###ミス4: providers のキーのスペル

```yaml
# 間違い
  custom1:
    ApiKey: "key"   # 大文字始まり
    BaseUrl: "url"  # 大文字始まり

# 正しい
  custom1:
    api_key: "key"   # snake_case
    base_url: "url"  # snake_case
```

###ミス5: timeout の型

```yaml
# 間違い
  timeout: "60"   # 文字列

# 正しい
  timeout: 60     # 数値
```

## デバッグ方法

### config.yaml の検証

```bash
# configを表示
hermes config

# 検証
hermes config check
```

### 環境変数を確認

```bash
# 設定済みの変数を表示
hermes env list

# 特定の値を確認
hermes env list OPENROUTER_API_KEY
```

### ログの確認

```bash
# 最新ログ
hermes logs

# エラーログ
hermes logs --level=ERROR

# セッション履歴
hermes logs --session <session_id>

# follow mode
hermes logs --follow
```

### config.yaml の移行

```bash
# 移行実行
hermes config migrate

# 移行表示 (インタラクティブ)
hermes config wizard
```

### デバッグモード

```bash
hermes chat --yolo  # approvalなしでテスト
hermes chat --debug  # デバッグ詳細出力
```

## メッセージングプラットフォーム固有のトラブル

### Discord

- ボットが返信しない → DISCORD_TOKENを確認
- DMが反応しない → allowed_messages を設定
- チャンネルが反応しない → allowed_channels を設定

### Telegram

- ボットが返信しない → TELEGRAM_TOKENを確認
- グループで反応しない → allowed_chats を設定
- トピックで反応しない → channel_topics を設定

### Slack

- ボットが反応しない → SLACK_BOT_TOKEN(XOXB) を確認
- フリーメッセージ → free_response_channels を設定

## よくある失敗例

### 失敗例1: provider に空文字列

```yaml
model:
  provider: ""  # 空文字列 → auto-detect (期待と違う結果になる場合がある)
```

### 失敗例2: custom_providers.list のまま

```yaml
# v12以降は自動マイグレーションされるが、手動で修正が必要
custom_providers:  # 非推奨
  - name: "my_provider"
```

### 失敗例3: timeout の値が小さすぎる

```yaml
auxiliary:
  compression:
    timeout: 5  # ローカルモデルには短すぎる
```

## 公式ドキュメント上の根拠

- `website/docs/reference/faq.md`
- `website/docs/reference/environment-variables.md`
- `website/docs/guides/cron-troubleshooting.md`

## ソースコード上の根拠

- `hermes_cli/config.py:3141` migrate_config 関数
- `hermes_cli/config.py:1000` config_version
