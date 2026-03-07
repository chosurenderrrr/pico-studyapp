# pico-studyapp 仕様書

## Context
ローカルフォルダにある問題素材（画像・テキスト）を読み込んで多択クイズとして解答できるWebアプリ。
サーバー不要でブラウザだけで動作し、自分で問題を作って繰り返し学習することを目的とする。

---

## クイズセットのフォルダ構成

```
quiz-set-name/
├── 001/
│   ├── question.png  または  question.txt
│   ├── correct.png   または  correct.txt
│   ├── wrong1.png    または  wrong1.txt
│   └── wrong2.png    ...（wrong3, wrong4 と増やせる）
├── 002/
│   └── ...
```

- ルートフォルダ = クイズセット
- サブフォルダ（001, 002...）= 各問題
- テキストと画像はどちらでもOK

---

## 画面仕様

### 1. ホーム画面 (views/home.js)
- [フォルダを開く] ボタン → `showDirectoryPicker()` 起動
- 登録済みクイズセット一覧（カード形式）: セット名・問題数・最終スコアを表示
- [解答する] → クイズ解答画面へ遷移
- [削除] → 一覧から除去（ファイルは消さない）
- [作成] → クイズ作成画面へ遷移

### 2. クイズ解答画面 (views/quiz.js)
- 進捗バー（「3 / 10」形式）
- 問題エリア: `question.png` or `question.txt` を表示
- 選択肢ボタン（グリッドレイアウト、画像・テキスト両対応）
- 選択後: 正解=緑、不正解=赤にハイライト
- [次へ] ボタン（最終問題では [結果を見る]）
- [終了] ボタン（確認ダイアログあり）

### 3. 結果画面 (views/result.js)
- スコア表示（「7 / 10 正解」+ パーセント）
- 問題別正誤一覧（○×マーク）
- [もう一度] → 同じセットで再挑戦（シャッフルし直し）
- [ホームへ] ボタン
- 最高スコア更新時はバッジ表示

### 4. クイズ作成画面 (views/create.js)
- 保存先フォルダを `showDirectoryPicker()` で選択
- 問題入力: テキスト入力 or 画像ファイル選択
- 正解・誤答も同様（[誤答を追加] で動的に行を追加）
- [保存] → File System Access API の `createWritable()` で書き出し
- 問題番号は既存最大+1で自動採番（3桁ゼロパディング）
- 既存フォルダを選択すると継続編集が可能

---

## データ構造

### メモリ上

```js
Question = {
  id: string,              // "001"
  dirHandle: FileSystemDirectoryHandle,
  question: { type: "text"|"image", content: string, fileName: string },
  correct:  { type: "text"|"image", content: string, fileName: string },
  wrongs:   [{ type: "text"|"image", content: string, fileName: string }]
}

Session = {
  setName: string,
  questions: Question[],   // シャッフル済み
  currentIndex: number,
  answers: [{ questionId, selectedIndex, isCorrect, shuffledOptions }]
}
```

### localStorage

```js
// "pico-studyapp-sets"
[{ name, questionCount, lastScore, bestScore, lastPlayedAt }]

// "pico-studyapp-history"
[{ setName, playedAt, score, total, details: [{ questionId, isCorrect }] }]

// "pico-studyapp-settings"
{ questionCount: number|"all", shuffleQuestions: bool, shuffleOptions: bool }
```

※ `FileSystemDirectoryHandle` はlocalStorageに保存不可 → メタデータのみ保存。
　再起動後はホーム画面でフォルダを再選択してもらう。

---

## 実装順序

| フェーズ | 内容 | ステータス |
|---|---|---|
| 1. 基盤 | index.html / store.js / app.js（ルーター） | 未着手 |
| 2. FS連携 | fileSystem.js（読み込み・書き出し・権限管理） | 未着手 |
| 3. ホーム | views/home.js | 未着手 |
| 4. 解答 | quiz.js（ロジック） + views/quiz.js + views/result.js | 未着手 |
| 5. 作成 | views/create.js | 未着手 |
| 6. 仕上げ | CSS整備・エラーハンドリング・設定画面 | 未着手 |

---

## 重要な実装注意点

- `showDirectoryPicker()` はユーザー操作イベントハンドラ内でのみ呼び出せる
- 書き込み前に `requestPermission({ mode: 'readwrite' })` が必要
- `URL.createObjectURL()` で生成したBlob URLは画面遷移時に `revokeObjectURL()` で解放
- File System Access API は Chrome/Edge 86以上が必要（Firefox/Safari非対応）。非対応ブラウザには警告バナーを表示
