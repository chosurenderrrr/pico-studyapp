# TODO

## タスク一覧

### Feature 1: エディタ：選択肢行の画像対応

#### 選択肢行の UI 変更（共通化）
- [x] `_makeChoiceRow(...)` ヘルパーを作成：テキスト/画像タブ付きの行を生成
- [x] `addCorrectItem(data)` を新 UI に置き換え
- [x] `addWrongChoice(data)` を新 UI に置き換え
- [x] `addOrderItem(data)` を新 UI に置き換え
- [x] `addOrderDummy(data)` を新 UI に置き換え
- [x] `setChoiceType(btn, type)` を追加：タブ切り替え処理
- [x] `pickChoiceImage(btn)` を追加：画像選択 → dataURL を data-img にセット

#### リナンバー処理の修正
- [x] `removeCorrectItem` / `removeWrongChoice` / `removeOrderDummy` のリナンバーを新構造に対応

#### Editor.save のデータ収集変更
- [x] `#order-items` 収集：行の type/img を判定して `{ type, content }` を構築
- [x] `#order-dummies` 収集：同上
- [x] `#correct-choices` 収集：同上
- [x] `#wrong-choices` 収集：同上

### Feature 2: クイズ：選択肢の画像表示

- [x] `renderChoiceContent(choice)` ヘルパー関数を追加
- [x] `renderMultipleChoice`：`mcChoicesData` に `type` フィールドを含める
- [x] `renderMultipleChoice`：ボタン内の選択肢を `renderChoiceContent` に変更
- [x] `renderOrder`：`orderAllChoices` に `type` フィールドを含める
- [x] `_renderOrderChoices`：`btn.textContent` → `btn.innerHTML = renderChoiceContent(choice)` に変更
- [x] `_renderOrderSlots`：スロット内の content 表示を `renderChoiceContent` に変更
- [x] `confirmMultipleChoice`：不正解時の正解リスト表示を画像対応
- [x] `confirmOrder`：不正解時の正解順番表示を画像対応

### 動作確認
- [ ] 選択問題：正解に画像を設定して保存・読み込みできること
- [ ] 選択問題：誤答に画像を設定して保存・読み込みできること
- [ ] 選択問題クイズ：画像選択肢が表示され、正誤判定が正しく動作すること
- [ ] 整序問題：正解選択肢に画像を設定して保存・読み込みできること
- [ ] 整序問題：ダミーに画像を設定して保存・読み込みできること
- [ ] 整序問題クイズ：画像選択肢がボタン・スロット両方で表示されること
- [ ] テキストと画像が混在する問題が正常に動作すること
