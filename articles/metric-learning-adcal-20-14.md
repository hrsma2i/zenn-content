---
title: "metric leraning のバッチ活用具合をグラム行列の視点から可視化する"
emoji: "📏"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["metriclearning"]
published: true 
---

:::message
こちらは [metric learning Advent Calendar 2020](https://adventar.org/calendars/5596) の記事です。
:::

# この記事の目的は？

metric learning に興味のある方に、既存手法が「どれくらいバッチを活用できているか」をグラム行列の視点から可視化する方法を紹介します。

contrastive loss や triplet loss より N-pair loss の方がバッチ内の情報を活用できることはご存知かと思いますが、そのようなことを直感的に理解するための方法を紹介します。


# グラム行列とは

一言で言うなら、 **内積の行列** です。

以下のような特徴量 $\bm{x}_i \in \mathbb{R}^d$ を並べた $d \times B$ 行列があるとします。この行列を、この記事ではミニバッチとみなします。

$$
X = [\bm{x}_1, \bm{x}_2, \dots, \bm{x}_B]
$$

グラム行列とは、以下の $B \times B$ 行列のことです。

$$
X^TX =  \left[
  \begin{array}{ccc}
    \bm{x}_1^T\bm{x}_1 & \bm{x}_1^T\bm{x}_2 & \dots & \bm{x}_1^T\bm{x}_B\\
    \bm{x}_2^T\bm{x}_1 & \bm{x}_2^T\bm{x}_2 & \dots & \bm{x}_2^T\bm{x}_B\\
    \vdots & \vdots & \ddots & \vdots\\
    \bm{x}_B^T\bm{x}_1 & \bm{x}_B^T\bm{x}_2 & \dots & \bm{x}_B^T\bm{x}_B\\
  \end{array}
\right]
$$

各要素は、各特徴量の組の内積になります。

内積ではなくユークリッド距離を要素にした行列でも、これから紹介する方法の本質は変わらないので、以降、簡単のため、すべて内積で説明します。

# contrastive loss

まず、 contrastive loss のバッチ活用具合を可視化します。 contrastive loss の損失関数は以下です[^1]。

[^1]: わかりやすさのため、本質でない部分を削ぎ落としたり、表記を変えたり、若干の変更を加えています。

$$
\sum_{k=1}^{B_{\rm con}}{
  (1-q_{k}) \bm{x}_k^T\bm{x}^\prime_k
  - q_{k} \bm{x}_k^T\bm{x}^\prime_k
}
$$

- $B_{\rm con}$: バッチサイズ
- $(\bm{x}_k, \bm{x}^\prime_k)$: バッチ内で $k$ 番目の、特徴量のペア
- $q_k \in \{0, 1\}$: $k$ 番目のペアがポジティブ（同じクラスなど）なら1、ネガティブ（異なるクラスなど）なら0をとるラベル

上の損失関数を最小化すると、ポジティブペアの内積を大きく、ネガティブペアの内積を小さくします。

contrastive loss のバッチは下図のような構成になります。

![](https://storage.googleapis.com/zenn-user-upload/cx1wmxkko5ifh2cf8yi02kbdnept =310x)

起点となる特徴量 $\bm{x}$ を **アンカー** 、対になる特徴量 $\bm{x}^\prime$ を **ポジティブサンプル** 、 **ネガティブサンプル** と呼ぶことにします。

今、このバッチ内の特徴量を1列に並べた新しいバッチを考えます。

![](https://storage.googleapis.com/zenn-user-upload/enn7yb39xyfqafe9nm18b874vp1p =100x)

ここで、新しいバッチのグラム行列を考えると、損失関数に反映される内積は、下図の赤と青の要素だけであることがわかります。バッチ内から得られるはずの特徴量の組合せ（15通り）のうち、ほんの一部（3通り）しか活用できていません。

![](https://storage.googleapis.com/zenn-user-upload/wubxbtwmwkhlti7dyeyvjsksavn5 =450x)

- 赤い要素：ポジティブペアの内積
- 青い要素：ネガティブペアの内積
- 太い黒枠：元のバッチでのペアの関係
- 灰色の要素：考慮しない（対角は自分自身との内積なのと、グラム行列は対称行列なため）


参考

http://yann.lecun.com/exdb/publis/pdf/hadsell-chopra-lecun-06.pdf

# triplet loss

triplet loss の損失関数は以下です[^1]。

$$
\sum_{k=1}^{B_{\rm tri}}{
  \max{(0, 
    - \bm{x}_{a_k} ^T \bm{x}_{p_k}
    + \bm{x}_{a_k} ^T \bm{x}_{n_k}
    + m
  )}
}
$$

- $B_{\rm tri}$: バッチサイズ
- $(\bm{x}_{a_k}, \bm{x}_{p_k}, \bm{x}_{n_k})$: バッチ内で $k$ 番目の、特徴量のトリプレット。左から順にアンカー、ポジティブサンプル、ネガティブサンプル。
- $m \in \mathbb{R}_{>0}$: マージン（ハイパーパラメータ）

上の損失関数を最小化すると、 ポジティブの内積がネガティブより $m$ だけ差をつけて大きくなる（ $\bm{x}_{a_k} ^T \bm{x}_{p_k} \geq \bm{x}_{a_k} ^T \bm{x}_{n_k} + m$ を満たす）ように学習します。

triplet loss のバッチは下図のような構成になります。

![](https://storage.googleapis.com/zenn-user-upload/xp5fulsft20f294wonfvm1lkg2fv =310x)


今、このバッチ内の特徴量を1列に並べた新しいバッチを考えます。


![](https://storage.googleapis.com/zenn-user-upload/jsy6ic82b9w7wo1n6m3t63y1a7eu =100x)

ここで、新しいバッチのグラム行列を考えると、4/15通りしか損失関数に反映されません。

![](https://storage.googleapis.com/zenn-user-upload/ofgu3ugo84m0d06kg2obl1ucw5z0 =450x)

- 太い黒枠：元のバッチでのトリプレットの関係

さらに、 triplet loss ではポジティブペアとネガティブペアの内積の組合せも考える必要があります。 なぜなら、 triplet loss の各項は、 ポジティブとネガティブの内積の差がヒンジ関数（ $max(0, \cdot)$ ）によって囲まれているため、 ポジティブとネガティブに切り分けられないからです。

下図の右側がポジティブとネガティブの内積の組合せを表しており、白い要素の組合せ、つまり2/4通りしか損失関数に反映されません。

![](https://storage.googleapis.com/zenn-user-upload/yt556a28h82kvjzscc40sl0bqvmb =300x)

参考

- [Jiang Wang, et al. "Learning Fine-grained Image Similarity with Deep Ranking" CVPR, 2014](https://arxiv.org/pdf/1404.4661.pdf)


# N-pair loss

N-pair loss の損失関数は以下です[^1]。

$$
- \sum_{k=1}^{B_{\rm Npair}}{
  \log{
    \frac{
      \exp{
        ( \bm{x}_k ^T \bm{x}^\prime_k)
      }
    }{
      \sum_{l=1}^{B_{\rm Npair}}{
        \exp{
          ( \bm{x}_k ^T \bm{x}^\prime_l)
        }
      }
    }
  }
}
$$


N-pair loss のバッチは下図のような構成になります。何番目をアンカーとみなすかによって、他のサンプルのポジティブ、ネガティブが変わります。

![](https://storage.googleapis.com/zenn-user-upload/cehoa7sg7eo5o33pcrha6ecwj09e =300x)

このバッチを1列に変形したときのグラム行列を考えると、下図の右端のように9/15通りが損失関数に反映されます。

![](https://storage.googleapis.com/zenn-user-upload/eby09nfbykhk2sa9mlzvswn6xo4k =550x)

ちなみに、 N-pair loss では内積の組合せを考える必要はありません。なぜなら、以下のように式変形でき、 各項を「ポジティブな内積を大きくするための項（左）」と「ネガティブな内積を小さくするための項（右）」に分解できるからです。

$$
\sum_{k=1}^{B_{\rm Npair}}{\Biggl\{
  - \bm{x}_k ^T \bm{x}^\prime_k
  +
  \log{
      \sum_{l=1}^{B_{\rm Npair}}{
        \exp{
          ( \bm{x}_k ^T \bm{x}^\prime_l)
        }
      }
  }
\Biggr\}}
$$


参考

https://papers.nips.cc/paper/2016/file/6b180037abbebea991d8b1232f8a8ca9-Paper.pdf

# lifted structured loss

lifted structured loss の損失関数は以下です(※1)。

$$
\sum_{k=1}^{B_{\rm LS}}{
  \max{(0, 
    \max{(
      \max_{l=1;l \neq k}^{B_{\rm LS}}{(m + \bm{x}_k ^T \bm{x}^\prime_l)},
      \max_{l=1;l \neq k}^{B_{\rm LS}}{(m + \bm{x}_l ^T \bm{x}^\prime_k)}
    )}
    - \bm{x}_k ^T \bm{x}^\prime_k
  )}
}
$$

バッチは下図のような構成になり、どのサンプルをアンカーとみなすかによって、他のサンプルがポジティブなのかネガティブなのかが決まります。

![](https://storage.googleapis.com/zenn-user-upload/opdgk6v3uwm9tng7n9smktxbto5d =500x)

このバッチを1列に変形したとき、グラム行列の各要素は下図のように計算されます。

![](https://storage.googleapis.com/zenn-user-upload/rz6apoisidnma9oihcitroqdcuyp)

上の6つを合わせると、下図のように、すべての組合せの情報が損失関数に反映されていることがわかります。

![](https://storage.googleapis.com/zenn-user-upload/vtt26mvqbvbqaiufn0psxrjne7nu =150x)

また、 lifted structured loss もポジティブとネガティブの内積の組合せを考える必要があります。その理由を説明するために、まずは上の損失関数の式を、もう少しわかりやすい形に変形します。上の損失関数を以下に再掲します。

$$
\sum_{k=1}^{B_{\rm LS}}{
  \max{(0, 
    \max{(
      \max_{l=1;l \neq k}^{B_{\rm LS}}{(m + \bm{x}_k ^T \bm{x}^\prime_l)},
      \max_{l=1;l \neq k}^{B_{\rm LS}}{(m + \bm{x}_l ^T \bm{x}^\prime_k)}
    )}
    - \bm{x}_k ^T \bm{x}^\prime_k
  )}
}
$$

バッチを1列に変形すると、以下のように表せます。

$$
\sum_{i=1}^{B}{
  \max{(0, 
    \max_{n \in \mathcal{N}_i}{(m + \bm{x}_i ^T \bm{x}_n)}
    - \bm{x}_i ^T \bm{x}_{p_i}
  )}
}
$$

- $B$: 1列に変形した後のバッチサイズ（上の図の例だと、 $B_{\rm LS}=3, B=6$）
- $\mathcal{N}_i$: サンプル $i$ をアンカーとしたときの、バッチ内のネガティブサンプルすべての集合
- $p_i$: サンプル $i$ をアンカーとしたときの、バッチ内のポジティブサンプル

ただし、特徴量を $\bm{x}, \bm{x}^\prime$ に分けず、 $\bm{x}$ に統一しました。

さらに、加算は $\max$ に対して分配則が成り立つので、ポジティブな内積を $\max$ の中に入れます。

$$
\sum_{i=1}^{B}{
  \max{(0, 
    \max_{n \in \mathcal{N}_i}{(
      m
      + \bm{x}_i ^T \bm{x}_n
      - \bm{x}_i ^T \bm{x}_{p_i}
    )}
  )}
}
$$

さらに $\max$ を入れ替えます。

$$
\sum_{i=1}^{B}{
  \max_{n \in \mathcal{N}_i}{\Biggl\{
    \max{(0, 
      m
      + \bm{x}_i ^T \bm{x}_n
      - \bm{x}_i ^T \bm{x}_{p_i}
    )}
  \Biggr\}}
}
$$

すると、 triplet loss とかなり近いことをやっているのがわかります。 triplet loss 同様に内積の差がヒンジ関数で囲われてるので、内積の組合せを考える必要があるわけです。

そして、損失関数に反映される内積の組合せは下図の右側の行列の白い要素（24/36通り）となり、すべての組が反映されるわけではありません。

![](https://storage.googleapis.com/zenn-user-upload/t8rj6cvdoq5ek3kywzekkpwr29p2 =700x)

参考

https://arxiv.org/pdf/1511.06452.pdf 

# quadruplet loss

lifted structured loss では、下図のように内積の組合せすべてが損失関数に反映されるわけではありませんでした（白い組合せのみ）。

![](https://storage.googleapis.com/zenn-user-upload/k2g4kf9duithqarugtsfpubwqjxe =300x)

損失関数に反映されない灰色の組合せは、内積のアンカーが異なり、4つのサンプルから構成されます（他は3つ）。例えば、上の図の灰色の組 $(p_1, n_5)$ は以下のように4つのサンプルから構成されます。

$$
p_1 = \bm{x}_1^T\bm{x}_4
$$
$$
n_5 = \bm{x}_2^T\bm{x}_3
$$

この灰色の組が損失関数に反映されないと下図のように、クラスB-C間の距離より、クラスA内の距離のほうが遠くなるといった問題が起きる可能性があります。

![](https://storage.googleapis.com/zenn-user-upload/xm2icugj1t4t2d3qj0dgxtqgy5s9 =400x)

出典：https://qiita.com/tancoro/items/35d0925de74f21bfff14#quadruplet-loss

そこで、下図のように、すべての内積の組を反映させたものが [quadruplet loss](https://arxiv.org/pdf/2002.11644.pdf) になります。

![](https://storage.googleapis.com/zenn-user-upload/975jvi7u6ajzyfkqdrh6fry35kck =300x)

# pytorch-metric-learning との関係

「グラム行列のどの要素を損失関数に反映させるか」は、 [pytorch-metric-learning](https://github.com/KevinMusgrave/pytorch-metric-learning) の「labels」にあたります。グラム行列は「dist_mat (distance matrix)」に相当します。

![](https://storage.googleapis.com/zenn-user-upload/hsv3i69akyk7lsj4dr3l95ql4yc8)

出典：https://kevinmusgrave.github.io/pytorch-metric-learning/

# まとめ

metric learning の手法のバッチ活用具合を、グラム行列の視点から可視化する方法を紹介しました。また、ヒンジ関数などによって内積の差が囲まれている場合は、どの内積の組合せが損失関数に反映されるかも考える必要がありました。

以下の表に、紹介した各手法の可視化結果をまとめました。「グラム行列」の図では、赤と青の要素が多いほどバッチを活用しており、「内積の組」の図では、白い要素が多いほどバッチを活用しています。

|手法|グラム行列|内積の組|
|:-|:-|:-|
|contrastive|![](https://storage.googleapis.com/zenn-user-upload/74gb1r3sjrw85gs75f921nyawtj7 =100x)|（なし）|
|triplet|![](https://storage.googleapis.com/zenn-user-upload/yovqiurxid31y97yb5xl5xm5klcr =100x)|![](https://storage.googleapis.com/zenn-user-upload/z9it7ornf4xl0bgzczr6gngy268f =50x)|
|N-pair|![](https://storage.googleapis.com/zenn-user-upload/ca8163yswotc33nysl6mfi52llms =100x)|（なし）|
|lifted structured|![](https://storage.googleapis.com/zenn-user-upload/xf5ky7n0c0zbenjf17bcw0sz9sg5 =100x)|![](https://storage.googleapis.com/zenn-user-upload/q22hddwsi9ij4gpp070hcjr46qxt =200x)|
|quadruplet|（同上）|![](https://storage.googleapis.com/zenn-user-upload/975jvi7u6ajzyfkqdrh6fry35kck =200x)|


# 次回は？

metric learning の推論を高速・軽量化する方法をいくつか紹介します。
