# web / browser ツール設定

## 本章で扱うこと

1. web 設定 (search / extract backend)
2. browser tool 設定
3. browser provider (Browserbase, browser-use, Firecrawl)
4. SearXNG / Brave / DuckDuckGo の差異
5. ブラウザーの検出と利用方法

## web 設定

### web.backend

- **型**: string
- **既定値**: `""` (空文字列)
- **意味**: web_search および web_extract の共有フォールバックバックエンド

### web.search_backend

- **型**: string
- **既定値**: `""`
- **意味**: web_search 専用のバックエンド

### web.extract_backend

- **型**: string
- **既定値**: `""`
- **意味**: web_extract 専用のバックエンド

### web.search_backend / extract_backend の優先順位

1. `web.search_backend` (または `extract_backend`) が設定 → そのバックエンドを使用
2. `web.backend` が設定 → そのバックエンドを使用
3. デフォルトのバックエンド (環境に依存)

**例**:
```yaml
web:
  backend: "searxng"
  search_backend: "brave_free"    # search_backend が優先
```

この場合: web_searchは Brave Free、web_extract は SearXNG を使用する。

## サポートされているバックエンド

## web_search のサポートバックエンド

| バックエンド | 要件 | 説明 |
|--|-|-|
| searxng | SEARXNG_URL | ソースインストール必須 |
| brave_free | BRAVE_SEARCH_API_KEY | Brave Search（無料プラン） |
| ddgs | なし | DuckDuckGo（無料） |
| tavily | TAVILY_API_KEY | Tavily API |
| exa | EXA_API_KEY | Exa Search API |
| parallel | PARALLEL_API_KEY | Parallel API |
| firecrawl | FIRECRAWL_API_KEY | Firecrawl（検索機能） |

### web_extract バックエンド

| バックエンド | 要件 | 説明 |
|--|-|-|
| native | なし | デフォルトのHTMLパース |
| firecrawl | FIRECRAWL_API_KEY | Firecrawl のページ抽出 |
| tavily | TAVILY_API_KEY | Tavily の検索結果 |
| auxiliary | auxiliary.web_extract | LLMによる要約 |

## web_search で SearXNG だけを使う場合

### SearXNG だけでできること

- web_search ツールで検索結果を取得
- SearXNG の API エンドポイントを直接使用
- 設定: `web.search_backend: "searxng"`

```yaml
web:
  search_backend: "searxng"

providers:
  searxng:
    api: "http://localhost:8080"
```

### SearXNG だけではできないこと

- HTMLページ内容の抽出 (→ extract_backend も設定が必要)
- ブラウザーレンダリング (→ browser tool を設定が必要)
- LLMの要約 (→ auxiliary.web_extract を設定が必要)
-画像の検索 (SearXNG に依存)

### web_extract と auxiliary.web_extract の違い

| 項目 | web_extract | auxiliary.web_extract |
|--|-|-|
| 用途 | HTMLの直接抽出 | LLMによる要約出力 |
| バックエンド | HTMLパース / Firecrawl / Tavily | LLM |
| modelの使用 | なし | auxiliary.web_extract.model を使用 |
| timeout | 環境依存 | auxiliary.web_extract.timeout (既定: 360秒) |

## browser 設定

### browserの主要設定 (DEFAULT_CONFIG 基準)

| キー | 型 | 既定値 | 説明 |
|--|-|-|-|
| inactivity_timeout | int | 120 | アイドルタイムアウト(秒) |
| command_timeout | int | 30 | コマンドタイムアウト(秒) |
| record_sessions | bool | False | セッション録画 (WebM) |
| allow_private_urls | bool | False | 内部IP/localhostの許可 |

## browser.engine の値

**source_code / website/docs/での正確なengineの値は未確認。**
一般的にサポートされるエンジン:

| 値 | 挙動 |
|--|-|--|
| "auto" | Chromeを使用（デフォルト） |
| "lightpanda" | Lightpanda（1.3-5.8倍高速、スクリーンショット不可） |
| "chrome" | Chromeを明示的に要求 |
| "AGENT_BROWSER_ENGINE" | 環境変数でエンジン指定 |

**注**: sourceコードで正確なengineの値を確認していない。正確な値は source/hermes-agent/tools/browser_tool.py を参照。

## 環境変数によるブラウザエンジン指定

環境変数 `AGENT_BROWSER_ENGINE` でもエンジン指定可能:
- "auto"（デフォルト）: Chrome
- "lightpanda": Lightpanda
- "chrome": Chrome

### browser.provider の省略時の挙動

1. CDP エンドポイントが設定 → その CDP に接続
2. ローカルブラウザが検出 → ローカルモードで起動
3. Browserbase API キー → Cloud Browserbase
4. Firecrawl API キー → Cloud Firecrawl
5. エラー

### browser tool と Playwright の違い

| 項目 | browser tool | Playwright |
|--|-|-|
| インストール | agent-browser (Pythonパッケージ) | separateパッケージ |
| 設定 | config.yaml で制御 | コード内で制御 |
| スクリーンショット | 組み込み | 組み込み |
| JavaScript | レンダリング | レンダリング |
| CDP | 標準サポート | CDP経由 |

### headless Chromium の検出方法

1. **システム Chromium/Chrome** を検出
2. **CDP** (Chrome DevTools Protocol) エンドポイントを介して接続
3. **agent-browser** の internal chromium を使用する

```bash
# チェック方法
chromium --version
google-chrome --version
which chromium
which google-chrome
```

## browser backend の関係

### CDP (Chrome DevTools Protocol)

- Chromeまたは Chromium のデバッグポートに接続
- 既存のブラウザインスタンスを制御 (browser connect)
- `browser.cdp_url` に設定

### agent-browser

- Pythonパッケージとしてインストール
- Chromeを自動検出
- `AGENT_BROWSER_ENGINE` 環境変数でエンジン指定

### Chromium / Chrome

- ローカルブラウザとしてのデフォルトエンジン
- `browser.engine` で明示指定可能

### Lightpanda

- Chromeの代わりに高速レンダリングエンジン
- スクリーンショット不可
- `browser.engine: "lightpanda"`

### Cloud browser

| Provider | 要件 |
|--|-|
| Browserbase | BROWSERBASE_API_KEY + BROWSERBASE_PROJECT_ID |
| Firecrawl | FIRECRAWL_API_KEY |
| browser-use | 未確認 |
| Camofox | 未確認 |

## よくある誤解

1. " browser provider を省略すると常にローカルブラウザになる" → CDPやCloudBrowserbaseが検出可能な場合、そのCloudプロバイダーにフォールバック
2. "SearXNGでスクリーンショット取れる" → SearXNGは検索のみ。HTMLのスクレイピングは `web.extract_backend` の対象
3. "agent-browser = Playwright" → agent-browserはPlaywrightとは別のパッケージ。agent-browser内部でChromium/Chromeを制御する

## 設定例

### ローカルブラウザのセットアップ

```yaml
browser:
  engine: "auto"
  allow_private_urls: True
  auto_local_for_private_urls: True
  cdp_url: ""
```

### Cloud browser のセットアップ

```yaml
browser:
  engine: "auto"
  allow_private_urls: True
  auto_local_for_private_urls: False

providers:
  browserbase:
    api: "https://www.browserbase.com"
    api_key: "${BROWSERBASE_API_KEY}"
  firecrawl:
    api: "https://api.firecrawl.dev"
    api_key: "${FIRECRAWL_API_KEY}"
```

### web_search の設定例

```yaml
web:
  search_backend: "brave_free"
  extract_backend: "native"

providers:
  brave_free:
    api_key: "${BRAVE_SEARCH_API_KEY}"
```

## 公式ドキュメント上の根拠

- `website/docs/user-guide/features/web-search.md`
- `website/docs/user-guide/features/browser.md`
- `website/docs/user-guide/features/provider-routing.md`

## ソースコード上の根拠

- `hermes_cli/config.py:163` web section
- `hermes_cli/config.py:169` browser section
- `tools/web_tools.py` web_backendsの実装
- `tools/browser_tool.py` browser implementation
- `tools/web_providers/searxng.py` SearXNG impl
- `tools/browser_providers/browserbase.py` Browserbase impl
- `tools/browser_providers/firecrawl.py` Firecrawl impl
