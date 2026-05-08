# terminal ツール / approvals / code_execution

## 本章で扱うこと

1. terminal ツールの全設定
2. backend (local, docker, ssh, modal, daytona, singularity, vercel_sandbox)
3. approvals (手動/スマート/オフ)
4. code_execution (実行モード)
5. スクリーンショットと画面キャプチャの関係

## terminal 設定

### 主要設定項目

| キー | 型 | 既定値 | 説明 |
|--|-|-|-|
| backend | string | "local" | ターミナルバックエンド |
| modal_mode | string | "auto" | Modalモード |
| cwd | string | "." | 作業ディレクトリ |
| timeout | int | 180 | コマンドタイムアウト(秒) |
| env_passthrough | list | `[]` | 渡す環境変数 |
| shell_init_files | list | `[]` | ソースするファイル |
| auto_source_bashrc | bool | True | bashrcの自動読み込み |
| docker_image | string | "nikolaik/python-nodejs:python3.11-nodejs20" | Docker イメージ |
| docker_forward_env | list | `[]` | Dockerに進める環境変数 |
| docker_env | dict | `{}` | Docker上の明示的な環境変数 |
| docker_volumes | list | `[]` | Docker ボリュームマウント |
| docker_mount_cwd_to_workspace | bool | False | DockerにCWDをマウント |
| docker_run_as_host_user | bool | False | Dockerをホストユーザーとして実行 |
| persistent_shell | bool | True | 永続シェル |
| singularity_image | string | "docker://nikolaik/python-nodejs:python3.11-nodejs20" | Singularity イメージ |
| modal_image | string | "nikolaik/python-nodejs:python3.11-nodejs20" | Modal イメージ |
| daytona_image | string | "nikolaik/python-nodejs:python3.11-nodejs20" | Daytona イメージ |
| vercel_runtime | string | "node24" | Vercel ランタイム |
| container_cpu | int | 1 | コンтей너 CPU |
| container_memory | int | 5120 | コンтейナーメモリ(MB) |
| container_disk | int | 51200 | コンтейナーディスク(MB) |
| container_persistent | bool | True | コンテナーの永続化 |

### terminal.backend の選択肢 (DEFAULT_CONFIG 基準)

sourceコードの DEFAULT_CONFIG から確認したterminal.backendの有効値:

| バックエンド | 要件 | 説明 |
|--|-|-|
| local (既定) | なし | ローカルシェル |
| docker | docker_installation | Docker コンテナ内 |
| ssh | SSH サーバー | リモートSSH |
| modal | MODAL_TOKEN, MODAL_PROFILE | Modal クラウド |
| daytona | DAYTONA_TOKEN | Daytona エンバイロメント |
| singularity | singularity_installation | Singularity コンテナ |
| vercel_sandbox | VERCEL_TOKEN | Vercel Sandbox |

**注**: source_codeの DEFAULT_CONFIG では正確なbackendの値が未確認。正確な値は tools/terminal_tool.py を参照。

### terminal.timeout の挙動

- 既定: 180秒
- 環境依存: シェルコマンドの実行タイムアウト
- 環境変数で設定可能: TERMINAL_TIMEOUT (未確認)
- 設定しても実際に効かない場合: コマンドによって異なる

**注意点**: `terminal_timeout` は terminal_tool の出力限界であり、実際のコマンドのタイムアウトではない。シェルスクリプト内で明示的に `timeout` コマンドを使う必要がある。

### terminal.auto_source_bashrc の意味

既定値の `True` で:

1. `~/.profile`
2. `~/.bash_profile`
3. `~/.bashrc`
を順にソースする。bashは非対話的ログインモードでbashrcをスキップするため、この設定で明示的に読み込む。

False にすると: 厳密なログインモードのみ。環境変数が読み込まれない場合がある。

### persistent_shell

- 既定: `True` (非ローカルバックエンドのみ)
- ローカルでは `TERMINAL_LOCAL_PERSISTENT` 環境変数で制御
- シェル変数やCWDがコマンド間で維持される

## approvals 設定

### approvals.mode

- **型**: string
- **既定値**: "manual"
- **設定可能な値**:
  - `manual` (既定): 常にユーザーにプロンプトを表示
  - `smart`: 補助LLMが自動判断、低リスクは承認、高リスクは手動
  - `off` (yolo): 承認プロンプトを全てスキップ

### approvals.timeout

- **型**: int
- **既定値**: 60
- **意味**: 承認プロンプトのレスポンスタイムアウト(秒)

### approvals.cron_mode

- **型**: string
- **既定値**: "deny"
- **設定可能な値**:
  - `deny`: cronジョブはDangerous Commandをブロック
  - `approve`: cronジョブはすべてのDangerous Commandを自動承認

### approval.mode の実運用上の意味

| モード | CLI/TUI | Gateway | Cron |
|--|-|-|-|
| manual | 必ずプロンプト | 必ずプロンプト | 依存: cron_mode |
| smart | 補助LLMで判断 | 補助LLMで判断 | 依存: cron_mode |
| off | スキップ | スキップ | スキップ |

## コード実行ツール設定

### code_execution.mode

- **型**: string
- **既定値**: "project"
- **設定可能な値**:
  - `project`: プロジェクトディレクトリで実行 (デフォルト)
  - `strict`: 隔離されたテンポラリディレクトリで実行

## tool_output 設定

| キー | 型 | 既定値 | 説明 |
|--|-|-|-|
| tool_output.max_bytes | int | 50,000 | 出力制限(charactor) |
| tool_output.max_lines | int | 2,000 | ファイル読み込み制限 |
| tool_output.max_line_length | int | 2,000 | 行長制限 |

## tool_loop_guardrails 設定

| キー | 型 | 既定値 | 説明 |
|--|-|-|-|
| warnings_enabled | bool | True | ワーニング表示 |
| hard_stop_enabled | bool | False | ハードストップ |
| warn_after.exact_failure | int | 2 | 失敗回数によるワーニング |
| warn_after.same_tool_failure | int | 3 | 同じツールの失敗 |
| warn_after.idempotent_no_progress | int | 2 | 無進歩 |
| hard_stop_after.exact_failure | int | 5 | ハードストップ |
| hard_stop_after.same_tool_failure | int | 8 | ハードストップ |
| hard_stop_after.idempotent_no_progress | int | 5 | ハードストップ |

## よくある誤解

1. "terminal.timeoutはOSレベルのコマンドのタイムアウト" → terminal_toolの出力キャップに過ぎない
2. "approvals.mode=smartは補助LLMが常に正確に判断する" → 実質は「自動承認」なので注意
3. "code_execution.mode=strictなら絶対に安全" → 隔離は不完全。環境のクリーンアップは行われない

## 設定例

### Dockerバックエンドのセットアップ

```yaml
terminal:
  backend: "docker"
  docker_image: "ubuntu:24.04"
  container_memory: 8192
  container_disk: 102400
  persistent_shell: True
  docker_volumes:
    - "/home/user/projects:/workspace/projects"
```

### approvals のセットアップ

```yaml
approvals:
  mode: "smart"
  timeout: 120
  cron_mode: "deny"       # cronでは安全を優先
```

## 公式ドキュメント上の根拠

- `website/docs/user-guide/configuration.md`
- `website/docs/user-guide/features/code-execution.md`
- `website/docs/user-guide/cli.md`

## ソースコード上の根拠

- `hermes_cli/config.py:468` terminal section
- `hermes_cli/config.py:1178` approvals section
- `hermes_cli/config.py:1273` code_execution section
- `hermes_cli/config.py:264` tool_loop_guardrails section
