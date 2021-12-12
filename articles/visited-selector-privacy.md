---
title: ":visited selector に対するスタイリングは一部制限されているらしい"
emoji: "🖇️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["privacy", "CSS", "visited"]
published: true
---

:::message
本記事は「[Recruit Engineers Advent Calendar 2021](https://adventar.org/calendars/6663)」 13 日目の記事です。
:::

## ある日の会議にて

ある日の会議にて、 `:visited` selector への CSS 指定が上手くいかないという相談がありました。

以下の CodeSandbox は状況を簡略化したものです。
一度 Link を踏んでから Link の style を確認してみてください。

[@codesandbox](https://codesandbox.io/s/visited-selector-styling-nk6x4?file=/index.html)

`:visited` selector への `color` に半透明な赤色を指定しているのにも関わらず、実際に表示されている `<a>` は不透明な赤色となっています。

## もしかして: セキュリティ

なぜ透明度が無視されてしまうのかの具体的な原因はわかりませんでしたが、透明度を利用した攻撃があることを記事で読んだことがありました。

<https://blog.mozilla.org/attack-and-defense/2021/01/11/leaking-silhouettes-of-cross-origin-images/>

この記事の中では、 Cross-Origin から持ってきた画像の上に半透明な pixel を置いた際の描画のパフォーマンスから、画像の pixel 情報を抜き出す脆弱性 ([CVE-2020-16012](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2020-16012)) について説明しています。

このことから、「`:visited` に透明度が付与できてしまうと、何らかの情報が抜き出せる可能性があるのではないか？」と考えました。

## プライバシーと `:visited` セレクター

調べてみると、 `:visited` を介して閲覧履歴が漏れないようにするための仕様であることがわかりました。

例えば以下のような HTML をユーザーが開くと、そのユーザーが `https://example.com` にアクセスしたことがあるかどうかがわかってしまいます。

```html
<a id="link" href="https://example.com">link</a>

<script>
    if (document.querySelector('#link:visited') !== null) {
        console.log('The user has already visited to https://example.com')
    }
</script>
```

閲覧履歴はプライバシー情報であり、これがサイト側から読めてしまうことを防ぐ必要があります。

### JavaScript や selector に対する制限

閲覧履歴が読めないようにいくつかの制限がかけられています。

> ユーザーのプライバシーを保護するために、Firefox や他のブラウザーは特定の状況下でウェブアプリケーションに嘘をつきます。
> - `window.getComputedStyle` メソッドや、 `element.querySelector` などの類似の関数は、ユーザーがページ上のリンクを訪れたことがないことを示す値を常に返します。
> - `:visited + span` のような兄弟セレクターを使用した場合、隣接する要素 (この例では `span`) は、リンクが未訪問であるかのようにスタイル付けされます。
> - 稀なケースですが、入れ子になったリンク要素を使用していて、マッチしている要素が履歴の中の存在がテストされているリンクとは異なる場合、要素はリンクが未訪問であったかのようにレンダリングされます。
>
> [プライバシーと :visited セレクター - CSS: カスケーディングスタイルシート | MDN](https://developer.mozilla.org/ja/docs/Web/CSS/Privacy_and_the_:visited_selector) より引用

### スタイリングに対する制限

上記の制限だけでは、例えば `background-image` を使うことでユーザーの閲覧履歴を取得できてしまいます。
ユーザーが以下の HTML を開いた場合を考えてみましょう。

```html
<style>
    #link:link    { background-image: url("/link"); }
    #link:visited { background-image: url("/visited"); }
</style>

<a id="link" src="https://example.com">link</a>
```

`https://example.com` にアクセスしたことがない場合は `/link` にリクエストが飛び、アクセスしたことがある場合は `/visited` にリクエストが飛びます。
結果として、リクエスト元のユーザーの閲覧履歴がサイト側から取得できてしまいました。

このようなケースを防ぐために、訪問済みリンクへのスタイリングは以下のプロパティのみが利用できます。

- `color`
- `background-color`
- `border-color`
- `column-rule-color`
- `outline-color`
- `fill` , `stroke` property の color 関連

また、これに加えて透明度にも制限があります。

> さらに、訪問済みリンクにセットできるプロパティであっても、未訪問リンクと訪問済みリンク間で不透明度を変えることはできません。これは、別の状況なら、`rgba()` や `hsla()` のカラー値、もしくは `transparent` キーワードを使ってできた操作でした。
>
> [プライバシーと :visited セレクター - CSS: カスケーディングスタイルシート | MDN](https://developer.mozilla.org/ja/docs/Web/CSS/Privacy_and_the_:visited_selector) より引用

### なぜ透明度に対する制限が必要なのか？

色の中でも透明度だけは別個に防がれているということは、これを利用して閲覧履歴を盗み出せる可能性があるということでしょうか？

制限されている理由についての詳しい言及を見つけることは残念ながらできませんでしたが、関連しそうな言及として CSS を用いたパフォーマンスへの影響を用いて閲覧履歴を盗み出す手法が触れられていました。

> They can use CSS features such that matching selectors, doing layout, or doing painting would take a different amount of time depending on whether the link is visited or unvisited, and then run a performance test.
>
> [Preventing attacks on a user's history through CSS :visited selectors](https://dbaron.org/mozilla/visited-privacy#problem-statement) より引用

これは冒頭の「透明度を利用して描画のパフォーマンスを低下させて情報をリークさせる手法」に近いです。
これを考慮した結果として、訪問済みリンクへの透明度の指定を無視する使用にになったのではないでしょうか。

## まとめ

本記事では、 `:visited` selector にいくつかの制限があることを紹介しました。

チーム内ではブラウザのバグだったり謎の挙動だと考えられていたことにここまで深い理由と仕様があるとは思いませんでした。
CSS の透明度ですら攻撃に利用できてしまうなんて、 Web Security の世界はなんて広くて深いんでしょうか...。

Web Security を勉強すれば「これはバグじゃなくて仕様なのかも？」と捉えることができるようになるかもしれません。
また、この記事が `:visited` selector にスタイルが付けられない理由をデザイナーさんに説明する際の一助になれれば幸いです。
