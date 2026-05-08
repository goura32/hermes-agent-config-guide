# auxiliary モデル

## 本章で扱うこと

1. auxiliary モデルとは
2. 各 auxiliary タスクの設定項目
3. provider: auto/main/custom の挙動
4. タイムアウトの実効性
5. フォールバック動作

## auxiliary モデルとは

auxiliary モデルは、メインチャットモデルとは別に、補助的なタスクに使用されるモデルである。以下のようなタスクで利用される:

- 画像分析 (vision)
- web 要約 (web_extract)
- コンテキスト圧縮 (compression)
- セッション検索 (session_search)
- スキルハブ操作 (skills_hub)
- コマンド承認判断 (approval)
- MCP 操作 (mcp)
- タイトル生成 (title_generation)
- トライジ仕様化 (triage_specifier)
- スキルキューレーター (curator)

## 設定項目一覧

各 auxiliary タスクは以下のキーを持つ:

| キー | 型 | 既定値 | 説明 |
|--|-|-|-|
| provider | string | "auto" | 使用するプロバイター |
| model | string | "" | モデル識別子 |
| base_url | string | "" | 直接エンドポイント |
| api_key | string | "" | クレデンシャル |
| timeout | int | タスク依存 | タイムアウト秒数 |
| extra_body | dict | `{}` | 追加のリクエストフィールド |

## 各タスクの既定値

### vision

| キー | 既定値 | 説明 |
|--|-|-|
| provider | "auto" | モデル選択 |
| model | "" | 使用モデル |
| base_url | "" | 直接エンドポイント |
| api_key | "" | APIキー |
| timeout | 120 | LLM APIタイムアウト |
| extra_body | `{}` | 追加フィールド |
| download_timeout | 30 | 画像DLタイムアウト |

### web_extract

| キー | 既定値 | 説明 |
|--|-|-|
| provider | "auto" | モデル選択 |
| model | "" | 使用モデル |
| base_url | "" | 直接エンドポイント |
| api_key | "" | APIキー |
| timeout | 360 | web要約タイムアウト |
| extra_body | `{}` | 追加フィールド |

### compression

| キー | 既定値 | 説明 |
|--|-|-|
| provider | "auto" | モデル選択 |
| model | "" | 使用モデル |
| base_url | "" | 直接エンドポイント |
| api_key | "" | APIキー |
| timeout | 120 | 圧縮タイムアウト |
| extra_body | `{}` | 追加フィールド |

### session_search

| キー | 既定値 | 説明 |
|--|-|-|
| provider | "auto" | モデル選択 |
| model | "" | 使用モデル |
| base_url | "" | 直接エンドポイント |
| api_key | "" | APIキー |
| timeout | 30 | 検索タイムアウト |
| extra_body | `{}` | 追加フィールド |
| max_concurrency | 3 | 並列処理上限 |

### skills_hub

| キー | 既定値 | 説明 |
|--|-|-|
| provider | "auto" | モデル選択 |
| model | "" | 使用モデル |
| base_url | "" | 直接エンドポイント |
| api_key | "" | APIキー |
| timeout | 30 | タスクタイムアウト |
| extra_body | `{}` | 追加フィールド |

### approval

| キー | 既定値 | 説明 |
|--|-|-|
| provider | "auto" | モデル選択 |
| model | "" | 高速軽量推奨 |
| base_url | "" | 直接エンドポイント |
| api_key | "" | APIキー |
| timeout | 30 | 承認判断タイムアウト |
| extra_body | `{}` | 追加フィールド |

### mcp

| キー | 既定値 | 説明 |
|--|-|-|
| provider | "auto" | モデル選択 |
| model | "" | 使用モデル |
| base_url | "" | 直接エンドポイント |
| api_key | "" | APIキー |
| timeout | 30 | MCP操作タイムアウト |
| extra_body | `{}` | 追加フィールド |

### title_generation

| キー | 既定値 | 説明 |
|--|-|-|
| provider | "auto" | モデル選択 |
| model | "" | 使用モデル |
| base_url | "" | 直接エンドポイント |
| api_key | "" | APIキー |
| timeout | 30 | タイトル生成タイムアウト |
| extra_body | `{}` | 追加フィールド |

### triage_specifier

| キー | 既定値 | 説明 |
|--|-|-|
| provider | "auto" | モデル選択 |
| model | "" | 高速軽量推奨 |
| base_url | "" | 直接エンドポイント |
| api_key | "" | APIキー |
| timeout | 120 | トライジ処理タイムアウト |
| extra_body | `{}` | 追加フィールド |

### curator

| キー | 既定値 | 説明 |
|--|-|-|
| provider | "auto" | モデル選択 |
| model | "" | 使用モデル |
| base_url | "" | 直接エンドポイント |
| api_key | "" | APIキー |
| timeout | 600 | スキルレビュータイムアウト |
| extra_body | `{}` | 追加フィールド |

## provider: auto vs main vs custom の違い

### provider: auto (デフォルト)

- 利用可能な最初のプロバイターを自動選択
- 費用を抑えられる
- visionやweb_extractは高速なモデル(gemini-flash等)を使用する傾向

### provider: main

- メインチャットモデルと同じモデルを使用
- 設定が最もシンプル
- メインモデルが重い場合は遅い/高額

### provider: custom

- 任意の名前付きプロバイターを指定
- `providers.<custom_id>` の形式
- ロールごとに最適なプロバイターを使用可能

### 名前付きプロバイター指定時

```yaml
auxiliary:
  compression:
    provider: "my-local-ollama"  # ← providersセクションで定義済みの名前
```

この場合、`providers.my_local_ollama` の base_url と api_key が使用される。

### base_urlのみを設定した場合

base_urlのみに値を設定すると:

1. APIキーは OPENAI_API_KEY がフォールバック
2. 自動的に OpenAI互換モードが有効になる
3. mainチャットモデルのプロバイターは影響を受けない

### api_key省略時の挙動

api_keyを省略すると:

1. config.yaml に api_key が含まれない
2. .env ファイルから環境変数が読み込まれる
3. OPENAI_API_KEY が最終的なフォールバック
4. 一部のプロバイターはAPIキー不要(Ollama等)

## 実運用上の推奨設定

### 費用を抑える場合

```yaml
model:
  default: "google/gemini-3-flash-preview"
  provider: openrouter

auxiliary:
  vision:
    provider: "auto"        # gemini-flash にフォールバック
    download_timeout: 60    # 遅い接続にも対応
  compression:
    provider: "auto"
    timeout: 120
  web_extract:
    provider: "auto"
    timeout: 360
  approval:
    provider: "auto"        # gemini-flash 推奨
```

### ローカルモデルを補助タスクに使用する場合

```yaml
providers:
  ollama:
    api: "http://localhost:11434/v1"
    name: "My Local Ollama"
    default_model: "qwen3:latest"

auxiliary:
  compression:
    provider: "ollama"       # ← ローカルモデルを使用
  title_generation:
    provider: "ollama"
```

### メインモデルの代わりに補助タスクに専用モデルを使用する場合

```yaml
model:
  default: "anthropic/claude-sonnet-4-20250514"
  provider: anthropic

auxiliary:
  vision:
    provider: "openrouter"
    model: "google/gemini-2.5-flash"      # visionには専用モデル
  approval:
    provider: "openrouter"
    model: "google/gemini-2.5-flash"       # approvalにも軽量モデル
  curator:
    provider: "openrouter"
    model: "google/gemini-2.5-flash"       # curatorにも軽量モデル
    timeout: 600                            # 長時間がかかるので余裕を持たせる
```

## よくある誤解

1. "auxiliary.vision.timeoutがwebのtimeoutと混同される" → vision.timeoutはLLMへの呼び出しのみ、画像のダウンロードにはdownload_timeoutが別
2. "auxiliary.compression.mainチャットのtimeoutと混同される" → 独立した設定。compression.timeoutは圧縮処理のタイムアウト
3. "title_generationのtimeoutが本当に効くか" → 実装依存。実装で実際に参照されるか確認が必要

## よくある失敗例

### failure 1: base_urlのみの設定

```yaml
# 間違い: api_keyを省略するとプロバイターのデフォルトが使用される
providers:
  myprovider:
    api: "https://example.com/v1"
    # api_key がない → OPENAI_API_KEY にフォールバック
```

### failure 2: modelの省略

```yaml
# 間違い: modelが空文字列 → プロバイターのデフォルトが使用される
auxiliary:
  compression:
    provider: "custom"
    model: ""   # ← 空文字列はプロバイターのデフォルトになる
```

### failure 3: timeoutの小さすぎる設定

```yaml
# 間違い: compressionで30秒は短すぎる
auxiliary:
  compression:
    provider: "local-ollama"
    timeout: 30   # ← ローカルモデルには遅すぎる
```

## 公式ドキュメント上の根拠

- `website/docs/user-guide/configuration.md`
- `website/docs/user-guide/features/credential-pools.md`
- `website/docs/guides/local-ollama-setup.md`

## ソースコード上の根拠

- `hermes_cli/config.py:716` auxiliary DEFAULT_CONFIG定義
- `hermes_cli/config.py:2631` `_normalize_custom_provider_entry` (custom_providersの検証)
- `agent/auxiliary_client.py` (auxiliary解決処理)
- `agent/model_metadata.py` (組み込みプロバイダー)
