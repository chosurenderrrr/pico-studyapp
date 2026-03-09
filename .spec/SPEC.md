# SPEC - 技術仕様・要件定義

## 機能概要
アプリ起動時に前回使用したフォルダを IndexedDB に記憶し、次回起動時に1タップで再開できるようにする。

---

## 技術方針
- `FileSystemDirectoryHandle` は IndexedDB に保存可能（localStorage 不可）
- ブラウザのセキュリティ制約により、パーミッション復元にはユーザー操作（タップ）が必要
- `handle.queryPermission()` で現在のパーミッション状態を確認
- `handle.requestPermission({ mode: 'readwrite' })` でパーミッションを要求

---

## ウェルカム画面 UI

### 初回起動（IndexedDB にハンドルなし）
```
📚 StudyApp
通勤中でも手軽に勉強できる
あなただけの学習アプリ

[📁 フォルダを選択して始める]
```

### 2回目以降（IndexedDB にハンドルあり）
```
📚 StudyApp
前回のフォルダ: {フォルダ名}

[▶ 前回のフォルダで始める]   ← メインボタン（primary）
[📁 別のフォルダを選択]       ← サブボタン（secondary）
```

---

## 実装詳細

### IndexedDB 操作
- DB名: `studyapp-db`
- ストア名: `workspace`
- キー: `'handle'`
- 値: `FileSystemDirectoryHandle` オブジェクト

### 起動フロー
1. `App.init()` で IndexedDB から handle を読み込む
2. handle が存在する場合 → ウェルカム画面を「前回フォルダあり」モードで表示
3. 「前回のフォルダで始める」タップ
   - `handle.queryPermission({ mode: 'readwrite' })` を確認
   - `'granted'` なら即座に開始
   - それ以外なら `handle.requestPermission({ mode: 'readwrite' })` を呼び出す
   - 許可されれば開始、拒否されれば「別のフォルダを選択してください」トーストを表示
4. 「別のフォルダを選択」タップ → 従来通り `showDirectoryPicker()` を呼び出す
5. フォルダ選択成功時は常に IndexedDB に handle を上書き保存

### エラー対応
- フォルダが削除・移動されていた場合：パーミッション失敗 → トースト表示、通常選択へ誘導

---

## 非機能要件（継続）
- 単一HTMLファイル（`index.html`）にインライン実装
- 外部ライブラリ禁止
- Android Chrome タッチ操作対応
