---
title: "分布を見るためのSQL"
emoji: "📊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["sql", "bigquery"]
published: true
---

# この記事の目的

データ分析において平均などの統計値より先に分布を確認することが多いので、 以下のような[度数分布表](https://toukeigaku-jouhou.info/2015/08/23/distribution-frequency-table/) を出すSQLの備忘録です。なお、階級に分ける処理については書いてません。


![](https://toukeigaku-jouhou.info/wp-content/uploads/2015/08/dosuubunpuhyou1.png)

出典：https://toukeigaku-jouhou.info/2015/08/23/distribution-frequency-table/


# サンプルクエリ

以下の商品ごとの注文回数がのっているテーブル（ `tbl_item_order` ）から、注文回数の分布を見たいとします。

|item_name|num_orders|
|:-|:-|
|りんご|1|
|さくらんぼ|1|
|ナシ|1|
|オレンジ|2|
|ヤシの実|2|
|もも|3|

以下の SQL で度数分布表を出せます。

```sql
with tbl_freq as (
  select
    num_orders
    , count(1) as freq
  from tbl_item_order
  group by num_orders
)
, tbl_rate as (
  select
    num_orders
    , freq
    , freq / sum(freq) over ()　as rate
  from tbl_freq
)

select
  -- 注文回数（N回注文されたか）
  num_orders
  -- 頻度、絶対度数（N回注文された商品は何種類あるか）
  , freq
  -- 累積度数
  , sum(freq) over (order by num_orders range between unbounded preceding and current row) as cum_freq
  -- 割合、相対度数（N回注文された商品は全商品のうち、どれくらいの割合なのか）
  , rate
  -- 累積相対度数
  , sum(rate) over (order by num_orders range between unbounded preceding and current row) as cum_rate
from tbl_rate
```

結果

|num_orders|freq|cum_freq|rate|cum_rate|
|:-|:-|:-|:-|:-|
|1|3|3|0.5|0.5|
|2|2|5|0.3333333333333333|0.8333333333333333|
|3|1|6|0.16666666666666666|0.9999999999999999|
