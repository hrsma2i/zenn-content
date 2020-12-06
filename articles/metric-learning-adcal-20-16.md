---
title: "metric learning の推論における高速化・軽量化"
emoji: "📏"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["metriclearing", "kNN", "ANN", "Elasticsearch", ]
published: true
---

:::message
こちらは [metric learning Advent Calendar 2020](https://adventar.org/calendars/5596) の記事です。
:::

# この記事の目的は？

metric learning のモデルを Web アプリに載せたい方に、 推論における高速化・軽量化の工夫を3つ紹介します。

機械学習を Web アプリに載せる際は、精度だけでなく、推論速度や容量の小ささも重要になってきます。

# 推論方法

metric learing のモデルの推論方法は主に2つあり、それぞれの利用例を以下の表にまとめました。

|推論方法|利用例|概要|
|:-|:-|:-|
|最近傍探索（Nearest neighbor search: NNS）|画像による商品検索、ニュースの記事推薦|クエリの商品・記事に近いものをいくつか返す|
|k最近傍法（k-Nearest neighbor: kNN）|商品画像の色・カテゴリー予測|クエリの商品に近い商品の色・カテゴリーから多数決で決める|


# early fusion ではなく late fusion を使う

metric learning のモデルのアーキテクチャは early fusion と late fusion に分けられます。 early fusion はデータを結合してからネットワークに通して計量を計算する方法です。一方、 late fusion はデータをそれぞれネットワークに通してから、得られた特徴量で計量を計算する方法です。

（図）

early fusion より **late fusion の方が速く推論しやすい** です。 なぜなら、 early fusion だと計算の度にデータをネットワークに通す必要がありますが、 late fusion だと予め抽出しておいた特徴量からユークリッド距離やコサイン類似度など比較的 **簡単な計量の計算だけで済む** からです。また、次のセクションで紹介する **近似最近傍探索が使える** という利点もあります。

ちなみに、 early/late fusion は metric learning というよりマルチモーダル学習で一般的な用語ですが、 同アドベントカレンダーの11日目の記事でマルチモーダル学習と metric learning の繋がりを紹介しています。

# 近似最近傍探索

商品検索などでは NNS で探索する商品数が膨大になることがあります。このような場合、クエリと探索範囲の商品すべての組に対し、計量を愚直に計算すると相当な時間がかかってしまいます。そこで、 近似最近傍探索（approximate nearest neighbor: ANN）という方法なら、精度が若干落ちるものの、 **膨大な探索範囲から高速に探索** できます。

ANN のライブラリはいくつかありますが、どのライブラリを使えばいいかは、以下のスライドが参考になると思います。

- [[CVPR20 Tutorial] Billion-scale Approximate Nearest Neighbor Search - Speaker Deck](https://speakerdeck.com/matsui_528/cvpr20-tutorial-billion-scale-approximate-nearest-neighbor-search?slide=115)
- [近似最近傍探索の最前線 - Speaker Deck](https://speakerdeck.com/matsui_528/jin-si-zui-jin-bang-tan-suo-falsezui-qian-xian?slide=7)

また、以下のように分散処理で ANN を行うライブラリもあります。

|ライブラリ|ANN バックエンド|備考|
|:-|:-|:-|
|[Milvus](https://github.com/milvus-io/milvus)|いろいろ|[令和時代のサーチエンジンになるか？　気鋭のベクトル検索OSS Milvus についてまとめてみた - Taste of Tech Topics](https://acro-engineer.hatenablog.com/entry/2019/12/26/114500)|
|[Vald](https://github.com/vdaas/vald)|[NGT](https://github.com/yahoojapan/NGT)|Kubernetes 上で動かせる。|
|[k-NN plugin - Open Distro for Elasticsearch](https://opendistro.github.io/for-elasticsearch-docs/docs/knn/)|[NMSLIB](https://github.com/nmslib/nmslib)|[Amazon Elasticsearch Service で k 近傍法 (k-NN) 類似検索エンジンが構築可能に](https://aws.amazon.com/jp/about-aws/whats-new/2020/03/build-k-nearest-neighbor-similarity-search-engine-with-amazon-elasticsearch-service/)|

Elasticsearch での最近傍探索については以下の資料も参考になります。

- [Elasticsearchでマルチモーダル画像検索 1 - riktorのメモ](https://riktor.hatenadiary.jp/entry/2019/11/02/220336)
- [Elasticsearch における類似度ベクトル検索のベストプラクティスを求めて/es-vector-search - Speaker Deck](https://speakerdeck.com/takahiko03/es-vector-search)


ところで、近似最近傍探索の問題の1つとして、 **インデックスを作った後に動的に探索範囲を絞れない** ことが挙げられます。そこで、それを可能にしたのが [matsui528/rii](https://github.com/matsui528/rii) というライブラリになります。これにより **値段で絞った商品の画像検索** などを実現しやすくなります。



# 特徴量の平均を重みベクトルとする

商品画像の色・カテゴリー予測などを高速化・軽量化する方法として kNN+ANN も考えられますが、より簡単な「特徴量の平均を重みベクトルとする」方法があります。 そのアルゴリズムを以下に記します。

まず、あらかじめ以下の用意をしておきます。

1. 予測したい商品の色・カテゴリーごとに商品画像をいくつか（1000個とか）サンプリングする。
2. それらの画像特徴量を metric learning で学習させたモデルで抽出する。
3. 色・カテゴリーごとに特徴量の平均をとり、その色・カテゴリーの重みベクトルとする。

（図）

推論時は以下のように行います。

1. クエリ画像の特徴量を抽出する。
2. クエリ特徴量と用意しておいた重みベクトルの計量を計算する。
3. 最も近かった重みベクトルの色・カテゴリーを予測結果とする。

（図）

イメージとしては、 metric learning のモデルで抽出した特徴量をもとに、1つのFC層を持つ線形分類器を学習させていることに近いです。あらかじめ計算する重みベクトルが、FC層の重みに相当します。


# まとめ

metric learning の推論における高速化・軽量化の工夫として以下の3つを紹介しました。

- early fusion ではなく late fusion を使う
- 近似最近傍探索
- 特徴量の平均を重みベクトルとする

# 次回は？

「metric learning のファッション分野における活躍」を紹介します。
