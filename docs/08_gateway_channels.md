# gateway / チャネル設定

## 本章で扱うこと

1. gateway 実行の設定
2. Discord プラットフォーム設定
3. Telegram プラットフォーム設定
4. Slack プラットフォーム設定
5. WhatsApp プラットフォーム設定
6.その他のメッセンジャー (Signal, Matrix, Feishu, WeChat等)

## gateway実行

### agent セクションの gateway関連設定

| キー | 型 | 既定値 | 説明 |
|--|-|-|-|
| agent.max_turns | int | 90 | 最大ターン |
| agent.gateway_timeout | int | 1800 | gatewayアイドルタイムアウト(秒) |
| agent.restart_drain_timeout | int | 180 | 再起動時ドロンタイムアウト(秒) |
| agent.api_max_retries | int | 3 | APIリトライ |
| agent.service_tier | string | "" | サービスティア |
| agent.tool_use_enforcement | string/List | "auto" | ツール使用の強制 |
| agent.gateway_timeout_warning | int | 900 | タイムアウトワーニング(秒) |
| agent.gateway_notify_interval | int | 180 | 通知間隔(秒) |
| agent.gateway_auto_continue_freshness | int | 3600 | 自動継続の新規性(秒) |
| agent.image_input_mode | string | "auto" | 画像入力モード |
| agent.disabled_toolsets | list | `[]` | 無効にするツールセット |

### agent.gateway_timeout の意味

- gateway のアイドルタイムアウト
- 30分(1800秒)後にエージェントが停止
- 空いている間もAPI呼び出ししている場合は継続
- 0 = 無制限

### agent.tool_use_enforcement の値

| 値 | 挙動 |
|--|-|-|
| "auto" (既定) | gpt/codex モデルに適用 |
| true | 全てのモデルに適用 |
| false | 適用しない |
| ["gpt", "codex"] | 指定されたモデルにのみ適用 |

## Discord 設定

### discord の設定項目

| キー | 型 | 既定値 | 説明 |
|--|-|-|-|
| discord.require_mention | bool | True | サーバーチャンネルでメンション必要 |
| discord.free_response_channels | string | "" | メンション不要のチャンネルID |
| discord.allowed_channels | string | "" | 応答するチャンネルのホワイトリスト |
| discord.auto_thread | bool | True | @mentionでスレッド自動作成 |
| discord.reactions | bool | True | 処理中リアクション |
| discord.channel_prompts | dict | `{}` | チャンネル固有のプロンプト |
| discord.dm_role_auth_guild | string | "" | DMのロール認証ギルドID |
| discord.server_actions | string | "" | エージェントのサーバーアクション |

### Discord 環境変数

| 環境変数 | 説明 |
|--|- |
| DISCORD_TOKEN | ボットトークン |
| DISCORD_HOME_CHANNEL | ホームチャンネルID |
| DISCORD_HOME_CHANNEL_NAME | ホームチャンネル名 |

### Discord 設定例

```yaml
discord:
  require_mention: True
  free_response_channels: "1234567890,0987654321"
  auto_thread: True
  reactions: True
  channel_prompts:
    "1234567890": "你是一个技术专家，请用中文回答"
```

## Telegram 設定

### telegram の設定項目

| キー | 型 | 既定値 | 説明 |
|--|-|-|-|
| telegram.reactions | bool | False | 処理中リアクション |
| telegram.channel_prompts | dict | `{}` | チャット/トピック固有のプロンプト |
| telegram.allowed_chats | string | "" | 応答するチャットIDのホワイトリスト |

### Telegram 環境変数

| 環境変数 | 説明 |
|--|-|
| TELEGRAM_TOKEN | ボットトークン |
| TELEGRAM_HOME_CHANNEL | ホームチャットID |
| TELEGRAM_HOME_CHANNEL_NAME | ホームチャット名 |

### Telegram 設定例

```yaml
telegram:
  reactions: True
  channel_prompts:
    "-1001234567890": "请使用中文交流"
```

## Slack 設定

### slack の設定項目

| キー | 型 | 既定値 | 説明 |
|--|-|-|-|
| slack.require_mention | bool | True | チャンネルでメンション必要 |
| slack.free_response_channels | string | "" | メンション不要のチャンネルID |
| slack.allowed_channels | string | "" | 応答するチャンネルのホワイトリスト |
| slack.channel_prompts | dict | `{}` | チャンネル固有のプロンプト |

### Slack 環境変数

| 環境変数 | 説明 |
|--|-|
| SLACK_BOT_TOKEN | ボットトークン(xoxb-) |
| SLACK_APP_TOKEN | アプリトークン(xapp-) |
| SLACK_HOME_CHANNEL | ホームチャンネル |
| SLACK_HOME_CHANNEL_NAME | ホームチャンネル名 |

### Slack 設定例

```yaml
slack:
  require_mention: True
  free_response_channels: "C1234567890"
```

## WhatsApp 設定

### WhatsApp 環境変数

| 環境変数 | 説明 |
|--|-|
| WHATSAPP_MODE | mode ("self-hosted", "cloud") |
| WHATSAPP_ENABLED | enabled flag |
| WHATSAPP_ALLOWED_USERS | 許可ユーザー |

### WhatsApp 設定項目

| キー | 型 | 既定値 | 説明 |
|--|-|-|-|
| whatsapp.reply_prefix | string | "" | 返信プレフィックス |

## その他のメッセンジャー

### Signal

| 環境変数 | 説明 |
|--|-|
| SIGNAL_ACCOUNT | アカウント |
| SIGNAL_HTTP_URL | HTTP URL |
| SIGNAL_ALLOWED_USERS | 許可ユーザー |
| SIGNAL_GROUP_ALLOWED_USERS | グループ許可ユーザー |
| SIGNAL_HOME_CHANNEL | ホームチャット |
| SIGNAL_HOME_CHANNEL_NAME | ホームチャット名 |

### Matrix

| 環境変数 | 説明 |
|--|-|
| MATRIX_PASSWORD | パスワード |
| MATRIX_ENCRYPTION | 暗号化 |
| MATRIX_DEVICE_ID | デバイスID |
| MATRIX_HOME_ROOM | ホームルーム |
| MATRIX_REQUIRE_MENTION | メンション必要 |
| MATRIX_FREE_RESPONSE_ROOMS | 自由応答ルーム |
| MATRIX_DM_AUTO_THREAD | DMスレッド自動 |

### Feishu (飛書)

| 環境変数 | 説明 |
|--|-|
| FEISHU_APP_ID | アプリID |
| FEISHU_APP_SECRET | アプリシークレット |
| FEISHU_ENCRYPT_KEY | 暗号化キー |
| FEISHU_VERIFICATION_TOKEN | 検証トークン |
| FEISHU_HOME_CHANNEL | ホームチャット |
| FEISHU_HOME_CHANNEL_NAME | ホームチャット名 |

### WeChat (微信)

| 環境変数 | 説明 |
|--|-|
| WEIXIN_ACCOUNT_ID | アカウントID |
| WEIXIN_TOKEN | トークン |
| WEIXIN_BASE_URL | ベースURL |
| WEIXIN_HOME_CHANNEL | ホームチャット |
| WEIXIN_DM_POLICY | DMポリシー |
| WEIXIN_GROUP_POLICY | グループポリシー |

### その他のメッセンジャー

- **Wecom (企業微信)**: WECOM_BOT_ID, WECOM_SECRET
- **DingTalk (釘釘)**: DINGTALK_CLIENT_ID, DINGTALK_CLIENT_SECRET
- **QQ/QQBot**: QQ_APP_ID, QQ_CLIENT_SECRET
- **BlueBubbles**: BLUEBUBBLES_SERVER_URL, BLUEBUBBLES_PASSWORD
- **Mattermost**: MATTERMOST_HOME_CHANNEL
- **Yuanbao**: YUANBAO_HOME_CHANNEL

## チャネル固有の表示設定

`display.platforms` でプラットフォーム毎の表示設定を上書き:

```yaml
display:
  platforms:
    telegram:
      tool_progress: "off"    # Telegramでは進行状況を表示しない
      runtime_footer:
        enabled: true    # footerを表示
    discord:
      tool_progress: "new" # Discordで新しいメッセージのみ表示
    slack:
      tool_progress: "all"
```

## よくある誤解

1. "telegram.reactions = True の場合リアクションのステータスが更新される" = 実質的に新しいメッセージへのリアクションが追加されるだけ
2. "slack.allowed_channelsのチャンネルIDはチャンネル名" → CHANNEKID (Cで始まる)である。名前ではない。
3. "whatsapp.reply_prefix を設定するとヘッダーが追加される" → `""` (空文字列) を設定するとヘッダーが**削除**される

## 設定例

### 全メッセンジャーの連携

```yaml
telegram:
  reactions: True
  channel_prompts:
    "-1001234567890": "请用中文交流"

discord:
  auto_thread: True
  reactions: True

slack:
  require_mention: True
  allowed_channels: "C12345678, C98765432"
```

## 公式ドキュメント上の根拠

- `website/docs/user-guide/messaging/index.md`
- `website/docs/user-guide/messaging/telegram.md`
- `website/docs/user-guide/messaging/discord.md`
- `website/docs/user-guide/messaging/slack.md`
- `website/docs/user-guide/messaging/whatsapp.md`

## ソースコード上の根拠

- `hermes_cli/config.py:1117` slack section
- `hermes_cli/config.py:1125` discord section
- `hermes_cli/config.py:1156` telegram section
- `hermes_cli/config.py:1149` whatsapp section
- `hermes_cli/config.py:1614` additional platform sections
