# 設定の優先順位

## 優先順位体系

Hermes Agent は下層の設定を上層の値で上書きする。

```
優先度 低 ──────────────────▶ 高

1. ソース内 DEFAULT_CONFIG
      │
      │ load_config() による deep_merge
      ▼
2. config.yaml 上の設定
      │
      │ _expand_env_vars() による環境変数展開
      ▼
3. 環境変数 (config.yaml 内の ${VAR} 展開)
      │
      │ .env ファイルの読み込み
      ▼
4. ~/.hermes/.env
      │
      │ プロファイル固有 env の注入
      ▼
5. プロファイルenv ({HERMES_HOME}/profile.env)
      │
      │ プラットフォーム固有 env の注入
      ▼
6. プラットフォーム固有 env
      │
      │ CLI引数
      ▼
7. CLI引数 (最優先)
```

## 各層の詳細

### Layer 1: ソース内 DEFAULT_CONFIG (最低優先度)

`hermes_cli/config.py` の `DEFAULT_CONFIG` に全設定項目の既定値が定義される。

**重要**: これは「デフォルト」であり、ユーザーが設定しない場合に適用される値。

**既定値の主要項目**:
```yaml
model: ""                              # 空 → auto-detect
providers: {}
toolsets: ["hermes-cli"]
agent.max_turns: 90
agent.gateway_timeout: 1800            # 30分
agent.api_max_retries: 3
terminal.backend: "local"
terminal.timeout: 180
web.backend: ""                        # 空 → バックエンド自動選択
web.search_backend: ""
web.extract_backend: ""
browser.inactivity_timeout: 120
browser.command_timeout: 30
compression.enabled: True
compression.threshold: 0.50
tts.provider: "edge"                   # 無料
tts.edge.voice: "en-US-AriaNeural"
stt.enabled: True
stt.provider: "local"
stt.local.model: "base"
voice.record_key: "ctrl+b"
voice.max_recording_seconds: 120
display.personality: "kawaii"
display.language: "en"
display.tui_status_indicator: "kaomoji"
checkpoints.enabled: False
checkpoints.auto_prune: True
approvals.mode: "manual"
approvals.cron_mode: "deny"
cron.wrap_response: True
kanban.dispatch_in_gateway: True
security.redact_secrets: True
memory.memory_enabled: True
delegation.max_iterations: 50
```

### Layer 2: config.yaml (~/.hermes/config.yaml)

ユーザーが直接編集する設定ファイル。DEFAULT_CONFIG と deep_merge される。

**マージルール**:
- config.yaml に存在するキーのみ上書き
- 存在しないキーは DEFAULT_CONFIG の値を保持
- 入れ子構造も再帰的にマージ
- 配列は置換（マージなし）

**例**:
```yaml
# config.yaml
model:
  default: "google/gemini-3-flash-preview"
  provider: openrouter

terminal:
  timeout: 300
  backend: "docker"
```

この場合:
- `model.default` → "google/gemini-3-flash-preview" (上書き)
- `model.provider` → "openrouter" (上書き)
- `model.base_url` → "" (DEFAULT_CONFIGの値を保持)
- `agent.max_turns` → 90 (DEFAULT_CONFIGの値を保持)
- `terminal.timeout` → 300 (上書き)
- `terminal.docker_image` → "nikolaik/python-nodejs:..." (保持)

### Layer 3: 環境変数 (config.yaml 内の ${VAR})

config.yaml 内の値が環境変数で展開される:

```yaml
# config.yaml
providers:
  custom1:
    api_key: "${CUSTOM_API_KEY}"     # ← 環境変数で展開
    base_url: "${CUSTOM_BASE_URL}"
```

env_loader.py は以下のように.envファイルを自動でロードする:

```
優先高 ──────────▶ 優先低

ユーザー ~/.hermes/.env
      │
      ▼
プロジェクト .env (開発フォールバック)
```

### Layer 4: CLI引数 (最優先)

CLI引数はプロセス単位で一時的に設定を Overrides:

```bash
# model を設定
hermes model /google/gemini-3-flash-preview

# provider を設定
hermes --provider openrouter

# yoloモード (approvals.off)
hermes --yolo

# checkpoint有効化
hermes --checkpoints

# toolsetの制限
hermes --toolsets web,browser
```

## model/provider の解決順

### model.default の解決

```
1. CLI: /model <slug>
2. .env: HERMES_MODEL
3. config.yaml: model.default
4. providerのデフォルトモデル
5. auto-detect-best (利用可能な最初のもの)
6. None (エラー)
```

### model.provider の解決

```
1. CLI: /provider <name>
2. .env: HERMES_INFERENCE_PROVIDER
3. config.yaml: model.provider
4. 利用可能な最初のプロバイダー (auto-detect)
5. None
```

### 各プロバイダーの対応

| プロバイダー | 必須環境変数 |
|--|-|
| openrouter | OPENROUTER_API_KEY |
| anthropic | ANTHROPIC_API_KEY |
| nous | NOUS_API_KEY (OAuth) |
| google/gemini | GOOGLE_API_KEY or GEMINI_API_KEY |
| openai-codex | OPENAI_API_KEY (OAuth) |
| copilot | COPILOT_API_TOKEN |
| copilot-acp | COPILOT_API_TOKEN |
| zai | GLM_API_KEY or ZAI_API_KEY |
| kimi-coding | KIMI_API_KEY |
| bedrock | AWS credentials |
| custom/custom_providers | 各自の API key |

## auxiliary モデル解決

各 auxiliary タスクの provider 値:

| 値 | 挙動 |
|--|-|
| auto | 利用可能な最初のプロバイターを自動選択 |
| main | メインチャットモデルを使用 |
| custom | 任意の名前付きプロバイター (config.yaml に定義) |
| openrouter/anthropic | 特定のプロバイターを使用 |
| "" | mainと同様 |

**fallback**: どの auxiliary タスクも設定したプロバイターが利用不可の場合、`openrouter:google/gemini-3-flash-preview` にフォールバック。

### main vs auto の違い

- `main`: 常にメインチャットモデルにルーティング。メインモデルがauxiliaryに最適でない場合は遅い/高額。
- `auto`: 軽量なモデル(通常 gpt-4o-mini や gemini-flash)を自動選択。コスト削減に最適。

### auto で選択アルゴリズム

**sourceコードでの正確な順序は agent/model_metadata.py で定義。**
このファイルでの provider のビルド順が決定する。一般的な優先順位としては:

1. anthropic (ANTHROPIC_API_KEY / ANTHROPIC_TOKEN)
2. openrouter (OPENROUTER_API_KEY)
3. google-gemini-cli (GOOGLE_API_KEY or GEMINI_API_KEY)
4. openai-codex (OPENAI_API_KEY, OAuth)
5. copilot (COPILOT_API_TOKEN)
6. その他の組み込みプロバイダー

**注**: 正確な順序は agent/model_metadata.py を参照。このファイルで定義される組み込みプロバイダーの優先順位が auto-detect の決定要因となる。

## timeout の解決

設定から取得 → 環境変数で展開 → 実装で適用。

### 主要な timeout 項目

| 設定項目 | 既定値 | 環境変数 | 対象 |
|--|-|-|-|
| provider.request_timeout_seconds | プロバイダー依存 | 未確認 | 各API呼び出し |
| provider.stale_timeout_seconds | 未確認 | 未確認 | ストリーム stale |
| auxiliary.vision.timeout | 120s | 未確認 | 画像分析 |
| auxiliary.vision.download_timeout | 30s | 未確認 | 画像DL |
| auxiliary.web_extract.timeout | 360s | 未確認 | web要約 |
| auxiliary.compression.timeout | 120s | 未確認 | コンテキスト圧縮 |
| auxiliary.session_search.timeout | 30s | 未確認 | セッション検索 |
| auxiliary.title_generation.timeout | 30s | 未確認 | タイトル生成 |
| auxiliary.curator.timeout | 600s | 未確認 | スキルキューレター |
| agent.gateway_timeout | 1800s | HERMES_AGENT_TIMEOUT? | gatewayタイムアウト |
| agent.gateway_notify_interval | 180s | HERMES_AGENT_NOTIFY_INTERVAL? | 通知間隔 |
| agent.api_max_retries | 3 | 未確認 | APIリトライ |
| terminal.timeout | 180s | 未確認 | 端末出力 |
| browser.inactivity_timeout | 120s | 未確認 | ブラウザーアイドル |
| browser.command_timeout | 30s | 未確認 | ブラウザーコマンド |
| browser.dialog_timeout_s | 300s | 未確認 | ダイアログ自動Dismiss |
| approvals.timeout | 60s | 未確認 | 承認タイムアウト |
| cron (scheduler) | 未確認 | 未確認 | cronジョブ |
| delegation.child_timeout_seconds | 600s | 未確認 | サブエージェント |

**注意**: HERMES_STREAM_READ_TIMEOUT, HERMES_STREAM_STALE_TIMEOUT, HERMES_API_TIMEOUT, HERMES_API_CALL_STALE_TIMEOUT, HERMES_AGENT_TIMEOUT, HERMES_AGENT_NOTIFY_INTERVAL は config.py の OPTIONAL_ENV_VARS または REQUIRED_ENV_VARS には定義されていない。これらの環境変数が実際にどこで参照されているかは run_agent.py や agent の実装を直接調査する必要がある。

## provider 解決の流れ

```
設定値取得
    │
    ▼
CLI引数 (/provider) → プロバイター直接使用
    │
    ▼
環境変数 (HERMES_INFERENCE_PROVIDER) → 直接使用
    │
    ▼
config.yaml (model.provider) → 直接使用
    │
    ▼
空文字列 → auto-detect (利用可能な最初のもの)
    │
    ▼
None → エラー
```

**注意**: `custom` は予約語ではなく、任意の名前付きプロバイターとしての使い方。

## auto/main/custom の違い

| 設定値 | 意味 | クレデンシャル | 費用目安 |
|--|-|-|--|
| auto | 軽量モデルを自動選択 | mainと同じ | 低 (flashモデル) |
| main | メインチャットモデルを再利用 | mainと同じ | 高 (mainと同じ) |
| custom | 名前付きプロバイター | そのプロバイターのもの | 各プロバイター次第 |

## 設定項目間の依存関係

### 必須の依存関係

| 設定 | 依存 |
|--|-|
| model.default | model.provider |
| auxiliary.*.provider | 利用可能なプロバイター |
| terminal.backend | backend 固有の設定 |
| browser.provider | browser/backend 依存の設定 |
| cron.enabled | クレデンシャル |
| delegation.* | 親のプロバイター |
| discord.* | DISCORD_TOKEN |
| telegram.* | TELEGRAM_TOKEN |
| slack.* | SLACK_BOT_TOKEN |

### 相互影響する設定

| 影響関係 | 説明 |
|--|-|
| compression.enabled + auxiliary.compression.timeout | compression が enabled=false の場合 auxiliary.compression は無効 |
| model.provider + provider.api_key | 対応する API キーが必要 |
| web.backend + web.search_backend | search_backend は backend の設定を上書き |
| web.backend + web.extract_backend | extract_backend は backend の設定を上書き |
| approvals.mode + approvals.cron_mode | Cronではcron_modeが優先 |
| display.language + platform | プラットフォーム固有の上書きが可能 |

## バージョン差分

### version 23 → 22→ 21 → 20 → 19→ 18→ 17 → 16→ 15 → 14→ 13→ 12 → 11→ 10→ 9 → 8→ 7→ 6→ 5→ 4→ 3→ 2→ 1

version 12: `custom_providers`(list) → `providers`(dict) にマイグレーション
version 17: `compression.summary_*` → `auxiliary.compression` にマイグレーション
version 21: プラグインがオプトインに変更
version 22~23: curator デフォルトのシード

## 設定ファイルと環境変数の優先順位 (まとめ)

```
環境変数 > config.yaml > DEFAULT_CONFIG

ただし:
- .env ファイル内の値は config.yaml 内の ${VAR} に展開
- CLI引数は .env よりも優先
- model.default のデフォルトは空文字列 (auto-detect)
- 環境変数が存在しない場合、config.yaml に記述される
```
