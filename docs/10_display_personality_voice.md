# display / personality / voice / TTS / STT

## 本章で扱うこと

1. display 設定
2. personality / skin
3. voice mode (録音)
4. TTS (text-to-speech) 設定
5. STT (speech-to-text) 設定

## display 設定

### 主要設定項目

| キー | 型 | 既定値 | 説明 |
|--|-|-|-|
| display.compact | bool | False | 簡易表示 |
| display.resume_display | string | "full" | セッション再開時の表示 |
| display.busy_input_mode | string | "interrupt" | 使用中の入力モード |
| display.tui_auto_resume_recent | bool | False | TUI自動再開 |
| display.personality | string | "kawaii" | キャラクター |
| display.streamming | bool | False | ストリーミング表示 |
| display.final_response_markdown | string | "strip" | 最終レスポンスのフォーマット |
| display.persistent_output | bool | True | 永続出力 |
| display.persistent_output_max_lines | int | 200 | 永続出力の行数 |
| display.inline_diffs | bool | True | 差分表示 |
| display.show_cost | bool | False | 費用表示 |
| display.skin | string | "default" | スキン |
| display.tui_status_indicator | string | "kaomoji" | TUIインジケーター |
| display.language | string | "en" | UI言語 |
| display.interim_assistant_messages | bool | True | ガットウェイの中間メッセージ |
| display.tool_progress_command | bool | False | /verboseコマンド有効 |
| display.tool_preview_length | int | 0 | ツールプレビューの長さ |
| display.ephemeral_system_ttl | int | 0 | システム通知のTTL |
| display.runtime_footer.enabled | bool | False | runtime footer |
| display.runtime_footer.fields | list | ["model, "context_pct", "cwd"] | runtime footerのフィールド |
| display.copy_shortcut | string | "auto" | コピーショートカット |

### display.busy_input_mode

| 値 | 説明 |
|--|-|-|
| interrupt (既定) | 使用中にユーザーが送信した入力をインタープット |
| queue | 待機して順番に処理 |
| steer | ステアリングモード (UX) |

### display.personality

既定値の "kawaii" は組み込みのデフォルトのキャラクター。

### display.language

- **既定値**: "en"
- **対応言語**: en, zh, ja, de, es, fr, tr, uk
- **注意点**: エージェントのレスポンスには影響しない。静的なUIメッセージのみの影響。

### display.runtime_footer

- gatewayでターン終了時のメタデータフッター
- デフォルト: 無効
- enabled: true にすると、メッセージの最後に `model · 68% · ~/path` が表示

### display.copy_shortcut

| 値 | 説明 |
|--|-|-|
| auto | プラットフォームのデフォルト |
| ctrl_c | Ctrl+C |
| ctrl_shift_c | Ctrl+Shift+C |
| disabled | 無効 |

## personality / skin

### custom_personities

```yaml
personalities:
  my_char:
    name: "マイキャラクター"
    description: "技術的な質問に答えるAI"
    system_prompt: "あなたは技術的な質問に答え..."
    tone: "カジュアル"
    style: "簡潔に"
```

### skin

- `hermes_cli/skin_engine.py` で管理
- data-driven CLI theming
- `display.skin` でデフォルトのスキンを指定

## voice mode 設定

### voice の主要設定

| キー | 型 | 既定値 | 説明 |
|--|-|-|-|
| voice.record_key | string | "ctrl+b" | 録音キー |
| voice.max_recording_seconds | int | 120 | 最大録音時間 |
| voice.auto_tts | bool | False | TTS自動応答 |
| voice.beep_enabled | bool | True | ビープ音 |
| voice.silence_threshold | int | 200 | 静音閾値(0-32767) |
| voice.silence_duration | float | 3.0 | 静音検知秒数 |

## TTS 設定

### tts 主要設定

| キー | 既定値 | 説明 |
|--|-|-|
| tts.provider | "edge" | TTSプロバイター |

### TTS プロバイター一覧

| プロバイター | 費用 | 説明 |
|--|-|-|
| edge (既定) | 無料 | Edge browser TTS |
| elevenlabs | 有料 | エレメンラボTTS |
| openai | 有料 | OpenAI TTS |
| xai | 有料 | xAI TTS |
| minimax | 有料 | MiniMax TTS |
| mistral | 有料 | Mistral TTS |
| gemini | 有料 | Gemini TTS |
| neutts | 無料 | ローカル(ニューティーズ) |
| kittentts | 無料 | ローカル(キティTTS) |
| piper | 無料 | ローカル(Piper) |

### プロバイター別の設定

#### edge

| キー | 既定値 | 説明 |
|--|-|-|
| voice | "en-US-AriaNeural" | 音声 |

#### elevenlabs

| キー | 既定値 | 説明 |
|--|-|-|
| voice_id | "pNInz6obpgDQGcFmaJgB" | 音声ID (Adam) |
| model_id | "eleven_multilingual_v2" | モデル |

## STT 設定

### stt 主要設定

| キー | 既定値 | 説明 |
|--|-|-|
| stt.enabled | True | STT有効 |
| stt.provider | "local" | STTプロバイター |

### STT プロバイター

| プロバイター | 費用 | 説明 |
|--|-|-|
| local (既定) | 無料 | faster-whisper |
| groq | 有料 | Groq STT |
| openai | 有料 | OpenAI Whisper API |
| mistral | 有料 | Mistral Voxtral Transcribe |

### stt.local 設定

| キー | 既定値 | 説明 |
|--|-|-|
| stt.local.model | "base" | faster-whisperのモデル |
| stt.local.languae | "" | 言語 (自動検出) |

### faster-whisper モデル一覧

```
tiny, tiny.en, base, base.en, small, small.en,
medium, medium.en, large-v3, large
```

## 設定例

### カスタムキャラクター

```yaml
personalities:
  tech_expert:
    name: "技術専門家"
    system_prompt: "あなたは経験豊富な技術専門家..."
    tone: "プロフェッショナル"
    style: "簡潔に"
display:
  personality: "tech_expert"
```

### TTSを使用する場合

```yaml
tts:
  provider: "edge"
  edge:
    voice: "ja-JP-KazumiNeural"

stt:
  enabled: True
  provider: "local"
  local:
    model: "medium"
    language: "ja"

voice:
  auto_tts: True
```

## よくある誤解

1. "display.languageをjaにするとエージェントも日本語で話す" → UIのみ。エージェントのレスポンスはmodelやSOUL.mdに依存。
2. "TTSとSTTは自動で検出される" → スイッチして明示的に設定する必要がある。

## 公式ドキュメント上の根拠

- `website/docs/user-guide/configuration.md`
- `website/docs/user-guide/features/personality.md`
- `website/docs/user-guide/features/vision.md`
- `website/docs/user-guide/features/voice-mode.md`
- `website/docs/user-guide/features/tts.md`

## ソースコード上の根拠

- `hermes_cli/config.py:426` display section
- `hermes_cli/config.py:884` tts section
- `hermes_cli/config.py:930` stt section
- `hermes_cli/config.py:945` voice section
- `hermes_cli/skin_engine.py` スキンエンジン
