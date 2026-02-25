# Forced Color Mode Demo — 仕様書

デジタル庁の記事「強制カラーモードへの対応」（2025-10-14）に基づき、Windows のハイコントラストモード（`forced-colors: active`）で壊れる UI と壊れない UI を並列比較するデモサイト。

---

## 技術スタック

| 項目           | 内容                          |
| -------------- | ----------------------------- |
| フレームワーク | Astro 4.x                     |
| 言語           | TypeScript（strict）          |
| スタイル       | SCSS（sass）                  |
| 出力           | Static（SPA構成の単一ページ） |

---

## 設計方針

- **強制カラーモード対応**: OS での有効化を推奨。シミュレーションなし。`@media (forced-colors: active)` のみ使用
- **比較レイアウト**: 左 Bad UI / 右 Good UI の2カラム並列表示
- **掲載ケース**: 記事の3例のみ（ラジオボタン、ドロワー、ボタン優先度）

---

## ファイル構成

```
forced-color-mode-demo/
├── astro.config.mjs
├── tsconfig.json
├── package.json
├── public/
│   └── favicon.svg
└── src/
    ├── env.d.ts
    ├── layouts/
    │   └── BaseLayout.astro
    ├── pages/
    │   └── index.astro
    ├── components/
    │   ├── Header.astro
    │   ├── Nav.astro
    │   ├── HowToEnable.astro
    │   ├── ComparisonSection.astro
    │   ├── Footer.astro
    │   └── cases/
    │       ├── RadioBadUI.astro
    │       ├── RadioGoodUI.astro
    │       ├── DrawerBadUI.astro
    │       ├── DrawerGoodUI.astro
    │       ├── ButtonBadUI.astro
    │       ├── ButtonGoodUI.astro
    │       ├── IconBtnBadUI.astro
    │       └── IconBtnGoodUI.astro
    └── styles/
        ├── global.scss
        ├── _variables.scss
        └── _mixins.scss
```

---

## 設定ファイル

### `astro.config.mjs`

```js
import { defineConfig } from "astro/config";

export default defineConfig({
  vite: {
    css: {
      preprocessorOptions: {
        scss: {
          // 全コンポーネントの <style lang="scss"> に変数・Mixin を自動注入
          additionalData: `
            @use "/src/styles/_variables.scss" as *;
            @use "/src/styles/_mixins.scss" as *;
          `,
        },
      },
    },
  },
});
```

### `tsconfig.json`

`astro/tsconfigs/strict` を継承。パスエイリアスを追加:

```json
{
  "extends": "astro/tsconfigs/strict",
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@components/*": ["src/components/*"],
      "@layouts/*": ["src/layouts/*"],
      "@styles/*": ["src/styles/*"]
    }
  }
}
```

---

## SCSS 設計

### 重要な注意点

`_mixins.scss` は Sass モジュールスコープの制約から、**先頭に `@use './variables' as *;` が必要**。
ミックスイン内で変数を参照するとき、変数は定義ファイル（`_mixins.scss`）のスコープで解決されるため。

### `_variables.scss` のトークン

```scss
// 色
$color-primary: #1a56db;
$color-secondary: #6b7280;
$color-bg: #ffffff;
$color-surface: #f3f4f6;
$color-border: #d1d5db;
$color-text: #111827;
$color-text-muted: #6b7280;
$color-selected-bg: #dbeafe;
$color-shadow: rgba(0, 0, 0, 0.15);
$color-overlay: rgba(0, 0, 0, 0.4);

// スペーシング: $space-xs(4px) 〜 $space-3xl(64px)
// タイポグラフィ: $font-size-xs 〜 $font-size-3xl、$font-family (Noto Sans JP)
// Z-index: $z-drawer: 200, $z-drawer-overlay: 100, $z-nav: 50
// ボーダー半径: $border-radius-sm(4px) 〜 $border-radius-full
```

### `_mixins.scss` の主要ミックスイン

| ミックスイン         | 用途                                              |
| -------------------- | ------------------------------------------------- |
| `container`          | `max-width: 1200px` + `margin-inline: auto`       |
| `card`               | border + border-radius + padding のカードスタイル |
| `visually-hidden`    | スクリーンリーダー専用の非表示                    |
| `focus-visible-ring` | `:focus-visible` アウトライン                     |
| `button-reset`       | ボタンのリセット + フォーカスリング               |
| `mobile` / `tablet`  | レスポンシブブレークポイント                      |

---

## コンポーネント設計

### `ComparisonSection.astro`（中核コンポーネント）

Props:

```ts
interface Props {
  caseNumber: number;
  title: string;
  description: string;
  badExplanation: string;
  goodExplanation: string;
  badLabel?: string; // default: "Bad UI"
  goodLabel?: string; // default: "Good UI"
}
```

Named slot で Bad/Good を受け取り、2カラムグリッドで並列表示:

```astro
<slot name="bad" />   // 左パネル（赤ラベル）
<slot name="good" />  // 右パネル（緑ラベル）
```

`@media (forced-colors: active)` でパネルとバッジに `border: 1px solid ButtonText` を付与し、構造を維持。

### `BaseLayout.astro`

`.forced-colors-active-banner` を配置 — JS なしで強制カラーモードの有効状態を確認できる固定バー:

```scss
.forced-colors-active-banner {
  display: none; // 通常時は非表示
  @media (forced-colors: active) {
    display: block; // 有効時のみ表示
    position: fixed;
    background-color: Highlight;
    color: HighlightText;
  }
}
```

---

## 3ケースの実装

### Case 1: ラジオボタンの選択状態

カスタム CSS ラジオボタン実装（ブラウザデフォルト `<input type="radio">` を `visually-hidden` で隠し、`<span class="radio-xxx__mark">` で描画）。

HTML 構造:

```html
<label class="radio-xxx__item">
  <input class="radio-xxx__input" type="radio" ... />
  <!-- visually-hidden -->
  <span class="radio-xxx__mark" aria-hidden="true"></span>
  <!-- カスタム円 -->
  <span class="radio-xxx__label-text">...</span>
</label>
```

選択状態は `--selected` クラスではなく CSS `:has()` で管理（ユーザー操作にも動的対応）。

|                  | Bad UI                                           | Good UI                                                             |
| ---------------- | ------------------------------------------------ | ------------------------------------------------------------------- |
| 行の選択表示     | `background-color` のみ                          | `background-color` + `:checked` 時に `border-color`                 |
| 内側ドット       | `background-color` のみ                          | `background-color` + `border: 2px solid transparent`                |
| 強制カラーモード | 背景色・ドットが `Canvas` になり選択状態が消える | `transparent` ボーダーが `ButtonText` になり残る + `outline` で補強 |

Good UI のキー実装:

```scss
// 行: background-color + border-color で選択状態を表現
.radio-good__item:has(.radio-good__input:checked) {
  background-color: $color-selected-bg;
  border-color: $color-primary;
}

// forced-colors: フォーカス時に outline で確実に伝える
@media (forced-colors: active) {
  .radio-good__item:has(.radio-good__input:focus-visible) {
    outline: 2px solid Highlight;
    outline-offset: 2px;
  }
}

// 内側ドット: transparent border → forced-colors で ButtonText の輪郭として残る
.radio-good__item:has(.radio-good__input:checked) .radio-good__mark::after {
  content: "";
  position: absolute;
  inset: 3px;
  border-radius: 50%;
  background-color: $color-primary;
  border: 2px solid transparent;
  @media (forced-colors: active) {
    background-color: ButtonText; // ドットを明示的に維持
  }
}
```

### Case 2: ドロワーの境界線

`<dialog>` 要素 + `showModal()` / `.close()` でモーダルとして実装。`::backdrop` は非表示（`opacity: 0`）のためオーバーレイなし。

|                  | Bad UI                                                    | Good UI                                                |
| ---------------- | --------------------------------------------------------- | ------------------------------------------------------ |
| 境界             | `box-shadow` のみ                                         | `border-left: 2px solid transparent` を追加            |
| 強制カラーモード | `box-shadow` が消えてドロワーとページの境界が不明瞭になる | `transparent` ボーダーが `ButtonText` になり境界が残る |

Good UI のキー実装:

```scss
.drawer-good {
  box-shadow: -8px 0 24px $color-shadow;
  border-left: 2px solid transparent; // forced-colors で ButtonText になり境界として残る
}

// ヘッダー区切り線も forced-colors で明示
.drawer-good__header {
  @media (forced-colors: active) {
    border-bottom: 1px solid ButtonText;
  }
}
```

### Case 3: ボタンの優先度表現

|                  | Bad UI                                          | Good UI                                                          |
| ---------------- | ----------------------------------------------- | ---------------------------------------------------------------- |
| 優先度表示       | 色のみで区別（border なし）                     | プライマリに `border: 5px double transparent` を追加             |
| 強制カラーモード | 全ボタンが `ButtonFace/ButtonText` で同一見た目 | `transparent` ボーダーが `ButtonText` の二重線として残り区別可能 |

Good UI のキー実装:

```scss
.btn-good--primary {
  background-color: $color-primary;
  color: $color-text-on-primary;
  border: 5px double transparent; // forced-colors で ButtonText の二重ボーダーになり視覚的に強調される
}

.btn-good--secondary {
  border: 1px solid $color-border; // 通常の単線ボーダー → forced-colors でも ButtonText で残る
}
```

### Case 4: アイコン付きボタン

SVG アイコンの色指定方法による forced-colors での挙動の違い。ボタン（白背景・黒文字・ボーダー）にシェブロン矢印を添えたシングルボタンで比較。

|                  | Bad UI                                                           | Good UI                                                                   |
| ---------------- | ---------------------------------------------------------------- | ------------------------------------------------------------------------- |
| アイコン描画     | インライン SVG に `stroke="#111"` をハードコード                 | インライン SVG に `stroke="currentColor"`                                 |
| 強制カラーモード | 固定色が強制テーマと合わない場合にアイコンが背景と同化して消える | `currentColor` が CSS `color` を継承し、`ButtonText` として常に表示される |

Good UI のキー実装:

```html
<!-- Good: stroke="currentColor" で CSS color プロパティを継承 -->
<button class="icon-btn-good">
  次へ進む
  <svg stroke="currentColor" aria-hidden="true" focusable="false" ...>
    <path d="M6 3l5 5-5 5" />
  </svg>
</button>
```

```scss
// forced-colors では color が ButtonText に強制される
// → currentColor を使う SVG の stroke も自動的に ButtonText になる
.icon-btn-good {
  color: $color-text; // 通常時: 黒
  // forced-colors: ButtonText に強制 → SVG も追従
}
```

---

## ページ構造

```
/
├── <Header>        — タイトル + forced-colors の概要説明
├── <Nav>           — スティッキー: #how-to-enable / #case-1 / #case-2 / #case-3 / #case-4
├── <HowToEnable id="how-to-enable">
│   ├── Windows 高コントラスト（推奨）
│   ├── Chrome DevTools Rendering パネル
│   └── macOS（非推奨、DevTools を案内）
├── <ComparisonSection id="case-1">  ラジオボタン
├── <ComparisonSection id="case-2">  ドロワー
├── <ComparisonSection id="case-3">  ボタン優先度
├── <ComparisonSection id="case-4">  アイコン付きボタン
└── <Footer>
```

---

## 強制カラーモードで使用するシステムカラー

| システムカラー  | 用途                       |
| --------------- | -------------------------- |
| `Canvas`        | ページ背景                 |
| `CanvasText`    | 本文テキスト               |
| `ButtonFace`    | ボタン背景                 |
| `ButtonText`    | ボタンテキスト・ボーダー   |
| `Highlight`     | 選択状態・アクセントカラー |
| `HighlightText` | Highlight 上のテキスト     |
| `LinkText`      | リンクカラー               |

---

## 検証方法

1. `npm run dev` → `localhost:4321` で起動確認
2. Chrome DevTools → Rendering タブ → "Emulate CSS media feature forced-colors" → `active`
3. 各ケースで Bad UI が「壊れる」、Good UI が「機能する」ことを目視確認
4. スティッキーナビゲーションのアンカーリンク動作確認
5. モバイル幅（375px）で比較グリッドが縦積みになること確認
6. （任意）Windows 実機の高コントラストモードで最終確認
