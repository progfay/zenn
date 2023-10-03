# `HTMLVideoElement` の "DOMException: The play() request was interrupted by a call to pause()" の原因を調査する

## TL; DR

`HTMLVideoElement.prototype.play()` による動画読み込み中に `HTMLVideoElement.prototype.pause()` を呼ぶと `DOMException` が発生します。
この原因の調査が難航した場合には、 `prototype` を wrap してログを埋め込むような動的解析をすると特定の一助になるかもしれません。

## DOMException: The play() request was interrupted by a call to pause()

[`HTMLVideoElement.prototype.play()`](https://developer.mozilla.org/docs/Web/API/HTMLMediaElement/play) は [`<video>` tag](https://developer.mozilla.org/docs/Web/HTML/Element/video) の再生を開始する関数です。
`HTMLVideoElement.prototype.play()` は呼ばれるとまず動画の読み込みを開始し、それが終わり次第再生を行います。
この関数は返り値として、読み込みが終わり次第 resolve される `Promise<void>` を返します。

しかし、読み込みの途中で同一インスタンスの [`HTMLVideoElement.prototype.pause()`](https://developer.mozilla.org/docs/Web/API/HTMLMediaElement/pause) が呼ばれると、この `Promise<void>` は reject され、 [`DOMException`](https://developer.mozilla.org/docs/Web/API/DOMException) を発生させます。
エラーメッセージは以下の通りです。

> _Uncaught (in promise) DOMException: The play() request was interrupted by a call to pause().

ref. [DOMException - The play() request was interrupted - Chrome for Developers](https://developer.chrome.com/blog/play-request-was-interrupted/#what-is-causing-this)

## `play()` / `pause()` のスパゲッティ

仕様や実装によっては一つの `<video>` tag に対して様々なタイミングで `play()` や `pause()` を呼ぶこともあり得るかと思います。

実際に僕が開発に携わっていたアプリケーション上では `play()` / `pause()` が至る所から呼ばれており、まるでスパゲッティコードのようになっていました。
そんな中、 `DOMException` が発生してしまいました。

さらに、動画再生 library として使用している [`bitmovin-player` package](https://www.npmjs.com/package/bitmovin-player) は build 前のコードが公開されていません。
[build 後のコード](https://unpkg.com/bitmovin-player@8.134.0/bitmovinplayer.js) はとてもじゃないですが読めたものではありません。

しかし、仮に使用している library が build 前のコードを公開していたとしても、それを読んで原因を調査するのは骨が折れる作業です。

## 動的解析を試してみる

`play()` や `pause()` がスパゲッティのようになっていても、 `bitmovin-player` のような動画再生のための library を使用していてもエラーの原因は突き止めたいです。

しかし、コードを読んで挙動を把握する静的解析では原因の調査は難しそうでした。

そこで、コードを動かして挙動を把握する動的解析に挑戦してみます。

前述の `DOMException` には `HTMLElement.prototype.play` と `HTMLElement.prototype.pause` が関係しています。
この 2 つのメソッドがどのような経緯で呼ばれて、どのタイミングでエラーとなるのかを log として出せれば解析の一助になりそうです。

以下は 2 つのメソッドを wrap して log を仕込むようなコードです。
```js
const HTMLVideoElement_pause = HTMLVideoElement.prototype.pause
HTMLVideoElement.prototype.pause = function (...args) {
	if (args.length === 0) {
		console.trace('HTMLVideoElement#pause', this.src)
	} else {
		console.debug('HTMLVideoElement#pause', this.src, ...args)
	}
	return HTMLVideoElement_pause.call(this, ...arguments)
}

const HTMLVideoElement_play = HTMLVideoElement.prototype.play
HTMLVideoElement.prototype.play = function (...args) {
	if (args.length === 0) {
		console.trace('HTMLVideoElement#play', this.src)
	} else {
		console.debug('HTMLVideoElement#play', this.src, ...args)
	}
	return HTMLVideoElement_play.call(this, ...arguments).catch(e => {
		console.error('HTMLVideoElement#play', this.src, ...args)
		throw e
	})
}
```

このコードでは [`HTMLVideoElement`](https://developer.mozilla.org/docs/Web/API/HTMLVideoElement) の [`prototype`](https://developer.mozilla.org/docs/Glossary/Prototype) に生えた `play()` と `pause()` を wrap しています。
`play()` や `pause()` が呼ばれたタイミングで stack trace を出力し、元の処理を実行します。
また、 `play()` が reject されたタイミングでも stack trace を出力しています。

複数動画がある場合はどの `<video>` tag に対する操作かが分かりづらいため、 [`this.src`](https://developer.mozilla.org/docs/Web/HTML/Element/video#src) を log に出しています。

stack trace が出すぎると読みづらいため、 `video.play('timing 1')` のように記述すると stack trace の代わりに `'timing 1'` が log に出るようにしました。

この動的解析によって、実際に 2 つの `DOMException` の原因を突き止めて修正することができました！

## この動的解析手法は他にも応用できる

Production のコードで `prototype` を書き換えることはあまり好ましくはないと思いますが、エラーやバグの調査などの用途で `prototype` を wrap して log を仕込んだり返り値をいじったりする手法は有効だと思います。

以下のコードを雛形として自分で処理を差し込めるようにしておくと、どこかで役立つかもしれません。
```js
const Class_method = Class.prototype.method
Class.prototype.method = function (...args) {
	return Class_method.call(this, ...arguments)
}
```
