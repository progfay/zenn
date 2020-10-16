---
title: "ISUCON10予選の作問を担当しました"
emoji: "🪑"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["ISUCON"]
published: true
---

![thumbnail](https://storage.googleapis.com/zenn-user-upload/ngc2fn4xm2lik9gh281ers3czagl)

この度、ISUCON10の運営に携わる機会を頂き、予選問題の作成を担当しました。作問の際に考えていたことや狙いなどを、記録として記事に残すこととします。

Github: [isucon/isucon10-qualify](https://github.com/isucon/isucon10-qualify)

## 運営に携わることになった経緯

私は株式会社リクルートの2020年度新卒としてリクルートテクノロジーズに入社しました。 ~~新人研修の一環として実施されるOJTでは、私を含めて3名の新人が予選問題の作成チームにアサインされました。~~ (通常の新人研修から外れ、ISUCONの運営を手伝う経緯になりました。)

私がアサインされたタイミングでは、以下の二点が決まっているだけでした。

- テーマがISUUMOというイスと物件を購入できるサービスである
- 地図上をなぞって物件を探せる「なぞって検索」という目玉機能がある

[@yosuke_furukawa](https://twitter.com/yosuke_furukawa)さんを含む3名にメンターをしていただきながら、新卒組で仕様から決めていきました。

## 予選問題の狙い

ISUCONでは、チューニングの対象としてアプリケーションやデータベースにボトルネックを作ります。当初はどこにボトルネックを作るかの議論をしましたが、これを考えながら設計するのは非常に困難でした。「これをやったら重そうだよね」というアタリを付けておき、実際にコードを書いていくと重たいアプリケーションが完成しました。

興味深いのは、これが意図したボトルネックと意図していなかったボトルネックが生まれることでした。例えば今回の予選問題で言うと、「なぞって検索」は意図したボトルネックでしたが、昇順と降順を組み合わせたソートは意図せず生まれた天然のボトルネックでした。こういう偶然生まれたボトルネックも問題として取り入れながら問題難易度の調整をしました。実際に問題を作成してから解いてみると、想像以上に難しくスコアが全く上がらなかったため、難易度を下げるために一部ボトルネックを削除したり負荷の掛け方を変えたりと試行錯誤を行いました。

また個人的な狙いですが、予選問題は決勝に進出するチームを選出するためのふるいの役割であるため、計測ができなければスコアが上がらないような問題を意識して作成しました。

## 実装の解説

次に私が担当した部分を、何を考えながら実装したかを交えて解説していきます。

### フロントエンド

ISUCONのフロントエンド部分は動作の確認に用いられることが多く、特に今回はベンチマークの対象としなかったため簡易的に作成しました。「なぞって検索」は、線が交差しても検索が行えるように凸法に変換するなどの処理を加えていますが、それ以外に特殊なことはしていません。イス検索や物件検索の条件の数や、「なぞって検索」の機能が「なんとなく重そうだ」ということを意識してもらえるようにアプリケーションを組みました。

今回のベンチマーカーはトップページから順にサイトを訪問するようなシナリオを3つ持っていました。また、フロントエンドに競技者の注意を向けさせないために、Next.jsでSSGをしてnginxから静的ファイル配信をするのみに留めました。

### バックエンド

今回の問題では、データベース側のボトルネックを重点的に作りました。これは複数言語で実装されるISUCONにおいて、どの言語でも同様の課題に直面するであろうと考えたからです。実際に競技者の方が書いた記事などを見ても、データベースをどうチューニングするかという問題に向き合っていました。

### ベンチマーカー

予選問題でのスコア計算式は以下の通りでした。

```
スコア = (イスの購入件数 + 物件の資料請求件数) - 減点
```

これはISUUMOというサイトにおけるコンバージョン数を示しており、即ち「ユーザーに購入や資料請求をしてもらえるようなサイト」である必要があります。しかし、現実のサービスでは、サイトの表示に時間が掛かるとユーザーは離脱してしまいます。(参考: [Why does speed matter? | web.dev](https://web.dev/why-speed-matters/))

今回の問題は、Webサイトのトップページから順に以下の3種類のシナリオが実行されます。Webサイトを表示するためのAPIが1秒以内に返ってこない場合はシナリオをやり直す仕様になっていました。

- イス検索シナリオ: トップページ → イス検索 → イス詳細 → イス購入 → イスによるおすすめ物件の詳細 → 物件の資料請求
- 物件検索シナリオ: トップページ → 物件検索 → 物件詳細 → 物件の資料請求
- なぞって検索シナリオ: トップページ →  なぞって検索 → 物件詳細 → 物件の資料請求

また、これとは別に以下のシナリオがありました。

- ボットシナリオ: イス検索と物件検索を定期的に実行する
- イス入稿シナリオ: 特定のタイミングでイスのCSV入稿を行う
- 物件入稿シナリオ: 特定のタイミングで物件のCSV入稿を行う

これらのシナリオを用いて自動負荷上昇を実現しました。ワーカーがシナリオを実行し、シナリオが成功すれば即座にシナリオを再実行、失敗すれば1, 2秒ほど待ってからシナリオを再実行します。負荷の上昇はスコア (= コンバージョン数) が一定値を超えたタイミングで行われます。この設定は[isucon/isucon10-qualify/bench/parameter/parameter.go](https://github.com/isucon/isucon10-qualify/blob/master/bench/parameter/parameter.go#L33-L146)にまとまっているので、そちらを参照してください。

## 負荷レベルごとの解説

ここからは、この負荷レベルごとの狙いやボトルネックについて解説していきます。

### Level 0 (1 ~ 299)

スコアに繋がる3つのシナリオは、全てトップページにアクセスするところから始まります。そして、トップページでは最安のイス・物件を取得し表示しています。このため、SQLのSlow Queryを調べると、`GET /api/chair/low_priced`と`GET /api/estate/low_priced`がボトルネックとして現れます。

```sql
mysql> EXPLAIN SELECT * FROM chair WHERE stock > 0 ORDER BY price ASC, id ASC LIMIT 20;
+----+-------------+-------+------------+------+---------------+------+---------+------+-------+----------+-----------------------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows  | filtered | Extra                       |
+----+-------------+-------+------------+------+---------------+------+---------+------+-------+----------+-----------------------------+
|  1 | SIMPLE      | chair | NULL       | ALL  | NULL          | NULL | NULL    | NULL | 29545 |    33.33 | Using where; Using filesort |
+----+-------------+-------+------------+------+---------------+------+---------+------+-------+----------+-----------------------------+
1 row in set, 1 warning (0.01 sec)

mysql> EXPLAIN SELECT * FROM estate ORDER BY rent ASC, id ASC LIMIT 20;
+----+-------------+--------+------------+------+---------------+------+---------+------+-------+----------+----------------+
| id | select_type | table  | partitions | type | possible_keys | key  | key_len | ref  | rows  | filtered | Extra          |
+----+-------------+--------+------------+------+---------------+------+---------+------+-------+----------+----------------+
|  1 | SIMPLE      | estate | NULL       | ALL  | NULL          | NULL | NULL    | NULL | 29221 |   100.00 | Using filesort |
+----+-------------+--------+------------+------+---------------+------+---------+------+-------+----------+----------------+
1 row in set, 1 warning (0.01 sec)
```

`ORDER BY`に時間が掛かっているようなので、INDEXを貼ります。

```sql
CREATE INDEX price_id_idx on chair(price, id);
CREATE INDEX rent_id_idx on estate(rent, id);
```

```sql
mysql> EXPLAIN SELECT * FROM chair WHERE stock > 0 ORDER BY price ASC, id ASC LIMIT 20;
+----+-------------+-------+------------+-------+---------------+--------------+---------+------+------+----------+-------------+
| id | select_type | table | partitions | type  | possible_keys | key          | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-------+------------+-------+---------------+--------------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | chair | NULL       | index | NULL          | price_id_idx | 8       | NULL |   20 |    33.33 | Using where |
+----+-------------+-------+------------+-------+---------------+--------------+---------+------+------+----------+-------------+
1 row in set, 1 warning (0.01 sec)

mysql> EXPLAIN SELECT * FROM estate ORDER BY rent ASC, id ASC LIMIT 20;
+----+-------------+--------+------------+-------+---------------+-------------+---------+------+------+----------+-------+
| id | select_type | table  | partitions | type  | possible_keys | key         | key_len | ref  | rows | filtered | Extra |
+----+-------------+--------+------------+-------+---------------+-------------+---------+------+------+----------+-------+
|  1 | SIMPLE      | estate | NULL       | index | NULL          | rent_id_idx | 8       | NULL |   20 |   100.00 | NULL  |
+----+-------------+--------+------------+-------+---------------+-------------+---------+------+------+----------+-------+
1 row in set, 1 warning (0.00 sec)
```

このチューニングでLevelが上がるという想定でした。本ボトルネックは、計測とチューニングをしっかり行えれば解けるという意味で初心者向けに用意しました。

### Level 1 (300 ~ 599)

次にボトルネックとなってくるのは、検索時などに出てくる`ORDER BY popularity DESC`です。これもINDEXを貼れば解決されますが、落とし穴としてMySQL 5.7では降順INDEXがサポートされていません。そのため、MySQLのバージョンを上げたり、GENERATE COLUMNを用いてpopularityに-1を掛けたCOLUMNを用意したりと、テクニックが必要になります。実際に`CREATE INDEX popularity_id_idx on estate(popularity, id);`を貼っただけでは`Using filesort`となってしまいます。計測→チューニング→計測のループを回し、正しくチューニングが行えているのかの確認が大切であることを問うボトルネックでした。

Level 1からは「なぞって検索」を行うシナリオが走り、ここもボトルネックとして表出していきます。各言語のソースコードに、明らかにN+1となっている箇所があるため、これを修正します。

```sql
-- before
SELECT * FROM estate WHERE latitude <= ? AND latitude >= ? AND longitude <= ? AND longitude >= ? ORDER BY popularity DESC, id ASC;
SELECT * FROM estate WHERE id = ? AND ST_Contains(ST_PolygonFromText(%s), ST_GeomFromText(%s));

-- after
SELECT * FROM estate
  WHERE latitude <= ? AND latitude >= ? AND longitude <= ? AND longitude >= ? AND ST_Contains(ST_PolygonFromText(%s), ST_GeomFromText(%s))
  ORDER BY popularity DESC, id ASC;
```

また、ここで[SPATIAL INDEX](https://dev.mysql.com/doc/refman/5.6/ja/creating-spatial-indexes.html)の存在に気付いた参加者の方もいらっしゃったでしょう。SPATIAL INDEXを貼ることで、バウンディングボックスによる絞り込み (`latitude <= ? AND latitude >= ? AND ...`) も不要になり、高速化が図れます。

```sql
CREATE TABLE isuumo.estate
(
    -- skip
    latitude    DOUBLE PRECISION    NOT NULL,
    longitude   DOUBLE PRECISION    NOT NULL,
    point       POINT AS (POINT(latitude, longitude)) STORED NOT NULL,
    -- skip
);

ALTER TABLE estate ADD SPATIAL INDEX estate_point_idx(point);

SELECT id, thumbnail, name, description, latitude, longitude, address, rent, door_height, door_width, features, popularity FROM estate
  WHERE ST_Contains(ST_PolygonFromText(%s), point)
  ORDER BY popularity_desc, id;
```

MySQLでSPATIAL INDEXを取り扱っている参加者は少ないと想定しており、初めて見たチューニング方法に対して短い競技時間内で対応できるかを問うボトルネックでした。

### Level 2 (600 ~ 799)

Level 2からは、イス・物件のCSV入稿とボットのシナリオが走ります。

CSV入稿はイス・物件が一気に500件ずつ増えるAPIです。このリクエストは5秒以内に返さないとタイムアウトし、ベンチマークがfailする仕様となっています。アプリケーション側でCSVをパースし、1件ずつINSERTを行っているため、タイムアウトしてしまう可能性も十分にありました。BULK INSERTで複数件を一気に入れるなどの方法で負荷を軽減し、タイムアウトしないようなチューニングが必要でした。

ボットは初期状態では走行しておらず、Level 2に到達して初めて出現します。そのため、当日マニュアルを見て実装するだけではパフォーマンスが改善されないようになっています。計測してからのチューニングが大切であることを示すためのボトルネックでした。

しかし、競技開始直後は運営側の不手際でベンチマークが実行できない状況でした。このため、とりあえずボットを弾く処理から手をつけた競技者も多かったのではないでしょうか。

ボットに対する正規表現マッチがアプリケーション側のボトルネックとなるように作問しました。実はボットのUser-Agentは、走行しているボット1台に対して固定で設定されています。User-Agentとボット判定の結果をキャッシュすることでアプリケーション側のボトルネックを解消できると想定していました。しかし、データベースのボトルネックが大きすぎたため、ボット判定が問題となることはありませんでした。

### Level 3 ~ (800 ~)

ここからはレベルアップの度に全シナリオに対して1つのワーカーが増えるようになっています。

予選問題では3台のサーバーが用意されていたため、アプリケーション、イスDB、物件DBの構成にするとデータベースへの負荷が分散してスコアが大きく伸びます。

また、イス・物件検索の検索条件はレベルが高ければ高いほど複雑になる仕様となっています。イス・物件検索のチューニングには明確なチューニング方法がなく、実際に計測とチューニングを繰り返し行う必要があります。検索条件には[多少の偏りが生まれるようになっており](https://github.com/isucon/isucon10-qualify/blob/master/bench/scenario/searchQuery.go)、これを狙ってチューニングを行うことでより高いスコアを狙うことができます。

### その他

予選終了後のブログにて想定外の解法が書かれており、出題者としては非常に嬉しかったです。
その中から1つだけピックアップして紹介させていただきます。

競技プログラミングの要素としてイスがドアを通過可能かの判定するボトルネックを用意していました。

```sql
SELECT * FROM estate
  WHERE
    (door_width >= ? AND door_height >= ?) OR
    (door_width >= ? AND door_height >= ?) OR
    (door_width >= ? AND door_height >= ?) OR
    (door_width >= ? AND door_height >= ?) OR
    (door_width >= ? AND door_height >= ?) OR
    (door_width >= ? AND door_height >= ?)
  ORDER BY popularity DESC, id ASC
  LIMIT ?;
```

初期実装では全通りを調べるようになっています。想定解は、椅子の最短の辺がドアの短い方の辺より短く、椅子の2番目に短い辺がドアの長い方の辺より短いことを調べることでした。

しかし、予選を3位で通過したチームFetchDecodeExecWriteはwriteup記事にて、より効率的なアルゴリズムを用いてこの問題を解き、スコアアップを実現していました。アルゴリズムについては僕自身の能力では理解できなかったので、興味のある方は以下の記事を参照してください。

[ISUCON10予選を3位で通過しました (vs descending index) - Lを探す日常](http://akouryy.hatenablog.jp/entry/2020/09/13/130415)

## 最後に

予選問題では計測とチューニングを高速に繰り返すことが重要だったことを述べました。しかしながら、予選では[運営側の不手際](http://isucon.net/archives/55084867.html)によりベンチマーク走行が終わらなかったり結果が何も表示されないままFailになったりと、非常にご迷惑をお掛けしました。自身の経験不足を補うべく、今後も精進していきます。

最後になりましたが、10回目のISUCONという節目のタイミングで運営に携わらせて頂けたことに非常に感謝しています。ISUCON10の予選問題作成の手伝いをするという貴重な経験ができました。

ISUCON11は参加者として、ISUCONを思いっきり楽しみます！
