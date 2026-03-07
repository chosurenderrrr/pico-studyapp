# pico-studyapp 仕様書（v2）

## Context
ローカルフォルダから問題素材を読み込んで多択クイズとして学習できるSPA。
HTML + CSS + JavaScript のみ。サーバー・フレームワーク不使用。
学習時間記録・カレンダー・苦手対策モードを追加して学習管理アプリとして拡充する。

---

## ユーザーのワークスペースフォルダ構成

```
[workspace]/             ← 起動時にユーザーが1回選ぶルートフォルダ
├── data/                ← アプリが自動生成
│   ├── study-log.json   ← 日別勉強時間・セッション記録
│   ├── quiz-stats.json  ← 問題ごとの正答率・解答履歴
│   └── calendar.json    ← カレンダーの予定
└── [quiz-set-name]/     ← クイズセット（複数可）
    └── 001/
        ├── question.png/txt
        ├── correct.png/txt
        └── wrong1.png/txt
```

---

## アプリコードのファイル構成

```
pico-studyapp/           ← gitリポジトリ
├── index.html
├── style.css
└── js/
    ├── app.js           ← SPAルーター・ワークスペース初期化
    ├── store.js         ← 状態管理・localStorage連携
    ├── fileSystem.js    ← File System Access API（JSON読み書き含む）
    ├── quiz.js          ← クイズロジック・フィルタリング
    ├── timer.js         ← [新規] 学習タイマー
    ├── statsEngine.js   ← [新規] 統計計算・集計
    ├── chartRenderer.js ← [新規] Canvas グラフ描画
    └── views/
        ├── home.js      ← ホーム・モード選択モーダル
        ├── quiz.js      ← クイズ解答
        ├── result.js    ← 結果・統計保存
        ├── create.js    ← 問題作成
        ├── stats.js     ← [新規] 振り返り画面
        └── calendar.js  ← [新規] カレンダー画面
```

---

## 画面仕様

### 1. ホーム画面 (views/home.js)
- 初回起動時: ワークスペースフォルダ選択（以降はlocalStorageに記憶して省略）
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

## fileSystem.js 追加メソッド

```js
ensureDataFolder()                          // data/ フォルダ自動生成
readJSON(relativePath, defaultValue)        // なければデフォルト値を返す
writeJSON(relativePath, data)               // 上書き保存
mergeJSON(relativePath, partial, default)   // 読み込み→マージ→書き込み
```

書き込み競合対策: 呼び出し側で Promise チェーンによるキュー管理を行う。

---

## 実装フェーズ

| フェーズ | 内容 | ステータス |
|---|---|---|
| 1. 基盤 | fileSystem.js 拡張 / app.js ワークスペース選択フロー / store.js | 未着手 |
| 2. タイマー | timer.js 新規 / quiz.js・result.js に連携 | 未着手 |
| 3. 統計保存 | statsEngine.js 新規 / result.js 呼び出し / quiz.js フィルタ | 未着手 |
| 4. 基本画面 | home.js / quiz.js / result.js / create.js | 未着手 |
| 5. 振り返り | chartRenderer.js 新規 / stats.js 新規 | 未着手 |
| 6. カレンダー | calendar.js 新規 | 未着手 |
| 7. モード拡張 | home.js モーダル / 苦手・誤答モード繋ぎ込み | 未着手 |
| 8. 仕上げ | CSS整備・エラーハンドリング・非対応ブラウザ警告 | 未着手 |

---

## 重要な実装注意点

- `showDirectoryPicker()` はユーザー操作イベントハンドラ内でのみ呼び出せる
- 書き込みは `requestPermission({ mode: 'readwrite' })` が必要
- `URL.createObjectURL()` の Blob URL は画面遷移時に `revokeObjectURL()` で解放
- File System Access API は Chrome/Edge 86以上（Firefox/Safari非対応）→ 起動時に警告
- `visibilitychange` で hidden 時は即 localStorage バッファ書き込み
