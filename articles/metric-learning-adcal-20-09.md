---
title: "自然言語処理のモデルを metric learning の観点からとらえる"
emoji: "📏"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["metriclearning"]
published: false
---

# この記事の目的は？

自然言語処理または metric learning に興味がある人に、 **metric learning と自然言語処理には関係がある** ということを伝えることです。

次のセクションから、いくつかの自然言語処理の手法を metric learning の観点からとらえてみます。


# Word2Vec

Word2Vec は単語の連続的ベクトル表現を獲得する手法です。様々な手法が提案されてますが、この記事では CBoW (Continuous Bag of Words) と Skip-Gram について考えていきます。

## CBoW

CBoW は下図のように文中のターゲットとなる単語 $w_t$ をその周辺語 $w_{t-k}, w_{t-k+1}, \dots , w_{t+k}$ から予測します。

![](https://storage.googleapis.com/zenn-user-upload/8qtjzqksjc3jza3au8jjl79950xz =600x)

CBoW の学習では以下の最適化問題を解きます。

$$
\min_{U, V}{- \sum_{t=1}^T{\log{\frac{\exp{(\bm{v}_{w_t}^T\bm{c}_t})}{\sum_{w \in W}\exp(\bm{v}_w^T\bm{c}_t)}}}}
$$

$$
\bm{c}_t = \sum_{-k \leq j \leq k, j \neq 0}{\bm{u}_{w_{t+j}}}
$$

- $(w_1, w_2, \dots, w_t, \dots, w_T)$: 文（単語の列）
- $\bm{u}_w, \bm{v}_w \in \mathbb{R}^d$: 単語 $w$ の $d$ 次元特徴量ベクトル
- $W$: 学習に使う単語すべての集合
- $U, V \in \mathbb{R}^{d \times |W|}$: それぞれ単語ベクトル $\bm{u}_w, \bm{v}_w$ を並べた重み行列
- $\bm{c}_t$: 文脈ベクトル。単語 $w_t$ の周辺語 $w_{t+j}$ の特徴量を和によって reduce したもの。 周辺語をいくつ使うかは、 ウィンドウサイズ $k$ によって決める。

この式だけでもわかりますが、2回目の記事でも紹介したとおり、Softmax Cross Entropy (SCE) を使っているので、 **CBoW は単語×文脈の metric learning** ととらえられます。 特徴量空間において、 **周辺語による文脈ベクトルとターゲットの単語ベクトルを近づけ、ターゲット以外の単語ベクトルは離す** よう学習します。 下図は metric learning の観点から描きなおしたアーキテクチャです。

![](https://storage.googleapis.com/zenn-user-upload/yexwv23kigzidlqakvff8r2f2o50)

## Skip-Gram

Skip-Gram は下図のとおり、CBoWとは逆に、ターゲットとなる単語 $w_t$ から周辺語 $w_{t-k}, w_{t-k+1}, \dots, w_{t+k}$ を予測します。

![](https://storage.googleapis.com/zenn-user-upload/rdklb4206i1h9c0zp13bampxued5 =600x)

Skip-Gram の学習では以下の最適化問題を解きます。

$$
\min_{U, V}{- \sum_{t=1}^T{\sum_{-k \leq j \leq k, j \neq 0}{\log{\frac{\exp{(\bm{u}_{w_{t+j}}^T\bm{v}_{w_t}})}{\sum_{w \in W}\exp(\bm{u}_w^T\bm{v}_{w_t})}}}}}
$$

上の式より、 **Skip-Gram は単語×単語の metric learing** ととらえられます。特徴量空間において、 **ターゲットの単語ベクトルと周辺語ベクトルを近づけ、周辺語でない単語ベクトルは離す** ように学習します。下図は metric learning の観点から描きなおしたアーキテクチャ図です。

![](https://storage.googleapis.com/zenn-user-upload/n8exsde8iyd3mtovrqwx0j5ghi3x =450x)

以上より、 CBoW と Skip-Gram 、 どちらの Word2Vec も metric learning としてとらえられそうです。

参考

- https://arxiv.org/pdf/1301.3781v3.pdf
- https://proceedings.neurips.cc/paper/2013/file/9aa42b31882ec039965f3c4923ce901b-Paper.pdf
- [絵で理解するWord2vecの仕組み - Qiita](https://qiita.com/Hironsan/items/11b388575a058dc8a46a)
- [単語と図で理解する自然言語処理（word2vec, RNN, LSTM）前編 - ギークなエンジニアを目指す男](https://www.takapy.work/entry/2019/01/06/203042#%E6%9C%80%E5%BE%8C%E3%81%AB)


# RNN による言語モデル

RNN による言語モデル（RNNLM）は、 下図のように直前の単語列 $w_1, w_2, \dots, w_t$ から、次の単語 $w_{t+1}$ を予測する手法です。

![](https://storage.googleapis.com/zenn-user-upload/za2mmlleb8sohp49tnbq22zlo3n9)

RNNLM の学習では、以下の最適化問題を解きます。

$$
\min_{U, \Theta, V}{
  - \sum_{t=1}^{T-1}{
    \log{
      \frac{
        \exp{(
          \bm{v}_{w_{t+1}} ^T \bm{c}_t
        )}
      }{
        \sum_{w \in W}{
          \exp{(
            \bm{v}_w ^T \bm{c}_t
          )}
        }
      }
    }
  }
}
$$

$$
\bm{c}_t = {\rm RNN}(\bm{u}_{w_t}, \dots, \bm{u}_{w_2}, \bm{u}_{w_1}; \Theta)
$$

- $\bm{c}_t$: 文脈ベクトル。RNN の隠れ層の状態。$t$ 番目までの単語ベクトルたちを RNN で reduce したもの。
- $\Theta$: RNN のパラメータ

上の式より、CBoWと同様に **RNNLM は単語×文脈の metric learning** ととらえられます。 CBoW との違いは、文脈を構成する単語と、それらを reduce する関数です。その違いを以下の表にまとめました。

||CBoW|RNNLM|
|:-|:-|:-|
|文脈を構成する単語|予測対象の両側の単語いくつか|予測対象よりも前の単語すべて|
|reduce する関数|sum（学習不可能）|RNN（学習可能）|

また、下図は RNNLM を metric learning の観点から描きなおしたものになります。

![](https://storage.googleapis.com/zenn-user-upload/7ffsbl1lvdlsxfn5j8k89yhvri4c)

参考

- https://www.fit.vutbr.cz/research/groups/speech/publi/2010/mikolov_interspeech2010_IS100722.pdf

# まとめ

既存の自然言語処理のモデルを metric learning の観点からとらえてみることで、 自然言語処理処理と metric learning に繋がりがあることを説明しました。

ちなみに、トピックモデルである LSI も文書×単語の共起行列を SVD で行列分解するので、 metric learning としてとらえられないか考えているところです（埋め込み写像を、回転行列と対角行列に分解できるように正則化をかけたらできるのだろうか？）。

# 次回は？

推薦や自然言語処理に加えて、他にも metric learning と関係のある分野をいくつか紹介します。

