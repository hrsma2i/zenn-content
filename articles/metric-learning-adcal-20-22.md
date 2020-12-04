---
title: "metric learning のファッション分野における活躍"
emoji: "👗"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["metriclearning", "fashionMachineLearning", "compatibilityLearning", "attributeManipulation", ""]
published: true
---

# この記事の目的は？

ファッション3つの研究分野において、 metric learning がどう使われているかを説明し、関連文献もいくつか紹介します。 metric learning やファッションの研究に興味を持たれた方が、研究を始めやすくなればと考えています。


# street-to-shop image retrieval

## どんな研究か？

**ファッションアイテムの自撮り画像から、ECサイトで使われるような商品画像を検索** するための研究です。ファッションに限らない、一般的な呼び方だと cross-domain image retrieval と呼んだりもします。

![](https://storage.googleapis.com/zenn-user-upload/h03ol9yuoib570ukg9q9u67y4dl5 =400x)
図：自撮り画像の例

![](https://storage.googleapis.com/zenn-user-upload/2g8c1a2eojyku572nw11knr9bxi8 =400x)
図：商品画像の例

出典: [(M. Hadi Kiapour et al., 2015, ICCV)  Where to Buy It: Matching Street Clothing Photos in Online Shops](https://www.cv-foundation.org/openaccess/content_iccv_2015/papers/Kiapour_Where_to_Buy_ICCV_2015_paper.pdf) 

## metric learing はどう使われてるか？

**同じアイテムの自撮り画像と商品画像の特徴量を近づける** のに使われます。

![](https://storage.googleapis.com/zenn-user-upload/unerwes9bbeevduffr07gafmo7ry =600x)

どのように検索するかというと、クエリの画像特徴量の近傍を検索結果として返します。ECサイトなどでは商品数が多いため、近似近傍探索が使われたりします。

アパレルECサイトの [ZOZOTOWN](https://zozo.jp/) や、ファッションSNSの [WEAR](https://wear.jp) の画像検索機能の裏では、まさに metric learning で学習したモデルが動いています。ZOZOTOWN の検索システムについては以下の記事で解説しています。

- [ZOZOTOWN、AIを活用し、閲覧商品と似ている商品を検索できる「類似アイテム検索機能」を本日より導入 - 株式会社ZOZO](https://corp.zozo.com/news/20190826-8586/)
- [あなたの"欲しい"にたどりつく！WEARの超便利機能に注目🕵｜WEAR｜note](https://note.com/wear_official/n/n254031853e32)
- [deep metric learningによるcross-domain画像検索 - ZOZO Technologies TECH BLOG](https://techblog.zozo.com/entry/metric_learning)：学習アルゴリズムの話
- [Google Cloud TPUを使った計量学習の高速化事例の紹介 - ZOZO Technologies TECH BLOG](https://techblog.zozo.com/entry/tpu_metric_learning)：TPUで学習を速くした話
- [類似アイテム検索機能についてGoogle Cloud Next '19 in Tokyoで技術発表をしました - ZOZO Technologies TECH BLOG](https://techblog.zozo.com/entry/cloudnext19tokyo-imagesearch)：アーキテクチャの話（マイクロサービス, gRPC, GKE, terraform, 監視）
- [メルカリ・ヤフー・ZOZO開発者が語る「画像検索」の最前線！　 Bonfire Data & Science #1 イベントレポート - Yahoo! JAPAN Tech Blog](https://techblog.yahoo.co.jp/entry/20191114780128/)：レスポンスのキャッシュによる高速化の話
- [近似最近傍探索Indexを作るワークフロー - ZOZO Technologies TECH BLOG](https://techblog.zozo.com/entry/ann-workflow)：Cloud Composer (Airflow) による機械学習ワークフローの話

## 関連文献

- [(J. Lasserre et al., 2018, ICPRAM) Studio2Shop: from studio photo shoots to fashion articles](https://arxiv.org/abs/1807.00556)：ヨーロッパの EC、Zalando の研究所が出した論文。既存手法の違いをまとめた比較表がわかりやすい。商品画像の特徴抽出には学習可能なモデルを使っていない。また、 early fusion を使っている(※2, 論文中だと non-linear matching に相当。late fusion は linear matching に相当。)
- [(A. Veit et al., 2017, CVPR) Conditional similarity networks](https://arxiv.org/pdf/1603.07810.pdf)：類似性には、「色が似ている」や「形が似ている」など複数の観点がある。観点で条件づけた特徴量で学習させることにより、特定の観点にもとづいた検索を可能にした。
- [(Q. Dong et al., 2017, WACV) Multi-Task Curriculum Transfer Deep Learning of Clothing Attributes](https://arxiv.org/pdf/1610.03670.pdf)：2段階のカリキュラム学習を行う。1段階目は商品画像の色やカテゴリーを予測するという比較的簡単なタスクで学習し、2段階目で商品画像と自撮り画像の metric learning をする。参考文献の [t分布を用いた metric leanring](https://lvdmaaten.github.io/publications/papers/MLSP_2012.pdf) が面白い。損失関数にL2距離とシグモイド関数を使った metric learning は、正規分布を使ってるとみなせるので、それをt分布にすることでロバストにした。
- [(X. Wang et al., 2016, ICML) Matching User Photos to Online Products with Robust Deep Features](http://yugangjiang.info/publication/icmr16-productSearch.pdf)：あまり似ていない正例のペア（異常値）に対する勾配を大きくしない robust contrastive loss を提案。
- [(S. Jiang et al., 2016, MM) Deep Bi-directional Cross-triplet Embedding for Cross-Domain Clothing Retrieval](http://www1.ece.neu.edu/~yuewu/files/2016/p52-jiang.pdf)： lifted structured loss (※4) とほぼ同じで、 同じドメインの画像間の計量も考慮する。
- [(Q. Chen et al., 2015, CVPR) Deep Domain Adaptation for Describing People Based on Fine-Grained Clothing Attributes](http://rogerioferis.com/publications/DDANCVPR2015.pdf)： multi-label classification loss に加え、CNNの最終層の出力だけでなく、中間層の出力も近づけるような階層的な metric learning を行っている。
- [(J. Huang et al., 2015, ICCV) Cross-domain Image Retrieval with a Dual Attribute-aware Ranking Network](https://arxiv.org/pdf/1505.07922.pdf)：triplet loss 系と、色とカテゴリーを予測する classification loss を併用している (※3)。
- [(M. Hadi Kiapour et al., 2015, ICCV) Where to Buy It: Matching Street Clothing Photos in Online Shops](https://www.cv-foundation.org/openaccess/content_iccv_2015/papers/Kiapour_Where_to_Buy_ICCV_2015_paper.pdf)：自撮り画像と商品画像の例がわかりやすい。 contrastive loss 系と early fusionを使ってる（※2）。データセットのノイズがひどい。

**ドメイン適応** の研究も関連してるようです。

# attribute manipulation

## どんな研究か？

アイテムの画像特徴量をテキストによって操作する研究です。大雑把に言うと、 **Word2Vec の、意味による演算を画像と単語でやる** のが目的です。

応用先として、 **画像検索のテキストによる補正** などに使えます。例えば、長袖のボーダーTシャツを検索したいのに、半袖のボーダーTシャツの画像しか持っていなくても、その画像と「長袖」というテキストを与えるだけで検索できるようにすることです。

![](https://storage.googleapis.com/zenn-user-upload/4gqjrfh35yt18wkkqig16d6rrru6 =500x)

## metric learing はどう使われてるか？

**同じアイテムの画像と属性テキストの特徴量を近づける** のに使われます。属性テキストとは、アイテムの **袖の長さや模様** などを表した文や単語のことです(※1)。

![](https://storage.googleapis.com/zenn-user-upload/ccnfcanqimywrq0oc0v2i89rysc6 =600x)

どのように画像検索の補正を行うかというと、クエリ画像（e.g., 半袖のボーダーTシャツの画像）の特徴量に、属性テキスト（e.g, 「長袖」というキーワード）の特徴量を加算したりして補正します。あとは画像検索同様に、近傍を検索結果として返すだけです。


## 関連文献

- [(Y. Chen et al., 2020, ECCV) Learning Joint Visual Semantic Matching Embeddings for Language-guided Retrieval](https://www.ecva.net/papers/eccv_2020/papers_ECCV/papers/123670137.pdf) ：（まだ読めてない）
- [(G. Sadeh et al., 2019) Joint Visual-Textual Embedding for Multimodal Style Search](http://arxiv.org/abs/1906.06620)： metric learning の損失関数に加え、属性を予測する classificaiton loss も付けた。
- [(M. Shin et al., 2019) Semi-supervised Feature-Level Attribute Manipulation for Fashion Image Retrieval](http://arxiv.org/abs/1907.05007)： GAN によって補正後の特徴量を生成する。 画像と補正用属性から、補正後の特徴量を生成し、逆に補正後の特徴量と本来の属性から、補正前の特徴量を生成する cycle consisitensy loss を使っている。
- [(K. E. Ak et al., 2018, CVPR) Learning Attribute Representations with Localization for Flexible Fashion Search](https://openaccess.thecvf.com/content_cvpr_2018/papers/Ak_Learning_Attribute_Representations_CVPR_2018_paper.pdf)：属性予測の学習で得られた活性化マップを用いて、画像のある属性に関連する領域だけを残し、同じ属性をもつ画像どうしの metric learning を行う。さらに属性ごとに別々に抽出した画像特徴量を結合・圧縮した特徴量でさらに global な metric learning を行う。 triplet loss 系を用いてる。
- [(B. Zhao et al., 2017, CVPR) Memory-augmented attribute manipulation networks for interactive fashion search](https://openaccess.thecvf.com/content_cvpr_2017/papers/Zhao_Memory-Augmented_Attribute_Manipulation_CVPR_2017_paper.pdf) ：学習データとして (あるアイテムの画像, そのアイテムの属性) の組が与えられるのではなく、 (クエリ画像, 補正用属性, 正解画像) の組が与えられる問題設定（自己教師あり学習ではなく、教師あり学習）。補正後のクエリの画像特徴量と正解の画像特徴量で metric learing する。 tirplet loss 系と classification loss の併用。
- [(X. Han et al., 2017, ICCV) Automatic Spatially-aware Fashion Concept Discovery](http://openaccess.thecvf.com/content_ICCV_2017/papers/Han_Automatic_Spatially-Aware_Fashion_ICCV_2017_paper.pdf) ：属性が key-value 形式（e.g., "袖":"長袖", "模様":"ボーダー"）で与えられていない問題設定（e.g., "長袖", "ボーダー”）において、無理やり key-value 形式にする方法を提案。具体的には、属性ベクトルをクラスタリングして得られたクラスターをkeyとする。

**マルチモーダル学習の visual semantic embeddings** という分野も関連しているようです。


# compatibility learning

## どんな研究か？

**アイテムの相性** を学習させる研究です。例えば、デニムジャケットにはボーダーのTシャツが合う、というようなことを学習させます。 **コーデの採点** や **アイテムからアイテムの推薦** に使うことができます。

![](https://storage.googleapis.com/zenn-user-upload/aj7qt2mojkwqwb8mj3xfdvoaooeq =600x)


## metric learing はどう使われてるか？

**同じコーデで使われているアイテムの特徴量どうしを近づける** のに使われます。または、同時購入されたアイテムどうしを近づける場合もあります。特徴量としては画像が使われることが多く、特に visual compatibility と呼ばれたりしています。

![](https://storage.googleapis.com/zenn-user-upload/xv7hkpd99udivtgbaaox227npdds =600x)

どうやってコーデの採点を行うかというと、コーデ内のアイテムの組合せすべてについて、特徴量の類似度または距離を計算し、特徴量が近いほど高い値をとる採点スコアに変換します。

また、どうやってアイテムからアイテムの推薦を行うかというと、画像検索と同様にクエリとなるアイテム特徴量の近傍を推薦結果として返します。

ちなみに、アイテム×アイテムだけでなく、 **アイテム×スタイル** の場合もあります。スタイルとは、カジュアル、きれいめ、コンサバ、ガーリーなど、コーデの雰囲気のことです。以下の論文では Bi-LSTM によって **コーデ内のアイテム特徴量を reduce して得られる文脈ベクトルをスタイル** とみなし、 metric learning をしています(※5)。

[(X. Han et al., 2017, MM) Learning Fashion Compatibility with Bidirectional LSTMs](https://arxiv.org/pdf/1707.05691.pdf)

![](https://storage.googleapis.com/zenn-user-upload/iyst9i65q4xzb3y3hjx2elb7y496)

## 関連文献

- [(X. Yang et al., 2020, MM) Learning tuple compatibility for conditional outfitrecommendation](https://arxiv.org/pdf/2008.08189.pdf)：既存手法のアーキテクチャがまとまっていてわかりやすい。スタイルベクトルをカテゴリーで条件付けることで推薦するアイテムのカテゴリーを制御できるようにした。夏なのに冬服を推薦するといった問題を回避できるようになった。
- [(P. Tangseng et al., 2020, WACV) Toward Explainable Fashion Recommendation](https://openaccess.thecvf.com/content_WACV_2020/papers/Tangseng_Toward_Explainable_Fashion_Recommendation_WACV_2020_paper.pdf)：アイテムの画像特徴量に、輪郭やメインカラーといった人間にわかりやすい情報を使うことで解釈性を上げた。オシャレじゃないと判定したコーデにおいて、使われてるアイテムの形と色どちらがおかしいのか？まで解釈できるようになった。
- [(Z. Cui et al., 2019, WWW) Dressing as a Whole: Outfit Compatibility Learning Based on Node-wise Graph Neural Networks](http://arxiv.org/abs/1902.08009)：コーデをアイテムの集合とみなし、集合関数によってオシャレかどうかを予測する。early fusion (※2)。カテゴリーの共起情報に基づき、各アイテム間の相性にアテンションを張ることで、重要なカテゴリーの組ほどオシャレさに反映されるようにした。カテゴリーの組合せごとの重み行列を、カテゴリーごとの重み行列の積で表現することでパラメータを大幅削減。
- [(W. Chen et al., 2019, KDD) POG: Personalized Outfit Generation for Fashion Recommendation at Alibaba iFashion](http://arxiv.org/abs/1905.01866)： Alibaba の論文。コーデを系列でなく集合とみなし、1つだけアイテムを抜いて、 Transformer によって抜いたアイテムを予測することで学習。また、ユーザーが閲覧したアイテムの系列で条件づけることによりパーソナライズされたコーデを推薦できる。
- [(W. Yu et al., 2019, TKDE) Visually-aware Recommendation with Aesthetic Features](http://arxiv.org/abs/1905.02009)：既存手法の多くはアイテムの共起情報しか用いてないが、人間の視覚的な美学に基づいた特徴量を用いてる。
- [(Y.-S. Shih et al., 2018, AAAI) Compatibility Family Learning for Item Recommendation and Generation](https://arxiv.org/pdf/1712.01262.pdf)： GAN を使っていることよりも、「あるアイテムと相性の良いアイテムは複数存在する」という仮定のもと、アイテムの画像を1点ではなく、複数点に埋め込んでいるのが面白い。損失関数に min を使うことで、複数の埋め込みのうち、1点でも正解のアイテムに近ければ良いようにしている。
- [(M. I. Vasileva et al., 2018, ECCV) Learning Type-Aware Embeddings for Fashion Compatibility](https://arxiv.org/abs/1803.09196)：既存手法では類似性と相性を計る空間が同じだったが、カテゴリーの組ごとに相性を計る空間を用意することで分離した。具体的にはカテゴリーの組ごとに特徴量のマスクを用意し、マスクした特徴量で metric learning を行う。
- [(W.-L. Hsiao et al., 2018, CVPR) Creating Capsule Wardrobes from Fashion Images](https://arxiv.org/pdf/1712.02662.pdf): オシャレかつ多様なコーデを、持っているアイテムの中からいくつか推薦する手法を提案。持っているアイテムの部分集合をコーデととらえ、「オシャレさ」と「多様さ」を目的関数とした劣モジュラ最適化を行っている。多様さの計算には選んだコーデのスタイルを使うが、コーデ×属性（e.g., 花柄やレースなど）の共起行列に対して Correlated Topic Model を用いて得られるトピック分布をスタイルとしている。
- [(H. Lee et al., 2017) Style2Vec: Representation Learning for Fashion Items from Style Sets](https://arxiv.org/pdf/1708.04014.pdf) ：アイテム画像の系列を文とみなし、 Word2Vec (Skip-Gram) で学習させた(※5)。
- [(X. Song et al., 2017, MM) NeuroStylist: Neural compatibility modeling for clothing matching](http://neurostylist.farbox.com/) ：画像とテキストの特徴量を結合し、 Bayesian Personalized Raking (BPR) (※6) と visual-semantic embeddings (※7) と Auto-Encoder の合わせ技で学習。
- [(Q. Liu et al., 2017, SIGIR) DeepStyle: Learning User Preferences for Visual Recommendation](http://www.shuwu.name/sw/DeepStyle.pdf)： Matrix Factorization のアイテム特徴量にコンテンツ（画像）ベースのスタイルベクトルを加算したもの。スタイルベクトルは、アイテムの画像特徴量からカテゴリーの埋め込みを減算したものと定義している。BPR(※6)で学習させている。
- [(Y. Li et al., 2016, TMM) Mining Fashion Outfit Composition Using An End-to-End Deep Learning Approach on Set Data](https://arxiv.org/pdf/1608.03016.pdf)：アイテム × スタイルの metric learning を行っている。スタイルベクトルに reduce する関数として、 mean, max, RNN を比較している。また、他の論文ではあまり言及されない ealy fusion と late fusion についても議論している(※8)。


**コンテンツベースの推薦分野** も関連しているようです。

# まとめ

metric learning が活用されているファッションの研究分野を3つ紹介しました。関連文献の要約に書きましたが、 metric learning だけでなく **推薦、自然言語処理、マルチモーダル学習などとも関連しています。**

紹介した3つの分野で metric learning がどう使われてるかを下の表にまとめました。


|研究|応用先|学習データ|正例・負例の基準|
|:-|:-|:-|:-|:-|:-|
|**street-to-shop image retrieval**|画像検索|自撮り画像 × 商品画像|同じアイテムの画像かどうか|
|**attribute manipulation**|画像検索のテキストによる補正|アイテム画像 × 属性テキスト|同じアイテムの画像と属性かどうか|
|**compatibility learning**|コーデ採点、アイテムからアイテムの推薦|アイテム画像 × （アイテム画像 or スタイル）|同じコーデで使われているアイテムかどうか|


---
(※1) 一般的な用語ではありません。説明を簡単にするため、こう表現しました。

(※2) early fusion とは、特徴量どうしを結合してからネットワークに通して計量を計算する方法です。一方、特徴量をネットワークに通して得られた特徴量の計量を計算する方法を late fusion と言います。どちらもマルチモーダル学習の分野で一般的な用語です。 metric learning の分野では late fusion を用いる例が多い気がします。

(※3) classification loss も metric learning です。同アドベントカレンダーの3日目の記事が参考になると思います。

(※4) 同アドベントカレンダーの14日目の記事で lifted structed loss について説明してます。

(※5) 同アドベントカレンダーの9日目の記事で Word2Vec や RNN による言語モデルも metric learning であることを説明しています。

(※6) 同アドベントカレンダーの7日目の記事で BPR を metric leanrning の観点からとらえています。

(※7) 対応する画像とテキストを近くに埋め込むマルチモーダル学習の一分野です。これも metric learning をしています。同じアドベントカレンダーの11日目の記事で説明しています。また、この記事の attribute manipulation の学習とも近いです。

(※8) マルチモーダル学習における early/late fusion とは若干、異なります。この論文おける early fusion は「あるアイテム特徴量と、残りのアイテムベクトルを reduce したスタイルベクトルの metric learning」を表し、 late fusion は「アイテムのペアごとに計量を計算し、損失関数で統合すること」を表します。
