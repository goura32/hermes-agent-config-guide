# 付録：ソースコード上の根拠とドキュメント差分

## ソースコードのコミット情報

- **リポジトリ**: https://github.com/NousResearch/hermes-agent
- **コミット**: `7d66d30`
- **説明**: feat(kanban): add tooltips and docs link across dashboard (#21541)
- **日付**: 2026-05-08
- **config schema version**: 23
- **ブランチ**: main

## 主要の設定ファイル

| ファイル | 行数 | 役割 |
|--|-|-|
| `hermes_cli/config.py` | ~5200 | DEFAULT_CONFIG, loading, migration |
| `hermes_cli/env_loader.py` | ~175 | .env loading |
| `hermes_constants.py` | ~350 | PATHS, env constants |
| `run_agent.py` | ~12000 | AIAgent (core loop) |
| `model_tools.py` | ~36800 | tool discovery |
| `toolsets.py` | ~27952 | toolset definitions |
| `agent/auxiliary_client.py` | ~182515 | auxiliary model resolution |
| `agent/model_metadata.py` | ~60598 | built-in providers |

## 公式ドキュメント一覧

| カテゴリ | ファイル数 | 主要ファイル |
|--|-|-|
| developer-guide | 15 | `gateway-internals.md`, `architecture.md`, `prompt-assembly.md` |
| getting-started | 5 | `installation.md`, `quickstart.md`, `updating.md` |
| guides | 14 | `automation-templates.md`, `tips.md`, `local-ollama-setup.md` |
| reference | 10 | `model-catalog.md`, `environment-variables.md`, `cli-commands.md` |
| user-guide | 62 | `configuration.md`, `configuring-models.md`, `sessions.md` |
| integrations | 2 | `providers.md`, `index.md` |
| messaging | 13 | `telegram.md`, `discord.md`, `slack.md`, `whatsapp.md` |
| **合計** | **121** | (website/docs内のmarkdownファイル) |

## ドキュメントとソースコードの差分

### 差分1: custom_providers の形式

| 分野 | ドキュメント | ソースコード |
|--|-|-|
| custom_providers | list形式で説明されている場合がある | v12以降はdict形式にマイグレーション |
| 移行 | 言及されているが詳細がない | `migrate_config` で詳細な移行ロジック |

### 差分2: model.provider の既定値

| 分野 | ドキュメント | ソースコード |
|--|-|-|
| model.provider | "openrouter"と説明されている場合がある | DEFAULT_CONFIGでは空文字列 (`""`) |

### 差分3: compression.enabled 既定値

| 分野 | ドキュメント | ソースコード |
|--|-|-|
| compression.enabled | 文脈によって異なる | DEFAULT_CONFIGでは `True` |

### 差分4: environment variables

| 分野 | ドキュメント | ソースコード |
|--|-|-|
| HERMES_STREAM_READ_TIMEOUT | ドキュメントに記載あり | OPTIONAL_ENV_VARS に記載なし |
| HERMES_STREAM_STALE_TIMEOUT | ドキュメントに記載あり | OPTIONAL_ENV_VARS に記載なし |
| HERMES_API_TIMEOUT | ドキュメントに記載あり | OPTIONAL_ENV_VARS に記載なし |

### 差分5: timeout の実装

| 分野 | ドキュメント | ソースコード |
|--|-|-|
| プロバイダー単位のtimeout | `providers.<id>.request_timeout_seconds` | 実装で実際に参照されるか要確認 |
| auxiliary.timeout | ドキュメントに記載あり | `auxiliary_client.py` で参照 |

### ドキュメントのみ記載 (ソース未確認)

1. HERMES_STREAM_READ_TIMEOUT
2. HERMES_STREAM_STALE_TIMEOUT
3. HERMES_API_TIMEOUT
4. HERMES_API_CALL_STALE_TIMEOUT
5. HERMES_AGENT_TIMEOUT
6. HERMES_AGENT_NOTIFY_INTERVAL
7. browser.image_timeout
8. web.sitemap_timeout

## マイグレーション履歴

| version | major change |
|--|-|
| 3 | tool progress migration |
| 4 | timezone migration |
| 5 | ANN TOKEN cleanup |
| 11 | custom_providers to providers |
| 12 | dead LLM_MODEL / OPENAI_MODEL cleanup |
| 13 | legacy flat stt.model migration |
| 14 | gateway interim-message gate |
| 15 | tool_progress_overrides → display.platforms |
| 16 | compression.summary_* → auxiliary.compression |
| 17 | plugins are now opt-in |
| 20 | curator defaults + logs/curator/ creation |
| 22 | curator defaults + logs/curator/ creation |
| 23 | current version |

## 未確認項目表

### 必須確認項目

以下はソースコードでの直接確認が必要な項目。

1. **HERMES_AGENT_TIMEOUT**: `run_agent.py` のどこで参照されるか
2. **HERMES_AGENT_NOTIFY_INTERVAL**: `run_agent.py` のどこで参照されるか
3. **HERMES_STREAM_READ_TIMEOUT**: 実際のストリーム読み込み処理
4. **HERMES_API_TIMEOUT**: APIクライアントの呼び出し元
5. **custom_provider の timeout**: `request_timeout_seconds`, `stale_timeout_seconds` の実際の効果
6. **title_generation.timeout**: 実際に有効なtimeoutか
7. **mcp timeout**: MCPツールからの呼び出しでのtimeout
8. **delegation.timeout**: サブエージェントのタイムアウト
9. **browser.timeout**: 各browser backendでのtimeout

### 推奨確認方法

これらの項目は `hermes_cli/config.py` の OPTIONAL_ENV_VARS に定義されていないものが多い。
`run_agent.py` や `agent/` 内のソースコードで `os.getenv` / `os.environ.get` などで直接参照されている可能性が高い。

## 環境変数のカテゴリ別一覧

### Provider 関連

| 環境変数 | 用途 |
|--|-|
| OPENROUTER_API_KEY | OpenRouter API鍵 |
| ANTHROPIC_API_KEY | Anthropic API鍵 |
| ANTHROPIC_TOKEN | Anthropic トークン |
| OPENAI_API_KEY | OpenAI API鍵 |
| GOOGLE_API_KEY | Google API鍵 |
| GEMINI_API_KEY | Gemini API鍵 |
| GEMINI_BASE_URL | Gemini base_url |
| XAI_API_KEY | xAI API鍵 |
| XAI_BASE_URL | xAI base_url |
| NVIDIA_API_KEY | NVIDIA API鍵 |
| NVIDIA_BASE_URL | NVIDIA base_url |
| LM_API_KEY | LM Studio API鍵 |
| LM_BASE_URL | LM Studio base_url |
| GLM_API_KEY | Z.AI API鍵 |
| ZAI_API_KEY | Z.AI API鍵 |
| Z_AI_API_KEY | Z.AI API鍵 |
| NOUS_API_KEY | Nous Portal API鍵 |
| NOUS_BASE_URL | Nous Portal base_url |
| DASHSCOPE_API_KEY | DashScope API鍵 |
| DASHSCOPE_BASE_URL | DashScope base_url |
| KIMI_API_KEY | Kimi API鍵 |
| KIMI_BASE_URL | Kimi base_url |
| MINIMAX_API_KEY | MiniMax API鍵 |
| MINIMAX_BASE_URL | MiniMax base_url |

### Platform 関連

| 環境変数 | 用途 |
|--|-|
| TELEGRAM_TOKEN | Telegram ボットトークン |
| DISCORD_TOKEN | Discord ボットトークン |
| DISCORD_BOT_TOKEN | Discord ボットトークン (エイリアス) |
| SLACK_BOT_TOKEN | Slack ボットトークン |
| SLACK_APP_TOKEN | Slack アプリトークン |
| SENDING_ACCOUNT | メールアカウント |
| WHATSAPP_MODE | WhatsApp mode |
| WHATSAPP_ENABLED | WhatsApp enabled |
| WHATSAPP_ALLOWED_USERS | WhatsApp allowed users |

### Tool 関連

| 環境変数 | 用途 |
|--|-|
| SEARXNG_URL | SearXNG base URL |
| FIRECRAWL_API_KEY | Firecrawl API鍵 |
| TAVILY_API_KEY | Tavily API鍵 |
| EXA_API_KEY | Exa API鍵 |
| BRAVE_SEARCH_API_KEY | Brave Search API鍵 |

### System 関連

| 環境変数 | 用途 |
|--|-|
| HERMES_HOME | HERMES_HOME |
| HERMES_MODEL | モデル |
| HERMES_INFERENCE_PROVIDER | プロバイター |
