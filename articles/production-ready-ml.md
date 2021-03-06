---
title: "本番環境に移行しやすいMLエンジニアリング"
emoji: "🤖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["mlops", "python", "poetry", "machinelearning"]
published: true
---

# 概要

機械学習アルゴリズムを実装する企業研究者やデータサイエンティストが、本番環境に移行しやすい機械学習コードを書くための Tips を紹介します。

私は、最近、機械学習アルゴリズムの実装だけでなく、それを本番環境に載せる作業もするようになったので、その際に「 **こうしておけば本番環境への移行が楽だったな** 」と思ったことをまとめました。

「自分はエンジニアじゃないから細かいことはいいや」と思われる方にも以下のメリットがあると思います。

- エンジニアへの負担が減り、早くリリースすることができ、研究者またはデータサイエンティストとしての社内での評価や信頼を獲得しやすくなる
- 信頼を獲得できれば、次の仕事を回してもらいやすくなる、という好循環が生まれる
- 自分自身の生産性や開発体験が向上するので、より早く実験サイクルを回せる
- 再利用性や再現性などを上げることができ、コードを公開したときに多くの人に使ってもらいやすい



# パッケージマネージャ、フォーマッタを使う

パッケージマネージャは再現性のために、フォーマッタは他人が読みやすように最低限入れておきたいです。

イマドキな Python の開発環境で開発するには、以下の資料が参考になります。

[【2020年新人研修資料】ナウでヤングなPython開発入門 - Speaker Deck](https://speakerdeck.com/stakaya/2020nian-xin-ren-yan-xiu-zi-liao-nauteyankunapythonkai-fa-ru-men)

上記の資料によると、現在は、パッケージマネージャには [Poetry](https://github.com/python-poetry/poetry)、 フォーマッタには [black](https://github.com/psf/black) が良さそうです。ただ、この手の資料は枯れやすいので、1年に1回くらいは「Python 開発環境 2021」のように検索して開発環境を見直したいです。

あと、 [型アノテーション](https://docs.python.org/ja/3/library/typing.html) まで書いてあると、かなり読みやすく、開発スピードが上がると思います。

# リファクタリング用の極小データセットを用意する

本番環境に移行する際に他のエンジニアがリファクタリングを行うことは多いと思います。このとき怖いのが **リファクタリングによりモデルの性能（精度など）の劣化** です。それを防ぐには、小さな修正のたびにテストが必要です（機械学習のほうではなくソフトウェアエンジニアリングのほう）。その際に実験で使うような大きなデータセットでいちいちテストしていたら時間が非常にかかるので、極小のデータセットが必要です。

**ダミーデータの生成はスクリプト化** し再現性を保ちます。乱数を使う場合はシードを固定します。

また、この **テストデータを用意** するのはエンジニアではなく、 **アルゴリズムの実装者が適任** だと思います。なぜなら「モデルが改悪されてないかを、そのテストデータで本当に確かめられるのか」は、アルゴリズム実装者の方が判断に優れているからです。

リファクタリングするエンジニアであれば、いきなりリファクタリングするのではなく、このようなテストデータを用意してもらってから着手したほうが安全です。

# パスはCLIの引数として指定し、ハードコーディング or 環境変数での指定を避ける

ハードコーディングとは以下のようにパスをコード内に埋め込むようなことです。

```py
df_train = pd.read_csv("~/data/foo/bar.csv")
```

こうしてしまうとコードを他のデータセットに流用しづらくなります。

また、以下のように環境変数で指定すると、指定ヶ所が散在し、実行時に何を指定すればいいのか、ひと目でわからなくなります。

```py
df_train = pd.read_csv(os.environ["TRAIN_DATA_FILE"])
```

なので、 **CLIの引数** として窓口を1つに絞るのが良いと思います。


# データは CSV や JSON Lines など一般的な形式で保存し、pickle は避ける

pickle を用いたほうが容量を減らせますが、以下のような問題があります。

- Python 以外に対応できない（学習・推論をGoで書きたい、評価をBigQueryでやりたい、シェルスクリプトで軽くデータ処理したい、など）
- 中身を確認するのに手間がかかる

代わりに **CSV や JSON Lines** など一般的なものにします。ちなみに JSON より改行区切りの JSON Lines のほうが BigQuery やシェルスクリプトで扱いやすいです。 pandas で読み書きする場合は以下のようにします。

```py
# 読み込み
df = pd.read_json(json_file, lines=True)

# 書き込み
df.to_json(out_file, orient="records", lines=True)
```

# 学習済みモデルの再現に必要なものは全て1つにまとめて保存する

学習済みモデルの重み以外に、再現に必要なものは全て1つにまとめて保存します。 tar でまとめる必要はないと思いますが、1つのディレクトリを指定すれば再現できるくらいは最低限やっておきます。

保存し忘れがちなものの例

- ハイパーパラメータなど、モデルの設定
- アイテムID, ユーザーID, 単語ID ↔ 行列のインデックスの変換テーブル

# モデルの保存には joblib を使う

pickle や npy, npz でも十分ですが、 joblib のパフォーマンスが良いそうです。

![](http://gael-varoquaux.info/programming/attachments/persistence_lfw_bench.png)

出典：http://gael-varoquaux.info/programming/new_low-overhead_persistence_in_joblib_for_big_data.html


# データ取得のSQLも Git 管理し、ファイルとの対応を残す

BigQuery などから学習・テストデータを取得した場合、その SQL も必ず Git で管理します。 また、その結果が、どのファイルに対応するのかや、GCSやS3のどこにあるのかも README.md などに残しておきます。**再現性のため** です。

また、1つのデータファイルを学習・テスト用に分割した場合、分割の処理は前処理としてスクリプト化し、実験が目的ならシードも固定しておきます。


# テンソルの shape をコメントする

以下のように `numpy.array` や PyTorch のテンソルのなどの変数に対し、その `shape` をコメントしておくと可読性が上がり、改修しやすいです。

```py
class FooModel:

    def fit(self, X: np.ndarray):
        # X: (num_users, num_items)

        item_sims = X.T.dot(X)
        # item_sims: (num_items, num_items)
```


# なるべく BigQuery に任す

特に前処理や評価など、 **サンプルごとに独立しており、SQLで記述できる** 処理は BigQuery に任せたほうが速いです。

pandas の方が小回りが効くので実験段階ではそれで十分ですが、仕様が fix したら BQに移行します。推論スクリプトで推論結果を JSON Lines などで掃き出し、それを BQ にインポートして SQL や UDF で評価します。

ただ、 ベンダーロックインを回避するため、 UDF で複雑なことをやりすぎないように注意します。

# 前処理などを含め、推論をメソッド1つで呼べるようにしておく

以下のように、学習成果物をまとめたディレクトリを指定し、メソッド1つ呼べば推論ができるようにしておきます。多少、自社のデータセットに密結合になっても、引き継ぐエンジニアが簡単に使える形で実装します。密結合を避けたければ、汎用的なメソッド（`_predict`）を作り、それを利用先に合わせたメソッド（`predict`）でラップすればいいだけです。

```py
class FooModel:
    def __init___(self, model_dir):
        # load model artifacts

    def predict(self, user_id: str) -> str:
        u = self.user_id2index(user_id)
        i = self._predict(u)
        return self.item_index2id(i)
```

前処理などを分けてしまうと、引き継いだエンジニアの手間が増えたり、前処理を施し忘れて性能劣化のバグを生む可能性があります。

# 推論のループでは pandas の apply() を使い、 for + iterrows() は避ける

データセットの各サンプルについて推論を行う場合、 pandas の apply を使うのが速いです。ベクトル演算として表現できるなら、その方がより速いです。for + iterrows は遅いです。以下の資料では、5つのループの方法について比較しています。

https://engineering.upside.com/a-beginners-guide-to-optimizing-pandas-code-for-speed-c09ef2c6a4d6

さらなる高速化を手っ取り早く目指すなら、並列・分散処理のできる dask や vaex を使うという手があります。

![](https://global-uploads.webflow.com/5d3ec351b1eba4332d213004/5ef62ef85090a97b0c469fa9_image5.png)

出典：https://www.datarevenue.com/en-blog/pandas-vs-dask-vs-vaex-vs-modin-vs-rapids-vs-ray

Apache Spark や Beam という選択肢ありますが、この記事の範疇を超える気がしたので詳細は割愛します。

# 学習・推論・評価に分けてスクリプト化する

**最悪** なのが学習・推論・評価がすべて **1つの Jupyter notebook** にまとまっているパターンです。そうではなく、学習・推論・評価ごとにスクリプトを用意すると、 **再利用や改修をしやすくなります**。

例えば、モデルを継続的に学習させるパイプラインにおいては推論・評価は不要な場合が多いですし、サーバーやバッチで推論する場合には学習・評価は不要になります。なので、この3つに分けておくと再利用しやすいです。また、このように各フェーズを疎結合にしておくことで、「 **学習の処理を改修しただけなのに、推論や評価の処理がおかしくなった** 」ということも回避できます。

学習とテストに分けるだけでなく、 さらに **テストを推論と評価に** 分けます。先述のとおり、 推論だけ使う場面は多いですし、 学習・推論を **他のライブラリや言語に差し替える際に評価スクリプトを再利用** できます。

また、 **前処理も** 切り離したほうが前処理結果をストレージにキャッシュでき、 **学習・推論時のオーバーヘッド** を減らしたり、推論時に **前処理の施し忘れ** を防いだりできます。


# スクリプトを階層化する

各フェーズ（学習・推論・評価）ごとにスクリプト化する際に、 **データから実行する関数** → **パスから実行する関数** → **CLI** と階層化すると再利用しやすくなります。以下では学習スクリプトを例に、それぞれ説明しますが、推論・評価においても同様です。

## データから実行する関数

以下のように学習データの `numpy.ndarray` や `pandas.DataFrame` を受け取り、機械学習モデルのオブジェクトを返すような関数をまず作ります。つまり、入出力が Python オブジェクトとなる関数です。

```py
# train.py
from typing import Dict, Any

import pandas as pd

from mymodels import Model


def train_from_data(
    model_config: Dict[str, Any],
    df_train: pd.DataFrame,
) -> Model:
    model = Model(**model_config)
    model.fit(df_train)
    return model
```

## パスから実行する関数

次に、以下のように学習データファイルのパスや、学習成果物を保存するディレクトリのパスを受け取り、学習を実行する関数を作ります。先ほどの「データから実行する関数」のラッパーです。


```py
# train.py
from pathlib import Path

import yaml
import pandas as pd
import joblib

# ...

def train_from_path(
    model_config_file: Path,
    train_data_file: Path,
    out_dir: Path,
) -> None:
    # Load
    with model_config_file.open() as f:
        model_config = yaml.safe_load(f)

    df_train = pd.read_csv(train_data_file)

    # Train
    model = train_from_data(model_config, df_train)

    # Save
    out_dir.mkdir(parents=True, exist_ok=True)
    with (out_dir / "model.pkl").open("wb") as f:
        joblib.dump(model, f)
```

学習に **必要なものはファイルごとにパスを指定できる** ように します（`model_config_file`、`train_data_file`）。なぜなら、1つのディレクトリで指定しようとすると、 **そのディレクトリ構造以外に対応できなくなる** からです。また、 その構造を理解するためにコードを読む手間が増えます。大量の画像などはディレクトリを指定せざるを得ないですが、それでも、画像ディレクトリとCSVなどのパスは分けたほうが柔軟に対応できます。

逆に、学習の **成果物はまとめて1つのパスを指定する** ようにします（`out_dir`）。なぜなら、学習成果物をバラバラに保存できるようにしてしまうとモデルを管理・再現しづらくなるからです。 **入力は柔軟に、出力は厳格に** します。

## CLI
最後に、以下のように「パスから実行する関数」をコマンドラインインターフェースで実行できるようにします。CLIツール作成のライブラリには `argparse` や `click` などがありますが、個人的には FastAPI 作者による [Typer](https://github.com/tiangolo/typer) がオススメです。型アノテーションを利用するので記述量が少なくメンテしやすいです。

```py
# train.py
import typer

# ...

def main():
    typer.run(train_from_path)

if __name__ == "__main__":
    main()
```

```sh
$ train.py --help
```

とすれば、リッチなヘルプメッセージを確認できます。

## 3層に分けた理由

まず **「データから実行する関数」を設けた理由** は、 **ローカルにあるファイル以外からも実行できるようにするため** です。例えば、BigQueryを叩いた結果を `pandas.DataFrame` に入れて学習したい場合において `train_from_data` を再利用できます。


次に **「パスから実行する関数」を設けた理由** は、 **プログラムからの呼び出しにも対応できるようにするため** です。例えば、 SageMaker では Docker イメージを作らずとも `train_from_path` を再利用して SDK でジョブを叩けます。


# まとめ

本番環境に移行しやすい ML エンジニアリングの Tips を紹介しました。

- パッケージマネージャ（Poetry）、フォーマッタ（black）を使い、型アノテーションする
- リファクタリング用の極小データセットを用意する
- パスはCLIの引数として指定し、ハードコーディング or 環境変数での指定を避ける
- データは CSV や JSON Lines など一般的な形式で保存し、pickle は避ける
- 学習済みモデルの再現に必要なものは全て1つにまとめて保存する（ハイパーパラメータ、ID↔インデックスの変換テーブルなども）
- モデルの保存には joblib を使う
- データ取得の SQL も Git 管理して、ファイルとの対応を残す
- テンソルの shape をコメントする
- なるべく BigQuery に任す
- 前処理などを含め、推論をメソッド1つで呼べるようにしておく
- 推論のループでは pandas の apply() を使い、 for + iterrows() は避ける
- 学習・推論・評価に分けてスクリプト化する
- データから実行する関数 → パスから実行する関数 → CLI と階層化する

随時更新予定です。

より良い方法やご指摘があれば、コメントいただけると幸いです。
