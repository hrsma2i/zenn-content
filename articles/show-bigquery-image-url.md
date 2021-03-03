---
title: "BigQuery の画像 URLを表示するブックマークレット"
emoji: "🏞"
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
var t = document.querySelector("#main > div > div > central-page-area > div > div > pcc-content-viewport > div > div > ng2-router-root > div > ng-component > cfc-panel-container > div > div > cfc-panel.bq-main-container.cfc-panel.cfc-panel-center.cfc-panel-color-white.cfc-panel-orientation-vertical > div > div > cfc-panel-container > div > div > cfc-panel.cfc-panel.cfc-panel-center.cfc-panel-color-white.cfc-panel-orientation-horizontal.ng-star-inserted > div > div > bq-workspace > bq-tab-container > cfc-panel-container > div > div > cfc-panel > div > div > bq-tab-panel > bq-tab-content > div > bqui-query-panel > div > cfc-panel-container > div > div > cfc-panel.bqui-query-results.cfc-panel.cfc-panel-center.cfc-panel-color-white.cfc-panel-orientation-horizontal > div > div > cfc-panel-body > cfc-virtual-viewport > div.cfc-virtual-scroll-content-wrapper > bq-ng1-query-results-container-wrapper > xap-deferred-loader-outlet > bq-query-results-container-upgrade-wrapper > bq-query-results-container-upgrade > bq-query-results-container > bq-script-query-results > bq-query-results > div > div > section > div.p6n-bq-query-results-tab > bq-results-table > div > span > div.p6n-bq-results-table-scroll-container > table");
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
javascript:var%20t%20=%20document.querySelector("#main%20>%20div%20>%20div%20>%20central-page-area%20>%20div%20>%20div%20>%20pcc-content-viewport%20>%20div%20>%20div%20>%20ng2-router-root%20>%20div%20>%20ng-component%20>%20cfc-panel-container%20>%20div%20>%20div%20>%20cfc-panel.bq-main-container.cfc-panel.cfc-panel-center.cfc-panel-color-white.cfc-panel-orientation-vertical%20>%20div%20>%20div%20>%20cfc-panel-container%20>%20div%20>%20div%20>%20cfc-panel.cfc-panel.cfc-panel-center.cfc-panel-color-white.cfc-panel-orientation-horizontal.ng-star-inserted%20>%20div%20>%20div%20>%20bq-workspace%20>%20bq-tab-container%20>%20cfc-panel-container%20>%20div%20>%20div%20>%20cfc-panel%20>%20div%20>%20div%20>%20bq-tab-panel%20>%20bq-tab-content%20>%20div%20>%20bqui-query-panel%20>%20div%20>%20cfc-panel-container%20>%20div%20>%20div%20>%20cfc-panel.bqui-query-results.cfc-panel.cfc-panel-center.cfc-panel-color-white.cfc-panel-orientation-horizontal%20>%20div%20>%20div%20>%20cfc-panel-body%20>%20cfc-virtual-viewport%20>%20div.cfc-virtual-scroll-content-wrapper%20>%20bq-ng1-query-results-container-wrapper%20>%20xap-deferred-loader-outlet%20>%20bq-query-results-container-upgrade-wrapper%20>%20bq-query-results-container-upgrade%20>%20bq-query-results-container%20>%20bq-script-query-results%20>%20bq-query-results%20>%20div%20>%20div%20>%20section%20>%20div.p6n-bq-query-results-tab%20>%20bq-results-table%20>%20div%20>%20span%20>%20div.p6n-bq-results-table-scroll-container%20>%20table");var%20a%20=%20t.querySelectorAll("thead%20>%20tr%20>%20th");a.forEach((h,%20i)%20=>%20{if%20(h.innerText.includes("url"))%20{var%20b%20=%20t.querySelectorAll(`tbody%20>%20tr%20>%20td:nth-child(${i+1})%20>%20div`);b.forEach((r)%20=>%20{r.innerHTML%20=%20`<img%20src=${r.innerText}%20height=150>`;});}});
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
