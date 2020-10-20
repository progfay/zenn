---
title: "WriteCodeEveryDayを始めて500日が経過しました"
emoji: "👨‍💻"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["WriteCodeEveryDay"]
published: true
---

WriteCodeEveryDayを始めてから500日が立ったので、感想と今後の抱負を残します。WriteCodeEveryDayを始めた経緯等については以下の記事を参照してください。

[WriteCodeEveryDayを始めて1年経った - blogfay](https://progfay.hatenablog.com/entry/2020/06/10/221443)

## 社会人生活の中で書き続ける

私がWriteCodeEveryDayを始めたのは大学4年生の時で、卒論の執筆や共同研究と並行して毎日コードを書くのは大変でした。しかし、社会人になってからの方が自由に動ける時間が少なくなり、コードを書くための時間と体力を考えると制約が増えました。学生時代から始めていた恩恵として、毎日コードを書くことが習慣となっていたので短くても時間を確保し集中して書くことはできました。問題は体力面で、勢いに乗ってコードを書きすぎてしまい夜更かしをしないようにするなどのコントロールが必要になりました。ここで問題になったのは生活習慣で、就寝と起床の時間を気にしながらコードを書くようになったため負荷が増えました。

私は、100％の集中力でなくても一定のパフォーマンスを出せるようになるための訓練としてWriteCodeEveryDayを行っており、この負荷は私が期待していたものでした。仕事をしていく上で完全に集中してコードを書けるタイミングを待つより、60％の集中でもパフォーマンスを発揮できるエンジニアを目標としています。

## 感想・抱負

社会人になってリモートでの仕事というのもあって、生活習慣が段々固定されてきました。終業してから就寝までにコードを書く時間を長く取ることは難しく、30分から2時間ほどしか確保できていません。最近では[柴田 芳樹さん](https://twitter.com/yoshiki_shibata)による[プログラミング言語Go研修](https://yshibata.blog.ss-blog.jp/archive/c2305793699-1)に参加しており、これの練習問題をWriteCodeEveryDayの一環として日々解いています。大変ではありますが、小さなコードをたくさん書いて設計やテストを繰り返す中で浮かんだ疑問点を柴田さんに質問して解決するというプロセスを回せるため、Goの範囲に限らない学習ができていると実感しています。

一方で、他の開発にあまり時間を割けていないのが現状です。作りたいもののリストは書き溜めているので、いづれ開発に踏み切りたいと考えながらも足が重くなっているので、小さく始めていきたいです。

[Zenn](https://zenn.dev/)ではGitHubを通して記事を公開でき、執筆活動も技術に関するアウトプットとなると考えてWriteCodeEveryDayに含めました。今まではWriteCodeEveryDayで時間を取れていませんでしたが、これを機に記事執筆にも力を入れていきます。

WriteCodeEveryDayはコードを書くことに専念してしまいがちですが、私にはコードを読む能力も不足しています。
そのため、新たな試みとして「OSSのコードを読んで記事にする」活動をWriteCodeEveryDayの一環とすることを検討しています。
インプットをそのままアウトプットへと繋げるスピード感を今後の活動に反映していきたいです。

## 宣伝

GitHub Contribution Streak (連続でcontributionのある日数) を数えてくれるCLIツールを作りました。

[progfay/github-streaks: Check streak on GitHub from CLI](https://github.com/progfay/github-streaks)

![carbon](https://storage.googleapis.com/zenn-user-upload/bvcorb8x4qifee0s4a1op9lzsdi3)

```sh
npx @progfay/github-streaks [username]
```

最長のStreakや直近のStreakの他にも全Contributionの統計データも計算して表示しています。
是非、試しに一度自分のContributionを調べてみてください！

## 最後に

WriteCodeEveryDayの日数は本質的な目標ではないですが、継続のためのモチベーションになっています。今後もこの活動を絶やさずに続けていきますので、応援よろしくお願いします。次回は「WriteCodeEveryDayを始めて2年が経過しました」にて活動の経過報告をさせていただきます。
