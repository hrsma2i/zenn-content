---
title: "Poetryでプラットフォームごとにインストールするライブラリを変える"
emoji: "🐍"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["python", "poetry"]
published: false
---

# Poetry とは

Python ライブラリの依存管理ツールです。

- https://python-poetry.org/docs/ 
- [2020 年の Python パッケージ管理ベストプラクティス - Qiita](https://qiita.com/sk217/items/43c994640f4843a18dbe#pyenv--poetry)


# 課題：プラットフォームごとにインストールするライブラリを変えたい

プラットフォームごとにインストールするライブラリを変えたい場合があります。例えば、 Tensorflow 1.x では、 インストール先のマシンにGPUがあるかどうかによってライブラリ名を変える必要があります （ `tensoflow` と `tensorflow-gpu` ）。


# 解決策：`--extras` オプション

Poetry の `--extras` オプションで解決できます。最終的には以下のようなコマンドで切り替えることができます。

CPUの場合

```bash
$ poetry install -E cpu
```

GPUの場合

```bash
$ poetry install -E gpu
```

設定手順は以下です。

プラットフォームに依存するライブラリを追加するときに、以下のコマンドで追加します。

```bash
$ poetry add "$LIBRARY=$VERSION"[$EXTRAS] --optional
```

`$EXTRAS` はライブラリ群に対するタグのようなもので、今回の例ではプラットフォーム名などを記入します。

例）

```bash
$ poetry add "tensorflow-gpu=1.14"[gpu] --optional 
```


次に、以下のように `pyproject.toml` に、プラットフォームごとにインストールしたいライブラリのリストを追記します。

```toml:pyproject.toml
[tool.poetry.extras]
cpu = ["tensorflow"]
gpu = ["tensorflow-gpu"]
```

複数ライブラリある場合は `["lib1", "lib2"]` のように書きます。

あとは以下のように環境をセットアップする際に `--extras, -E` でプラットフォームを指定するだけです。

CPUの場合

```bash
$ poetry install -E cpu
```

GPUの場合

```bash
$ poetry install -E gpu
```

また、 [公式ドキュメント](https://python-poetry.org/docs/pyproject/#extras) にも書かれている通り、 extras はタグのように複数指定することもできます。

```bash
$ poetry install --extras "mysql pgsql"
$ poetry install -E mysql -E pgsql
```
