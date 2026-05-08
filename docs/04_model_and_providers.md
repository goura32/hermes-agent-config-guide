# model / providers / custom_providers

## 本章で扱うこと

1. model (primary inference settings)
2. providers (named providers)
3. custom_providers (legacy, for migration only)
4. fallback_providers (failover)
5. credential_pool_strategies

## 設定項目一覧

| 設定パス | 型 | 既定値 | 省略時 |
|--|-|-|-|
| `model` | string | `""` | auto-detect |
| `model.default` | string | `""` | auto-detect |
| `model.provider` | string | `""` | auto-detect |
| `providers` | dict | `{}` | なし |
| `fallback_providers` | list | `[]` | フェールバックなし |
| `credential_pool_strategies` | dict | `{}` | なし |

## model.設定項目

### model (トップレベル)

- **型**: string (空文字列デフォルト)
- **設定可能な値**: 任意のモデル識別子 (例: "google/gemini-3-flash-preview")
- **既定値**: 空文字列
- **省略時**: auto-detect (利用可能な最初のプロバイター)

### model.default

- **型**: string
- **既定値**: `""` (空文字列)
- **説明**: デフォルトのモデル。CLIの `/model <slug>` で上書き可能。

### model.provider

- **型**: string
- **既定値**: `""` (空文字列)
- **説明**: デフォルトのプロバイター。

### model.base_url

- **型**: string
- **既定値**: `""`
- **説明**: プロバイターごとのデフォルト base_url より優先。

### model.api_key

- **型**: string
- **既定値**: `""`
- **説明**: プロバイター固有の API キー。

### model.context_length

- **型**: integer
- **既定値**: 未確認 (auto-detect)
- **説明**: コンテキスト長指定。省略時はプロバイターが自動検出。

### model.max_tokens

- **型**: integer
- **既定値**: 未確認 (auto-detect)
- **説明**: 最大トークン数。省略時はプロバイターのデフォルトが使用される。

## providers 設定

### 形式

`providers` は dictionary 形式。各エントリには以下の設定が含まれる:

```yaml
providers:
  <id>:
    api: <string>      # base_url (required)
    name: <string>     # display name (optional)
    api_key: <string>  # API key (optional)
    default_model: <string> # default model for this provider
    model: <string>        # alias for default_model
    transport: <string>    # api_mode (OpenAI-compatible, Codex responses, etc.)
    api_mode: <string>     # alias for transport
    models: <dict>         # model catalog for this provider
    context_length: <int>  # context length hint
    rate_limit_delay: <int> # per-request delay (seconds)
    request_timeout_seconds: <int> # timeout for this provider
    stale_timeout_seconds: <int>   # stale timeout for this provider
```

### providers.<id> の利用可能なフィールド

| キー | 型 | 必須 | 説明 |
| url, api, base_url | string | はい（いずれか1つ） | base_url。3つのエイリアス、**どれか1つ**指定で十分。指定の場合、base_urlにマージされる |
| api_key | string | いいえ | API key (configに保存される) |
| key_env | string | いいえ | API key を含める環境変数名 |
| api_key_env | string | いいえ | key_env のエイリアス |
| name | string | いいえ | 表示名 |
| provider_key | string | いいえ | internal key |
| transport | string | いいえ | api_mode (OpenAI-compatible, Codex responses等) |
| api_mode | string | いいえ | transport (エイリアス) |
| model | string | いいえ | デフォルトモデル |
| default_model | string | いいえ | model のエイリアス |
| models | dict or list | いいえ | プロバイダーがサポートするモデル一覧 |
| context_length | int | いいえ | コンテキスト長 |
| rate_limit_delay | int/float | いいえ | リクエスト間の遅延 |
| request_timeout_seconds | int | いいえ | タイムアウト秒数 |
| stale_timeout_seconds | int | いいえ | stale タイムアウト秒数 |


### camelCase エイリアス

`providers` は camelCase のキーも受け入れる:

- `apiKey` → `api_key`
- `baseUrl` → `base_url`
- `apiMode` → `api_mode`
- `keyEnv` → `key_env`
- `apiKeyEnv` → `key_env`
- `defaultModel` → `default_model`
- `contextLength` → `context_length`
- `rateLimitDelay` → `rate_limit_delay`

### 既知のキー

```
name, api, url, base_url, api_key, key_env, api_key_env,
api_mode, transport, model, default_model, models,
context_length, rate_limit_delay
```

他のキーは無視され、警告ログが出力される。

## custom_providers (レガシー)

**重要**: version 12 マイグレーション以来、`custom_providers` は非推奨。
`providers` dict 形式を使用する。

### custom_providers から providers への移行

マイグレーション処理は以下の変換を行う:

1. `custom_providers.name` → `providers.<key>.name`
2. `custom_providers.base_url` → `providers.<key>.api`
3. `custom_providers.api_key` → `providers.<key>.api_key`
4. `custom_providers.api_mode` → `providers.<key>.transport`
5. `custom_providers.model` → `providers.<key>.default_model`

## custom_providers.name と providers.<id> の違い

| 項目 | custom_providers.name | providers.<id> |
|--|-|-|
| 形式 | list 内の dict | dict のキー |
| キー | "name" | <dictのキー> |
| base_url | "base_url" | "api" / "url" / "base_url" |
| migration | v12以降自動移行 | 標準 |

**注意**: `custom_providers.name` と `providers` のキーの名前は異なる可能性がある。マイグレーション後、名前もキーも変わる場合がある。

## プロバイダー解決の流れ

```
provider 設定値
    │
    ▼
"auto" → 利用可能な最初のプロバイターを検索
    │
    ▼
"openrouter" → OPENROUTER_API_KEY が必要
    │
    ▼
"anthropic" → ANTHROPIC_API_KEY が必要
    │
    ▼
"nous" → NOUS_API_KEY or OAuthフロー
    │  ▼
    ▼
"google-gemini-cli" → GOOGLE_API_KEY or GEMINI_API_KEY が必要
    │
    ▼
"openai-codex" → OPENAI_API_KEY が必要 (OAuth)
    │
    ▼
"copilot" → COPILOT_API_TOKEN が必要
    │  ▼
    ▼
"copilot-acp" → COPILOT_API_TOKEN が必要
    │
    ▼
"zai" → GLM_API_KEY or ZAI_API_KEY が必要
    │
    ▼
"kimi-coding" → KIMI_API_KEY が必要
    │
    ▼
"custom" / 名前 → 指定された名前のプロバイターを使用 (APIキー必要)
    │
    ▼
"" → auto-detect
    │
    ▼
None → エラー
```

## API モードの違い

### api_mode / transport の値

値によって API クライアントの振る舞いが決定される:

1. `chat_completions` (デフォルト) - OpenAI互換API (chat.completions.create)
2. `codex_responses` - OpenAI Codex Responses API
3. それ以外 - エラー (またはプロバイター固有の処理)

### Ollama / LM Studio / vLLM / llama.cpp での違い

各ローカルエンドポイントで設定する必要があるもの:

| エンドポイント | base_url | api_mode |
|-|-|-|
| Ollama | `http://localhost:11434/v1` | chat_completions |
| LM Studio | `http://localhost:1234/v1` | chat_completions |
| vLLM | `http://localhost:8000/v1` | chat_completions |
| llama.cpp (cppserver) | `http://localhost:8080/v1` / `/completions` | 未確認 |

## custom は予約語か？

**答え: いえ、予約語ではない**。 `custom` はユーザーが定義できる名前として利用される。

ただし、"auto", "main", "custom" の三つの値には特別な意味がある:
- `auto`: automatic provider selection (予約語的な扱い)
- `main`: use the primary chat model (予約語的な扱い)
- `custom`: 任意の名前付きプロバイター名

## よくある誤解

1. "custom_providersはprovidersと独立した機能" → V12以降は同一のデータ構造に統合された
2. "api_mode省略時は常にchat_completions" → 実際の挙動はプロバイターのデフォルトに依存
3. "providersには複数のentryが設定できる" → providersはdictで各プロバイターに一意の名前が必要

## 実運用上の推奨設定

### OpenRouter をメインプロバイターとして使用

```yaml
model:
  default: "google/gemini-3-flash-preview"
  provider: openrouter

providers:
  openrouter:
    api: "https://openrouter.ai/api/v1"
  custom1:
    api: "http://localhost:11434/v1"
    name: "My Local Ollama"
```

### ローカルモデルをフェールバックとして

```yaml
model:
  provider: openrouter

providers:
  openrouter:
    api: "https://openrouter.ai/api/v1"
    api_key: "${OPENROUTER_API_KEY}"
  ollama:
    api: "http://localhost:11434/v1"
    name: "My Ollama"
```

## 公式ドキュメント上の根拠

- `website/docs/user-guide/configuration.md`
- `website/docs/user-guide/configuring-models.md`
- `website/docs/integrations/providers.md`
- `website/docs/user-guide/features/provider-routing.md`
- `website/docs/user-guide/features/fallback-providers.md`

## ソースコード上の根拠

- `hermes_cli/config.py:1385` DEFAULT_CONFIGのmodel/providers定義
- `hermes_cli/config.py:2631` `_normalize_custom_provider_entry` 関数
- `hermes_cli/env_loader.py:14-17` `_CREDENTIAL_SUFFIXES`
- `hermes_constants.py:342` OPENROUTER_BASE_URL
