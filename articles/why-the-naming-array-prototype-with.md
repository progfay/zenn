---
title: "Array.prototype.with の命名について"
emoji: "📛"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["tc39", "javascript"]
published: true
---

## `tc39/proposal-change-array-by-copy`

`tc39/proposal-change-array-by-copy` は TC39 に提案されている 2022-12-27 現在で Stage3 の proposal です。
https://github.com/tc39/proposal-change-array-by-copy

本 proposal では、配列に対して破壊的な変更を行うメソッドに対して同様の処理を非破壊的に行えるメソッドを追加することが提案されています。

- `Array.prototype.toReversed() -> Array`
- `Array.prototype.toSorted(compareFn) -> Array`
- `Array.prototype.toSpliced(start, deleteCount, ...items) -> Array`
- `Array.prototype.with(index, value) -> Array`

### `Array.prototype.with` って？

`toReversed`, `toSorted`, `toSpliced` は名前から挙動の推測がつきますが、 `with` だけはわかりませんでした。

proposal 内の Example には `Array.prototype.with` が以下のような挙動をするようです。

```js
const correctionNeeded = [1, 1, 3];
correctionNeeded.with(1, 2); // => [1, 2, 3]
correctionNeeded; // => [1, 1, 3]
```

[Spec](https://tc39.es/proposal-change-array-by-copy/#sec-array.prototype.with) も合わせて読むと、挙動は以下のようなものになりそうです。

- `arr.with(index, value)` は `arr` のうち、 `index` 番目を `value` に置き換えたものを `arr` を破壊することなく生成する
- 第一引数には [`Array.prototype.at`](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Array/at) と同様に `Array` の index を相対的に指定できる
  - 例: `-1` を指定すると `Array` の末尾を置き換え位置の対象にできる

Chrome 108 から Developer Trial が開始しているため、 `enable-javascript-harmony` を有効化することで動かすことができます。
(`chrome://flags/#enable-javascript-harmony` から有効化できます。)

![Chrome DevTools Console](https://storage.googleapis.com/zenn-user-upload/c22dff7106f6-20221227.png)

## なぜ `Array.prototype.with` という命名なのか？

他の命名もあったはずですが、なぜ `with` という抽象度が高い命名になったのかが気になったので調査しました。

proposal の issue を漁ってみると丁度ドンピシャな質問をしている issue がありました。

https://github.com/tc39/proposal-change-array-by-copy/issues/103

> why `with()` and not something like `replaceValue()`?

> Some of the reasons for choosing with were brevity and also because of the similarities to [its use in the Temporal proposal](https://tc39.es/proposal-temporal/#sec-temporal.plaindate.prototype.with).

ということで「`Temporal.PlainDate.prototype.with` と似ているから」が理由らしいです。

## `Temporal.PlainDate.prototype.with` の命名理由も調べてみる

`Temporal.PlainDate.prototype.with` が spec に追加されたのは[この commit](https://github.com/tc39/proposal-temporal/commit/50a84efee15a1fa01b99e5e4cacb340a5c0848ac) からのようですが、命名については議論されてなさそうでした。
おそらく [`Temporal.PlainDate.prototype.with`](https://tc39.es/proposal-temporal/#sec-temporal.plaindate.prototype.with) は [`Temporal.PlainDate.prototype.withCalendar`](https://tc39.es/proposal-temporal/#sec-temporal.plaindate.prototype.withcalendar) をより汎用化する過程で `Calendar` を取ってできた命名だったのではないでしょうか。

(`with` という単語が検索難易度を格段に引き上げているため調査が大変でした...)

## おまけ: `Array.prototype.pushed` はどこいった？

Stage 2 の途中までは `Array.prototype.push` を非破壊的に行う `Array.prototype.pushed` などがあったはずですが、気がついたら以下の議論の末に消えてました。

https://github.com/tc39/proposal-change-array-by-copy/issues/24
https://github.com/tc39/proposal-change-array-by-copy/issues/27
