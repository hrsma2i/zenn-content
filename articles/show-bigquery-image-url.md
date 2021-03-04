---
title: "BigQuery の画像 URLを表示するブックマークレット"
emoji: "🏞️"
type: "tech"
topics: ["bookmarklet", "bigquery"]
published: true
---

# この記事の目的

データ分析の際、 BigQuery の画像 URL のカラムの画像を確認したいことがあります。例えば、機械学習による商品の推薦結果を BigQuery に吐き出して、定性評価を行う場合などです。その際、いちいちURLをコピペして開くのは面倒なので、一気に表示できるブックマークレットを紹介します。

↓こんなかんじに表示できます。

![](https://storage.googleapis.com/zenn-user-upload/56pwfhumnms8bsu6pfw0g8jkggv7)


# ブックマークレット

```js
var t = document.querySelector("table");
var a = t.querySelectorAll("thead > tr > th");
a.forEach((h, i) => {
    if (h.innerText.includes("url")) {
        var b = t.querySelectorAll(`tbody > tr > td:nth-child(${i+1}) > div`);
        b.forEach((r) => {
            r.innerHTML = `<img src=${r.innerText} height=150>`;
        });
    }
});
```

# 使い方

上のコードを平坦化したものを、ブラウザのお気に入りなどに登録し、BigQueryの実行結果の画面で起動すれば、画像が表示されます。

カラム名に `url` が入ったカラムが対象になります。

平坦化したコード

```js
javascript:var%20t%20=%20document.querySelector("table");var%20a%20=%20t.querySelectorAll("thead%20>%20tr%20>%20th");a.forEach((h,%20i)%20=>%20{if%20(h.innerText.includes("url"))%20{var%20b%20=%20t.querySelectorAll(`tbody%20>%20tr%20>%20td:nth-child(${i+1})%20>%20div`);b.forEach((r)%20=>%20{r.innerHTML%20=%20`<img%20src=${r.innerText}%20height=150>`;});}});
```

ちなみに、以下のスクリプトで平坦化しました。

```sh
#!/bin/sh -eu
printf "javascript:"
sed "s/    //g" main.js | sed "s/ /%20/g" | tr -d "\t" | tr -d "\n"
```

**オススメの使い方** は **Chrome の検索エンジンに登録して URL バーから呼び出す** 方法です。

chrome://settings/searchEngines にアクセスし、 `その他の検索エンジン > 追加` で以下のように追加すると、 URL バーに `bqimg` と打つだけで起動します。

![](https://storage.googleapis.com/zenn-user-upload/cle9qxhchxssvegbfelu6o5tz0vh)
