# pico-studyapp 仕様書（v3）

## Context
ローカルフォルダから問題素材を読み込んで多択クイズとして学習できるSPA。
HTML + CSS + JavaScript のみ。サーバー・フレームワーク不使用。
学習時間記録・カレンダー・苦手対策モードを追加して学習管理アプリとして拡充する。

### 方針（v3 2026-03-07）
- **スマートフォン利用前提**: UIはモバイルファースト設計
- **単一HTMLファイル**: `index.html` 1ファイルに CSS・JS をすべてインライン実装
- **ローカルフォルダが主**: File System Access API でデバイス上のフォルダを読み書き
- **Google Drive は同期先**: 認証不要・公開フォルダ＋APIキーで差分確認＋ダウンロード
- **ログイン不要**: `file://` で直接開いて使える

### ストレージ役割分担
| データ | 保存先 | 理由 |
|---|---|---|
| クイズセット（問題画像・テキスト） | ローカルフォルダ | 高速・オフライン動作 |
| 学習ログ・正答率・カレンダー | ローカルフォルダ内 `data/` サブフォルダ（JSON） | フォルダと一体管理 |
| Driveフォルダ設定・APIキー | localStorage | 毎回入力不要 |
| フォルダハンドル | IndexedDB | 再起動後も使えるよう永続化 |

---

## ユーザーのワークスペースフォルダ構成

```
[workspace]/               ← 初回起動時にユーザーが選択（File System Access API）
├── data/                  ← アプリが自動生成
│   ├── study-log.json
│   ├── quiz-stats.json
│   └── calendar.json
└── [quiz-set-name]/       ← クイズセット（複数可）
    └── 001/
        ├── question.png/txt
        ├── correct.png/txt
        └── wrong1.png/txt （wrong2, wrong3 も可）
```

---

## Google Drive 同期フロー

### 役割
- Drive フォルダは**配布・バックアップ用**（ローカルへのダウンロード元）
- Drive は**読み取り専用**（APIキー＋公開フォルダ）
- 書き込みは行わない（ユーザーが手動で Drive に置く）

### フォルダ構成（Drive側）
ローカルと同じ構成。ユーザーが Drive 上に手動で配置する。

```
[Drive公開フォルダ]/         ← フォルダIDをアプリに登録
└── [quiz-set-name]/
    └── 001/
        ├── question.png
        └── ...
```

### 差分チェック・ダウンロードフロー
1. ユーザーが「Drive と同期」ボタンをタップ
2. Drive フォルダのサブフォルダ一覧を取得（APIキーで認証なし）
3. ローカルのセット名一覧と比較
4. 差分を表示：
   - Drive にあってローカルにない → 「ダウンロード可能」リスト
   - ローカルにあって Drive にない → 「ローカルのみ」リスト
5. ユーザーがダウンロードしたいセットを選択→ダウンロード実行
6. ダウンロード: Drive のファイルを fetch → ローカルフォルダに書き込み

---

## アプリコードのファイル構成

```
pico-studyapp/
└── index.html    ← CSS・JS をすべてインライン
```

### index.html 内部の論理モジュール構成

```
<script> 内
├── idb             ← IndexedDB ラッパー（フォルダハンドル永続化）
├── localFS         ← File System Access API ラッパー
├── driveAPI        ← Drive API v3 読み取り専用（APIキー使用）
├── syncEngine      ← 差分計算・ダウンロード処理
├── store           ← 状態管理
├── timer           ← 学習タイマー
├── statsEngine     ← 統計保存（ローカルJSON）
├── quizLogic       ← フィルタ・シャッフル・ロード
├── chart           ← Canvas グラフ描画
├── h()             ← DOM ヘルパー
└── views/
    ├── viewSetup       ← 初回フォルダ選択・Drive設定
    ├── viewHome        ← ホーム・クイズセット一覧
    ├── viewQuizLoading ← 問題読み込み中
    ├── viewQuiz        ← クイズ解答
    ├── viewResult      ← 結果・統計保存
    ├── viewStats       ← 振り返り
    ├── viewCalendar    ← カレンダー
    ├── viewCreate      ← 問題作成
    └── viewSync        ← Drive差分・ダウンロード
```

---

## 画面仕様

### 0. セットアップ画面（初回起動・フォルダ未選択時）
- 「ワークスペースフォルダを選択」ボタン（`showDirectoryPicker`）
- Drive同期設定（任意）: フォルダID・APIキーの入力欄
- 設定は localStorage に保存

### 1. ホーム画面
- ワークスペース内のクイズセット一覧（セット名・最終スコア）
- [解答する] → モード選択モーダル → クイズ
- 下部ナビ: ホーム / 振り返り / カレンダー / 作成 / Drive同期

### 2. クイズ解答画面
- ローカルファイルから問題・選択肢を読み込み
- 画像・テキスト両対応

### 3. 結果画面
- スコア表示・問題別正誤
- 統計をローカルJSONに保存

### 4. 問題作成画面
- セット名・問題番号・ファイル選択
- ローカルフォルダに直接書き込み

### 5. 振り返り画面
- 日別勉強時間（棒グラフ）
- 問題別正答率・スコア推移（折れ線）

### 6. カレンダー画面
- 月表示・予定追加削除・学習時間バッジ

### 7. Drive同期画面
- Drive設定（フォルダID・APIキー）の確認・変更
- 差分リスト表示（Drive のみ / ローカルのみ）
- ダウンロードボタン（Drive→ローカル）

---

## localFS モジュール メソッド

```js
localFS.tryRestoreHandle()          // IDB からハンドル復元、パーミッション確認
localFS.requestPermission()         // 再許可ダイアログ表示
localFS.pickFolder()                // showDirectoryPicker → IDB に保存
localFS.initDataDir()               // data/ サブフォルダ取得 or 作成
localFS.listDirs(handle)            // サブディレクトリ一覧
localFS.listAll(handle)             // ファイル＋フォルダ一覧
localFS.readText(dirHandle, name)   // テキストファイル読み込み
localFS.readBlob(dirHandle, name)   // ファイルを Blob URL で取得
localFS.readJSON(name, default)     // data/ から JSON 読み込み
localFS.writeJSON(name, data)       // data/ に JSON 書き込み
localFS.writeFile(dirHandle, name, data) // 任意のフォルダにファイル書き込み
localFS.createDir(parentHandle, name)    // サブフォルダ作成
```

## driveAPI モジュール メソッド

```js
driveAPI.isConfigured()             // APIキー＋フォルダID が設定済みか
driveAPI.listChildren(folderId)     // フォルダ内アイテム一覧（APIキー使用）
driveAPI.downloadBlob(fileId)       // ファイルをBlobで取得
```

## syncEngine モジュール メソッド

```js
syncEngine.getDiff()                // {onlyInDrive, onlyInLocal} を返す
syncEngine.downloadQuizSet(item)    // Drive → ローカルへセット全体をDL
```

---

## JSONデータ構造

### data/study-log.json
```json
{
  "version": 1,
  "sessions": [
    { "sessionId": "uuid", "date": "2026-03-07", "endTime": "...", "durationSeconds": 2712,
      "quizSetName": "英単語A", "mode": "normal", "questionCount": 20, "correctCount": 15 }
  ],
  "dailySummary": {
    "2026-03-07": { "totalSeconds": 5400, "sessionIds": ["uuid"] }
  }
}
```

### data/quiz-stats.json
```json
{
  "version": 1,
  "quizSets": {
    "英単語A": {
      "questions": {
        "001": { "totalAttempts": 12, "correctAttempts": 9, "correctRate": 0.75,
                 "lastAttemptDate": "2026-03-07", "lastResult": "correct", "history": [...] }
      },
      "sessionScores": [{ "sessionId": "uuid", "date": "2026-03-07", "score": 15, "total": 20 }]
    }
  }
}
```

### data/calendar.json
```json
{
  "version": 1,
  "events": {
    "2026-03-15": [{ "eventId": "evt-001", "type": "exam", "title": "英検2級", "note": "" }]
  }
}
```

---

## タイマー (timer)

IDLE → RUNNING ⇄ PAUSED → IDLE
visibilitychange(hidden) 時に localStorage バッファに書き込み。

---

## 苦手対策モードのロジック

```js
filter(all, 'weak', setName, stats, threshold)
// correctRate <= threshold の問題のみ。未解答は含める。

filter(all, 'wrong_last', setName, stats)
// lastResult === 'wrong' の問題のみ
```

---

## 実装フェーズ

| フェーズ | 内容 | ステータス |
|---|---|---|
| 1. 基盤 | idb / localFS / store / セットアップ画面 | 完了 |
| 2. クイズ | quizLogic / viewHome / viewQuiz / viewResult | 完了 |
| 3. 統計 | statsEngine / chart / viewStats | 完了 |
| 4. カレンダー | viewCalendar | 完了 |
| 5. 作成 | viewCreate | 完了 |
| 6. Drive同期 | driveAPI / syncEngine / viewSync | 完了 |
| 7. 仕上げ | モバイルCSS・エラー処理・非対応ブラウザ警告 | 完了 |

---

## 重要な実装注意点

### File System Access API
- `showDirectoryPicker()` はユーザー操作イベント内でのみ呼べる
- `file://` では Chrome/Edge で動作する（Firefoxは非対応・Safariは部分対応）
- フォルダハンドルは IndexedDB に保存し、再起動時は `requestPermission()` で再許可
- `FileSystemWritableFileStream` で画像・JSONを書き込み可能
- `URL.createObjectURL()` の Blob URL は画面遷移時に `revokeObjectURL()` で解放

### Drive API（読み取り専用）
- APIキーのみ使用（OAuth不要）
- Drive フォルダは「リンクを知っている全員 - 閲覧者」で共有必須
- GCP Console で Drive API を有効化 + APIキーを制限なし or Drive API に限定

### ホスティング
- 基本動作（クイズ）: `file://` で直接開くだけで動く
- Drive同期: `file://` でも動く（APIキーの fetch に制約なし）
- CORS: Drive API は `mode: 'cors'` で fetch 可

### スマホ対応
- Android Chrome: File System Access API 対応
- iOS Safari: File System Access API **非対応** → 将来的に `<input webkitdirectory>` フォールバック追加を検討
- タップターゲット最低 44×44px
