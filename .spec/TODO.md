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

### 2026-03-22：出題モード選択・続きから機能

- [x] HTML：`view-mode-select` 画面を追加
- [x] HTML：`view-problem-select` 画面を追加
- [x] CSS：モード選択・1問選択画面のスタイル追加
- [x] `Data.saveProgress()` / `clearProgress()` を追加
- [x] `Data.init()` で `quiz-progress.json` を読み込み
- [x] `App.openSet()` の遷移先を `view-mode-select` に変更
- [x] `ModeSelect` モジュールを追加（モード選択・続きから・1問選択・苦手・前回ミス）
- [x] `ProblemSelect` モジュールを追加（1問選択画面）
- [x] `App.startQuizWithMode(modeObj)` を実装
- [x] `Quiz.start()` の modeObj 対応
- [x] 回答ごとに `Data.saveProgress()` を呼び出す（`Quiz.next()`）
- [x] 全問完了時に `Data.clearProgress()` を呼び出す（`Quiz.finish()`）
- [x] 1問選択後の出題・結果後の view-problem-select への復帰
- [x] `Quiz.backFromResult()` / `retryFromResult()` を追加

### 2026-03-22：回答後の自動スクロール

- [x] `confirmMultipleChoice` の末尾に `scrollToExplanation()` を追加
- [x] `confirmOrder` の末尾に `scrollToExplanation()` を追加

### 2026-03-22：GitHub フォルダ形式アップロード・ダウンロード

- [x] `GitHubAPI.uploadFile(path, jsonData, message)` を追加
- [x] `GitHubAPI.getTree()` を追加（`git/trees/HEAD?recursive=1`）
- [x] `GitHubAPI.getBlob(sha)` を追加
- [x] `GitHubAPI.listQuizSets()` を `type === 'dir'` に変更
- [x] `GitHubAPI.upload()` / `download()` を削除（uploadFile / getBlob に統合）
- [x] `GHSync.uploadSet()` を問題別ループ形式に変更（進捗表示付き）
- [x] `GHSync.downloadSet(setName)` をフォルダ形式に変更（getTree + getBlob）
- [x] `_renderImportList()` の引数を `setName` のみに変更

### 2026-03-22：ウェルカム画面スマホ表示修正

- [x] `welcome-new` に `width:100%` を追加（ボタンが全幅表示されない問題を修正）
