# SPEC - 技術仕様・要件定義

## 機能概要
クイズ解答後に画面下部に固定アクションバーを表示し、スクロールなしで操作できるようにする。

---

## 解答後アクションバー（#quiz-action-bar）

### 表示タイミング
- 回答確定（confirmMultipleChoice / confirmOrder）後に表示
- 次の問題に進む（render）時に非表示に戻す

### 配置
- `#view-quiz` の `.main` の兄弟要素として `.main` の下に配置（flex column の末尾）
- `.main` が残りの高さを占め、アクションバーは画面下部に固定表示される

### バーの内容（上から順）
1. **結果ラベル**：「✓ 正解！」（緑）または「✗ 不正解」（赤）
2. **ボタン行（横並び）**：
   - 「解説を見る」ボタン：クリックで `#quiz-explanation-content` までスクロール（解説がない場合は非表示）
   - 「次へ →」ボタン（最後の問題は「結果を見る →」）
3. **「🏠 ホームへ」ボタン**：全幅、セカンダリスタイル

### アニメーション
- 表示時：下から滑り上がる（`slideUp` 0.2s ease-out）

---

## 既存コードの変更

### HTML
- `#quiz-answer-area` 内の `quiz-next-btn` / ホームへボタンを削除
- `#quiz-action-bar` を `.main` の外（兄弟要素）として追加
- `quiz-answer-area` は解説内容（ラベル + テキスト）のみに縮小

### JS
- `confirmMultipleChoice` / `confirmOrder`：`quiz-answer-area.show` に加え `quiz-action-bar.show` を追加
  - 結果ラベルテキストを設定（正解/不正解）
  - 解説の有無で「解説を見る」ボタンの表示を切り替え
- `render`：`quiz-action-bar` の `.show` を除去

### CSS
- `#quiz-action-bar`：アニメーション付きのスタイルを追加
- `#view-quiz .main`：アクションバー表示時に下部パディング不要（flex で自然に収まる）

---

## 非機能要件（継続）
- 単一HTMLファイル（`index.html`）にインライン実装
- 外部ライブラリ禁止
- Android Chrome タッチ操作対応
