# Progress Log

このプロジェクトの実作業の進捗を記録するドキュメント。
新しい Claude Code セッションを始めるとき、Claude がここを読んで「どこまで進んだか」を把握できるようにする。

## 現在地

- **更新日**: 2026-05-24
- **目標**: ゴールA（ローカルでブラウザに表示）
- **直近完了**: ステップ1（package.json セットアップ）
- **次のステップ**: ステップ2（GitHub API を `fetch` で叩いて `console.log`）

## 完了したステップ

### ステップ1 — package.json セットアップ（2026-05-24）

- `package.json` を作成
  - `type: "module"`（ESM 構文を有効化）
  - `private: true`（npm publish 誤爆防止）
  - `engines.node: ">=18"`（標準 `fetch` を使うため）
  - `scripts.build: "node build.js"`（build.js 本体は未作成）
- `.gitignore` は既に `node_modules/` / `dist/` / `.env` を含む状態だったので追記不要
- `npm run build` を実行し「Cannot find module build.js」エラーまで到達 → 配線確認 ✓
- Commit: `07c04a2 chore: add minimal package.json (ESM, private, node >=18)`

## ロードマップ（docs/DESIGN.md §11 より）

ゴールA（ローカルでブラウザに表示）:

- [x] 1. `package.json` 作成・最小構成
- [ ] 2. GitHub API を `fetch` で叩いて `console.log`
- [ ] 3. topic で絞り込み、必要項目だけ抽出
- [ ] 4. テンプレート展開関数の実装
- [ ] 5. HTML 全体を組み立てて `dist/index.html` に出力
- [ ] 6. ブラウザで `dist/index.html` を開いて確認

ゴールB（GitHub Pages で公開）:

- [ ] 7. `.portfolio/meta.json` 対応
- [ ] 8. GitHub Actions ワークフロー作成
- [ ] 9. GitHub Pages 有効化
- [ ] 10. テスト用アプリで E2E 動作確認

## 再開時の使い方

新しい Claude Code セッションを開いたら、最初に以下を指示する:

```
docs/PROGRESS.md を読んで現在地を把握してください。
次のステップに進む準備をしてください。コードはまだ書かないでください。
```

`CLAUDE.md` と `docs/DESIGN.md` は Claude Code が自動で読み込みます。
このファイル（`docs/PROGRESS.md`）も `CLAUDE.md` から `@` 参照されているため、一緒に読み込まれます。

## このファイルの更新ルール

- 各ステップを完了したら **「完了したステップ」セクションに追記**
- 「現在地」セクションの **更新日・直近完了・次のステップ** を最新化
- ロードマップのチェックボックスを更新
- これらは Claude にお願いしてもよいし、自分で書いてもよい（コミットの一部としてやるのが綺麗）

## 開発スタイルのリマインダー

詳細は `CLAUDE.md` 参照。要点だけ:

- Claude はコードを書く人ではなく、**ペアプロの相手・解説者・レビュアー**
- 一度に出すスニペットは **1関数 or 30行以内**
- 新しい概念・API・記法が出たら一言解説を添える
- 設計上の意思決定は必ず確認を取る
