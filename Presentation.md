---
marp: true
theme: default
paginate: true
style: |
  :root {
  --fgColor-accent: #1a56db;
  }
  section {
    font-family: "Noto Sans JP", sans-serif;
    font-size: 1.1rem;
  }
  h1 { color: #1a56db; border-bottom: 3px solid #1a56db; padding-bottom: 0.2em; }
  h2 { color: #1a56db; }
  code { background: #f3f4f6; padding: 0.1em 0.4em; border-radius: 4px; }
  pre { background: #f3f4f6; border-left: 4px solid #1a56db; }
  table { width: 100%; }
  th { background: #1a56db; color: white; }
  code { display: inline-block; background: #e3e3e3; color: #f04400; font-weight: bold; padding: 2px;}
  .bad  { color: #dc2626; font-weight: bold;}
  .good { color: #16a34a; font-weight: bold; }
  .center {text-align: center;}
---

# 強制カラーモードへの対応

Windows ハイコントラストモードで壊れるUIと壊れないUI

---

## 🔍 今回、調べるきっかけになった記事

![width:800](<CleanShot 2026-02-25 at 18.17.38@2x.png>)

> [デジタル庁デザインシステムの活用例：よくあるアクセシビリティの課題とその解決策](https://digital-agency-news.digital.go.jp/articles/2025-10-14)

---

## 🖥️ 強制カラーモードとは？

OSやブラウザで強制的にテキストや背景色を高コントラストの色に変更する機能。

- **Windows ハイコントラストモード**が代表例
- ページ上のすべての色が OS 定義のシステムカラーに置換される

---

## 🧑‍🦯 なぜ強制カラーモードは存在する？

> 強制カラーモードは、主に弱視・ロービジョンの方や視覚過敏、ディスレクシアの方がよく利用している機能です。まぶしい環境下やモニターの影響など、一時的に見にくいという問題で強制カラーモードを使う方もいるかと思います。
>
> ただし、「一時的に見にくい」という問題は、コントラスト比の低いウェブサイトの場合が多く、これは適切なコントラスト比（テキストと背景色の比率が4.5：1以上）を満たすことで基本的には解決可能です。そのため、ここでは強制カラーモードの主な利用者には含まないものとします。

[デジタル庁デザインシステムの活用例：よくあるアクセシビリティの課題とその解決策｜デジタル庁ニュース](https://digital-agency-news.digital.go.jp/articles/2025-10-14#index_Cp2WkXm4)より引用

---

## ⇄ 強制カラーによって上書きされるもの

- `color`
- `background-color`
- `text-decoration-color`
- `text-emphasis-color`
- `border-color`
- `outline-color`
- `column-rule-color`
- `-webkit-tap-highlight-color`
- SVG `fill` 属性
- SVG `stroke` 属性

→ 各タグによって自動的にシステムからーに変更される

---

## 💥 強制カラーによって特別な動作をするもの

- `box-shadow` は `none` に強制される
- `text-shadow` は `none` に強制される
- `background-image` は URL ベースでない値では `none` に強制される
- `color-scheme` は `light dark` に強制される
- `scrollbar-color` は `auto` に強制される

→ `box-shadow`や`linear-gradient`は消えてしまうので注意が必要

---

## 🎨 システムカラー一覧

強制カラーモードで使用できる CSS キーワード

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

## 🐞 検証方法

Macではアクセシビリティのコントラストを変更してもこの機能は反映されないため、<br>Chrome DevTools でシミュレーションする必要がある

1. DevTools → ⌘ + ⇧ + P でコマンド検索
2. Run > **"Emulate CSS forced-colors: active"**

![width:500](<CleanShot 2026-02-25 at 17.53.34@2x.png>)

> Windows 実機での確認が最も正確だが、Chrome DevToolsでも十分に問題を検出できる

---

## 🚀 DEMO

![width:700 center](<CleanShot 2026-02-26 at 11.13.13@2x.png>)

[Forced Color Mode Demo - yskm_dev](https://forced-color-mode-demo.yskm.dev/)

---

## 💡 重要パターン1：transparent border

```scss
/* 通常時: 見えない（透明） */
border: 1px solid transparent;

/* forced-colors 時: colorプロパティを継承して 線が引かれる ✅ */
```

- 通常モードのデザインを壊さず、強制カラーモードでのみ境界線が現れる
- 塗りだけのボタンや`box-shadow` の代替として有効

---

## 💡 重要パターン2：currentColor

```html
<svg stroke="currentColor" aria-hidden="true">
  <path d="M6 3l5 5-5 5" />
</svg>
```

- アイコンが背景色と同化してしまうのを防ぐ
- currentColorはcolorプロパティを継承するため同化しない

---

## 💡 重要パターン3: @media (forced-colors: active)

```scss
@media (forced-colors: active) {
  background-color: CanvasText;
}
```

- 色を指定したい場合は `@media (forced-colors: active)` を使用する
- 指定する場合はシステムカラーで設定をする

---

## ⚠️ 注意： forced-color-adjust: none;

```scss
.box {
  background: rgb(83, 202, 36);
  forced-color-adjust: none;
}
```

- `forced-color-adjust: none` を使用すると色の置換をしないようにできる
- ただし、強制カラーモードの機能を壊す可能性がある
- <span class="bad">ユーザーの選択を尊重しないような使用はすべきではない</span>

---

## ✅ まとめ：forced-colors 対応チェックリスト

| テクニック                             | 効果                                            |
| -------------------------------------- | ----------------------------------------------- |
| `border: N solid transparent`          | 影・背景色の代替として境界線を残す              |
| SVGや背景色に `currentColor`を使用する | colorプロパティを継承して背景色との同化を避ける |
| `@media (forced-colors: active)`       | 必要に応じてシステムカラーで上書き              |

> **基本原則**: 色だけに頼らず、**境界線・アウトライン・形状**で情報を伝えることが重要

---

## 📚 参考サイト

- [デジタル庁デザインシステムの活用例：よくあるアクセシビリティの課題とその解決策](https://digital-agency-news.digital.go.jp/articles/2025-10-14)
- [強制カラーモードに対するメディアクエリforced-colorsの使い方 | フロントエンドBlog | ミツエーリンクス](https://www.mitsue.co.jp/knowledge/blog/frontend/202405/17_1401.html)
- [弱視・ロービジョンのユーザーのウェブ利用時の課題 | アクセシビリティ | SmartHR Design System](https://smarthr.design/accessibility/low-vision/)
- [アクセシビリティを気にし出したエンジニアへ、強制カラーモードの紹介 - Zenn](https://zenn.dev/schktjm/articles/c4239989992d1a)
