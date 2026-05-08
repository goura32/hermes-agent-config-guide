# memory / sessions / curator / delegation

## 本章で扱うこと

1. memory (永続メモリ)
2. sessions (セッション管理)
3. curator (スキル保守)
4. delegation (サブエージェント委任)
5. honcho (外部メモリ)

## memory 設定

### memory の設定項目

| キー | 型 | 既定値 | 説明 |
|--|-|-|-|
| memory_enabled | bool | True | メモリ有効 |
| memory_provider | str | "" | メモリプロバイダー |

### memory.enabled

- True (既定): メモリ機能が有効
- False: メモリ無効

### memory.memory_provider

- 空文字 (既定): デフォルトのプロバイダー（内部ストレージ）
- 指定: 外部メモリプロバイダー（例: "obsidian", "notion", "honcho" 等）

詳細は `website/docs/user-guide/features/memory-providers.md` を参照。

## sessions 設定

### sessions の設定項目

| キー | 型 | 既定値 | 説明 |
|--|-|-|-|
| sessions.auto_prune | bool | False | セッション自動削除 |
| sessions.retention_days | int | 90 | 保持日数 |
| sessions.vacuum_after_prune | bool | True | プルーン後の VACUUM |
| sessions.min_interval_hours | int | 24 | 最小インターバル（時間） |

### セッションの保存場所

- 保存先: `~/.hermes/state.db`（SQLite データベース）
- 全session history, tool call, FTS5索引を含む
- 大量のヘビーユーザー（68K+メッセージ）でも 384MB 以上になる

### sessions.auto_prune の意味

- True: ended セッションのみ保持
- False: すべてのセッションを保持（デフォルト）
- 注意: auto_prune=True は ended セッションのみ削除。active セッションは常に保持

## curator 設定

### curator の設定項目

| キー | 型 | 既定値 | 説明 |
|--|-|-|-|
| curator.enabled | bool | True | Curator 有効 |
| curator.interval_hours | int | 24*7（168） | Curator インターバル（時間） |
| curator.min_idle_hours | int | 2 | 最低アイドル時間 |
| curator.stale_after_days | int | 30 | stale 判定（日数） |
| curator.archive_after_days | int | 90 | archive 判定（日数） |
| curator.backup.enabled | bool | True | 自動バックアップ |
| curator.backup.keep | int | 5 | バックアップ保存数 |

### curator の動作

1. 利用可能な最初の curator パスにスキルをスキャン
2. long-unused スキルを stale としてマーク
3. 期限超過したスキルをアーカイブ（削除ではなくアーカイブ）
4. スキル重複を統合
5. スキル更新を検出すると削除

### .hermes/skills/.curator_backups/

- curator のバックアップは `.hermes/skills/.curator_backups/<date>/` に保存
- `hermes curator rollback` で復元可能

## delegation 設定

| キー | 型 | 既定値 | 説明 |
|--|-|-|-|
| delegation.max_iterations | int | 50 | 反復数（各サブエージェントの最大ツール呼び出し） |
| delegation.subagent_auto_approve | bool | False | サブエージェントの自動承認（注意：dangerous commands はデフォルト auto-deny） |
| delegation.max_concurrent_children | int | 3 | 並列数（各バッチの最大並列） |
| delegation.max_spawn_depth | int | 1 | 深さ制限（1=flat, 2=orchestrator→leaf, 3=3-level） |
| delegation.orchestrator_enabled | bool | True | オーケストレーター有効 |

### delegation.subagent_auto_approve の注意

- サブエージェントからの dangerous command approval はデフォルトで **auto-deny**
- `true` にすると auto-approve になり、audit message が出力される
- cron やバッチ処理で利用されるケースがある

## honcho 設定

| キー | 型 | 既定値 | 説明 |
|--|-|-|-|
| honcho | dict | `{}` | Honcho 設定（外部メモリ用） |

### honcho サブ設定

| キー | 型 | 既定値 | 説明 |
|--|-|-|-|
| honcho.apiKey | str | "" | Honcho API key |
| honcho.workspace | str | "" | Honcho ワークスペース |
| honcho.peerName | str | "" | ピア名 |
| honcho.session | str | "" | セッションID |

## よくある誤解

1. "sessions.auto_prune で active セッションも削除される" → ended セッションのみ。
2. "curator が古いスキルを削除する" → 削除ではなくアーカイブ。削除は手動。
3. "delegation.max_iterations はターン数" → 反復数（iteration count）。

## 設定例

### セッション履歴の長期保持

```yaml
sessions:
  auto_prune: False      # すべてのセッションを保持
```

### Curator の自動アーカイブ

```yaml
curator:
  enabled: True
  interval_hours: 24 * 7
  stale_after_days: 30
  archive_after_days: 90
```

## 公式ドキュメント上の根拠

- `website/docs/user-guide/features/memory.md`
- `website/docs/user-guide/features/memory-providers.md`
- `website/docs/user-guide/features/delegation.md`
- `website/docs/user-guide/sessions.md`

## ソースコード上の根拠

- `hermes_cli/config.py:1107` honcho section
- `hermes_cli/config.py:1354` onboarding section
- `hermes_cli/config.py:1328` sessions section
- `hermes_cli/config.py:1076` curator section
- `hermes_cli/config.py:993` delegation section
- `hermes_cli/config.py:1040` skills section
