# セッション開始ガイド

VS Code を開いてから、Claude Code で前回の続きを始めるまでの手順。
このファイルは **人間（私）向けのチェックリスト**。Claude は読まなくてよい。

## 前提（一度だけのセットアップ）

以下が済んでいる前提:

- VS Code がインストール済み
- Docker Desktop がインストール済み・起動中
- VS Code 拡張: `ms-vscode-remote.remote-containers`（Dev Containers）
- VS Code 拡張: `anthropic.claude-code`（Claude Code）
- GitHub 認証済み（`gh auth status` で確認可能）

## 手順

### 1. プロジェクトを開く

- VS Code を起動
- `File` > `Open Recent` から `my-portfolio` を選ぶ
- （または `File` > `Open Folder` で `my-portfolio` のディレクトリを選択）

### 2. Dev Container で再起動

- 右下に「Reopen in Container」通知が出る → クリック
- 通知が出ない場合: コマンドパレット（`Cmd/Ctrl + Shift + P`）→ `Dev Containers: Reopen in Container`
- 初回はビルドで数分かかる。2回目以降は数十秒程度

### 3. git の状態確認

ターミナル（`Ctrl + ` ` で開く）で以下を実行:

```bash
git status
git log --oneline -5
```

期待される状態:

- `On branch main`
- `working tree clean`
- `Your branch is up to date with 'origin/main'`

未コミットの変更があれば、必要に応じて対応してから次へ。

### 4. Claude Code を開く

- 左サイドバーの Claude アイコンをクリックしてパネルを開く
- パネルが反応しない場合: コマンドパレット → `Developer: Reload Window` で復旧することがある

### 5. 最初のプロンプトを貼る

Claude に以下を入力:

```
docs/PROGRESS.md を読んで現在地を把握してください。
次のステップに進む準備をしてください。コードはまだ書かないでください。
```

これで:

- `CLAUDE.md` と `docs/DESIGN.md` は Claude が自動で読む
- `docs/PROGRESS.md` も `CLAUDE.md` 内の `@` 参照経由で読まれる
- `docs/PROGRESS.md` の「次のステップ」を起点に作業再開

### 6. 作業を進める

`CLAUDE.md` の開発スタイルに従って Claude と対話しながら作業。
Claude が自分で勝手にコードを書き始めたら止めて、説明や設計判断を求めること。

## セッション終了時のチェックリスト

```bash
git status              # working tree clean か
git log --oneline -5    # 直近の commit を目視確認
git status              # "Your branch is up to date with 'origin/main'" か
```

完了したステップがあれば、`docs/PROGRESS.md` の「完了したステップ」と「現在地」を更新してから commit & push。
Claude に「PROGRESS.md を更新して」と頼んでもよい。

3つともクリアになっていれば VS Code を閉じて終了。
Dev container は停止しても起動したままでも、次回開いた時に同じ状態に戻る。

## トラブルシューティング

| 症状 | 対応 |
|---|---|
| `Reopen in Container` 通知が出ない | コマンドパレット → `Dev Containers: Reopen in Container` を手動実行 |
| コンテナビルドが失敗 | Docker Desktop が起動しているか確認。ログを見て原因特定 |
| `git push` で認証エラー | `gh auth status` で状態確認 → 必要なら `gh auth login` で再認証 |
| Claude Code パネルが空・反応しない | コマンドパレット → `Developer: Reload Window` |
| `node` コマンドが見つからない | Dev Container 内にいるか確認（ターミナルのプロンプトに `node@...` と出る） |
| `npm run build` で予期しないエラー | `node --version` で 18 以上か確認。`package.json` の `engines.node` と整合しているか |

## 参考

- `CLAUDE.md` - Claude Code 向けの作業ガイドライン
- `docs/DESIGN.md` - プロジェクトのアーキテクチャ・技術選定
- `docs/PROGRESS.md` - 進捗ログ・現在地
- `docs/DECISIONS.md` - 意思決定ログ（必要に応じて追記）
