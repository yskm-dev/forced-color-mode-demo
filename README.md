# Forced Color Mode Demo

デジタル庁の記事「強制カラーモードへの対応」（2025-10-14）に基づき、Windows のハイコントラストモード（`forced-colors: active`）で壊れる UI と壊れない UI を並列比較するデモサイト。

## 技術スタック

| 項目           | 内容                 |
| -------------- | -------------------- |
| フレームワーク | Astro 4.x            |
| 言語           | TypeScript（strict） |
| スタイル       | SCSS（sass）         |
| 出力           | Static（単一ページ） |

## 開発環境のセットアップ

```bash
npm install
npm run dev
```

`localhost:4321` で起動します。

## デモの内容

4つのケースで Bad UI（壊れる実装）と Good UI（対応済み実装）を左右に並列比較しています。

| Case | テーマ                                                 |
| ---- | ------------------------------------------------------ |
| 1    | カスタム CSS ラジオボタンの選択状態                    |
| 2    | ドロワーの境界線（`box-shadow` vs `border`）           |
| 3    | ボタンの優先度表現（色のみ vs `border` 併用）          |
| 4    | アイコン付きボタン（ハードコード色 vs `currentColor`） |

## 強制カラーモードの有効化

### Windows（推奨）

設定 → アクセシビリティ → コントラストテーマ からハイコントラストテーマを選択。

### Chrome DevTools（シミュレーション）

DevTools → Rendering タブ → **Emulate CSS media feature forced-colors** → `active`

## 設計のポイント

- `border: 1px solid transparent` — 通常時は透明、強制カラーモードで `ButtonText` 色のボーダーとして残る
- `stroke="currentColor"` — SVG アイコンが CSS `color` を継承し、`ButtonText` に追従する
- `@media (forced-colors: active)`

## ビルド

```bash
npm run build
```

静的ファイルが `dist/` に出力されます。
