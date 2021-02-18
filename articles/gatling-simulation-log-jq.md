---
title: "Gatling の simulation.log を jq でパースする"
emoji: "🔫"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Gatling", "負荷試験"]
published: true
---

# この記事の目的

Gatling では、負荷試験結果の統計情報は、結果ディレクトリ内の `js/stats.json` や `js/global_stats.json` から取得できますが、生ログデータを解析したい場合 `simulation.log` という TSV ファイルを見る必要があります。

ただ、この `simulation.log` が曲者で、ヘッダーがなかったり、行によってカラム数が異なったりと解析しづらいので、この記事では jq コマンドでパースして JSON として扱えるようにする方法を紹介します。


# 解決策

```sh
cat simulation.log |
jq -scR '
    split("\n")[:-1]
    |map(split("\t"))
    |map(
        if .[0] == "REQUEST" then
            {
                "record_header": .[0],
                "path": .[2],
                "start_at_msec": .[3],
                "end_at_msec": .[4],
                "status": .[5],
                "message": (if .[6] != " " then .[6] else "" end)
            }
        elif .[0] == "RUN" then
            {
                "record_header": .[0],
                "simulation_class": .[1],
                "simulation_id": .[2],
                "start_at_msec": .[3],
                "gatling_version": .[5]
            }
        elif .[0] == "USER" then
            {
                "record_header": .[0],
                "scenario": .[1],
                "event": .[2],
                "occured_at_msec": .[3]
            }
        else
            empty
        end)
    |.[]
'
```
