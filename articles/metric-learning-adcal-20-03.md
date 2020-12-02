---
title: "metric learningとsoftmax cross entropyを比べること自体間違っている"
emoji: "📏"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["metriclearning"]
published: true
---


# この記事の目的は？

「 **metric learning と softmax cross entropy (SCE) どっちを使えばいいんだろう〜？** 」と考えてる人に、「 **そもそも metric learning と SCE は比べるものではない** 」ということを伝えることです。

# なぜ比べるのは間違いか？

**SCE は metric learning に包含されるから** です。並列の関係ではないので比べられません。

実は SCE は暗黙的に metric learning をしています。それを次のセクションで解説します。

# なぜ SCE が metric learning なのか？

SCE は以下のように表せます(※1)。

$$
L_{\rm SCE} = - \sum_{i=1}^B \sum_{j=1}^B q_{(i,j)} \log{ P(y_i=y_j) }
$$

- $B$: バッチサイズ
- $i, j$: 1枚の画像など、バッチ内のサンプル(※2)の添字
- $y_i \in \{1, ..., C\}$: サンプル $i$ のクラス
- $q(i, j) \in \{0, 1\}$: サンプル $i$ のクラスがサンプル $j$ のクラスと同じなら1、そうでないなら0のラベル
- $P(y_i = y_j)$: サンプル $i$ のクラスがサンプル $j$ のクラスと同じ確率

$q(i,j)$ は、サンプル $i$ と $j$ のクラスが異なるとき0になるので、クラスが同じ項だけ残り、以下のようになります。

$$
L_{\rm SCE} = - \sum_{i=1}^B \sum_{j;y_j=y_i} \log{ P(y_i=y_j) }
$$

確率 $P$ は Softmax と内積で表せるので以下のように変形できます。

$$
L_{\rm SCE} = - \sum_{i=1}^B \sum_{j;y_j=y_i} \log{\frac{\exp{\bm{z}_i^T\bm{z}_j}}{\sum_{k=1}^B\exp{\bm{z}_i^T\bm{z}_k}}}
$$

- $\bm{z}_i \in \mathbb{R}^d$: サンプル $i$ の $d$ 次元特徴量ベクトル

さらに、2つの項に分解できます。

$$
L_{\rm SCE}= \sum_{i=1}^B \left\{ - \sum_{j;y_j=y_i} \bm{z}_i^T\bm{z}_j + \log{ \sum_{k=1}^B \exp{\bm{z}_i^T\bm{z}_k} } \right\}\\
 = \sum_{i=1}^B \left\{ - \sum_{j;y_j=y_i} \bm{z}_i^T\bm{z}_j + {\rm LogSumExp}_{k=1}^B (\bm{z}_i^T\bm{z}_k) \right\}
$$

![](https://storage.googleapis.com/zenn-user-upload/qqm0f2otm0jqynu079fvlrawwhu4 =350x)

図1：SCEのどの項がサンプル間にどうはたらくか
丸と五角形は特徴量空間におけるサンプルの位置。図形と色が同じなら同じクラス、異なるなら異なるクラス。学習によって矢印の向きに引っ張られる。

この SCE を最小化しようとすると、 1項目の同じクラス間の内積 $\bm{z}_i^T\bm{z}_j$ を大きくしようとします。 一方、 2項目の ${\rm LogSumExp}_{k=1}^B (\bm{z}_i^T\bm{z}_k)$ を小さくしようとします。 LogSumExp は max 関数のなめらかな近似であり、異なるクラス間の内積 $\bm{z}_i^T\bm{z}_k(y_i \neq y_k)$ が、同じクラス間の内積 $\bm{z}_i^T\bm{z}_k(y_i=y_k)$ より大きければ、小さくしようとします。

つまり、特徴量空間において、 図1のように **同じクラスのサンプルを1項目が近づけ、 異なるクラスのサンプルを2項目が離す** ような役割を果たしており、 SCE は暗黙的に metric learning をしていたわけです。

上記の解説は以下のブログ記事から抜粋しました(※3)。

[deep metric learningによるcross-domain画像検索 - ZOZO Technologies TECH BLOG](https://techblog.zozo.com/entry/metric_learning)

また、以下の論文では情報理論の観点から丁寧に証明されています。

[(M. Boudiaf et al., 2020, ECCV) A unifying mutual information view of metric learning: cross-entropy vs. pairwise losses](http://arxiv.org/abs/2003.08983) 

ところで、上記の SCE の式を見て、ロジットの計算方法に違和感を覚えられたかもしれません。実は、上記の SCE の使い方は、画像分類でよく見られる使い方とは若干異なります。次のセクションでは、その2つの違いを説明します。

# SCE の2つの使い方の違い

まず、 **(i)前のセクションで紹介した SCE の使い方** は以下の式のことでした。

$$
- \sum_{i=1}^B \sum_{j;y_j=y_i} \log{\frac{\exp{\bm{z}_i^T\bm{z}_j}}{\sum_{k=1}^B\exp{\bm{z}_i^T\bm{z}_k}}}
$$

一方、**(ii)画像分類でよく見られる使い方** とは以下の式のこととします(※1)。

$$
- \sum_{i=1}^B \log{\frac{\exp{\bm{z}_i^T\bm{\theta}_{y_i}}}{\sum_{y=1}^C\exp{\bm{z}_i^T\bm{\theta}_y}}}
$$

$\bm{z}$ が片方だけ $\bm{\theta}$ になり、$\sum$ が1つ外れました。$\bm{\theta_y}$ は最終FC層の重み $\Theta$ のうち、クラス $y$ に対応する重みベクトル（下の図2の緑の部分）です。 $\sum$ が外れる理由は、 サンプル $i$ が所属クラスは1つだけで、1項しか残らないからです。

![](https://storage.googleapis.com/zenn-user-upload/auylyq2r4rjmhjhmvv6di1dv288i =500x)

図2：(ii)のアーキテクチャと重みベクトル $\bm{\theta_y}$ の位置

「これでは (ii) は metric learning ではないのでは？」と思われるかもしれませんが、 (ii) も metric learning です。その理由を説明します。まず、図2のアーキテクチャは下の図3のようにも表せます。

![](https://storage.googleapis.com/zenn-user-upload/5h1u5ga71nv37vnm2uveplf7i5v6 =450x)

図3：(ii)と等価なアーキテクチャ

図3のアーキテクチャは、図2のアーキテクチャの最終FC層を、クラスIDの埋め込みとみなしたものです。あるサンプルの特徴量と、そのサンプルが属するクラスの埋め込みベクトルを近づけ、そうでないクラスを離すので、(ii)も metric learning です。

![](https://storage.googleapis.com/zenn-user-upload/ahzxl7abl4ws84ebvwy45dbnyafa =450x)

図4：(i)のアーキテクチャ

入力データのドメインが、 **(i)は画像×画像** 、 **(ii)は画像×クラス** と異なっていますが、どちらも同じ最適化問題を解いています。その証明は以下の論文でされています。論文中のPairwise Cross-Entropy (PCE)が(i)に、 Cross-Entropy (CE)が(ii)に相当します。

[(M. Boudiaf et al., 2020, ECCV) A unifying mutual information view of metric learning: cross-entropy vs. pairwise losses](http://arxiv.org/abs/2003.08983) 

それぞれの呼び方ですが、この記事では、以降、 **(i) を Pairwise Softmax Cross-Entropy (PSCE)** 、 **(ii) を Classification Softmax Cross-Entropy (CSCE)** と呼ぶことにします。

# PSCE と CSCE のどちらを使えばいいか

どちらを使えばいいかは状況によって変わります。

顔認識の研究では **CSCE ベースの手法 が話題** のようです。

- [モダンな深層距離学習 (deep metric learning) 手法: SphereFace, CosFace, ArcFace - Qiita](https://qiita.com/yu4u/items/078054dfb5592cbb80cc)
- [Softmax関数をベースにした Deep Metric Learning が上手くいく理由 - Qiita](https://qiita.com/tancoro/items/7ed5661488ab8cf340c7#uniform-loss)

また、 metric learning の研究では proxy 系の手法が話題ですが、 proxy とはクラスの重みベクトル $\bm{\theta}_y$ のようなものなので、 CSCE と近い手法と考えられます。

[CVPR2020読み会 Proxy Anchor Loss for Deep Metric Learning - Speaker Deck](https://speakerdeck.com/satokeiju/cvpr2020du-mihui-proxy-anchor-loss-for-deep-metric-learning)


こう見ると、 CSCE の方が良さそうですが、以下の論文の付録（On the limitations of cross-entropy）では **「相対的なラベルしかないとき」と「クラス数が多すぎるとき」 は PSCE** を推奨しています。相対的なラベルしかないというのは、データセット内の各サンプルのクラスが明確に定義されておらず、どの画像どうしが関係してるか・してないかだけが与えられてる状況です。また、クラス数が多すぎると、クラスベクトルの重み $\Theta$ のメモリ容量を確保するのが大変だからです。

[(M. Boudiaf et al., 2020, ECCV) A unifying mutual information view of metric learning: cross-entropy vs. pairwise losses](http://arxiv.org/abs/2003.08983) 

また、以下の論文では、pairwise （論文中だと instance-to-instance）と classification （論文中だと instance-to-class, instance-to-proxy）についてまとめられており、 この論文では pairwise の方を支持しています。

[(X. Wang et al., CVPR) Ranked List Loss for Deep Metric Learning](https://arxiv.org/abs/1903.03238) 

ちなみに [N-pair loss](https://papers.nips.cc/paper/2016/hash/6b180037abbebea991d8b1232f8a8ca9-Abstract.html) はSCEを使っており、入力データが画像×画像なので、 PSCEベースの手法です。

また、 PSCE と CSCE のどちらか2つに絞らずに、以下の方法などを試してみるのも面白いかもしれません。

- PSCE と CSCE 両方使う（同じ最適化問題を解いてるので変わらなそうですが）
- SCE に限らず、 ranking loss など、他の metric leaning の手法の classification (画像×ラベル) 版を考える（[このブログ記事](https://techblog.zozo.com/entry/metric_learning)では、 PSCE より ranking loss (記事中では N-pair hinge loss) の方が良い実験結果が報告されており、データによって適切な loss は変わりそう）
- [P2Grad](https://qiita.com/tancoro/items/7ed5661488ab8cf340c7#p2sgrad) のように loss を定義しない手法を使う


# SCE が metric learning だと何が嬉しいか？

SCE が metric learing だと嬉しいことは、 SCE を使えば良い、ということではなく、 **SCE を使っている他の分野に metric learning の知見を活かせる** ことだと思っています。例えば、SCEを使っている **Word2VecやRNNによる言語モデルも metric learning とみなせる** ので、metric learning の知見を自然言語処理(NLP)に活かせそうです。

後続の記事で、 **NLPや推薦のモデルを metric learning の観点からとらえる** 予定です。 異なる分野を metric learning という観点で繋げることで、それぞれの分野の知見を互いに流用できるかもしれません。

# まとめ

SCE の使い方として PSCE (画像×画像) と CSCE（画像×クラス）の2つが存在し、両方とも metric learning であるため、 SCE と metric learning を比較するのは間違っているということを紹介しました。

# 次回は？

推薦モデルを metric learning の観点からとらえることで、推薦分野と metric learning の繋がりを説明したいと思います。

---
(※1) バッチサイズの逆数によるスケーリングは本質ではないので省略しました。

(※2) この記事での「サンプル」は、画像1枚など、データのうちの1点を指します。標本点や結果(outcome)と表現するのが正確ですが、一般的ではなく想定読者にとってわかりづらいように思えたので「サンプル」と表現しました。

(※3) わかりやすいよう表記を多少変えています。
