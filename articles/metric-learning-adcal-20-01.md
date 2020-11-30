---
title: "metric learningは発展してないわけでもsoftmax cross cntropyに劣っているわけでもない"
emoji: "📏"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["metriclearning"]
published: false
---

metric learning アドベントカレンダー2020 2回目の記事です。「 **自分は画像系じゃないから metric learning とか関係ない〜** 」と思ってる方にとって、 このアドベントカレンダーが **metric learning を学ぶきっかけ** になったらと考えています。

# この記事の目的は？

「 **metric learning 実はそんな発展してなくて、 softmax cross entropy 使っとけばいいんでしょー？** 」と思ってる方の **誤解を晴らす** ことです。

# metric learning とは何か

metric learning の基本的なことについては、以下のわかりやすい記事に譲ります。

- [Metric Learning 入門 - copypasteの日記](https://copypaste-ds.hatenablog.com/entry/2019/03/01/164155)
- [Deep Metric Learning の定番⁈ Triplet Lossを徹底解説 - Qiita](https://qiita.com/tancoro/items/35d0925de74f21bfff14)
- [距離学習（Metric Learning）入門から実践まで｜はやぶさの技術ノート](https://cpp-learning.com/metric-learning/)


# 誤解のきっかけ

「metric learning って実はそんな発展してなくて、 softmax cross entropy(SCE) 使っとけばいいんでしょー？」というような誤解のきっかけは、おそらく以下の2つの論文が同時期に話題になったことだと考えています。

1. Musgrave, K. et al., 2020: [[2003.08505] A Metric Learning Reality Check](https://arxiv.org/abs/2003.08505)
2. Boudiaf, M. et al., 2020: [[2003.08983] A unifying mutual information view of metric learning: cross-entropy vs. pairwise losses](https://arxiv.org/abs/2003.08983)

どちらも ECCV2020 に通った論文です。

それぞれ要約すると、

1. Musgrave, K. et al., 2020: 先行研究のずさんな実験方法を整えて再実験したら、古典的な手法と大差なかった
2. Boudiaf, M. et al., 2020: metric learning も SCE も本質的には同じことをやっていて、実験的には SCE の方が metric learning より良かった

ですが、かといって **metric learning のここ数年の発展が無駄というわけでも、 SCE に劣るわけでもありません**（というより、比べること自体おかしいです）。

次のサブセクションで、それぞれの誤解しやすいポイントを解説します。

## Musgrave, K., 2020 の誤解ポイント

まず、 「 **古典的な手法** 」というのは contrastive loss と triplet loss のことを指しており、 **SCE のことではありません** 。

また、 この論文では **精度** (※1) **のみでしか評価されてません。** ここ数年の発展は、収束の速さなど、 **精度以外への貢献が大きい** と感じています。例として、以下の図のように収束が早く安定して学習できる手法が提案されてきました。

![](https://storage.googleapis.com/zenn-user-upload/v0bw5z7s91n3rujurbd9kpacx7ab)
出典： [[2003.13911] Proxy Anchor Loss for Deep Metric Learning](https://arxiv.org/abs/2003.13911.pdf)
参考： [CVPR2020読み会 Proxy Anchor Loss for Deep Metric Learning - Speaker Deck](https://speakerdeck.com/satokeiju/cvpr2020du-mihui-proxy-anchor-loss-for-deep-metric-learning)

この論文が本当に伝えたかったことは「 **実験条件をそろえて公平に比較しましょう** 」ということであり、決して metric learning が大して発展していなかった、というものではないと思います。

そもそも、この著者は Facebook research で metric learning を研究されており、 [pytorch-metric-learning](https://github.com/KevinMusgrave/pytorch-metric-learning) を開発されています。 そんな方が metric learning 自体を否定するとは考えづらく、おそらく metric learning への研究がずさんに進められてることに対して警鐘を鳴らしたかったのだと思います。同ライブラリ公開の目的も、透明で公平な比較を行いやすくするためだと捉えています。

## Boudiaf, M. et al., 2020 の誤解ポイント

実験的には SCE のほうが metric learning のいくつかの手法より良い結果が報告されています。しかし、 **手法ごとに異なるCNNアーキテクチャを使ってる** ので、 この実験結果から metric learing より **SCE の方が優れているとは言えない** はずです。著者自身も論文中で以下のように認めています。

> They have large differences in classification performances on ImageNet, but the impact on performances over DML benchmarks has rarely been studied in controlled experiments. As this is not the focus of our paper, we use ResNet-50 for our experiments. We concede that one may obtain better performances by modifying the architecture

（意訳：アーキテクチャによってImageNetでの分類結果に大きな差が出るが、 metric learning のベンチマークではほとんど調べられてないし、 この研究の本質ではないので、 手法間で別々のアーキテクチャを使っている。そのせいで SCE の方が良かったという可能性は認める）

また、付録に **SCE の限界** （On the limitations of cross-entropy）について注意書きされています。要約すると、 **相対的なラベルしかない場合や、クラス数が非常に多い場合には向かない** そうです。

この論文が本当に伝えたかったことは「 **SCE は metric learning と本質的には同じことをやっている** 」だと思います。実験条件は好ましくありませんが、 **SCE と metric learning は本質的には同じだということを情報理論の観点から丁寧に証明している点は非常におもしろい** と感じました。

以上より、「metric learning って実はそんな発展してなくて、 softmax cross entropy(SCE) 使っとけばいいんでしょー？」というような誤解は晴らせたかと思います。

# まとめ

「 metric learning 実はそんな発展してなくて、 softmax cross entropy 使っとけばいいんでしょー？」というような誤解を招いたであろう以下の2つの論文の誤解しやすいポイントを解説しました。

1. Musgrave, K. et al., 2020: [[2003.08505] A Metric Learning Reality Check](https://arxiv.org/abs/2003.08505) -> metric learning が発展してないわけではなく、 **実験条件をそろえて公平に比較すべき**
2. Boudiaf, M. et al., 2020: [[2003.08983] A unifying mutual information view of metric learning: cross-entropy vs. pairwise losses](https://arxiv.org/abs/2003.08983) -> metric learning が SCE に劣っているわけではなく、 **SCE は metric learning と本質的には同じ**

# 次回は？

次回のタイトルは「metric learning と softmax cross entropy を比べること自体間違っている」です。 SCE が metric learning と本質的には同じだということを簡単に説明し、それがわかると何が嬉しいのかについても紹介したいと思います。

---

- (※1) 正確には精度だけではないですが、「どれくらい予測が合っているか」を測る、精度のような指標のみでしか評価されておらず、また著者自身もそれらの指標を「accuracy」とまとめていたため、「精度」という言葉を使いました。
