---
title: "metric learning と関係のある分野まとめ"
emoji: "📏"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["metriclearning"]
published: true 
---

:::message
こちらは [metric learning Advent Calendar 2020](https://adventar.org/calendars/5596) の記事です。
:::

# この記事の目的は？

以下の分野と metric learing の繋がりをざっくり説明します。

- fine-grained image classification
- 顔認識
- few-shot learning
- マルチモーダル学習
- 異常検知
- learning to rank
- カーネル法
- 自己教師あり学習
- 推薦
- 自然言語処理

# fine-grained image classification

metric learning 手法が主に提案される分野は fine-grained image classification だと思います。 fine-grained image classification とは、車や犬というように大雑把に分類するのではなく、車種や鳥の種類など細かいクラスに分類する問題です。このように、 **クラス数が多く、1クラスあたりのサンプル数が少ない** 分類問題に metric learning は適しています。

metric learning のサーベイは、 以下の GitHub リポジトリが参考になると思います。

https://github.com/kdhht2334/Survey_of_Deep_Metric_Learning


また、実装は以下のライブラリにまとまっています。

https://github.com/KevinMusgrave/pytorch-metric-learning

ちなみに、extreme classification という似たような問題があります。こちらはマルチラベルの（1つの画像が複数のクラスに所属する）場合の話が多い気がしますが、以下の Ranked List Loss の論文では、 instance-to-instance な metric learning は extreme classification において良い解決策だと主張しています。

https://arxiv.org/pdf/1903.03238.pdf

extreme classification の概要については以下のブログ記事にまとまっています。

[数えきれないほどの分類を行うExtreme Classification - Technical Hedgehog](https://kamujun.hatenablog.com/entry/2018/06/22/180605)


# 顔認識

fine-grained image classification の他に、metric learning の手法がよく提案されるのは顔認識の分野だと思います。 人物をクラスと考えると、クラス数が多く、1クラスあたりのサンプル数が少ない状況なので、 fine-grained image classification の一例と考えられます。

![](https://storage.googleapis.com/zenn-user-upload/2wbunnrt5bzfhhk8t4qp4n799n6w)

出典：https://arxiv.org/abs/1704.08063

ちなみに顔認識の分野では Softmax Cross-Entropy ベースの手法が話題になっているようです。

- [モダンな深層距離学習 (deep metric learning) 手法: SphereFace, CosFace, ArcFace - Qiita](https://qiita.com/yu4u/items/078054dfb5592cbb80cc)
- [Softmax関数をベースにした Deep Metric Learning が上手くいく理由 - Qiita](https://qiita.com/tancoro/items/7ed5661488ab8cf340c7)


# few-shot learning

**1クラスあたりのサンプル数が少ない** といった意味では、 few-shot learning も関連しています。実際に metric learning ベースの手法が提案されています。

参考

- https://github.com/Bryce1010/Awesome-Few-shot


# マルチモーダル学習

マルチモーダル学習とは、画像とテキスト、画像と音など、異なるドメインのデータ間の関係を学習する手法です[^1]。例えば、 visual-semantic embeddings という分野では対応する画像とテキストの特徴量が近くなるように学習します。 [この論文](https://arxiv.org/pdf/1411.2539.pdf) では以下のような損失関数で学習します[^2]。

[^1]: 分布の多峰性のこともマルチモーダルと言いますが、それとは別の意味です。

[^2]: わかりやすくするため、表記を少し変えています。


$$
\min_{\Theta}{
  \sum_{i=1}^B{
    \sum_{j=1, j \neq i}^B{\Bigl\{
      \max{(0, m - s(\bm{x}_i, \bm{v}_i) + s(\bm{x}_i, \bm{v}_j)})
      + \max{(0, m - s(\bm{v}_i, \bm{x}_i) + s(\bm{v}_i, \bm{x}_j)})
    \Bigr\}}
  }
}
$$

- $B$: ミニバッチサイズ
- $\bm{x_i}$: ミニバッチ内で $i$ 番目の画像特徴量
- $\bm{v_i}$: 文ベクトル（$i$ 番目の画像に対応する文を LSTM で埋め込んだもの）
- $s$: コサイン類似度
- $\Theta$: モデルのパラメータ（画像特徴量を抽出するCNN、LSTM、など）
- $m$: マージンという正の実数のハイパーパラメータ

![](https://storage.googleapis.com/zenn-user-upload/55kqclw3h253wbngpqphspbtn9xo)

対応する画像と文のペアの類似度を、対応しないペアの類似度より $m$ だけ差をつけて大きくしようとしており、metric learning の手法である [triplet loss](https://arxiv.org/pdf/1404.4661.pdf) に似ています。

このようにマルチモーダル学習でも metric learning が使われています。


# 異常検知

あまり調べられてないのですが、異常検知でも metric learning が使われているようです。

- [深層距離学習(Deep Metric Learning)各手法の定量評価 (MNIST/CIFAR10・異常検知) - Qiita](https://qiita.com/daisukelab/items/5f9ec189959eaf1ef211)
- [【まとめ】ディープラーニングを使った異常検知 - Qiita](https://qiita.com/shinmura0/items/06d81c72601c7578c6d3)


# learning to rank

learning to rank とは、 主に検索のための学習方法で、 あるクエリに対する検索結果の適切な順位を学習します。詳細は以下のアドベントカレンダーが参考になると思います。

[ランク学習（Learning to Rank） Advent Calendar 2018 - Adventar](https://adventar.org/calendars/3357)

ポイントワイズとペアワイズの損失関数は、 metric learning の損失関数に近いと思います。

ポイントワイズの一例[^3]:

[^3]: 式は以下の[こちらのブログ記事](https://www.szdrblog.info/entry/2018/12/04/010611)を参考にさせていただきました。

$$
\sum_{q,d}\bigl\{{y_{q,d} - f(\bm{x}_{q,d})}\bigr\}^2
$$

- $q$: クエリ（単語など）
- $d$: 検索結果の1項目となる文書
- $y_{q,d}$: クエリ $q$ と文書 $d$ の関連度（教師ラベル。正の整数など。）
- $\bm{x}_{q,d}$: クエリ $q$ と文書 $d$ のなんらかの特徴量
- $f(\cdot)$: 関連度を予測する関数

ペアワイズの一例(※3):

$$
\sum_{q,i,j; y_{q,i} > y_{q,j}}\exp{(f(\bm{x}_{q,j}) - f(\bm{x}_{q,i}))}
$$

- $i, j$: 文書

どちらも関連度を類似度ととらえれば、それぞれ metric learning の contrastive, triplet 系の損失関数に似ています。

また、 metric learning の手法で learning to rank のしくみを取り入れた手法も提案されています。 learning to rank の評価指標である Average Precision の近似を損失関数とした FastAP という手法です。

https://openaccess.thecvf.com/content_CVPR_2019/papers/Cakir_Deep_Metric_Learning_to_Rank_CVPR_2019_paper.pdf




# カーネル法

metric learning はカーネル法と等価らしいです。詳細は以下のブログ記事が参考になると思います。

[距離計量学習とカーネル学習について - a lonely miner](http://conditional.github.io/blog/2013/04/20/distance-metric-learning-and-kernel-learning/)


# 自己教師あり学習

自己教師あり学習もかなり近いことをやっているようです。

![](https://qiita-user-contents.imgix.net/https%3A%2F%2Fimgur.com%2FJhoPLVG.png?ixlib=rb-1.2.2&auto=format&gif-q=60&q=75&w=1400&fit=max&s=c2d07d10ff6b5d3518bdcc60642a9299 =300x)

![](https://qiita-user-contents.imgix.net/https%3A%2F%2Fi.imgur.com%2F7EFjxSx.png?ixlib=rb-1.2.2&auto=format&gif-q=60&q=75&w=1400&fit=max&s=aa7ec5d50019baea055c21587342fbea =800x)

出典・参考：[2020年超盛り上がり！自己教師あり学習の最前線まとめ！ - Qiita](https://qiita.com/omiita/items/a7429ec42e4eef4b6a4d)

負例の獲得方法を工夫した4つアーキテクチャなどは、自己教師あり学習だけに限らず、他の分野でも活かせそうです。

# 推薦

[推薦モデルを metric leaning の観点からとらえる](https://zenn.dev/hrsma2i/articles/metric-learning-adcal-20-07) で、 MF (Matrix Factrization) や BPR (Bayesian Personalized Ranking) が metric learning の手法に似ていることや、 metric learning を使った推薦手法を紹介しました。


# 自然言語処理

[自然言語処理のモデルを metric learning の観点からとらえる](https://zenn.dev/hrsma2i/articles/metric-learning-adcal-20-09) で、 Word2Vec や RNN による言語モデル (RNNLM) が metric learning であることを説明しました。


# まとめ

metric learning と関係のある分野をリストアップしました。このように多くの分野と関係がありますが、 metric learning すごい！というより、 metric learning によって各分野の繋がりが強くなり、 **ある分野の知見を他の分野でも活かしやすくなる** ということが metric learning の面白さかなと思っています。

例えば、 Word2Vec や RNNLM の負例には、他の文では正例である単語が来る可能性もあります（「I have a pen」「I have a dog」という文は両方ともありえますが、前者において「dog」、後者において「pen」は負例になってしまいます）。 このように正例になる可能性がある負例は、推薦分野における _implicit feedback_ の負例に近いと考えています。そこで Word2Vec や RNNLM の Softmax Cross-Entropy の代わりに BPR や triplet loss などを使ってみても面白いのかなと考えています。

# 次回は？

次回は、 metric learning の既存手法がそれぞれ **「ミニバッチ内の情報をどくらい活用できているか」をグラム行列の観点から可視化** してみます。
