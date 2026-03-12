# KNOWLEDGE - ドメイン知識・調査結果

## 業務・ドメイン知識
## 調査・リサーチ結果
## 技術的な知見
- Canvas APIを使用した画像リサイズ: Imageオブジェクトをロードし、Canvasに描画することでリサイズ可能
- アスペクト比維持: `Math.min(maxWidth / w, maxHeight / h)` で比率を計算
- base64形式維持: `toDataURL`で元のフォーマット（JPEG/PNG）を維持

## 決定事項と理由
- 最大サイズ600x400px: スマホ画面での表示に適したサイズとして採用
- 外部ライブラリ不使用: 既存の制約に従い、Canvas APIで実装
