---
title: "Reactの公式ドキュメントを読む 〜Installation〜"
emoji: "👀"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["react", "docs", "reading"]
published: false
---

Reactを触り始めた時は、[公式ドキュメント](https://reactjs.org/docs)を読んで使い方を学習していました。しかし、当時はどうすれば動くかに集中していたため、公式ドキュメントに書かれている推奨された使い方などを見落としていました。現に今でも公式ドキュメントを調べ直して新たな気付きを得ることがしばしばあります。

この記事は、公式ドキュメントを読み直すことでReactに関する基礎知識を再構築するという試みの一環です。私が読み直した内容を書き残すための記事であり、得られた気付きなどをまとめていきます。そのため、事実とは誤った情報や個人的な解釈が含まれますが、ご容赦ください。また、誤りや指摘等はZennのDiscussionや[Twitter](https://twitter.com/progfay)にて教えていただけると幸いです。

今回の記事では、React公式ドキュメントの"Installation"を読んでいきます。

- Version: 17.0.1
- 原則として英語版と参考にする
- 解釈が難しい場合には日本語版を参照する

## [Getting Started](https://reactjs.org/docs/getting-started.html)

冒頭に以下のように書かれていたのが気になりました。

> React has been designed from the start for gradual adoption, and you can use as little or as much React as you need.

段階的に導入していけるということですが、私は別ライブラリなどからReactへの移行を経験したことがありません。例えばjQueryで書かれたWebサイトをReactに移行したい場合などはどのようにして部分的に置換を進めていけばいいのでしょうか。

[Integrating with Other Libraries](https://reactjs.org/docs/integrating-with-other-libraries.html)にはjQueryのライブラリをReactでWrapする例が書かれています。

> Although React is commonly used at startup to load a single root React component into the DOM, ReactDOM.render() can also be called multiple times for independent parts of the UI which can be as small as a button, or as large as an app.

とあるように、UIパーツごとに`ReactDOM.render()`を使って小さくReact Componentに置換できるということが主張したいのでしょう。

以下の記事では、step by stepでの安全なReact移行について解説してあり、`render() { return null }`から入るのが面白かったです。

[Migrating your front-end to React, step by step. — Xebia Blog](https://xebia.com/blog/migrating-to-react-step-by-step/)

他にもこのようなテクニックを使った移行に関する知見がどこかにまとまっているなら、読んでみたいです。
