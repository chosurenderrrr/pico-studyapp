# TODO

## タスク一覧

### 2026-03-19：数式表示機能

- [x] KaTeX CDNを `<head>` に追加（CSS + JS）
- [x] `renderMath(text)` ユーティリティ関数を実装
  - `$$...$$` ブロック数式の処理
  - `$...$` インライン数式の処理
  - KaTeXエラー時のフォールバック（テキスト表示）
- [x] `renderChoiceContent` を `renderMath` 対応に更新
- [x] `_renderQuestionPage` を `renderMath` 対応に更新
- [x] 解説・正解表示を `renderMath` 対応に更新
- [x] エディタ：問題文テキストエリアにプレビューエリアを追加
- [x] エディタ：解説テキストエリアにプレビューエリアを追加

### 2026-03-20：プルトゥリフレッシュ無効化・離脱防止ダイアログ

- [x] CSS に `overscroll-behavior-y: none` を追加（プルトゥリフレッシュ無効化）
- [x] `beforeunload` イベントで離脱防止ダイアログを実装

### 2026-03-20：GitHub設定のフォルダ内永続化

- [x] `Data` に `loadGitHubConfig()` / `saveGitHubConfig()` を追加
- [x] `Data.init()` で `loadGitHubConfig()` を呼び出し
- [x] `GitHubAPI.setToken()` で `saveGitHubConfig()` を呼び出し
- [x] `GitHubAPI.setRepo()` で `saveGitHubConfig()` を呼び出し
