# SPEC - 技術仕様・要件定義

## 機能概要：数式表示機能

### 対象箇所
- 問題文（テキストページ）
- 選択肢（正解・誤答・整序）
- 解説（テキスト）

### 記法
- インライン数式：`$...$`
- ブロック数式：`$$...$$`
- TeX記法（LaTeX互換）

### レンダリングライブラリ
- KaTeX（CDN経由で読み込み）
- CSS / JS の両方を `<head>` に追加

### 表示時（出題画面）
- `renderMath(text)` というユーティリティ関数を新設
  - `$$...$$` と `$...$` を検出し、KaTeXでレンダリングして返す
  - それ以外のテキストはそのまま返す（XSS考慮でエスケープ）
- 以下の箇所で `renderMath` を使用：
  - `_renderQuestionPage`：テキストページの表示
  - `renderChoiceContent`：選択肢テキストの表示
  - 解説テキストの表示（`quiz-explanation-content` への書き込み）
  - 正解選択肢の表示（不正解時の答え合わせ表示）

### 入力時（エディタ画面）
- 問題文テキストエリア（`#q-text`）の下にプレビューエリアを追加
  - `input` イベントで即時プレビュー更新
- 解説テキストエリア（`#ex-text`）の下にプレビューエリアを追加
  - `input` イベントで即時プレビュー更新
- 選択肢の各テキスト入力にはプレビューは設けない（UIが煩雑になるため）
  - 代わりに、選択肢テキストもレンダリング対象とし、出題画面で確認できる

### エラーハンドリング
- 不正なTeX記法の場合は、KaTeXのエラーをキャッチしてそのままテキスト表示

## 非機能要件（継続）
- 単一HTMLファイル（`index.html`）にインライン実装
- 外部ライブラリ禁止（KaTeX CDN は今回限り例外として許可）
- Android Chrome タッチ操作対応

---

## 機能概要：出題モード選択・続きから機能

### 要件まとめ
1. クイズセット選択後に「出題モード選択画面」を挿入する
2. 出題モード：順番（ID順）/ ランダム / 1問選択
3. 出題数（順番・ランダムのみ）：25 / 50 / 100 / 全部
4. 前回の続きがある場合、続きから再開できる
5. ランダム出題は順序を保存し、次回も同じ順番で再開
6. 次回開始時に「続きから」「初めから」を選べる
7. 1問選択モード：問題一覧から1問を選んでその1問だけ解く

---

### 画面フロー

```
クイズセット選択（view-home）
  ↓
モード選択画面（view-mode-select）  ← 新規追加
  ├─ [続きから始める] ← 前回の続きがある場合のみ表示
  ├─ [順番で出題] → 出題数選択 → 出題開始（view-quiz）
  ├─ [ランダムで出題] → 出題数選択 → 出題開始（view-quiz）
  └─ [1問選択] → 1問選択画面（view-problem-select） ← 新規追加
                   ↓
               問題を選択 → 出題開始（view-quiz）→ 結果後、一覧に戻る
```

---

### 出題モード選択画面（view-mode-select）

#### 表示内容
- セット名（タイトル）
- **「続きから始める」ボタン**（前回の続きがある場合のみ表示）
  - 例：「▶ 続きから再開（ランダム・50問 / 15問目）」
- **「順番で出題」ボタン**
- **「ランダムで出題」ボタン**
- **「1問を選んで解く」ボタン**
- 戻るボタン（ホームへ）

#### 出題数選択（順番・ランダム選択後にインライン表示）
- 「25問」「50問」「100問」「全部」のボタン群
- 問題数がボタン値より少ない場合はそのボタンをグレーアウト・無効化
- 「開始」ボタンで出題開始

---

### 1問選択画面（view-problem-select）

- 問題一覧をスクロールリスト表示（セット詳細と似た形式）
- 各項目：問題番号 + 問題の最初のテキスト（冒頭20文字程度）
- タップで即時出題開始（view-quizに遷移）
- 回答・結果確認後、view-problem-selectに戻る（Nav.backでなく明示的に戻る）
- 戻るボタン（モード選択へ）

---

### 続きから機能

#### 保存ファイル
- `{rootHandle}/data/quiz-progress.json`

#### データ構造
```json
{
  "セット名": {
    "mode": "order" | "random",
    "count": 25 | 50 | 100 | null,
    "sequence": [3, 1, 5, 2, ...],
    "currentIndex": 15
  }
}
```
- `count: null` = 全部
- `sequence`：出題する問題のnum（ID）の配列（順番・ランダム両方で使用）
- `currentIndex`：次に出題する問題のインデックス（0始まり）

#### 保存タイミング
- 各問題への回答後（currentIndex を +1 して保存）
- 中断時（ホームへ戻るボタン押下時）も currentIndex を保存済みなので対応不要

#### リセットタイミング
- 全問完了時（currentIndex が sequence.length に達した時）
- 「初めから始める」選択時（「続きから」を選ばずに新規開始した場合）

#### 「続きから」ボタン表示条件
- quiz-progress.json に対象セットのデータが存在し、かつ currentIndex < sequence.length

---

### Quiz モジュールの変更点

- `Quiz.start(problems, mode)` の `mode` を拡張：
  ```
  mode = {
    type: 'order' | 'random' | 'single',
    count: 25 | 50 | 100 | null,  // null = 全部
    sequence: [...] | null,  // 続きから時は保存済み順序を渡す
    startIndex: 0,            // 続きから時は前回のcurrentIndex
  }
  ```
- 回答ごとに `Data.saveProgress(setName, progressData)` を呼び出す
- single モード時は結果画面ではなく view-problem-select へ戻る

---

### Data モジュールの変更点

追加メソッド：
- `Data.loadProgress()` → quiz-progress.json を読み込み State.progress に格納
- `Data.saveProgress(setName, data)` → 対象セットの進捗を更新して保存
- `Data.clearProgress(setName)` → 対象セットの進捗を削除して保存

---

### 非機能要件
- 単一HTMLファイル（`index.html`）にインライン実装
- 外部ライブラリ禁止
- Android Chrome タッチ操作対応

---

## 機能概要：GitHub フォルダ形式アップロード・ダウンロード

### 要件
- アップロード：1セット＝複数ファイル（問題別）形式に変更
- ダウンロード：GitHub 上のフォルダ構造から問題別にダウンロード
- 旧 combined JSON 形式（`quiz-sets/セット名.json`）は廃止

### GitHub 上のパス構造
```
quiz-sets/
  セット名/
    001/
      problem.json
    002/
      problem.json
    ...
```
- 問題番号はゼロ埋め3桁（ローカルと同じ形式）

---

### アップロード変更点

#### 現状
- `PUT /repos/{repo}/contents/quiz-sets/{setName}.json`（1ファイル）

#### 変更後
- 問題ごとに `PUT /repos/{repo}/contents/quiz-sets/{setName}/{num}/problem.json`
- 既存ファイルの SHA を事前取得して上書き対応
- ローディング表示に進捗（`1/N` 形式）を表示
- 既存の combined JSON (`quiz-sets/{setName}.json`) は削除しない（残存しても問題なし）

#### 実装
```javascript
async uploadSet() {
  // 問題ごとに PUT
  for (let i = 0; i < problems.length; i++) {
    const num = String(problems[i].num).padStart(3, '0');
    const path = `quiz-sets/${setName}/${num}/problem.json`;
    await GitHubAPI.uploadFile(path, problem, `Update: ${setName}/${num}`);
    Loading.show(`アップロード中... ${i+1}/${problems.length}`);
  }
}
```

---

### ダウンロード変更点

#### 現状
- `GET /repos/{repo}/contents/quiz-sets/` でファイル一覧（`.json` ファイル）を取得

#### 変更後（2ステップ）

**ステップ1: セット一覧取得**
- `GET /repos/{repo}/contents/quiz-sets/` → `type === 'dir'` のみ取得

**ステップ2: セット内の問題を一括取得（git tree API 使用）**
- `GET /repos/{repo}/git/trees/HEAD?recursive=1`
- `quiz-sets/{setName}/XXX/problem.json` に一致するエントリを抽出
- 各エントリの SHA で `GET /repos/{repo}/git/blobs/{sha}` → base64 デコード → JSON パース
- ローカルの `ProblemIO.save()` で書き込み
- ローディング表示に進捗を表示

---

### GitHubAPI 変更点

追加メソッド：
- `GitHubAPI.uploadFile(path, jsonData, message)` — 汎用ファイル PUT（SHA 自動取得）
- `GitHubAPI.getTree()` — `GET /repos/{repo}/git/trees/HEAD?recursive=1`
- `GitHubAPI.getBlob(sha)` — `GET /repos/{repo}/git/blobs/{sha}` → JSON

変更メソッド：
- `GitHubAPI.listQuizSets()` — `type === 'dir'` に変更
- `GitHubAPI.upload()` は `uploadFile()` に統合（旧メソッドは削除）
- `GitHubAPI.download()` は `getTree()` + `getBlob()` に置き換え

---

### GHSync 変更点

- `uploadSet()` — 問題ごとにループして `uploadFile()` を呼び出す
- `downloadSet(setName)` — `getTree()` でパス一覧取得 → `getBlob()` で各問題取得 → `ProblemIO.save()` で保存
- インポートダイアログのセット選択部分は変更なし（セット名のみ渡す）

---

## バグ修正：GitHub ダウンロード時の stale ハンドルエラー

### 症状
- 100問程度の大きなクイズセットをGitHubから取り込む際に `An operation that depends on state cached in an interface object was...` エラーが発生する

### 原因
- Chrome の File System Access API の挙動として、`getDirectoryHandle({ create: true })` でサブディレクトリを作成すると、親ディレクトリハンドルのキャッシュ状態が無効化される
- 同じ `setHandle` を使い回して100問分のサブディレクトリを連続作成すると stale 状態になりエラーが発生する

### 修正方針
- `downloadSet()` のループ内で毎回 `State.rootHandle` から `setHandle` を再取得する
- `await FS.getDir(State.rootHandle, setName, true)` を各ループ先頭で呼び出す

---

## 機能概要：回答後の自動スクロール

### 要件
- 選択問題・整序問題どちらも、回答確定後に自動で解説エリアへスクロールする
- 正解・不正解に関わらず動作する

### 実装方針
- `confirmMultipleChoice` と `confirmOrder` の末尾（`bar.classList.add('show')` の直後）に `this.scrollToExplanation()` を呼び出す
- 既存の `scrollToExplanation()` をそのまま流用

---

## 機能概要：プルトゥリフレッシュ無効化・離脱防止ダイアログ

### 要件
1. スマホ（Android Chrome）でのプルトゥリフレッシュ（上スワイプによるページリロード）を全画面で無効化する
2. ページリロード・タブ閉じ・ブラウザバック等の離脱操作時に確認ダイアログを表示する

### 実装方針

#### プルトゥリフレッシュ無効化
- CSS に `html, body { overscroll-behavior-y: none; }` を追加
- Android Chrome 63+ でネイティブ対応しているため、JS不要

#### 離脱防止ダイアログ
- `window.addEventListener('beforeunload', ...)` でブラウザ標準の確認ダイアログを表示
- 常時有効（全操作中）
- ダイアログのメッセージ文はブラウザが制御するため、カスタムメッセージは表示されない（仕様）

### 対象範囲
- アプリ全体（全画面・全操作）

---

## 機能概要：GitHub設定のフォルダ内永続化

### 要件
- PAT（Personal Access Token）とリポジトリ名を、選択済みフォルダ内の `data/github-config.json` に保存する
- index.html を削除・差し替えた後でも、同じフォルダを選択すれば PAT・リポジトリ名が自動復元される

### 実装方針

#### 保存先
- `{rootFolder}/data/github-config.json`
- 内容：`{ "token": "ghp_...", "repo": "user/repo" }`

#### 読み込みタイミング
- `Data.init()` の最後に `loadGitHubConfig()` を呼び出し、ファイルから localStorage へ復元

#### 書き込みタイミング
- `GitHubAPI.setToken()` 実行時（localStorage への保存と同時にファイルへも書き込み）
- `GitHubAPI.setRepo()` 実行時（同上）

#### セキュリティ
- ファイルはユーザーが選択したローカルフォルダ内に保存される（localStorage と同等のセキュリティレベル）
- `data/` フォルダはクイズセット一覧から除外済みのため、UIには表示されない
