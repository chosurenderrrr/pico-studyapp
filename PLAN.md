# pico-studyapp 仕様書（v2）

## Context
ローカルフォルダから問題素材を読み込んで多択クイズとして学習できるSPA。
HTML + CSS + JavaScript のみ。サーバー・フレームワーク不使用。
学習時間記録・カレンダー・苦手対策モードを追加して学習管理アプリとして拡充する。

### 追加方針（2026-03-07）
- **スマートフォン利用前提**: UIはモバイルファースト設計
- **単一HTMLファイル**: `index.html` 1ファイルに CSS・JS をすべてインライン実装
- **Google Driveをストレージに変更**: File System Access API（ローカルフォルダ）をやめ、Google Drive API v3 でクラウドフォルダを参照・読み書きする
- **Google認証必須**: ユーザーは Google アカウントでログインしてから利用する（OAuth 2.0 / Google Identity Services）

---

## ユーザーのワークスペースフォルダ構成

フォルダ構成はローカル版と同じだが、**Google Drive 上のフォルダ**を使用する。

```
[workspace]/             ← Google Drive内の任意フォルダ（初回ログイン後にアプリが自動作成 or ユーザーが指定）
├── data/                ← アプリが自動生成（Google Drive上）
│   ├── study-log.json   ← 日別勉強時間・セッション記録
│   ├── quiz-stats.json  ← 問題ごとの正答率・解答履歴
│   └── calendar.json    ← カレンダーの予定
└── [quiz-set-name]/     ← クイズセット（複数可）
    └── 001/
        ├── question.png/txt
        ├── correct.png/txt
        └── wrong1.png/txt
```

### Google Drive 利用フロー
1. アプリ起動 → Google ログインボタン表示
2. OAuth 2.0 認証 → アクセストークン取得（スコープ: `drive.file` または `drive.appdata`）
3. ワークスペースフォルダを Google Drive API で検索 or 新規作成
4. 以降のファイル読み書きはすべて Google Drive API v3 経由

---

## アプリコードのファイル構成

**単一HTMLファイル構成**（スマホ配布・ホスティング簡易化のため）

```
pico-studyapp/           ← gitリポジトリ
└── index.html           ← CSS・JS をすべてインライン（<style>/<script>タグ）
```

### index.html 内部の論理モジュール構成（コメントで区切る）

```
<script> 内
├── gdriveApi.js   相当  ← Google Drive API v3 ラッパー（認証・CRUD）
├── store.js       相当  ← 状態管理・localStorage（トークン・設定キャッシュ）
├── quiz.js        相当  ← クイズロジック・フィルタリング
├── timer.js       相当  ← 学習タイマー
├── statsEngine.js 相当  ← 統計計算・集計
├── chartRenderer.js相当 ← Canvas グラフ描画
├── app.js         相当  ← SPAルーター・ワークスペース初期化
└── views/
    ├── home       相当  ← ホーム・モード選択モーダル
    ├── quiz       相当  ← クイズ解答
    ├── result     相当  ← 結果・統計保存
    ├── create     相当  ← 問題作成
    ├── stats      相当  ← 振り返り画面
    └── calendar   相当  ← カレンダー画面
```

---

## 画面仕様

### 1. ホーム画面 (views/home.js)
- 初回起動時: Google ログインボタン → 認証 → ワークスペースフォルダ自動セットアップ（以降はlocalStorageにフォルダIDを記憶して省略）
- クイズセット一覧カード（セット名・問題数・最終スコア）
- [解答する] → モード選択モーダルを表示してから解答画面へ
- [削除] / [作成] / [振り返り] / [カレンダー] ナビ

**モード選択モーダル（解答開始前）**
- 通常モード / 苦手対策モード / 前回誤答モード の3択
- 苦手対策モード選択時: 閾値スライダー（0〜100%、デフォルト60%）
  - 「対象問題数: X問 / 全Y問」をリアルタイム表示
  - 0問のとき [開始] ボタンを disabled + 警告メッセージ

### 2. クイズ解答画面 (views/quiz.js)
- タイマー連携: 問題表示時に `timer.startQuestion()` 呼び出し
- 進捗バー・問題エリア・選択肢ボタン（既存仕様に加算）
- 選択後: 正解=緑、不正解=赤ハイライト
- [次へ] / [終了]

### 3. 結果画面 (views/result.js)
- スコア表示・問題別正誤一覧
- 画面表示時に `timer.endSession()` + `statsEngine.updateStats()` を呼び出す
- [もう一度] / [ホームへ]

### 4. 問題作成画面 (views/create.js)
- 既存仕様のまま。保存先はワークスペースフォルダ直下に作成

### 5. 振り返り画面 (views/stats.js)
- セクション1: 日別勉強時間（過去30日・棒グラフ）
- セクション2: クイズセット選択
- セクション3: 問題別正答率テーブル（色分けバー付き）
- セクション4: セットごとのスコア推移（折れ線グラフ）
- カレンダーから日付付きで遷移した場合はその日を強調表示

### 6. カレンダー画面 (views/calendar.js)
- 月表示カレンダー（前月・次月ナビ）
- 予定がある日: タイプ別カラードット
- 学習記録がある日: 勉強時間バッジ
- 日付クリック → `<dialog>` モーダル
  - 既存予定のリスト（編集・削除）
  - 新規予定フォーム（タイプ・タイトル・メモ）
  - [振り返りを見る] → stats.js に日付パラメータ付きで遷移

**予定タイプ**
| type | 色 | 意味 |
|---|---|---|
| exam | #e74c3c（赤） | 試験・テスト |
| goal | #2ecc71（緑） | 学習目標 |
| other | #3498db（青） | その他 |

---

## JSONデータ構造

### study-log.json
```json
{
  "version": 1,
  "sessions": [
    {
      "sessionId": "uuid",
      "date": "2026-03-07",
      "startTime": "2026-03-07T09:00:00.000Z",
      "endTime": "2026-03-07T09:45:12.000Z",
      "durationSeconds": 2712,
      "quizSetName": "英単語A",
      "mode": "normal",
      "questionCount": 20,
      "correctCount": 15
    }
  ],
  "dailySummary": {
    "2026-03-07": { "totalSeconds": 5400, "sessionIds": ["uuid"] }
  }
}
```

### quiz-stats.json
```json
{
  "version": 1,
  "quizSets": {
    "英単語A": {
      "questions": {
        "001": {
          "totalAttempts": 12,
          "correctAttempts": 9,
          "correctRate": 0.75,
          "lastAttemptDate": "2026-03-07",
          "lastResult": "correct",
          "history": [
            { "sessionId": "uuid", "date": "2026-03-07", "result": "correct", "elapsedSeconds": 4.2 }
          ]
        }
      },
      "sessionScores": [
        { "sessionId": "uuid", "date": "2026-03-07", "score": 15, "total": 20 }
      ]
    }
  }
}
```
※ `history` は最新100件に制限

### calendar.json
```json
{
  "version": 1,
  "events": {
    "2026-03-15": [
      { "eventId": "evt-001", "type": "exam", "title": "英検2級", "note": "" }
    ]
  }
}
```

---

## タイマー実装 (timer.js)

**状態:** IDLE → RUNNING ⇄ PAUSED → IDLE

```js
// 公開インターフェース
timer.startSession(quizSetName, mode)
timer.startQuestion(questionId)
timer.endQuestion(result)        // → { questionId, elapsedSeconds, result }
timer.endSession(correct, total) // → ファイル書き込み
timer.handleVisibility()         // visibilitychange に登録
timer.recoverFromBuffer()        // 起動時に呼ぶ（localStorage バッファ回収）
```

**ロスト対策:**
1. 起動時: localStorage バッファ残があれば study-log.json に追記してクリア
2. visibilitychange(hidden): localStorage にバッファ書き込み
3. endSession(): ファイル書き込み成功後にバッファクリア

---

## 苦手対策モードのロジック (quiz.js)

```js
function filterQuestions(allQuestions, mode, setName, statsData, threshold) {
  if (mode === 'weak') {
    return allQuestions.filter(q => {
      const stat = statsData?.quizSets?.[setName]?.questions?.[q.id];
      if (!stat) return true;               // 未解答は苦手扱いで含める
      return stat.correctRate <= threshold;
    });
  }
  if (mode === 'wrong_last') {
    return allQuestions.filter(q => {
      const stat = statsData?.quizSets?.[setName]?.questions?.[q.id];
      return stat?.lastResult === 'wrong';
    });
  }
  return allQuestions; // normal
}
```

---

## グラフ実装 (chartRenderer.js)

Canvas APIを使用（SVGより DOM ノード数が増えないため選択）。

```js
ChartRenderer.drawBarChart(canvas, { labels, values, unit, color, highlightDate })
ChartRenderer.drawLineChart(canvas, { labels, datasets: [{ label, values, color }] })
```

---

## gdriveApi.js 相当のメソッド（旧 fileSystem.js）

Google Drive API v3 を使用してファイル操作を行う。

```js
// 認証
gdrive.login()                              // Google OAuth ポップアップ → アクセストークン取得
gdrive.logout()                             // トークン破棄・ログアウト
gdrive.isLoggedIn()                         // 認証済みかチェック

// フォルダ・ファイル操作
gdrive.ensureWorkspace(name)                // ルートワークスペースフォルダ取得 or 新規作成
gdrive.ensureDataFolder(workspaceFolderId)  // data/ サブフォルダ取得 or 新規作成
gdrive.listQuizSets(workspaceFolderId)      // クイズセットフォルダ一覧取得
gdrive.listQuestions(setFolderId)           // 問題番号フォルダ一覧取得
gdrive.readFileText(fileId)                 // テキストファイル内容取得
gdrive.readFileBlob(fileId)                 // 画像ファイルを Blob URL で取得
gdrive.readJSON(folderId, filename, def)    // JSON読み込み（なければデフォルト値）
gdrive.writeJSON(folderId, filename, data)  // JSON上書き保存（multipart upload）
gdrive.uploadFile(parentId, name, blob)     // ファイルアップロード（問題作成用）
```

書き込み競合対策: 呼び出し側で Promise チェーンによるキュー管理を行う。

### 認証スコープ
- `https://www.googleapis.com/auth/drive.file`  ← アプリが作成したファイルのみアクセス可（推奨）
- またはワークスペース選択が必要なら `drive` または `drive.readonly` + `drive.file` の組み合わせ

---

## 実装フェーズ

| フェーズ | 内容 | ステータス |
|---|---|---|
| 1. 基盤 | index.html 骨格 / Google OAuth 認証フロー / gdriveApi 実装 / store | 未着手 |
| 2. ワークスペース | ログイン後のフォルダセットアップ / クイズセット一覧取得 | 未着手 |
| 3. タイマー | timer 実装 / quiz・result に連携 | 未着手 |
| 4. 統計保存 | statsEngine 実装 / result 呼び出し / quiz フィルタ | 未着手 |
| 5. 基本画面 | home / quiz / result / create ビュー | 未着手 |
| 6. 振り返り | chartRenderer / stats ビュー | 未着手 |
| 7. カレンダー | calendar ビュー | 未着手 |
| 8. モード拡張 | home モーダル / 苦手・誤答モード繋ぎ込み | 未着手 |
| 9. 仕上げ | モバイルCSS整備・エラーハンドリング・オフライン警告・認証切れ対応 | 未着手 |

---

## 重要な実装注意点

### Google Drive / 認証
- Google Identity Services (GIS) の `google.accounts.oauth2.initTokenClient` を使用
- アクセストークンは有効期限1時間 → 期限切れ検知時に自動リフレッシュ or 再ログイン促す
- `drive.file` スコープはアプリが作成したファイルのみ操作可。既存のユーザーフォルダを参照させる場合は `drive.readonly` も追加が必要
- GCP コンソールで OAuth クライアントID・Drive API 有効化が必須（`index.html` 内に CLIENT_ID を埋め込む）
- **CORS**: Drive API はブラウザから直接呼べる（`fetch` で OK）

### 単一ファイル
- 外部 CDN スクリプトは `<script src>` で読み込み可（GIS ライブラリ等）
- CSS・アプリJS はすべて `<style>`/`<script>` タグにインライン
- 画像は Google Drive の fileId 経由で取得（Blob URL に変換して `<img src>` に渡す）

### スマホ対応
- `<meta name="viewport" content="width=device-width, initial-scale=1">` 必須
- タップターゲットは最低 44×44px
- `visibilitychange` で hidden 時は即 localStorage バッファ書き込み
- `URL.createObjectURL()` の Blob URL は画面遷移時に `revokeObjectURL()` で解放
