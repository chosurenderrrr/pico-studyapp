# TODO

## タスク一覧

### Feature 1: problem.json への完全移行

#### ProblemIO.load の書き換え
- [x] `type.txt` の読み込みを削除
- [x] .txt ファイル群（title/question/explanation/correct/wrong/item/dummy）の読み込みをすべて削除
- [x] `problem.json` を読み込んで JSON.parse し返す処理に置き換え

#### ProblemIO.save の書き換え
- [x] .txt ファイルへの writeText 処理をすべて削除
- [x] removeEntry の呼び出しをすべて削除
- [x] `problem.json` を JSON.stringify して1回で書き込む処理に置き換え

### Feature 2: 保存後の全問再読み込みを廃止

- [x] `Editor.save()` 内の `ProblemIO.loadAll()` 呼び出しを削除
- [x] 既存問題の場合：`State.problems` 内の該当オブジェクトを保存後の problem で置き換え
- [x] 新規問題の場合：`State.problems` に追加して `num` 順でソート
- [x] `App.renderProblemList()` を呼んで再描画（既存コードをそのまま使用）

### 動作確認
- [ ] 新規問題を作成・保存 → problem.json が生成されること
- [ ] 既存問題を編集・保存 → problem.json が更新されること
- [ ] 保存後に問題一覧が正しく更新されること（全問再読み込みなし）
- [ ] 読み込み：問題数が多いセットで速度が改善されること
- [ ] 画像あり問題（question/explanation が画像）で正常に保存・読み込みできること
- [ ] 整序問題（order タイプ）で正常に保存・読み込みできること
