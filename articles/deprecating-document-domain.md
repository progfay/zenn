---
title: "Deprecating document.domain setter"
emoji: "🚷"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Browser"]
published: true
---

[`document.domain`](https://developer.mozilla.org/en-US/docs/Web/API/Document/domain) は `Document` の Origin から domain の部分を参照できる property です。
この値を書き換えることで [Same-Origin Policy](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy) を緩和することができますが、この機能は非推奨となっています。

そして、この機能はセキュリティや [Cross-Origin Isolation](https://web.dev/cross-origin-isolation-guide/) の問題から利用できなくなることが計画されています。

https://groups.google.com/a/chromium.org/g/blink-dev/c/_oRc19PjpFo/

本記事では、 `document.domain` の書き換えができなくなる理由についてまとめます。

## `document.domain` setter

`document.domain` setter の MDN を参照してみましょう。

https://developer.mozilla.org/en-US/docs/Web/API/Document/domain#setter

> The setter for this property can be used to change a page's origin, and thus modify how certain security checks are performed. It can only be set to the same or a parent domain.

`document.domain` を変更することで Same-Origin Policy を緩和できます。

`document.domain` にセットできる値はサイト自体の domain から eTLD + 1[^1] までの親 domain までです。
例えば `a.b.example.com` の `document.domain` にセットできる値は、 `'a.b.example.com'`, `'b.example.com'`, `'example.com'` となります。

[^1]: [Public Suffix List](https://publicsuffix.org/list/) で管理されている eTLD の一つ下の階層のこと

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

### Cross-Origin Isolation についての問題

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

そこで `document.domain` setter の利用を非推奨とすることで Cross-Origin Isolation を推し進められるようになります。

## 代替案

上記の理由から `document.domain` setter は利用できなくなることになりました。
Chrome 101 からはデフォルトで `document.domain` が変更できないようになる予定です。

https://chromestatus.com/feature/5428079583297536

### `postMessage()` と Channel Message API

`document.domain` setter を置き換える方法として、 [`window.postMessage()`](https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage) か [Channel Messaging API](https://developer.mozilla.org/en-US/docs/Web/API/Channel_Messaging_API) があります。
これらの API を使うことで Cross-Origin 間の通信を実現することができます。

詳しい使い方については以下のサイトにて説明されています。

https://developer.chrome.com/blog/immutable-document-domain/#use-postmessage-or-channel-messaging-api

### `Origin-Agent-Cluster` Header

何らかの理由で `document.domain` setter の利用を続けたいケースがあるかもしれません。
その場合は [`Origin-Agent-Cluster`](https://html.spec.whatwg.org/multipage/origin.html#origin-keyed-agent-clusters) Header を使うことで引き続き `document.domain` の書き換えが可能になります。

`Origin-Agent-Cluster` Header を用いることで、対象の `Document` を同じ Origin を持つ他の `Document` と同じ process 上で処理するか否かを判断するためのヒントを与えることができます。
これは特に subdomain だけが異なるような Cross-Origin な `Document` の Isolation に役立ちます。

> The new Origin-Agent-Cluster header asks the browser to change this default behavior for a given page, putting it into an origin-keyed agent cluster, so that it is grouped only with other pages that have the exact same origin. In particular, same-site cross-origin pages will be excluded from the agent cluster.
>
> [Requesting performance isolation with the Origin-Agent-Cluster header](https://web.dev/origin-agent-cluster/#why-browsers-can't-automatically-segregate-same-site-origins) より引用

`Origin-Agent-Cluster` Header の値には [Structured Field Value](https://datatracker.ietf.org/doc/html/rfc8941) の boolean 値が設定できます。
`?1` で `true` を、 `?0` で `false` を示します。

> A Boolean is indicated with a leading "?" character followed by a "1" for a true value or "0" for false.
>
> [RFC 8941 - Structured Field Values for HTTP](https://datatracker.ietf.org/doc/html/rfc8941#section-3.3.6) より引用

Chrome による `document.domain` setter を廃止するにあたって、後方互換性を維持するために `Origin-Agent-Cluster: ?0` を明示的に送信することで `document.domain` setter を引き続き利用できるようにしています。

> We should deprecate it, by making it opt-in via `Origin-keyed agent clusters` (https://chromestatus.com/features/5683766104162304)
>
> The setter will remain, but the origin remains unchanged. In that case the compatibility risk is low.
>
> [Feature: Deprecate the `document.domain` setter.](https://chromestatus.com/feature/5428079583297536) より引用

しかしながら `document.domain` setter の利用は前述のようなセキュリティ的懸念があるため、何らかの強い理由がない限りは移行を検討するべきでしょう。
