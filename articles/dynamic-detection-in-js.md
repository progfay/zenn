---
title: "Headless Chromeを使ってプロパティアクセスや関数の実行を動的検出する"
emoji: "👀"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["JavaScript", "Chromium", "CDP", "同的検出"]
published: false
---

この記事は[Recruit Engineers Advent Calendar 2020](https://adventar.org/calendars/5166)の14日目の記事です。

:::message alert
**免責事項**
動的検出は対象となるウェブサイトに対する攻撃とみなされる可能性があります。当方はこれに対する一切の責任を負いません。
:::

ふとChrome DevTools Protocolで遊んでみようと思い立ち、試しにプロパティアクセスや関数の呼び出しの動的検出に挑戦してみたのでこれについてまとめておきます。

## Chrome DevTools Protocol

[Chrome DevTools Protocol](https://chromedevtools.github.io/devtools-protocol/)(以下、CDP)はChromiumと相互的に通信する際に用いられるプロトコルです。
Chromiumが処理しているNetworkやProfileなどの情報を取得したり、逆にChromiumに指定のウェブサイトを開かせたり任意のJavaScriptを実行させたりできます。
例えば、[Chrome DevTools](https://developers.google.com/web/tools/chrome-devtools)や[puppeteer/puppeteer](https://github.com/puppeteer/puppeteer)は内部的にCDPを使用しています。

## 今回の目標

`globalThis['eval']('document.write(navigator.userAgent)')`を[JavaScript Obfuscator Tool](https://obfuscator.io/)を使って難読化して読み込ませたウェブサイトを作成しました。

[@codesandbox](https://codesandbox.io/s/obfuscated-code-zglgt?file=/src/index.js)

今回は、このウェブサイトから以下の2つを動的に検出することを目的とします。

1. `navigator.userAgent`へのアクセスを検出する
2. `eval`の呼び出しを検出する (引数の値も取得する)

## 検出までの流れ

CDPの[`Page.addScriptToEvaluateOnNewDocument(source: string)`](https://chromedevtools.github.io/devtools-protocol/tot/Page/#method-addScriptToEvaluateOnNewDocument)は引数に指定したJavaScriptのコードをドキュメントを読み込む前のタイミングで実行することができます。これを利用して、プロパティアクセスや関数の実行の際に任意の関数を実行させることが可能です。

また、[`Runtime.consoleAPICalled`]はConsole APIが呼び出された際に発生するイベントです。今回は`console.log`を用いてプロパティアクセスや関数の実行を通知させるようにしました。

検出までの手順は以下の通りです。

1. `Page.addScriptToEvaluateOnNewDocument`でChromium側でプロパティアクセスや関数の実行時に`console.log`を実行するようにする
2. `Page.navigate`を用いて、Chromiumで対象のウェブサイトを開く
3. Chromium側でプロパティアクセスや関数の実行が行われ、対応する`console.log`が実行される
4. CDPを通して`Runtime.consoleAPICalled`イベントが発生し、検出に至る

## プロパティアクセスを検出する

https://github.com/progfay/prop-access-detector

### `Page.addScriptToEvaluateOnNewDocument`で実行するScript

```js
(function (target, prop) {
  let value = target[prop]
  const {
    get = () => value,
    set = v => { value = v },
  } = Object.getOwnPropertyDescriptor(target, prop) ?? {}
  Object.defineProperty(target, prop, {
    get: () => {
      console.trace({ mode: 'get', target, prop, value })
      return get()
    },
    set: v => {
      console.trace({ mode: 'set', target, prop, value })
      return set(v)
    },
  })
})(navigator, 'userAgent')
```

### 検出結果

```js
{
  message: 'detect accessing property: navigator["userAgent"]',
  arguments: [
    { name: 'mode', type: 'string', value: 'get' },
    { name: 'target', type: 'object', value: 'Navigator' },
    { name: 'prop', type: 'string', value: 'userAgent' },
    { name: 'value', type: 'string', value: '{{USER_AGENT_STRING}}' }
  ],
  stackTrace: [
    {
      functionName: 'get',
      scriptId: '11',
      url: '',
      lineNumber: 11,
      columnNumber: 20
    },
    {
      functionName: '',
      scriptId: '4',
      url: 'http://{{TARGET_WEBSITE}}/src/index.js',
      lineNumber: 3,
      columnNumber: 22
    }
  ]
}
```

## 関数の実行を検出する

https://github.com/progfay/eval-detector

### `Page.addScriptToEvaluateOnNewDocument`で実行するScript

```js
(function(target, prop) {
  const original = target[prop]
  target[prop] = function() {
    console.log(arguments)
    return original.apply(this, arguments)
  }
})(globalThis, 'eval')
```

### 検出結果

```js
{
  message: 'detect calling function: globalThis["eval"]',
  arguments: [
    {
      name: '0',
      type: 'string',
      value: 'document.write(navigator.userAgent)'
    },
    { name: 'callee', type: 'function', value: '' },
    { name: 'Symbol(Symbol.iterator)', type: 'function', value: '' }
  ],
  stackTrace: [
    {
      functionName: 'target.<computed>',
      scriptId: '3',
      url: '',
      lineNumber: 4,
      columnNumber: 20
    },
    {
      functionName: '',
      scriptId: '4',
      url: 'http://{{TARGET_WEBSITE}}/src/index.js',
      lineNumber: 117,
      columnNumber: 28
    }
  ]
}
```

### 終わりに

プロパティアクセスや関数の実行を同的検出する方法について解説しました。実際に実装してみた感想として、CDPは意外と簡単に取り扱えて幅広く活用できそうだと感じました。今後もCDPを使って色々遊んでみたいと思います。