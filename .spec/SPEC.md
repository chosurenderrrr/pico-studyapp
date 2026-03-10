# SPEC - 技術仕様・要件定義

## 機能概要
選択肢（正解・誤答・整序アイテム・ダミー）に画像を登録・表示できるようにする。
選択問題・整序問題の両方が対象。

---

## 1. エディタ：選択肢行の画像対応

### 変更対象
- `correct-choices`（選択問題：正解）
- `wrong-choices`（選択問題：誤答）
- `order-items`（整序問題：正解選択肢）
- `order-dummies`（整序問題：ダミー）

### 各選択肢行の UI
各行に「テキスト / 画像」タブを追加する（問題文・解説の切り替えと同じパターン）。

**テキストモード（デフォルト）：**
- 現状の textarea をそのまま使用

**画像モード：**
- 「📷 画像を選択」ボタン
- 選択後はプレビュー画像を表示
- 画像の dataURL を行の `data-img` 属性に保持
- プレビュー横に「✕ 画像を削除」ボタン（削除でテキストモードに戻る）

### データ収集（Editor.save）
現在の `querySelectorAll('textarea')` での収集を、行ごとに以下に変更：
- `data-type="text"` の行 → `{ type: 'text', content: textarea.value.trim() }`
- `data-type="image"` の行 → `{ type: 'image', content: row.dataset.img }`
- 画像モードで画像未選択の行は収集しない（スキップ）

### ロード時（Editor.load）
既存の `addOrderItem(data)` / `addCorrectItem(data)` 等に渡す `data` の
`type === 'image'` 判定を追加し、画像モードで行を初期化する。

---

## 2. クイズ：選択肢の画像表示

### 共通ヘルパー関数（新規追加）
```javascript
function renderChoiceContent(choice) {
  if (choice.type === 'image') {
    return `<img src="${choice.content}" style="max-width:100%;max-height:120px;border-radius:6px;display:block;margin:4px auto">`;
  }
  return `<span>${choice.content}</span>`;
}
```

### 選択問題（renderMultipleChoice）
現状：
```javascript
btn.innerHTML = `<span class="choice-badge">${labels[i]}</span><span style="flex:1">${c.content}</span>`;
```
変更後：
```javascript
btn.innerHTML = `<span class="choice-badge">${labels[i]}</span><span style="flex:1">${renderChoiceContent(c)}</span>`;
```
- `mcChoicesData` に `type` フィールドを含めるよう変更

### 整序問題（_renderOrderChoices / _renderOrderSlots）
- `btn.textContent = choice.content` → `btn.innerHTML = renderChoiceContent(choice)`
- スロットの `order-slot-content` 内も同様に `renderChoiceContent` を使用

### 不正解時の正解表示（confirmMultipleChoice）
現状：`・${c.content}`
変更後：画像の場合はインライン `<img>` で表示

---

## 3. CSS 調整

- 画像タイプの選択肢ボタン（`quiz-choice`, `order-choice`）が縦に広がっても崩れないよう `height: auto` を確認
- `order-slot` が画像を内包できるよう `min-height` の調整（必要に応じて）

---

## 非機能要件（継続）
- 単一HTMLファイル（`index.html`）にインライン実装
- 外部ライブラリ禁止
- Android Chrome タッチ操作対応
