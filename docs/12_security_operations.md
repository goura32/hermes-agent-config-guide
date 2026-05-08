# security / operations

## 本章で扱うこと

1. security 設定
2. privacy 設定
3. checks points
4. compression 設定
5. session 管理
6. logging 設定

## security 設定

### security の設定項目

| キー | 型 | 既定値 | 説明 |
|--|-|-|-|
| security.allow_private_urls | bool | False | 内部IPの許可 |
| security.redact_secrets | bool | True | シークレットのマスク |
| security.tirith_enabled | bool | True | Tirithセキュリティスキャン |
| security.tirith_path | string | "tirith" | Tirith パス |
| security.tirith_timeout | int | 5 | Tirithタイムアウト(秒) |
| security.tirith_fail_open | bool | True | Tirith fail-open |
| security.website_blocklist.enabled | bool | False | ウェブサイトブロック |
| security.website_blocklist.domains | list | [] | ブロックドメイン |
| security.website_blocklist.shared_files | list | [] | 共有ファイル |

### security.allow_private_urls

- False (既定): 内部IPへのアクセスをブロック
- True: 内部IPを許可 (OpenWrt, proxy, VPN使用時)

### security.tirith_fail_open

- True (既定): Tirithがタイムアウトしても失敗としない
- False: Tirithが失敗した場合はコマンドをブロック

## privacy 設定

### privacy の設定項目

| キー | 型 | 既定値 | 説明 |
|--|-|-|-|
| privacy.redact_pii | bool | False | PIIのマスク |

### privacy.redact_pii

- False (既定): PIIをそのまま保持
- True: ユーザーIDをハッシュ化、電話番号をマスク

## checkpoints 設定

### checkpoints の設定項目

| キー | 型 | 既定値 | 説明 |
|--|-|-|-|
| checkpoints.enabled | bool | False | チェックポイント有効 |
| checkpoints.max_snapshots | int | 20 | 最大スナップショット数 |
| checkpoints.max_total_size_mb | int | 500 | 最大総サイズ(MB) |
| checkpoints.max_file_size_mb | int | 10 | 最大ファイルサイズ(MB) |
| checkpoints.auto_prune | bool | True | 自動プルーン |
| checkpoints.retention_days | int | 7 | 保持日数 |
| checkpoints.delete_orphans | bool | True | ゾンリの削除 |
| checkpoints.min_interval_hours | int | 24 | 最小インターバル(時間) |

### checkpoints の使用

- `/checkpoint` で手動スナップショット
- `/rollback` で復元
- destructive file ops (write_file, patch) の前に自動取得

## compression 設定

### compression の設定項目

| キー | 型 | 既定値 | 説明 |
|--|-|-|-|
| compression.enabled | bool | True | コンテキスト圧縮有効 |
| compression.threshold | float | 0.50 | 圧縮の閾値 (50%) |
| compression.target_ratio | float | 0.20 | 保持比率 (20%) |
| compression.protect_last_n | int | 20 | 圧縮しない最後のメッセージ |
| compression.hygine_hard_message_limit | int | 400 | ハードリミットのメッセージ数 |

### compression.enabled 無効の場合

- 圧縮が行われない
- ただし auxiliary.compression の設定は引き続き可能
- メインチャットの文脈が自動的に圧縮されない

## logging 設定

### logging の設定項目

| キー | 型 | 既定値 | 説明 |
|--|-|-|-|
| logging.level | string | "INFO" | ログレベル |
| logging.max_size_mb | int | 5 | maxサイズ(MB) |
| logging.backup_count | int | 3 | バックアップ数 |

### ログファイル

- `~/.hermes/logs/agent.log`: INFO以上
- `~/.hermes/logs/errors.log`: WARNING以上

## onboarding 設定

### onboarding の設定項目

| キー | 型 | 既定値 | 説明 |
|--|-|-|-|
| onboarding.seen | dict | `{}` | 既に表示済み |

## updates 設定

### updates の設定項目

| キー | 型 | 既定値 | 説明 |
|--|-|-|-|
| updates.pre_update_backup | bool | False | アップデート前のバックアップ |
| updates.backup_keep | int | 5 | バックアップ保存数 |

## よくある誤解

1. "compression.enabled = false であればすべてが手動になる" → auxiliary.compressionの設定は独立して有効。
2. "security.tirithがfalseならすべてのコマンドが実行される" → TIRITHはデフォルトで fail-open (True)。falseならすべてのコマンドがブロックされる。

## 設定例

### 安全な設定

```yaml
security:
  allow_private_urls: True
  redact_secrets: True
  tirith_enabled: True
  website_blocklist:
    enabled: True
    domains: ["malicious.example.com"]

compression:
  enabled: True
  threshold: 0.70    # 余裕を持たせる
```

### チェックポイントの有効化

```yaml
checkpoints:
  enabled: True
  max_snapshots: 30
  retention_days: 14
```

## 公式ドキュメント上の根拠

- `website/docs/user-guide/security.md`
- `website/docs/user-guide/features/checkpoints-and-rollback.md`
- `website/docs/user-guide/configuration.md`
- `website/docs/developer-guide/context-compression-and-caching.md`

## ソースコード上の根拠

- `hermes_cli/config.py:279` compression section
- `hermes_cli/config.py:488` dashboard section
- `hermes_cli/config.py:490` privacy section
- `hermes_cli/config.py:207` checkpoints section
- `hermes_cli/config.py:1225` security section
- `hermes_cli/config.py:1288` logging section
- `hermes_cli/config.py:1354` onboarding section
- `hermes_cli/config.py:1359` updates section
- `hermes_cli/config.py:1328` sessions section
