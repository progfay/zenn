---
title: "Deprecating document.domain setter"
emoji: "🚷"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Browser", "Origin Isolation"]
published: false
---

[`document.domain`](https://developer.mozilla.org/en-US/docs/Web/API/Document/domain) は `Document` の Origin から domain の部分を参照できる property です。
この値を書き換えることで [Same-Origin Policy](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy) を緩和することができますが、この機能は非推奨となっています。

そして、この機能はセキュリティや Origin Isolation の問題から利用できなくなることが計画されています。

https://groups.google.com/a/chromium.org/g/blink-dev/c/_oRc19PjpFo/

本記事では、 `document.domain` の書き換えができなくなる理由についてまとめます。

## `document.domain` setter

`document.domain` setter の MDN を参照してみましょう。

https://developer.mozilla.org/en-US/docs/Web/API/Document/domain#setter

> The setter for this property can be used to change a page's origin, and thus modify how certain security checks are performed. It can only be set to the same or a parent domain.

どうやら `document.domain` を変更することで Same-Origin Policy を緩和できるようです。
セットできる値は同一ドメインか親ドメインに限られています。

緩和の詳細については HTML Standard に記載がありました。

https://html.spec.whatwg.org/multipage/origin.html#relaxing-the-same-origin-restriction

> Can be set to a value that removes subdomains, to change the origin's domain to allow pages on other subdomains of the same domain (if they do the same thing) to access each other. This enables pages on different hosts of a domain to synchronously access each other's DOMs.

サブドメインを外すことで Top-Level と iframe 間での DOM へのアクセスが可能となります。

![illustration](https://storage.googleapis.com/zenn-user-upload/b94912833e5f-20220123.png)

## なぜ書き換えができなくなるのか？

`document.domain` setter の利用ができなくなる理由は大きく分けて 2 つあります。

### Same-Origin Policy についての問題

Same-Origin Policy が緩められるというのは Security 的に問題があります。

> Avoid using the document.domain setter. It undermines the security protections provided by the same-origin policy. This is especially acute when using shared hosting;
>
> [HTML Standard](https://html.spec.whatwg.org/multipage/origin.html#relaxing-the-same-origin-restriction) より引用

例えば自分と第三者が `example.com` の subdomain 上で任意のサイトをホストできるような状況を考えてみましょう。
自分のサイト上 (`my.example.com`) で `document.domain = 'example.com'` を実行している場合、他の domain (ex. `evil.example.com`) から Same-Origin Policy を緩和して `my.example.com` の DOM にアクセスできます。

これにより異なる subdomain を持つサイトから DOM に対する read / write が可能となってしまいます。

### Origin Isolation についての問題

`document.domain` setter による挙動を実現するために、同一 Tab 上の Same-Site の frame (Top-Level, `<iframe>` など) は同一 process 上に配置されます。

> Within a single browser tab, frames from different sites are always in different render processes from each other, but frames from the same site are always in the same render process.
>
> [Overview of the RenderingNG architecture - Chrome Developers](https://developer.chrome.com/blog/renderingng-architecture/#cpu-processes) より引用

> Setting the document.domain accessor allows pages (within a site, e.g. user1.example.com and user2.example.com) access to each other. To implement this access, the data from those origins needs to be in the same process.
>
> [mikewest/deprecating-document-domain](https://github.com/mikewest/deprecating-document-domain#a-problem-and-a-solution) より引用

これにより Spectre などの side channel 攻撃に脆弱になってしまいます。
また、 `document.domain` setter は process を分離するための障壁となっています。

> The `document.domain` setter allows developers to relax the same-origin policy, complicating the fundamental security boundary we aim to maintain, and putting roadblocks in the way of post-Spectre changes to Chromium's process model.
>
> [Deprecate the `document.domain` setter. - Chrome Platform Status](https://chromestatus.com/feature/5428079583297536) より引用

そこで `document.domain` setter の利用を非推奨とすることで Origin Isolation を推し進められるようになります。

## 代替案

上記の理由から `document.domain` setter は利用できなくなることになりました。
Chrome 101 からはデフォルトで `document.domain` が変更できないようになる予定です。

https://chromestatus.com/feature/5428079583297536

`document.domain` setter を置き換えるために 2 つの手法が用意されています。

1. [`window.postMessage()`](https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage), [Channel Messaging API](https://developer.mozilla.org/en-US/docs/Web/API/Channel_Messaging_API) を利用する
2. [`Origin-Agent-Cluster: ?0`](https://web.dev/origin-agent-cluster/) Header を利用する

詳しくは [Chrome will disable modifying `document.domain` to relax the same-origin policy - Chrome Developers](https://developer.chrome.com/blog/immutable-document-domain/) にて詳しく説明されていますので、そちらを参照してください。
