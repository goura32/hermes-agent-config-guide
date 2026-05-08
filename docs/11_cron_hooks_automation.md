# cron / kanban / hooks / automation

## 本章で扱うこと

1. cron ジョブの設定
2. kanban マルチエージェントの設定
3. hooks スクリプトフックの設定

## cron ジョブ設定

### cron の設定項目

| キー | 型 | 既定値 | 説明 |
|--|-|-|-|
| cron.wrap_response | bool | True | 応答のラップ |
| cron.max_parallel_jobs | int/None | None (=無制限) | 並列ジョブ数 |

### cron.wrap_response

- True: ジョブ完了時の応答をヘッダーとフッターで包む
- False: クリーンな出力

### cron.max_parallel_jobs

- None/0: 無制限
- 1: シリアル (v0.9以前の動作)
- N: 最大N並列

## kanban 設定

### kanban の設定項目

| キー | 型 | 既定値 | 説明 |
|--|-|-|-|
| kanban.dispatch_in_gateway | bool | True | ゲートウェイ内でディスパッチ |
| kanban.dispatch_interval_seconds | int | 60 | ディスパッチ間隔(秒) |
| kanban.failure_limit | int | 2 | 連続失敗でブロックする回数 |

### kanban.dispatch_in_gateway

- True: ゲートウェイプロセス内でディスパッチ (デフォルト、コストは約300μs)
- False: 別のsystemdユニットでディスパッチ

## hooks 設定

### hooks の形式

```yaml
hooks:
  <event_name>:  # pre_tool_call, post_tool_call, etc.
    - matcher: <event_matcher>
      command: <shell_command>
      timeout: <timeout_in_seconds>
```

### hooks_auto_accept

| キー | 型 | 既定値 | 説明 |
|--|-|-|-|
| hooks_auto_accept | bool | False | フックの自動承認 |

## automationのよくある設定

### cronの定期実行例

```yaml
# .envファイルなどで設定
# cron ジョブの設定は CLI スラッシュコマンド で行う
# /cron schedule "0 9 * * *" --prompt "朝の天気予報をチェック"
```

### kanbanの設定例

```yaml
kanban:
  dispatch_in_gateway: True
  dispatch_interval_seconds: 30
  failure_limit: 3
```

## hooksのよくある使用例

### pre_tool_call フック

```yaml
hooks:
  pre_tool_call:
    - matcher:
        tool: "terminal"
      command: "echo '{tool_name} {tool_args}' >> ~/hooks/pre_tool_call.log"
      timeout: 10
```

### post_tool_call フック

```yaml
hooks:
  post_tool_call:
    - matcher:
        tool: "terminal"
        exit_code: 0
      command: "echo '{tool_output}' >> ~/hooks/post_tool_call.log"
      timeout: 10
```

## よくある誤解

1. "cronジョブの設定はconfig.yamlで直接行う" → CLIスラッシュコマンドで管理。config.yamlはcron自体の設定のみ。
2. "hooksはエージェントの処理に影響を与えない" → フックはエージェント処理の前後で実行され、環境状態を変更できる。

## 公式ドキュメント上の根拠

- `website/docs/guides/automation-templates.md`
- `website/docs/user-guide/features/cron.md`
- `website/docs/user-guide/features/kanban.md`
- `website/docs/user-guide/features/hooks.md`

## ソースコード上の根拠

- `hermes_cli/config.py:1239` cron section
- `hermes_cli/config.py:1256` kanban section
- `hermes_cli/config.py:1212` hooks section
- `hermes_cli/config.py:1218` hooks_auto_accept
- `cron/scheduler.py` cronスケージューラー
- `cron/jobs.py` cronジョブ定義
