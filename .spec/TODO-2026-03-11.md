# TODO

## タスク一覧

### Feature 1: 処理の高速化
- [x] `ProblemIO.loadAll`：全問題を `Promise.all` で並列読み込みに変更
- [x] `ProblemIO.load`：内部の独立したファイル読み込みを並列化
- [x] `ProblemIO.save`：独立した書き込みを `Promise.all` で並列化

### Feature 2: 「出題時の表示数」廃止
- [x] エディタから `#mc-display-count-section` の HTML を削除
- [x] `ProblemIO.save` から `display_count.txt` 書き込みを削除
- [x] `ProblemIO.load` から `display_count.txt` 読み込みを削除
- [x] クイズ処理（`Quiz`）で `displayCount` を使わず全選択肢を使用するよう変更

### Feature 3: 整序問題にダミー選択肢を追加
- [x] `ProblemIO.load`：`dummy1.txt`〜`dummyN.txt` を読み込み `p.dummies[]` に格納
- [x] `ProblemIO.save`：`dummies[]` を `dummy1.txt`〜`dummyN.txt` として書き込み
- [x] エディタ：「ダミーの選択肢」セクション HTML を追加
- [x] エディタ：`Editor.load` でダミー選択肢を読み込み表示
- [x] エディタ：`Editor.save` でダミー選択肢を収集
- [x] クイズ：整序問題の選択肢を `items + dummies` のシャッフルに変更
- [x] クイズ：スロット数を `items.length` のみに変更（ダミーはスロット対象外）

### Feature 4: 保存ボタンをコンテンツ末尾に追加
- [x] エディタ末尾（削除ボタンの上）に保存ボタンを追加

### 動作確認
- [ ] 高速化：問題数が多いセットで読み込み時間が短縮されることを確認
- [ ] 表示数廃止：既存データのあるセットで全選択肢が表示されることを確認
- [ ] ダミー選択肢：整序問題でダミーが混在して出題されることを確認
- [ ] 保存ボタン：末尾の保存ボタンで正常に保存されることを確認
