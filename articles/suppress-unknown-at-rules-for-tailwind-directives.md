---
title: "Tailwind CSSのディレクティブ定義時に「unknownAtRules」が問題として報告される現象の対処方法"
emoji: "⚠️"
type: "tech"
topics: ["tailwindcss", "vscode"]
published: true
---

Tailwind CSSのv4で定義されているディレクティブをCSSファイル内で使った際、VS Code内にて `unknownAtRules` という形で問題として報告されてしまう課題を解決した話です。

## 課題

Tailwind CSSでは、CSSの仕様に無い独自のat-rules（以下、ディレクティブ）が定義されています。

このディレクティブを使う場合、VS Code上でディレクティブに対し黄色い波線が表示されます。

![VS Code上で@themeディレクティブを定義したときに、黄色い波線が表示されている様子](/images/unknown-at-rule-problem.png)

ビルドプロセスには影響ないですが、ファイル内に黄色い波線が表示されているのは気持ち悪いですね。

## 解決方法

VS Codeに備わっている[Custom Data Extension](https://code.visualstudio.com/api/extension-guides/custom-data-extension)を使って、Tailwind CSSのディレクティブをVS Code側で認識できるように拡張します。

なお、この記事ではTailwind CSS v4以上を前提として書きます。v4未満の場合は関連リンクから方法を見て設定してください。

### ファイル構造

ルートディレクトリ上に `.vscode` ディレクトリを作成し、2つのファイルを配置します。

```text
/
└── .vscode
     ├── settings.json
     └── tailwind.json
```

### settings.jsonの設定

`.vscode/settings.json` に、Tailwind CSS向けCustom Data Extensionの読み込み設定を書きます。

```json
{
  "css.customData": [".vscode/tailwind.json"]
}
```

### tailwind.jsonの設定

`.vscode/tailwind.json` に、Tailwind CSSのディレクティブを認識させる定義を書きます。

```json
{
  "version": 1.1,
  "atDirectives": [
    {
      "name": "@theme",
      "description": "Use the `@theme` directive to define your project's custom design tokens, like fonts, colors, and breakpoints.",
      "references": [
        {
          "name": "Tailwind Documentation",
          "url": "https://tailwindcss.com/docs/functions-and-directives#theme-directive"
        }
      ]
    },
    {
      "name": "@source",
      "description": "Use the `@source` directive to explicitly specify source files that aren't picked up by Tailwind's automatic content detection.",
      "references": [
        {
          "name": "Tailwind Documentation",
          "url": "https://tailwindcss.com/docs/functions-and-directives#source-directive"
        }
      ]
    },
    {
      "name": "@utility",
      "description": "Use the `@utility` directive to add custom utilities to your project that work with variants like `hover`, `focus` and `lg`.",
      "references": [
        {
          "name": "Tailwind Documentation",
          "url": "https://tailwindcss.com/docs/functions-and-directives#utility-directive"
        }
      ]
    },
    {
      "name": "@variant",
      "description": "Use the `@variant` directive to apply a Tailwind variant to styles in your CSS.",
      "references": [
        {
          "name": "Tailwind Documentation",
          "url": "https://tailwindcss.com/docs/functions-and-directives#variant-directive"
        }
      ]
    },
    {
      "name": "@custom-variant",
      "description": "Use the `@custom-variant` directive to add a custom variant in your project.",
      "references": [
        {
          "name": "Tailwind Documentation",
          "url": "https://tailwindcss.com/docs/functions-and-directives#custom-variant-directive"
        }
      ]
    },
    {
      "name": "@apply",
      "description": "Use the `@apply` directive to inline any existing utility classes into your own custom CSS.",
      "references": [
        {
          "name": "Tailwind Documentation",
          "url": "https://tailwindcss.com/docs/functions-and-directives#apply-directive"
        }
      ]
    },
    {
      "name": "@reference",
      "description": "If you want to use `@apply` or `@variant` in the `<style>` block of a Vue or Svelte component, or within CSS modules, you will need to import your theme variables, custom utilities, and custom variants to make those values available in that context.\n\nTo do this without duplicating any CSS in your output, use the `@reference` directive to import your main stylesheet for reference without actually including the styles.",
      "references": [
        {
          "name": "Tailwind Documentation",
          "url": "https://tailwindcss.com/docs/functions-and-directives#reference-directive"
        }
      ]
    },
    {
      "name": "@config",
      "description": "Use the `@config` directive to load a legacy JavaScript-based configuration file.",
      "references": [
        {
          "name": "Tailwind Documentation",
          "url": "https://tailwindcss.com/docs/functions-and-directives#config-directive"
        }
      ]
    },
    {
      "name": "@plugin",
      "description": "Use the `@plugin` directive to load a legacy JavaScript-based plugin.",
      "references": [
        {
          "name": "Tailwind Documentation",
          "url": "https://tailwindcss.com/docs/functions-and-directives#plugin-directive"
        }
      ]
    }
  ]
}
```

定義を書き終えたら、VS Codeのコマンドパレット上で「Developer: Restart Extension Host」を実行します。

![コマンドパレット上で「Developer: Restart Extension Host」を選択している様子](/images/developer-restart-extension-host.png)

少し待つと無事に黄色い波線が表示されなくなります。良かったですね。

![VS Code上でTailwind CSSのディレクティブ定義に対し、黄色い波線が表示されなくなった様子](/images/suppress-unknown-at-rule-problem.png)

## 参考にしたページ

- [Functions and directives - Core concepts - Tailwind CSS](https://tailwindcss.com/docs/functions-and-directives#directives)
- [vscode-css-languageservice/docs/customData.md at 0a8ca4c · microsoft/vscode-css-languageservice](https://github.com/microsoft/vscode-css-languageservice/blob/0a8ca4c/docs/customData.md)
- [Tailwind CSS v4未満の拡張方法](https://github.com/tailwindlabs/tailwindcss/discussions/5258#discussioncomment-19793948)
- [Tailwind CSS v4以上の拡張方法](https://github.com/tailwindlabs/tailwindcss/discussions/5258#discussioncomment-13239940)
