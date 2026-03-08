# TODO

## タスク一覧

### Phase 1: データ層（ProblemIO）
- [x] `ProblemIO.load()` に `type.txt` 読み込みを追加（"order" → `problemType = 'order'`）
- [x] `ProblemIO.load()` に `item1.txt`〜`item20.txt` の読み込みを追加（orderの場合）
- [x] `ProblemIO.save()` に `type.txt` 書き込みを追加（orderの場合）
- [x] `ProblemIO.save()` に `item*.txt` 書き込みを追加（orderの場合）
- [x] `ProblemIO.save()` で order 保存時に `correct.txt` / `wrong*.txt` を削除

### Phase 2: CSS
- [x] 整序問題の回答スロットスタイル（`.order-slot`）を追加
- [x] 整序問題の選択肢ボタンスタイル（`.order-choice`）を追加
- [x] 正解・不正解スロットの色（`.order-slot.correct` / `.order-slot.wrong`）を追加
- [x] 「回答確定」ボタンスタイルを追加（無効時にグレーアウト）

### Phase 3: エディタ（Editor）
- [x] エディタHTML上部に「問題タイプ」タブ（4択 / 整序）を追加
- [x] エディタHTML に整序問題用の選択肢エリア（`#order-items`）を追加
- [x] `Editor` に `problemType` プロパティを追加
- [x] `Editor.setProblemType(type)` を実装（タブ切替で正解・不正解セクション表示切り替え）
- [x] `Editor.addOrderItem(data)` を実装（選択肢を1行追加、最大20個）
- [x] `Editor.removeOrderItem(btn)` を実装（行削除）
- [x] `Editor.load()` に整序問題の読み込み処理を追加（タブ切替 + items 展開）
- [x] `Editor.save()` に整序問題の保存処理を追加（items 収集・バリデーション・ProblemIO呼び出し）

### Phase 4: クイズ画面（Quiz）
- [x] クイズHTMLに整序問題用エリアを追加（`#order-answer-area`, `#order-choices-area`, `#order-confirm-btn`）
- [x] `Quiz.render()` に `problemType === 'order'` 分岐を追加（既存の4択UIを隠し、整序UIを表示）
- [x] `Quiz.renderOrder(p)` を実装：
  - items をシャッフルして選択肢エリアに表示
  - 空の回答スロットを items.length 個作成
  - 各選択肢タップ → `Quiz.selectOrderItem()` 呼び出し
- [x] `Quiz.selectOrderItem(item)` を実装（選択肢 → スロットへ移動、全埋まりで確定ボタン有効化）
- [x] `Quiz.deselectOrderItem(slot)` を実装（スロット → 選択肢エリアへ戻す）
- [x] `Quiz.confirmOrder()` を実装（回答確定：順序比較 → `Quiz.answer()` 相当の処理）
- [x] 不正解時のフィードバック：スロットを色分け＋正解順テキストを表示
- [x] `Quiz.next()` / `Quiz.answer()` が整序問題でも正しく動作するよう調整

### Phase 5: 問題一覧
- [x] `App.renderProblemList()` の問題プレビューに整序問題アイコン（🔢）を表示

### Phase 6: 動作確認
- [x] 整序問題の作成・保存・再読み込みが正常に動作する
- [x] 4択問題と整序問題が同一セット内に混在して出題される
- [x] 苦手モード・前回ミスモードで整序問題が正しく絞り込まれる
- [x] 不正解フィードバック（スロット色分け + 正解順表示）が正常に動作する
