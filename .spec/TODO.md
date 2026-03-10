# TODO

## タスク一覧

### Feature: 解答後アクションバー

- [x] CSS：`#quiz-action-bar` のスタイルを追加（slideUp アニメーション含む）
- [x] HTML：`#quiz-action-bar` を `#view-quiz` の `.main` 外に追加
- [x] HTML：`#quiz-answer-area` から `quiz-next-btn` / ホームへボタンを削除
- [x] JS：`render()` で `#quiz-action-bar` の `.show` を除去
- [x] JS：`confirmMultipleChoice()` でアクションバーを表示・結果ラベル設定・解説ボタン表示制御
- [x] JS：`confirmOrder()` で同上
- [x] JS：`scrollToExplanation()` メソッドを追加

### 動作確認
- [ ] 解答後にアクションバーが下から滑り上がって表示されること
- [ ] 「解説を見る」ボタンで解説位置にスクロールすること
- [ ] 解説がない問題では「解説を見る」ボタンが非表示になること（正解時のみ）
- [ ] 「次へ →」で次の問題に進み、アクションバーが消えること
- [ ] 最後の問題で「結果を見る →」と表示されること
- [ ] 「🏠 ホームへ」でホームに戻ること
