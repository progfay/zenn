---
title: "InteropのFocus Areaには優先順位がある"
emoji: "🔝"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Web", "Interop", "Browser"]
published: true
---

## Summary

- Interop 2024 の Focus Area を選ぶために、候補に順序付けを行っている
- この順序付けは批判の声を集める危険性があるとして、非公表となっている

## Interop 2024 Selection Process

Interop 2024 の Selection Process については以下に記載されています。

https://github.com/web-platform-tests/interop/blob/599822581c3a1de51e92760ab2dd3716780aa83c/2024/selection-process.md

大まかな流れは以下の通りです:

1. Focus Area の候補を公募する
2. Focus Area としての選出基準を満たさない候補を除外する
3. Interop メンバーの誰からも支持されない候補を除外する
4. 各メンバーが残った候補に順位付けを行い、スコアをつけて集計する
5. 順位付けされたリストを元に、議論を通して Focus Area を選出する
6. 前年度から引き継ぐ Focus Area と合わせて完成させる

## 秘匿性

この Selection Process では機密性を重視しています。
特に順位付けについては非公開情報が多いです。
各メンバーが提出する順位付けは Interop Team の秘匿情報となります。

また、各メンバーは候補の順位付けと共に Focus Area 候補に対する異議 ("objection") を申し立てることが可能です。
Focus Area 候補に対する異議を提出するにあたって、メンバーは理由は明記しなくても良いことになっています。

## 順位付けリストの公表について

各メンバーの順位付けが集約されて算出されたスコアによる順位付けリストは Interop Team が公表・非公表を選択することができます。

> At the end of Round 3, the Interop Team will have a clear list of all remaining Focus Area proposals, in order of collective priority. If the team decides they want to publish this ordered list, they may, as long as it’s presented as a collective decision of the Interop Team, and neither blame nor credit is assigned to any particular individual organization.

[DeepL](https://www.deepl.com/ja/translator) による日本語訳:

> ラウンド 3 が終了した時点で、Interop チームは、残りのすべてのフォーカスエリア提案の明確なリストを、集団的な優先順位の順に持つことになります。チームがこの優先順位リストを公表したいと決定した場合、Interop チームの集団的な決定として提示され、特定の個々の組織に対する非難や信用がない限り、公表することができます。

しかし 2024/12/31 現時点でこの順位付けリストの公表はなく、また過去の議論が公開されています。

https://github.com/web-platform-tests/interop/issues/632
https://github.com/web-platform-tests/interop/issues/633

要約すると以下のような内容です:

- 公表するならば、誰の意見かを特定できないような形で行いたい
- 順位付けリストの公表が憶測や批判を招いてしまう
- 最終的に、 Interop Team の合意は得られず非公表となった

## 個人の感想

個人的には、この順位付けリストは Web の動向を追うための非常に貴重で重要な情報だと考えています。
そのため可能であれば公表してほしいと思います。

また、公表が Interop という取り組みについての透明性を高め信頼性の向上に繋がったり、更なる議論が Web の発展に大きく寄与するかもしれないとも思います。

一方で順位付けリストの公表が大衆の憶測や批判を招き、 Interop という取り組みの勢いが削がれてしまう危険性もあるかと思います。
この尊い取り組みを絶やしてはいけないと考えれば、非公表にも頷けます。
