# Portfolio Hub - 設計ドキュメント

このドキュメントは「何を作るか・なぜそうするか」を記述する。
Claude Codeへの作業指示は `/CLAUDE.md` を参照。

---

## 1. 目的

- 新しい技術を試したアウトプットを継続的に積み上げる場
- HP(このリポジトリ)に各アプリのカードを並べ、クリックすると個別のデプロイ済みアプリに遷移する
- 拡張性重視：新しいアプリを追加するときに、HP側のコード変更を発生させない
- **学習重視**：フレームワークに頼らず、Web標準とNode.jsの基礎を理解しながら作る

## 2. アーキテクチャ：ハブ＆スポーク型

```
your-portfolio (このリポジトリ／HP・ハブ)
  └── アプリ一覧カタログUI(ビルド時にGitHub APIで自動生成)

app-001-xxx (個別リポジトリ／スポーク)  → 独立デプロイ
app-002-yyy (個別リポジトリ／スポーク)  → 独立デプロイ
...
```

### 採用理由

- 各アプリで自由に技術スタックを選べる(Next.js / Svelte / Python など混在OK)
- 1つのアプリが壊れても他に影響しない
- 各アプリが「作品」として独立し、リポジトリ単体でシェア可能
- HPは薄いカタログ層なので保守コストが低い

### モノレポを採用しなかった理由

- 「新しい技術を試す」用途では、アプリごとに依存関係を分離したい
- アプリ追加時に共通の依存に縛られたくない
- デプロイの独立性を優先

## 3. 技術スタック

**方針：フレームワーク不使用。Node.jsの素のスクリプトでビルドする。**

| 領域 | 採用 | 理由 |
|---|---|---|
| ビルド言語 | Node.js (JavaScript) | フレームワークの「魔法」なしで仕組みを理解 |
| HTTPクライアント | Node標準の `fetch` | Node 18+ で標準搭載。追加依存なし |
| テンプレートエンジン | 文字列テンプレートリテラル | 軽量・依存ゼロ・JSの基本機能 |
| 出力フォーマット | 静的 HTML + CSS + (必要なら) Vanilla JS | Web標準のみ |
| ホスティング | GitHub Pages | 無料・GitHub完結 |
| CI/CD | GitHub Actions | GitHub Pagesと自然統合 |
| Lint/Format | ESLint + Prettier(任意) | 後から追加でも可 |

### 検討した代替案と却下理由

- **Astro / Next.js**: 便利だが、フレームワーク学習が主目的になってしまう
- **Vite + Vanilla TS**: 良い選択肢だが、Viteを学ぶ範囲が広がる。後から移行可能
- **Octokit**: 公式ライブラリで便利だが、まずは素の `fetch` でAPI叩く経験を積む。必要になれば導入

### 学習の到達点

このプロジェクトを通じて理解できるようになる範囲：

- Node.js の基本(fetch、ファイルI/O、モジュール)
- GitHub REST API の叩き方・認証
- HTML を文字列から組み立てる仕組み(テンプレートエンジンが内部で何をしているか)
- ビルドプロセスの設計
- GitHub Actions ワークフロー
- GitHub Pages へのデプロイ

## 4. メタデータ戦略

### 基本方針

各アプリリポジトリに **topic `portfolio-app`** を付けることでカタログ掲載対象としてマークする。
HPはビルド時にこのtopicを持つリポジトリ一覧をGitHub APIから取得し、HTMLカードを生成。

### 使用するGitHub API

| 用途 | エンドポイント |
|---|---|
| ユーザーのリポジトリ一覧取得 | `GET /users/{username}/repos` |
| 特定topicでの検索 | `GET /search/repositories?q=user:{username}+topic:portfolio-app` |
| リポジトリ内ファイル取得 | `GET /repos/{owner}/{repo}/contents/{path}` |

> 検索APIの方が「topic絞り込み」が一発でできるので便利。ただしレート制限がやや厳しい(認証あり: 30req/min)

### 取得元マッピング

| 表示項目 | 取得元(レスポンスのフィールド) |
|---|---|
| アプリ名 | `name` |
| 説明文 | `description` |
| デモURL(カードのリンク先) | `homepage` |
| タグ | `topics`(`portfolio-app` を除いたもの) |
| ソースコードリンク | `html_url` |
| 並び順 | `updated_at` の降順 |

### リッチメタデータ(オプショナル)

リッチな表示が必要なアプリのみ、リポジトリのルートに `.portfolio/meta.json` を配置：

```json
{
  "title": "表示用タイトル(リポジトリ名を上書き)",
  "subtitle": "短いキャッチ",
  "thumbnail": "thumbnail.png",
  "tech": ["Next.js", "OpenAI API"],
  "highlights": ["特徴1", "特徴2"],
  "demo_video": "https://..."
}
```

HP側ロジック：
1. リポジトリリスト取得後、各リポジトリに対して `contents/.portfolio/meta.json` を取得試行
2. 404なら無視(リッチデータなしで通常表示)
3. 200なら base64 デコードして JSON.parse → カード表示にマージ

## 5. ビルド・デプロイ戦略

### ビルドの流れ

```
1. GitHub API でリポジトリ一覧取得
2. 各リポジトリの .portfolio/meta.json を試行取得
3. データを正規化(型を揃える)
4. テンプレート文字列に流し込んで HTML 生成
5. CSS・画像を dist/ にコピー
6. dist/ を GitHub Pages にデプロイ
```

### フェーズ1(最初はこれで進める)

HP側の GitHub Actions を以下で起動：
- `schedule`: 1日1回(例：cron `0 0 * * *`)
- `workflow_dispatch`: 手動実行ボタン

新アプリを追加したら手動でAction実行 → 反映。

### フェーズ2(必要になったら)

各アプリリポジトリの Actions から HP リポジトリに `repository_dispatch` イベントを送信 → 即時再ビルド。
最初から作り込むとアプリ追加のたびにセットアップが必要になるため後回し。

## 6. ディレクトリ構成

```
.
├── CLAUDE.md                  # Claude Code への作業指示
├── README.md
├── docs/
│   ├── DESIGN.md              # このファイル
│   └── DECISIONS.md           # 意思決定ログ(必要に応じて追記)
├── package.json
├── build.js                   # ビルドのエントリーポイント
├── src/
│   ├── lib/
│   │   ├── github.js          # GitHub API を叩く関数群
│   │   ├── metadata.js        # .portfolio/meta.json のパース
│   │   └── render.js          # HTMLレンダリング(テンプレート展開)
│   ├── templates/
│   │   ├── index.html         # ページ全体テンプレート
│   │   └── card.html          # カード1枚分のテンプレート
│   └── assets/
│       └── style.css          # スタイル
├── dist/                      # ビルド成果物(.gitignoreで除外)
└── .github/
    └── workflows/
        └── build-and-deploy.yml
```

### 設計上の意図

- `build.js` が全体の流れを制御する司令塔
- `src/lib/` の各ファイルは「役割で分離された純粋な関数」
- `src/templates/` の HTML ファイルには `{{title}}` のようなプレースホルダを書く
- `src/assets/` の静的ファイルはビルド時にそのまま `dist/` にコピー
- `dist/` は git 管理外。GitHub Actions がビルドして直接 Pages にデプロイ

## 7. テンプレートの仕組み(自前実装の方針)

ライブラリを使わず、文字列置換で実装する。

### card.html の例

```html
<article class="card">
  <h2><a href="{{homepage}}">{{title}}</a></h2>
  <p>{{description}}</p>
  <div class="tags">{{tags}}</div>
  <a class="source" href="{{html_url}}">Source</a>
</article>
```

### レンダリング関数の方針

`src/lib/render.js` に簡易テンプレート関数を実装：

```javascript
// 概念イメージ(実装は自分で書く)
function render(template, data) {
  return template.replace(/\{\{(\w+)\}\}/g, (_, key) => data[key] ?? '');
}
```

複雑な分岐やループが必要になった時点で考え直す。最初はこれで十分。

## 8. 環境変数

| 変数 | 用途 | 設定場所 |
|---|---|---|
| `GITHUB_TOKEN` | GitHub API認証(レート制限緩和) | Actions実行時は自動付与 / ローカルは `.env` |
| `GITHUB_USERNAME` | 対象ユーザー名 | `.env` および Actions secrets |

`.env` は `.gitignore` 必須。
`dotenv` パッケージを使うか、シェルから export して使うか選択。

## 9. 新アプリ追加の運用ルール

これは「アプリ側のリポジトリ」で行う作業：

1. `claude-template` から "Use this template" で新規リポジトリ作成
2. 実装、デプロイ(Vercel / Cloudflare Pages / Hugging Face Spaces など)
3. リポジトリの Settings:
   - `Description` を埋める
   - `Website` に本番URLを入れる
   - `Topics` に **`portfolio-app`** を追加(必須)。技術タグも追加推奨
4. リッチ表示にしたい場合のみ `.portfolio/meta.json` をコミット
5. HP側のActionを手動実行(または1日待つ) → 反映確認

## 10. スコープ外(やらないこと)

- アプリリポジトリ間のコード共有 → 必要になったら npm package に切り出す
- 認証付きプライベートアプリ対応
- 全文検索・カテゴリフィルタ → アプリが増えたら検討
- 各アプリのアナリティクス集約
- フェーズ2の `repository_dispatch` 自動化
- TypeScript化(後から段階的に導入可能)
- React等のフレームワーク導入(必要になったら検討)

## 11. 着手順(最小構成までのマイルストーン)

1. **`package.json` の作成と最小構成**
   - `npm init -y`
   - Node のバージョン指定(`engines` フィールド)
   - `.gitignore` に `node_modules/`, `dist/`, `.env` を追加

2. **GitHub API から1つだけリポジトリを取得して console.log する**
   - `src/lib/github.js` を新規作成
   - 認証なしfetchで `https://api.github.com/users/{username}/repos` を叩く
   - 結果を整形して console に表示
   - **ここまでで「APIが叩けた」確認**

3. **topicで絞り込んだリポジトリ一覧を取得する**
   - 検索APIに切り替え or クライアント側でフィルタ
   - 必要な項目だけ抽出(name, description, homepage, topics, html_url, updated_at)

4. **テンプレート展開関数の実装**
   - `src/lib/render.js` に簡易テンプレート関数
   - 単一カードのHTMLを生成して console に出力

5. **HTML 全体を組み立てて dist/index.html に出力**
   - `src/templates/index.html` と `src/templates/card.html` を作成
   - `build.js` で全部つなげる
   - CSS も `dist/` にコピー

6. **ローカルでブラウザで開いて確認**
   - `dist/index.html` をブラウザで直接開く

7. **`.portfolio/meta.json` 対応の追加**
   - 各リポジトリのファイル取得を試行
   - リッチデータをカードに反映

8. **GitHub Actions ワークフローの作成**
   - `.github/workflows/build-and-deploy.yml`
   - ビルド → `dist/` を GitHub Pages にデプロイ
   - schedule + workflow_dispatch トリガー

9. **GitHub Pages の有効化(リポジトリSettings)**

10. **テスト用アプリリポジトリで topic `portfolio-app` を付けて E2E 動作確認**

### 各マイルストーンで意識すること

- 1ステップ完了ごとにコミット
- 「次のステップに進む前に、今の動作を理解できているか」を自問
- 詰まったら Claude Code に「概念解説」を依頼(コードを書かせるのではなく)