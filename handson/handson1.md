CNDJP #1 ハンズオンチュートリアル
=================================
これは、CNDJP第1回勉強会のハンズオン（前半分）のチュートリアルです。
ローカルのPC上に、VirtualBoxの仮想マシンを利用してKubernetesのマルチノードクラスターを構成します。<br>
また、そのKubernetesクラスターに対して、kubectlをによる基本的な管理操作を試してみます。


    --------------------------
    |      k8s cluster       |  <-- (管理操作)
    --------------------------          |
    |   vm   |   vm  |   vm  |          |
    --------------------------     -----------
    | Hypervisor(VirtualBox) |     | kubectl |
    ------------------------------------------
    |                   OS                   |
    ------------------------------------------


前提条件
--------
このチュートリアルは、以下の環境で実施することを前提とします。

- OS: Windows/Mac/Linux
- インストールSW
    * Vagrant
    * VirtualBox[^1]

以降の手順は、ほとんどの操作をコマンドラインツールから実行します。Mac/Linuxの場合はターミナル、Windowsの場合はWindows PowerShell利用するものとして、手順を記述します。

[^1]: Linux(ubuntu 16.04)で、5.1.x以上のバージョンでは動作しない問題を確認しています（2017/11/12時点）。ご注意ください。


 1 . Kubernetesクラスターのセットアップ
-------------------------------------
コマンドラインツールを起動し、適当なディレクトリをカレントディレクトリにします。このディレクトリに、Kubernetesクラスターのインストールスクリプト一式をダウンロードします。

    > mkdir cndjp
    > cd cndjp

インストールスクリプトには、Core OSの仮想マシン上で実行されるものが含まれます。これらは`git clone`時に改行コードが変換されてしまうと、Core OS上で正しく動作しない場合があります。これを避けるため、改行コードの自動変換を無効化するようにGitを設定しておきます。

    > git config --global core.autocrlf false

[Kubernetesクラスターのインストールスクリプト](https://github.com/pires/kubernetes-vagrant-coreos-cluster)を取得します。

    > git clone https://github.com/pires/kubernetes-vagrant-coreos-cluster.git

cloneで作成されたフォルダのトップを、カレントディレクトリにします。

    > cd kubernetes-vagrant-coreos-cluster

今回作成するクラスターは、Kubernetesノードの共有ディスクとしてNFSを利用します。NFSがインストールされていなければ、新たにインストールする必要があります。<br>
以下のコマンドは、debian/ubuntu系のLinuxでNFSをインストールする例です。

    > apt-get install nfs-kernel-server

Windows、Macの場合は、この手順は必要ありません。WindowsではNFSと同等の機能を提供するVagrantプラグインが、自動的に利用されます。Macの場合は既にNFSがインストールされています。

また、NFSへの通信はFirewallによって遮断されてしまうことが多いので、ここで無効化しておきます。

最後にインストールスクリプトを実行します。

    > vagrant up

コンソールに以下のような内容が出力されれば、クラスターの構築が成功しています。

    > vagrant up
    Bringing machine 'master' up with 'virtualbox' provider...
    Bringing machine 'node-01' up with 'virtualbox' provider...
    Bringing machine 'node-02' up with 'virtualbox' provider...
    ==> master: Running triggers before up...
    ==> master: 2017-11-15 01:25:08 +0900: setting up Kubernetes master...
    ...（中略）
        node-02: Running: inline script
    ==> node-02: Running triggers after up...
    ==> node-02: Waiting for Kubernetes minion [node-02] to become ready...
    ==> node-02: 2017-11-15 01:45:39 +0900: successfully deployed node-02


最後に、全ての仮想マシンが稼働していることを確認します。

    > vagrant status
    Current machine states:
    
    master                    running (virtualbox)
    node-01                   running (virtualbox)
    node-02                   running (virtualbox)


2 . kubectlのセットアップ
------------------------
ここでは、Kubernetesの管理操作を行うためのCLIである、kubectlをセットアップします。

### kubectlのインストール(Windowsユーザー向け)
（Mac/Linuxの場合は自動でkubectlがインストールされるため、この手順は不要です。「Kubernetesクラスターへの疎通確認」に進んでください。）

kubectlのバイナリファイルを、以下からダウンロードします。

- [kubectl v1.8.0](https://storage.googleapis.com/kubernetes-release/release/v1.8.0/bin/windows/amd64/kubectl.exe)

次に、ダウンロードしたバイナリを適当なフォルダに配置し、PATHを通しておきます。

PowerShellを再起動の後、以下のコマンドを実行して、PATHの設定が正しいことを確認します。

    > kubectl version

続いて、Kubernetesクラスターにアクセスするための設定を行います。インストールスクリプト一式のトップをカレントディレクトリにします。

    > cd cndjp\kubernetes-vagrant-coreos-cluster

以下のコマンドを順に実行していきます。

    > kubectl config set-cluster default-cluster --server=https://172.17.8.101 --certificate-authority=%CD%/artifacts/tls/ca.pem
    > kubectl config set-credentials default-admin --certificate-authority=%CD%/artifacts/tls/ca.pem --client-key=%CD%/artifacts/tls/admin-key.pem --client-certificate=%CD%/artifacts/tls/admin.pem
    > kubectl config set-context default-cluster --cluster=default-cluster --user=default-admin
    > kubectl config use-context default-cluster

### Kubernetesクラスターへの疎通確認
以下のコマンドを実行して、Kubernetesクラスターへの疎通を確認します。

・クラスター情報の取得

    > kubectl cluster-info
    Kubernetes master is running at https://172.17.8.101
    KubeDNS is running at https://172.17.8.101/api/v1/namespaces/kube-system/services/kube-dns/proxy
    
    To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

・ノードの一覧の取得

    > kubectl get nodes
    NAME           STATUS                     AGE       VERSION
    172.17.8.101   Ready,SchedulingDisabled   12h       v1.7.5
    172.17.8.102   Ready                      12h       v1.7.5
    172.17.8.103   Ready                      12h       v1.7.5


2 . 最初のアプリケーションを動かしてみる
---------------------------------------
Kubernetesクラスターに、サンプルアプリケーションをデプロイしてみます。以下のコマンドを実行します。[^2]

    > kubectl run kubernetes-bootcamp --image=docker.io/jocatalin/kubernetes-bootcamp:v1 --port=8080

[^2]: このアプリケーションは、Kubernetesのインタラクティブ・チュートリアルと同じものを利用しています。

自動的に、このアプリケーションに対応するデプロイメントオブジェクトが作成されています。デプロイメントオブジェクトの情報を確認するには、以下のコマンドを実行します。

・デプロイメントオブジェクトのリスト

    > kubectl get deployments

・デプロイメントオブジェクトの詳細情報

    > kubectl describe deployments/kubernetes-bootcamp

アプリケーション稼働しているコンテナを公開します。ここでは、KubernetesのAPIサーバーに対するプロキシを構成し、APIサーバー経由でアプリケーションにアクセスします。
以下のコマンドで、プロキシを構成します。

    > kubectl proxy

APIサーバーへのアクセスを確認します。curlがインストールされている場合は以下のコマンドを実行します。インストールされていなければ、ブラウザで同じURLにアクセスします。

・Mac/Linux

    > curl http://localhost:8001/version

・Windows

    > Invoke-RestMethod -Uri "http://localhost:8001/version"

Kubernetesのバージョン情報が、以下のようなJSON形式で取得できます。

```json
{
  "major": "1",
  "minor": "7",
  "gitVersion": "v1.7.5",
  "gitCommit": "17d7182a7ccbb167074be7a87f0a68bd00d58d97",
  "gitTreeState": "clean",
  "buildDate": "2017-08-31T08:56:23Z",
  "goVersion": "go1.8.3",
  "compiler": "gc",
  "platform": "linux/amd
}
```

いよいよアプリケーションにアクセスしてみます。以下のコマンドを順に実行してください。

・Mac/Linux

    export POD_NAME=$(kubectl get pods -o go-template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}')
    curl http://localhost:8001/api/v1/proxy/namespaces/default/pods/$POD_NAME/

・Windows

    $POD_NAME=kubectl get pods -o go-template --template '{{range .items}}{{.metadata.name}}{{\"\n\"}}{{end}}'
    Invoke-RestMethod -Uri "http://localhost:8001/api/v1/proxy/namespaces/default/pods/$POD_NAME/"

正しく動作していれば、以下のようなレスポンスが返却されます。

    Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-2457653786-vdgr2 | v=1


ハンズオンの前半の内容は、以上です。


参考リンク
----------

- kubectl リファレンス(v1.7):
    * [https://v1-7.docs.kubernetes.io/docs/user-guide/kubectl/v1.7/](https://v1-7.docs.kubernetes.io/docs/user-guide/kubectl/v1.7/)
- Kubernetes API リファレンス(v1.7):
    * [https://v1-7.docs.kubernetes.io/docs/api-reference/v1.7/](https://v1-7.docs.kubernetes.io/docs/api-reference/v1.7/)
- Kubernetes Basics チュートリアル
    * [https://kubernetes.io/docs/tutorials/kubernetes-basics/](https://kubernetes.io/docs/tutorials/kubernetes-basics/)

