CNDJP #1 ハンズオンチュートリアル
=================================
これは、CNDJP第1回勉強会のハンズオン（後半分）のチュートリアルです。

前提条件
--------
[前半分のチュートリアル](https://github.com/oracle-japan/cndjp1/blob/master/handson/handson1.md)が完了していることを前提としています。


1.Podの状態の確認方法
---------------------

### Podの標準出力
Kubernetes上で動作するアプリケーションの動作状況を確認する上で、最もシンプルな方法は、Pod標準出力確認することです。<br>
Podの標準出力を表示するには、以下のコマンドを実行します。

    > kubectl logs [Podの名前]

チュートリアルの前半を実施済みの場合、$POD_NAMEに現在稼働しているアプリケーションの、Podの名前が保存されています。

    > kubectl logs $POD_NAME

### Podの環境変数
Podに設定されている環境変数を確認するには、Podの環境上で`env`コマンドを実行する必要があります。<br>
Podの環境で任意のコマンドを実行するには`kubectl exec`コマンドを用います。

    > kubectl exec [Podの名前] [コマンド]

これを`env`と組み合わせて、現在稼働しているPodの環境変数を出力するには、以下のようなコマンドを実行します。

    > kubectl exec $POD_NAME env

`kubectl exec`を利用すると、Podのシェルを利用することもできます。

    > kubectl exec -it $POD_NAME /bin/bash

### 複数のPodの標準出力を一度にtailする
Podをスケールアウトさせた際には、複数のPodの標準出力を観察する必要がありますが、Sternを使うとこの作業が容易になります。
Sternは複数のPodの標準出力を、一箇所のコンソール上でまとめてtailするものです。

- [Stern](https://github.com/wercker/stern)

#### Sternのインストール手順(Windows/Linux)
ビルド済みのバイナリを以下からダウンロードしてPATHを設定します。

- [Sternのバイナリ](https://github.com/wercker/stern/releases)

#### Sternのインストール手順(Mac)
homebrewを使ってインストールします。

    > brew install stern

インストールが完了したら、以下のコマンドを実行してsternが利用できることを確認します。

    > stern -v

sternはデフォルトでkubectlの設定情報を利用して動作しますので、前半のチュートリアルでkubectlの設定ができていれば、特に追加の作業は必要ありません。

以下のコマンドで、Podの標準出力が表示されることを確認してください。

    > stern $POD_NAME

複数のPodの標準出力を同時に確認する場合には、Podの名前にワイルドカードを制定することができます。

    > stern kubernetes-bootcamp-*


2 . Serviceを使ったアプリケーションの公開
-------------------------------------

### Serviceの作成
Serviceを使ってKubernetes上で動くコンテナを公開してみます。<br>
以下のコマンドを実行します。

    > kubectl expose deployment/kubernetes-bootcamp --type="NodePort" --port 8080

これは、サンプルアプリケーションが稼働しているデプロイメントに対してServiceオブジェクトを構成しています。
このデプロイメントをスケールアウトさせると、自動的にデプロイメント内の複数のPodにルーティングが行われます。

このコマンドでは、Serviceを"NodePort"モードで、各Podの8080番ポートにルーティングすることを意味しています。

以下のコマンドで、指定したServiceが構成されていることを確認します。

    > kubectl get services
    > kubectl describe services/kubernetes-bootcamp


### Serviceを介したアプリケーションへのアクセス
NodePortモードでServiceを構成していますので、各メンバーノードの特定のポート番号から、アプリケーションにアクセスすることができます。

以下、この後の手順を簡単にするため、必要な値を環境変数に入れておきます。

はじめに、ポート番号です。

- Mac/Linux

    > export NODE_PORT=$(kubectl get services/kubernetes-bootcamp -o go-template='{{(index .spec.ports 0).nodePort}}')

- Windows

    > $NODE_PORT=kubectl get services/kubernetes-bootcamp -o go-template='{{(index .spec.ports 0).nodePort}}'

続いて、メンバーノードのホスト名です。

- Mac/Linux

    > export NODE01=172.17.8.102

    > export NODE02=172.17.8.103

- Windows

    > $NODE01=echo '172.17.8.102'

    > $NODE02=echo '172.17.8.103'

サービスにアクセスすると、前半のチュートリアルと同じ、サンプルアプリケーションの応答が返却されます。

- Mac/Linux

    > curl http://${NODE01}:${NODE_PORT}/

- Windows

    > Invoke-RestMethod -Uri "http://${NODE01}:${NODE_PORT}/"

アクセス先のNodeを変更しても同じように応答が返却されることも確認してみてください。


3 . アプリケーションのスケールアウト
------------------------------------
この時点では、サンプルアプリケーションは単ノードで稼働している状態です。デプロイメントに対してレプリカの数を指定することによって、デプロイメントに紐づくPodの数を変更することができます。

以下のコマンドを実行して、Podを4つに拡張してみます。

    > kubectl scale deployments/kubernetes-bootcamp --replicas=4

Podの状態を確認すると、以下の様に4つのPodが構成されます（一部のPodは起動中）。

    > kubectl get pods
    NAME                                   READY     STATUS              RESTARTS   AGE
    kubernetes-bootcamp-2457653786-bvhgb   0/1       ContainerCreating   0          24s
    kubernetes-bootcamp-2457653786-kvqdj   0/1       ContainerCreating   0          24s
    kubernetes-bootcamp-2457653786-mlbb8   1/1       Running             0          52m
    kubernetes-bootcamp-2457653786-txsp5   1/1       Running             0          24s

少し時間が経過すると全てのPodがReadyのステータスになります。

sternで標準出力を確認すると、以下の様にPodが追加されていることがわかります。

    + kubernetes-bootcamp-2457653786-txsp5 › kubernetes-bootcamp
    kubernetes-bootcamp-2457653786-txsp5 kubernetes-bootcamp Kubernetes Bootcamp App Started At: 2017-11-20T03:42:40.704Z | Running On:  kubernetes-bootcamp-2457653786-txsp5
    kubernetes-bootcamp-2457653786-txsp5 kubernetes-bootcamp
    + kubernetes-bootcamp-2457653786-bvhgb › kubernetes-bootcamp
    kubernetes-bootcamp-2457653786-bvhgb kubernetes-bootcamp Kubernetes Bootcamp App Started At: 2017-11-20T03:43:06.276Z | Running On:  kubernetes-bootcamp-2457653786-bvhgb
    kubernetes-bootcamp-2457653786-bvhgb kubernetes-bootcamp
    + kubernetes-bootcamp-2457653786-kvqdj › kubernetes-bootcamp
    kubernetes-bootcamp-2457653786-kvqdj kubernetes-bootcamp Kubernetes Bootcamp App Started At: 2017-11-20T03:43:08.604Z | Running On:  kubernetes-bootcamp-2457653786-kvqdj
    kubernetes-bootcamp-2457653786-kvqdj kubernetes-bootcamp

sternは複数のPodからの標準出力を自動的に色分けして表示します。実際に各Podからの出力が色で識別できることを確認してください。

何度かServiceにアクセスしてみると、複数のPodから応答が帰ってきていることを確認できます。

- Mac/Linux

    > curl http://${NODE01}:${NODE_PORT}/

- Windows

    > Invoke-RestMethod -Uri "http://${NODE01}:${NODE_PORT}/"


同様にして、Podを2つに縮小することも可能です。

    > kubectl scale deployments/kubernetes-bootcamp --replicas=2


以上。
