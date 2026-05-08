# 設定項目リファレンス表

## 全設定項目の索引 (TOP LEVEL KEYS)

```
model              # Primary inference settings
providers          # Named providers
custom_providers   # Legacy (migrated to providers in v12)
credential_pool_strategies
fallback_providers
toolsets           # ["hermes-cli"]
agent              # Agent runtime settings
terminal           # Terminal backend settings
web                # Web search/extract backends
browser            # Browser settings
checkpoints        # Filesystem checkpoints
file_read_max_chars
tool_output        # Output truncation
tool_loop_guardrails
compression        # Context compression
prompt_caching     # Anthropic prompt caching
openrouter         # OpenRouter-specific settings
bedrock            # AWS Bedrock settings
auxiliary          # Auxiliary models (vision, compression, etc.)
display            # UI settings
dashboard          # Dashboard settings
privacy            # Privacy settings
tts                # Text-to-speech settings
stt                # Speech-to-text settings
voice              # Voice mode settings
human_delay        # Human-like delay
context            # Context engine
memory             # Persistent memory
delegation         # Subagent delegation
prefill_messages_file
goals              # Cross-turn goals
skills             # External skill directories
curator            # Skill maintenance
honcho             # Honcho AI memory
timezone           # IANA timezone
slack              # Slack platform settings
discord            # Discord platform settings
whatsapp           # WhatsApp platform settings
telegram           # Telegram platform settings
mattermost         # Mattermost platform settings
matrix             # Matrix platform settings
approvals          # Approval settings
command_allowlist
quick_commands
hooks              # Shell script hooks
hooks_auto_accept
personalities      # Custom personalities
security           # Security settings
cron               # Cron scheduler settings
kanban             # Kanban multi-agent settings
code_execution     # Execute_code settings
logging            # Logging settings
model_catalog      # Model catalog settings
network            # Network settings
sessions           # Session storage
onboarding         # Onboarding hints
updates            # Update behavior
```

## detailed_setting_table

### model / providers

| 設定パス | 型 | 既定値 | 説明 |
| model | str | `""` | デフォルトモデル |
| provider | str | `""` | デフォルトプロバイダー |
| providers | dict | `{}` | named providers |
| fallback_providers | list | `[]` | フェールバック |
| credential_pool_strategies | dict | `{}` | クレデンシャルプール |

### agent

| 設定パス | 型 | 既定値 | 説明 |
|--|-|-|-|
| agent.max_turns | int | 90 | 最大ターン |
| agent.gateway_timeout | int | 1800 | timeout (秒) |
| agent.restart_drain_timeout | int | 180 | ドレインタイムアウト |
| agent.api_max_retries | int | 3 | APIリトライ回 |
| agent.service_tier | str | `""` | service tier |
| agent.tool_use_enforcement | str/list | `"auto"` | ツール使用の強制 |
| agent.gateway_timeout_warning | int | 900 | 警告インターバル |
| agent.gateway_notify_interval | int | 180 | 通知インターバル |
| agent.gateway_auto_continue_freshness | int | 3600 | 継続の新規性(秒) |
| agent.image_input_mode | str | `"auto"` | 画像入力モード |
| agent.disabled_toolsets | list | `[]` | 無効ツールセット |

### terminal

| 設定パス | 型 | 既定値 | 説明 |
|--|-|-|-|
| terminal.backend | str | `"local"` | backend |
| terminal.modal_mode | str | `"auto"` | Modalモード |
| terminal.timeout | int | 180 | timeout (秒) |
| terminal.cwd | str | `"."` | 作業ディレクトリ |
| terminal.auto_source_bashrc | bool | True | bashrc自動読み込み |
| terminal.docker_image | str | `"nikolaik/python-nodejs:python3.11-nodejs20"` | Docker イメージ |
| terminal.container_memory | int | 5120 | コンテナーメモリ (MB) |
| terminal.container_persistent | bool | True | コンテナー永続 |

### web/browser

| 設定パス | 型 | 既定値 | 説明 |
|--|-|-|-|
| web.backend | str | `""` | shared backend |
| web.search_backend | str | `""` | search backend |
| web.extract_backend | str | `""` | extract backend |
| browser.engine | str | `"auto"` | ブラウザーエンジン |
| browser.command_timeout | int | 30 | command timeout |
| browser.inactivity_timeout | int | 120 | idle timeout |
| browser.allow_private_urls | bool | False | 内部IP許可 |

### auxiliary

各タスク (vision, web_extract, compression, session_search, approval, title_generation, curator, mcp, skills_hub, triage_specifier):
`provider, model, base_url, api_key, timeout, extra_body` のキーを持つ。

### display

| 設定パス | 型 | 既定値 | 説明 |
|--|-|-|-|
| display.personality | str | `"kawaii"` | キャラクター |
| display.language | str | `"en"` | UI言語 |
| display.show_cost | bool | False | 費用表示 |
| display.streaming | bool | False | ストリーミング |
| display.tui_status_indicator | str | `"kaomoji"` | TUIインジケーター |

### approval / security / privacy

| 設定パス | 型 | 既定値 | 説明 |
|--|-|-|-|
| approvals.mode | str | `"manual"` | 承認モード |
| approvals.timeout | int | 60 | 承認タイムアウト |
| approvals.cron_mode | str | `"deny"` | cron承認モード |
| security.redact_secrets | bool | True | シークレットマスク |
| security.allow_private_urls | bool | False | 内部IP許可 |
| security.tirith_enabled | bool | True | Tirithスキャン |
| privacy.redact_pii | bool | False | PIMASK |

### cron/kanban

| 設定パス | 型 | 既定値 | 説明 |
|--|-|-|-|
| cron.wrap_response | bool | True | 応答ラップ |
| cron.max_parallel_jobs | int/None | None | 並列ジョブ数 |
| kanban.dispatch_in_gateway | bool | True | ゲートウェいで実行 |
| kanban.dispatch_interval_seconds | int | 60 | ディスパッチ間隔 |
| kanban.failure_limit | int | 2 | 失敗ブロック回数 |

### tts/stt/voice

| 設定パス | 型 | 既定値 | 説明 |
|--|-|-|-|
| tts.provider | str | `"edge"` | TTSプロバイダー |
| stt.enabled | bool | True | STT有効 |
| stt.provider | str | `"local"` | STTプロバイダー |
| voice.record_key | str | `"ctrl+b"` | 録音キー |
| voice.auto_tts | bool | False | 自動TTS |
| voice.max_recording_seconds | int | 120 | 最大録音時間 |

### compression

| 設定パス | 型 | 既定値 | 説明 |
|--|-|-|-|
| compression.enabled | bool | True | コンテキスト圧縮有効 |
| compression.threshold | float | 0.50 | 圧縮閾値 |

### checkpoints

| 設定パス | 型 | 既定値 | 説明 |
|--|-|-|-|
| checkpoints.enabled | bool | False | チェックポイント有効 |
| checkpoints.auto_prune | bool | True | 自動プルーン |
| checkpoints.max_snapshots | int | 20 | 最大スナップショット数 |
| checkpoints.retention_days | int | 7 | 保持日数 |

### sessions

| 設定パス | 型 | 既定値 | 説明 |
|--|-|-|-|
| sessions.auto_prune | bool | False | セッション自動削除 |
| sessions.retention_days | int | 90 | 保持日 |
| sessions.vacuum_after_prune | bool | True | プルーン後の VACUUM |
| sessions.min_interval_hours | int | 24 | 最小インターバル(時間) |

### logging

| 設定パス | 型 | 既定値 | 説明 |
|--|-|-|-|
| logging.level | str | `"INFO"` | ログレベル |
| logging.max_size_mb | int | 5 | maxサイズ(MB) |
| logging.backup_count | int | 3 | バックアップ数 |

## 環境変数リスト (OPTIONAL_ENV_VARS から抽出)

### Provider キー

| 環境変数 | カテゴリ | 説明 |
|--|-|-|
| OPENROUTER_API_KEY | provider | OpenRouter API key |
| ANTHROPIC_API_KEY | provider | Anthropic API key |
| ANTHROPIC_TOKEN | provider | Anthropic token |
| OPENAI_API_KEY | provider | OpenAI API key |
| GOOGLE_API_KEY | provider | Google AI Studio API key |
| GEMINI_API_KEY | provider | Gemini API key (alias for GOOGLE_API_KEY) |
| GEMINI_BASE_URL | provider | Gemini base URL override |
| XAI_API_KEY | provider | xAI API key |
| XAI_BASE_URL | provider | xAI base URL override |
| NVIDIA_API_KEY | provider | NVIDIA NIM API key |
| NVIDIA_BASE_URL | provider | NVIDIA NIM base URL override |
| LM_API_KEY | provider | LM Studio bearer token |
| LM_BASE_URL | provider | LM Studio base URL override |
| GLM_API_KEY | provider | Z.AI / GLM API key |
| ZAI_API_KEY | provider | Z.AI API key (alias for GLM_API_KEY) |
| Z_AI_API_KEY | provider | Z.AI API key (alias for GLM_API_KEY) |
| GLM_BASE_URL | provider | Z.AI base URL override |
| NOUS_API_KEY | provider | Nous Portal API key |
| NOUS_BASE_URL | provider | Nous Portal base URL override |
| DASHSCOPE_API_KEY | provider | Alibaba DashScope API key |
| DASHSCOPE_BASE_URL | provider | DashScope base URL |
| KIMI_API_KEY | provider | Kimi / Moonshot API key |
| KIMI_BASE_URL | provider | Kimi base URL override |
| KIMI_CN_API_KEY | provider | Kimi / Moonshot China API key |
| STEPFUN_API_KEY | provider | StepFun Step Plan API key |
| STEPFUN_BASE_URL | provider | StepFun base URL override |
| ARCEEAI_API_KEY | provider | Arcee AI API key |
| ARCEE_BASE_URL | provider | Arcee base URL override |
| GMI_API_KEY | provider | GMI Cloud API key |
| GMI_BASE_URL | provider | GMI Cloud base URL override |
| MINIMAX_API_KEY | provider | MiniMax API key (international) |
| MINIMAX_BASE_URL | provider | MiniMax base URL override |
| MINIMAX_CN_API_KEY | provider | MiniMax API key (China endpoint) |
| MINIMAX_CN_BASE_URL | provider | MiniMax (China) base URL override |
| DEEPSEEK_API_KEY | provider | DeepSeek API key |
| DEEPSEEK_BASE_URL | provider | DeepSeek base URL override |
| HERMES_QWEN_BASE_URL | provider | Qwen Portal base URL override |
| HERMES_GEMINI_CLIENT_ID | provider | Google OAuth client ID |
| HERMES_GEMINI_CLIENT_SECRET | provider | Google OAuth client secret |
| HERMES_GEMINI_PROJECT_ID | provider | GCP project ID |
| OPENCODE_ZEN_API_KEY | provider | OpenCode Zen API key |
| OPENCODE_ZEN_BASE_URL | provider | OpenCode Zen base URL override |
| OPENCODE_GO_API_KEY | provider | OpenCode Go API key |
| OPENCODE_GO_BASE_URL | provider | OpenCode Go base URL override |
| HF_TOKEN | provider | Hugging Face token |
| HF_BASE_URL | provider | HF Inference Providers base URL |
| OLLAMA_API_KEY | provider | Ollama Cloud API key |
| OLLAMA_BASE_URL | provider | Ollama Cloud base URL override |
| XIAOMI_API_KEY | provider | Xiaomi MiMo API key |
| XIAOMI_BASE_URL | provider | Xiaomi MiMo base URL override |
| AWS_REGION | provider | AWS region for Bedrock |
| AWS_PROFILE | provider | AWS profile for Bedrock |
| AZURE_FOUNDRY_API_KEY | provider | Azure Foundry API key |
| AZURE_FOUNDRY_BASE_URL | provider | Azure Foundry base URL |

### Web / Tool キー

| 環境変数 | カテゴリ | 説明 |
|--|-|-|
| BRAVE_SEARCH_API_KEY | web | Brave Search API key |
| SEARXNG_URL | web | SearXNG instance URL |
| TAVILY_API_KEY | web | Tavily API key |
| EXA_API_KEY | web | Exa Search API key |
| PARALLEL_API_KEY | web | Parallel API key |
| FIRECRAWL_API_KEY | web | Firecrawl API key |
| FIRECRAWL_API_URL | web | Firecrawl self-hosted URL |
| FIRECRAWL_GATEWAY_URL | web | Firecrawl gateway URL (Nous Subscribers) |
| TOOL_GATEWAY_DOMAIN | web | Tool-gateway domain suffix |
| TOOL_GATEWAY_SCHEME | web | Tool-gateway URL scheme |
| TOOL_GATEWAY_USER_TOKEN | web | Tool-gateway user token |
| BROWSERBASE_API_KEY | browser | Browserbase API key |
| BROWSERBASE_PROJECT_ID | browser | Browserbase project ID |
| BROWSER_USE_API_KEY | browser | Browser Use API key |
| AGENT_BROWSER_ENGINE | browser | Browser engine for local mode |
| CAMOFOX_URL | browser | Camofox browser server URL |
| FAL_KEY | media | FAL API key for image generation |
| TENOR_API_KEY | tool | Tenor API key for GIF search |
| TINKER_API_KEY | tool | Tinker API key for RL training |
| WANDB_API_KEY | tool | Weights & Biases API key |

### Platform / Messaging キー

| 環境変数 | カテゴリ | 説明 |
|--|-|-|
| TELEGRAM_TOKEN | platform | Telegram bot token |
| TELEGRAM_BOT_TOKEN | platform | Telegram bot token (alias) |
| TELEGRAM_ALLOWED_USERS | platform | Telegram allowed users |
| TELEGRAM_PROXY | platform | Telegram proxy |
| DISCORD_TOKEN | platform | Discord bot token |
| DISCORD_BOT_TOKEN | platform | Discord bot token (alias) |
| DISCORD_ALLOWED_USERS | platform | Discord allowed users |
| DISCORD_REPLY_TO_MODE | platform | Discord reply threading mode |
| SLACK_BOT_TOKEN | platform | Slack bot token |
| SLACK_APP_TOKEN | platform | Slack app token |
| WHATSAPP_MODE | platform | WhatsApp mode |
| WHATSAPP_ENABLED | platform | WhatsApp enabled |
| WHATSAPP_ALLOWED_USERS | platform | WhatsApp allowed users |
| MATTERMOST_URL | platform | Mattermost server URL |
| MATTERMOST_TOKEN | platform | Mattermost bot token |
| MATTERMOST_ALLOWED_USERS | platform | Mattermost allowed users |
| MATTERMOST_REQUIRE_MENTION | platform | Mattermost require mention |
| MATTERMOST_FREE_RESPONSE_CHANNELS | platform | Mattermost free response channels |
| MATRIX_HOMESERVER | platform | Matrix homeserver URL |
| MATRIX_ACCESS_TOKEN | platform | Matrix access token |
| MATRIX_USER_ID | platform | Matrix user ID |
| MATRIX_ALLOWED_USERS | platform | Matrix allowed users |
| MATRIX_REQUIRE_MENTION | platform | Matrix require mention |
| MATRIX_FREE_RESPONSE_ROOMS | platform | Matrix free response rooms |
| MATRIX_AUTO_THREAD | platform | Matrix auto thread |
| MATRIX_DM_AUTO_THREAD | platform | Matrix DM auto thread |
| MATRIX_DEVICE_ID | platform | Matrix device ID for E2EE |
| MATRIX_RECOVERY_KEY | platform | Matrix recovery key |
| BLUEBUBBLES_SERVER_URL | platform | BlueBubbles server URL |
| BLUEBUBBLES_PASSWORD | platform | BlueBubbles password |
| BLUEBUBBLES_ALLOWED_USERS | platform | BlueBubbles allowed users |
| IRC_SERVER | platform | IRC server hostname |
| IRC_CHANNEL | platform | IRC channel |
| IRC_NICKNAME | platform | IRC nickname |
| IRC_SERVER_PASSWORD | platform | IRC server password |
| IRC_NICKSERV_PASSWORD | platform | NickServ password |
| QQ_APP_ID | platform | QQ Bot App ID |
| QQ_CLIENT_SECRET | platform | QQ client secret |
| QQ_ALLOWED_USERS | platform | QQ allowed users |
| QQ_ALLOW_ALL_USERS | platform | QQ allow all users |
| QQ_GROUP_ALLOWED_USERS | platform | QQ group allowed users |
| QQ_SANDBOX | platform | QQ sandbox |
| QQBOT_HOME_CHANNEL | platform | QQBot home channel |
| QQBOT_HOME_CHANNEL_NAME | platform | QQBot home channel name |

### System / Gateway キー

| 環境変数 | カテゴリ | 説明 |
|--|-|-|
| HERMES_HOME | system | HERMES home directory |
| HERMES_MODEL | system | Model slug |
| HERMES_INFERENCE_PROVIDER | system | Provider |
| HERMES_MAX_ITERATIONS | system | Max iterations per conversation |
| HERMES_PREFILL_MESSAGES_FILE | system | Prefill messages file path |
| HERMES_EPHEMERAL_SYSTEM_PROMPT | system | Ephemeral system prompt |
| HERMES_LANGFUSE_PUBLIC_KEY | monitoring | Langfuse public key |
| HERMES_LANGFUSE_SECRET_KEY | monitoring | Langfuse secret key |
| HERMES_LANGFUSE_BASE_URL | monitoring | Langfuse server URL |
| SUDO_PASSWORD | system | Sudo password |
| API_SERVER_ENABLED | gateway | Enable API server |
| API_SERVER_KEY | gateway | API server key |
| API_SERVER_PORT | gateway | API server port |
| API_SERVER_HOST | gateway | API server host |
| API_SERVER_MODEL_NAME | gateway | API model name |
| GATEWAY_PROXY_URL | gateway | Gateway proxy URL |
| GATEWAY_PROXY_KEY | gateway | Gateway proxy key |
| GATEWAY_ALLOW_ALL_USERS | gateway | Gateway allow all users |
| WEBHOOK_ENABLED | webhook | Enable webhook |
| WEBHOOK_PORT | webhook | Webhook port |
| WEBHOOK_SECRET | webhook | Webhook HMAC secret |
| AIRTABLE_API_KEY | tool | Airtable token |
| NOTION_API_KEY | tool | Notion token |
| LINEAR_API_KEY | tool | Linear API key |
| HONCHO_API_KEY | memory | Honcho API key |
| VERCEL_TOKEN | tool | Vercel token |
