# pico-studyapp - Claude Code ガイド

## プロジェクト概要
ローカルフォルダから問題素材を読み込んで多択クイズとして学習できるブラウザアプリ。
HTML + CSS + JavaScript のみ。サーバー・フレームワーク不使用。

## 技術スタック
- HTML / CSS / JavaScript (ES Modules, `type="module"`)
- File System Access API（ローカルフォルダの読み書き）
- localStorage（メタデータ・履歴の永続化）
- 対応ブラウザ: Chrome / Edge 86以上

## ファイル構成
```
pico-studyapp/
├── index.html          ← SPAのシェル
├── style.css           ← グローバルスタイル（CSS変数でテーマ管理）
└── js/
    ├── app.js          ← ルーティング・画面管理
    ├── store.js        ← 状態管理・localStorage連携
    ├── fileSystem.js   ← File System Access API ラッパー
    ├── quiz.js         ← クイズロジック（シャッフル・採点）
    └── views/
        ├── home.js     ← ホーム画面
        ├── quiz.js     ← クイズ解答画面
        ├── result.js   ← 結果画面
        └── create.js   ← クイズ作成画面
```

## コーディング規約
- JSモジュールは ES Modules で分割（`import/export`）
- 状態変更は必ず `store.js` 経由で行う
- File System API 操作は `fileSystem.js` に集約する
- `URL.createObjectURL()` で生成した Blob URL は画面遷移時に `revokeObjectURL()` で解放する
- `showDirectoryPicker()` はユーザー操作イベントハンドラ内でのみ呼び出す

## localStorage キー
| キー | 内容 |
|---|---|
| `pico-studyapp-sets` | 登録済みクイズセットのメタデータ |
| `pico-studyapp-history` | 解答履歴 |
| `pico-studyapp-settings` | アプリ設定（出題数・シャッフル有無） |

## クイズセットのフォルダ規則
```
quiz-set-name/        ← ルートフォルダ（クイズセット名）
├── 001/              ← 問題（3桁ゼロパディング）
│   ├── question.png  ← 問題（.pngまたは.txt）
│   ├── correct.png   ← 正解（.pngまたは.txt）
│   ├── wrong1.png    ← 誤答1
│   └── wrong2.png    ← 誤答2（以降 wrong3, wrong4...）
```
