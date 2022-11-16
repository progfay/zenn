---
title: "email@example.com はドキュメントに使っていいのか？"
emoji: "📨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["email", "domain", "RFC", "documentation"]
published: true
---

## Summary

結論: RFC で推奨されていた

## ドキュメント専用の IP アドレスが用意されている

以前、 IP アドレスの例として `xxx.xxx.xxx.xxx` を使わない方がいいという記事を読みました。

https://qiita.com/Targoyle/items/1c5454a41ea4519b0c5f

こちらの記事の中で

> Wikipedia内にIPv4#特別用途のアドレスの表があります。こちらのドキュメント用のものをつかうか、

という記述があり、以降は IP アドレスの例をドキュメントに書く際にはこちらを参照するようにしています。

## 調査の発端

[Hacktoberfest](https://hacktoberfest.com/) で [github/docs](https://github.com/github/docs) に Contribution しようと思って docs を読み漁っている際に、 `email@example.com` という記述を見つけました。

https://github.com/github/docs/blob/68dad52f10e271066d92030e50f4883ab4df0d6e/data/ui.yml#L78

今まで特に深く考えたことはありませんでしたが、なんとか Contribution がしたいという一心でこの Email アドレスが本当に使っていいものなのかを調べてみることにしました。

## `email@example.com` はドキュメントに使っていいのか？

RFC 2606 の "Reserved Example Second Level Domain Names" という Section に書かれていました。

https://datatracker.ietf.org/doc/html/rfc2606#section-3

> The Internet Assigned Numbers Authority (IANA) also currently has the
> following second level domain names reserved which can be used as
> examples.
>
>   example.com
>   example.net
>   example.org

ということで、 `@example.com` は例として使って良さそうだということが定義されていました。

## 結論

結論として `example.com`, `example.net`, `example.org` などのドメイン名はドキュメントなどに使っていいことがわかりました。

しかし `sample.com` などは記述がなかったので、 RFC に準拠するのであれば使用を避けた方が良いかもしれないですね。

## 余談

結局 [github/docs](https://github.com/github/docs) への Contribution は別の方法で達成しました。

https://github.com/github/docs/pull/21133

あと [Hacktoberfest](https://hacktoberfest.com/) は僕のやる気が続かなくて途中で諦めちゃいました 😇
