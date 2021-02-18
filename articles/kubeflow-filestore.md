---
title: "Kubeflow で Filestore を使って画像の受け渡しをする"
emoji: "🐙"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["kubeflow", "GKE", "filestore"]
published: true
---

# はじめに

この記事では、Kubeflow で Filesotre を使い、コンポーネント間で画像を受け渡す手順を簡単な例とともに紹介します。

Kubeflow で Filestore を使う場面として、 画像を入力としたモデルの学習・推論をするため、画像をダウンロードするコンポーネントから、学習・推論をするコンポーネントへ画像を受け渡したいときなどがあげられます。

他の選択肢として GCS もありますが、 時間がかかるので Filestore を選択しました。


# 手順

事前に GCP の Cloud Console なり、 terraform なりで Filestore のインスタンスを立ておきます。

## Kubernetes リソースの作成

Kubernates の PersistentVolume と PersistentVolumeClaim リソースを作成します。 まず、以下の yaml を手元のマシンなどに作成します。

```yaml:k8s/pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: <PV NAME>
  namespace: kubeflow
spec:
  capacity:
    storage: 1T
  accessModes:
    - ReadWriteMany
  nfs:
    server: <NFS SERVER IP>
    path: <NFS PATH>
```

`<NSF SERVER IP>`, `<NFS PATH>` は、 **Cloud Console > Filestore インスタンスの詳細画面 > NFS マウントポイント** で確認できます（ `<NFS SERVER IP>:<NFS PATH>` という形式になっています）。 `<PV NAME>` は任意に決めてください。

```yaml:k8s/pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: <PVC NAME>
  namespace: kubeflow
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: ""
  volumeName: <PV NAME>
  resources:
    requests:
      storage: 10G
```

`<PVC NAME>` も任意に決めてください。 `<PV NAME>` は上の `pv.yaml` で決めた名前を指定してください。

以下のコマンドでリソースを作成します。

```bash
$ kubectl create -f k8s/
```

※ `k8s/` には上の2つの yaml が入っています。


## Pipeline の作成と Run の実行

Python の [Kubeflow Pipelines SDK](https://kubeflow-pipelines.readthedocs.io/en/latest/) を用いて、 Pipeline の作成と Run を実行します。

以下の python スクリプトは Pipline の作成と Run を実行するスクリプトの例になります。画像の受け渡しを想定していますが、以下のスクリプトでは簡単のため、テキストファイルを Filestore 経由で渡しています。

```py:build_run.py
import kfp
import kfp.dsl as dsl


def write_op():
    """
    foo と書かれた file1 を Filestore に保存する。
    """
    return dsl.ContainerOp(
        name="write",
        image="library/bash:4.4.23",
        command=["sh", "-c"],
        arguments=["echo foo > /mnt/file1; echo /mnt/file1 > /tmp/output1.txt"],
        pvolumes={"/mnt": dsl.PipelineVolume(pvc="<PVC NAME>")},
        file_outputs={
            "output1": "/tmp/output1.txt",
        },
    )


def read_op(path):
    """
    Filestore に保存された file1 の中身をログに表示する。
    """
    return dsl.ContainerOp(
        name="read",
        image="library/bash:4.4.23",
        command=["cat", path],
        pvolumes={"/mnt": dsl.PipelineVolume(pvc="<PVC NAME>")},
    )


@dsl.pipeline(name="VolumeOp Basic", description="A Basic Example on VolumeOp Usage.")
def write_and_read():
    write_task = write_op()
    # file1 のパスを次のコンポーネントにわたす
    read_task = read_op(write_task.outputs["output1"])


if __name__ == "__main__":
    # build
    pipeline_file = "filestore-example-pipeline.yaml"
    kfp.compiler.Compiler().compile(write_and_read, pipeline_file)
    # run
    client = kfp.Client()
    client.create_run_from_pipeline_package(
        pipeline_file=pipeline_file,
        arguments={},
    )

```

わざわざ `file1` のパスを `read_op` に渡してる理由は、 渡さないと 「`write_op` → `read_op`」という順で実行されず、並列で実行されてしまうからです。 `write_op` から `read_op` に何かしらの output を渡せば依存関係を定義できるので、渡す値は `file1` のパスでなくても大丈夫です。画像の場合は、画像が入ったディレクトリのパスなどを渡せばよいと思います。


以下のコマンドを実行することで、 `kfp.Client()` の引数が空でも Kubeflow にアクセスできるようにします。

```bash
gcloud containers clusters get-credentials $CLUSTER_NAME
```


`build_and_run.py` を実行することで、 Kubeflow 上で Run が実行されます。

Kubeflow の Web UI から `read_op` の Logs に `file1` の内容が表示されてることを確認します。

```
foo
```
