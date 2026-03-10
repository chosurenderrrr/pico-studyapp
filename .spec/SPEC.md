# SPEC - 技術仕様・要件定義

## 機能概要
1. 問題データを `problem.json` 1ファイルに完全移行（.txt 形式を廃止）
2. 問題保存後の全問再読み込みを廃止（保存処理の高速化）

---

## 1. problem.json 形式への完全移行

### 背景・目的
現状、1問あたり最大43回のファイルアクセスが発生している。
`problem.json` 1本に統合して 1回に削減する。.txt 形式は完全廃止。

### JSONデータ構造
各問題フォルダ（例: `001/`）に `problem.json` を格納する。

```json
{
  "problemType": "multiple-choice",
  "title": "タイトル",
  "questionType": "text",
  "question": "問題文 or base64 dataURL",
  "corrects": [{"type": "text", "content": "正解1"}],
  "wrongs":   [{"type": "text", "content": "不正解1"}],
  "items":    [{"type": "text", "content": "並び替え項目1"}],
  "dummies":  [{"type": "text", "content": "ダミー1"}],
  "explanationType": "text",
  "explanation": "解説文 or base64 dataURL"
}
```

- 画像は `question` / `explanation` フィールドに **base64 data URL 文字列**として格納

### 読み込み変更（ProblemIO.load）
- `problem.json` を読み込み、JSON.parse して返す
- .txt ファイルの読み込み処理はすべて削除

### 書き込み変更（ProblemIO.save）
- `problem.json` として1回のJSON書き込みで保存
- .txt ファイルへの書き込み・removeEntry はすべて削除

### 効果試算
| タイミング | 改善前 | 改善後 |
|---|---|---|
| ロード（500問） | ~21,500回のファイルアクセス | **500回** |
| 問題保存 | 多数の write + ~40回の removeEntry | **1回の JSON write** |

---

## 2. 保存後の全問再読み込みを廃止

### 現状の問題
`Editor.save()` が保存後に `ProblemIO.loadAll()` を呼んでいる。
→ 1問保存するたびに全問（500問）を再読み込みしている。

### 改善内容
`Editor.save()` 内で `ProblemIO.loadAll()` を呼ぶのをやめ、以下に変更：

- **既存問題の更新**：`State.problems` 配列の該当問題オブジェクトを直接置き換える
- **新規問題の追加**：`State.problems` に追加して `num` 順でソート
- 最後に `App.renderProblemList()` を呼ぶ

---

## 非機能要件（継続）
- 単一HTMLファイル（`index.html`）にインライン実装
- 外部ライブラリ禁止
- Android Chrome タッチ操作対応
