# TODO

## タスク一覧

### Phase 1: IndexedDB ユーティリティ
- [x] `IDB` オブジェクトを追加（`open()`, `saveHandle()`, `loadHandle()`）

### Phase 2: ウェルカム画面 HTML 変更
- [x] 「前回フォルダなし」モードのHTMLを維持（初期状態）
- [x] 「前回フォルダあり」モード用の要素を追加（フォルダ名表示・「前回で始める」ボタン・「別のフォルダ」ボタン）

### Phase 3: App.init() 変更
- [x] IndexedDB から handle を読み込み、存在すればウェルカム画面を「前回フォルダあり」モードに切り替え

### Phase 4: 「前回のフォルダで始める」処理
- [x] `App.resumeWorkspace()` を実装（queryPermission → requestPermission → 成功時 Data.init() → loadHome()）
- [x] 失敗時のトースト表示とフォールバック

### Phase 5: フォルダ選択時の handle 保存
- [x] `App.selectWorkspace()` 成功時に IndexedDB へ handle を保存

### Phase 6: 動作確認
- [ ] 初回：フォルダ選択 → 保存される
- [ ] 2回目：「前回のフォルダで始める」で即開始できる
- [ ] フォルダ変更：「別のフォルダを選択」で上書き保存される
- [ ] Android Chrome で動作確認
