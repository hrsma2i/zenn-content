---
title: "推薦モデルを metric leaning の観点からとらえる"
emoji: "📏"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["metriclearning"]
published: false
---


# この記事の目的は？

推薦または metric learning に興味がある人に、 **metric learning と推薦には関係がある** ということを伝えることです。

次のセクションから、いくつかの推薦手法を metric learning の観点からとらえてみます。

# MF (Matrix Factrization)

MF とは、以下の図のようにユーザーによるアイテムの評価行列を低ランクの行列に分解する手法です。

![](https://storage.googleapis.com/zenn-user-upload/okumnjreu41eoljnj5dq8p9qqhda =300x)

以下の最適化問題を解くことで、ユーザー行列とアイテム行列を学習します。

$$
\min_{P, Q} \sum_{u,i} \| R - P^TQ\|^2_F
$$
$$
\Leftrightarrow \min_{P, Q} \sum_{u,i} (r_{u,i} - \bm{p}_u^T\bm{q}_i)^2
$$

- $u \in \{1, ..., U\}$: ユーザーの添字
- $i \in \{1, ..., I\}$: アイテムの添字
- $R \in \mathbb{R}^{U \times I}$: 評価行列（ユーザー $u$ によるアイテム $i$ の評価値 $r_{i,j}$ の行列。 $r_{i,j}$ は5段階評価や ${0, 1}$ など。）
- $P \in \mathbb{R}^{K \times U}$: ユーザー行列（ユーザーの $K$ 次元特徴量ベクトル $\bm{p}_u$ を並べたもの）
- $Q \in \mathbb{R}^{K \times U}$: アイテム行列（アイテムの $K$ 次元特徴量ベクトル $\bm{q}_i$ を並べたもの）
- $\|\cdot\|^2_F$: フロベニウスノルム

上の式では、 ユーザー特徴量とアイテム特徴量の内積を、そのユーザーによるそのアイテムの評価値に近づけようとします。つまり、 **高い評価のユーザーとアイテムの特徴量どうしを近づけ、低い評価のものを離す** ように学習するので、 **評価値を計量とみなした、線形写像を学習する metric learning** ととらえることもできます(※1)。下図は、 それを表したものです。


![](https://storage.googleapis.com/zenn-user-upload/k0n7kfs2rtf40mvoteglwfbtb1jl =400x)

参考
- [Matrix Factorizationとは - Qiita](https://qiita.com/ysekky/items/c81ff24da0390a74fc6c)
- [PyTorchでもMatrix Factorizationがしたい！ | takuti.me](https://takuti.me/ja/note/pytorch-mf/)

# BPR (Bayesian Personalized Raking)

BPR もユーザーによるアイテムの評価を予測しますが、 評価値が implicit feedback の場合を考えます。 implicit feedback とは購入や閲覧など、ユーザーが暗黙的に与える評価のことで、逆に5段階の☆をつけるなど、ユーザーが明示的に与える評価は explicit feedback と言います。implicit feedback では、購入・閲覧されなかったアイテムだからといって、ユーザーが嫌っているかはわかりません。そこで、購入・閲覧されなかったアイテムより、されたアイテムの評価値の方が相対的に高くなるように学習させるのが BPR のキモです。

BPR における評価値の予測方法は自由です。 MF で評価値を予測し、 MF のパラメータ $P, Q$ を最尤推定(※2)する場合は、以下の最小化問題を解くことになります。

$$
\min_{P, Q}{
  - \sum_{(u,i,j)}{
      \log{
        \sigma ( \bm{p}_u^T\bm{q}_i - \bm{p}_u^T\bm{q}_j)
      }
  }
}
$$

- $i$: ポジティブサンプル（ユーザー $u$ が購入・閲覧などをしたアイテム）
- $j$: ネガティブサンプル（ユーザー $u$ が購入・閲覧などをしなかったアイテム）
- $\sigma(\cdot)$: シグモイド関数

(残りの表記は上のセクションと同じ意味です。)

上の式は、 **metric learning の手法である triplet loss にそっくり** です(※3)。

$$
\min_{P, Q} \sum_{(u,i,j)}{ \max(0, - \bm{p}_u^T\bm{q}_i + \bm{p}_u^T\bm{q}_j+m)}
$$

- $m$: マージンという正の実数

どちらも、ユーザーとポジティブサンプルの内積を、ネガティブサンプルとの内積よりも相対的に大きくするように学習しており、 特徴量空間において **ユーザーとポジティブサンプルを近づけ、ネガティブサンプルを離す** 点は同じです。

異なる点は、内積の差に施している関数です。 BPR では $\log{\sigma(\cdot)}$ 、 triplet loss ではマージン付きのヒンジ関数 $\max(0, \cdot +m)$ が施されています。つまり、 BPR では学習によって十分差のついたトリプレット $(u, i, j)$ の項は0にならず、 loss に反映され続けますが、 triplet loss では $m$ だけ差がついた（$\bm{p}_u^T\bm{q}_i \geq \bm{p}_u^T\bm{q}_j+m$ を満たした）項は0になり、 loss に反映されなくなります。

細かな違いはあるものの、どちらも本質的にやっていることは同じなので、 BPR も metric learning に関係があると言えそうです。

参考

- [Steffen Rendle, et al. "BPR: Bayesian Personalized Ranking from Implicit Feedback" UAI, 2009.](https://arxiv.org/pdf/1205.2618.pdf)
- [Jiang Wang, et al. "Learning Fine-grained Image Similarity with Deep Ranking" CVPR, 2014](https://arxiv.org/pdf/1404.4661.pdf)


# SMORe (Sampler Mapper Optimizer for Recommendation)

RecSys2019 で提案された SMOReでは、既存の推薦システムの手法を3つのモジュールに分割することで、統一的に表現できる枠組みです。

協調フィルタリングベースの手法は、学習データはユーザーやアイテムをノードとしたグラフであると考えることができます。モデルの学習は、 ユークリッド空間などに、 **隣接したノード同士は近くに、そうでないノードどうしは遠くに埋め込む** ことであると解釈できます。

以上の解釈を踏まえ、著者らは既存の手法を以下の 3 つのモジュールで表現することを提案しました。

![](https://storage.googleapis.com/zenn-user-upload/h23588gwh1n3nuw1kihn768atoue)

出典：https://www.slideshare.net/changecandy/recsys19-smore

- Sampler：ユーザー・アイテムのグラフからノードをサンプルする方法
- Mapper：サンプルしたノードを特徴量空間に埋め込む学習可能な写像
- Optimizer：類似度または距離と、損失関数


前のセクションで紹介した MF と BPR を SMORe で表すと下図のようになります。


![](https://storage.googleapis.com/zenn-user-upload/viyajv58dcm92h6dkqx4hnbxut42)

出典：https://www.slideshare.net/changecandy/recsys19-smore

BPR は SMORe の Loss の選択肢の1つとしてとらえています。

![](https://storage.googleapis.com/zenn-user-upload/6364i7rkk8jurrih6l3mlys3w9st)

出典：https://www.slideshare.net/changecandy/recsys19-smore


SMORe のモジュールの分け方は、 **metric learning のモジュールの分け方に似ています** 。

[deep metric learningによるcross-domain画像検索 - ZOZO Technologies TECH BLOG](https://techblog.zozo.com/entry/metric_learning) における分け方との対応は以下のようになると考えています。

|SMORe|ブログ|
|:-|:-|
|Sampler|サンプリング|
|Mapper|ネットワークの構造|
|Optimizer|計量と損失関数|

また、 [KevinMusgrave/pytorch-metric-learning](https://github.com/KevinMusgrave/pytorch-metric-learning) におけるモジュールの分け方は以下の図のようになっています。

![](https://storage.googleapis.com/zenn-user-upload/hsv3i69akyk7lsj4dr3l95ql4yc8)

出典：https://kevinmusgrave.github.io/pytorch-metric-learning/

SMOReのモジュールとこれらの対応は以下のようになると考えています。

|SMORe|pytorch-metric-learning|
|:-|:-|
|Sampler|Sampler （Lossも若干）|
|Mapper|Trainer の中の model（CNNやEmbeddingなど）|
|Optimizer|Loss|

上記のようにモジュールの分け方が似ていることからも、 **協調フィルタリングベースの手法と metric learning はかなり近いもの** と考えられそうです。



ちなみに、 SMORe と pytorch-metric-learning は OSS として公開されており、これらを積極的に活用することで、実装の効率化や、実験の透明化・公平化が期待できます。

- [cnclabs/smore: SMORe: Modularize Graph Embedding for Recommendation](https://github.com/cnclabs/smore/tree/master)
- [KevinMusgrave/pytorch-metric-learning: The easiest way to use deep metric learning in your application. Modular, flexible, and extensible. Written in PyTorch.](https://github.com/KevinMusgrave/pytorch-metric-learning)


# すでに metric learning は使われている

前のセクションでは、既存の推薦手法を metric learning の観点からとらえましたが、  **すでに推薦分野で metric learning として明示的に使われている手法**  もあります。Collaborative Metric Learng などが有名ですが、以下の記事に関連研究がまとまっています。

[Collaborative Metric Learningの関連研究まとめ - Qiita](https://qiita.com/guglilac/items/3b938bcb811245df39a2)

また、コンテンツベースの手法でも明示的に使われています。コンテンツベースの手法は、推薦のコールドスタート問題の解決方法の1つです。以下の2つの論文では、アイテムからアイテムを推薦する問題設定において、アイテムの画像を特徴量として使う手法になります。協調フィルタリングベースの手法では、ユーザーやアイテムのIDの埋め込みベクトルどうしを近づけたり・遠ざけたりしましたが、こちらは **アイテムの画像特徴量どうしを近づけたり・遠ざけたりします** 。

- https://cseweb.ucsd.edu/~jmcauley/pdfs/icdm16b.pdf
- https://cseweb.ucsd.edu/~jmcauley/pdfs/sigir15.pdf


# まとめ

既存の推薦手法を metric learning の観点からとらえてみたり、 すでに推薦分野で metric learning が使われている例を紹介することで、 推薦分野と metric learning に繋がりがあることを説明しました。

両分野の手法において些細な違いはあるものの、その些細な違いをお互いの分野に持ち込むことで、良い影響を与え合えるのではないかと考えています。

# 次回は？

今度はWord2VecやRNNといった自然言語処理のモデルと metric learning の繋がりを説明します。

---

(※1) 本来、評価値（内積）は計量（距離）ではありませんが、 metric learning の定義が明確に定まっておらず、 内積や類似度を使った手法も含めて metric learning と呼ぶ研究もあります。ユークリッド距離を展開するとわかりますが、ユークリッド距離による metric learning は内積を大きく/小さくしています。

(※2) Bayesian と謳ってますが、論文中では MAP 推定をしています。 $P, Q$ の事前分布を多次元正規分布 $\mathcal{N}(\bm{0}, \lambda I)$ と仮定した場合、MAP推定をすると、目的関数に $P, Q$ の L2 正則化項が付きますが、 BPR の構成要素のうち metric learning の観点からとらえたいのは、内積を相対的に最適化する部分なので、余計な正則化項を外すために最尤推定にしました。

(※3) triplet loss の論文では、計量にユークリッドとしていますが、便宜上、内積を計量としています。内積にしても metric learing と読んでよさそうです(※1)。
