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

- OS: Windows/MacOS X/Linux
- インストールSW
    * MacOS X:
        - Homebrew
        - Vagrant 1.8.6+
        - VirtualBox
    * Windows
        - Vagrant 1.8.6+
        - VirtualBox
    * Linux
        - Vagrant 1.8.6+
        - VirtualBox[^1]

以降の手順は、ほとんどの操作をコマンドラインツールから実行します。Mac/Linuxの場合はターミナル、Windowsの場合はWindows PowerShell利用するものとして、手順を記述します。

[^1]: Linux(ubuntu 16.04)で、VirtualBox 5.1.x以上のバージョンでは動作しない問題を確認しています。当該OSを利用する場合は5.0.xの利用を推奨します。


 0 . 勉強会の現場で作業される方へ
---------------------------------
_この章は[Kubernetes ときどき Serverless -- cndjp第1回](https://cnd.connpass.com/event/69971/)の現地で作業される方向けの手順です。それ以外の方は、「1 . Kubernetesクラスターのセットアップ」に進んでください。_

KubernetesクラスターのインストールスクリプトをUSBメモリで配布しています。適当なフォルダにコピーしてください。

コマンドラインツールを起動し、インストールスクリプトのトップをカレントディレクトリに変更してください。

    > cd ~/cndjp/kubernetes-vagrant-coreos-cluster-cndjp1

配布したインストールスクリプトには、インストールを短縮するため、CoreOSのVagrant Boxが同梱してあります。以下のコマンドで、boxをインポートしてください。

    > vagrant box add --name=coreos-alpha-cndjp1 boxes\coreos-alpha-cndjp1.box

以上の手順が終わったら、「wgetのインストール（Macのみ）」に進んでください。


 1 . Kubernetesクラスターのセットアップ
-------------------------------------

### インストールスクリプトの取得
コマンドラインツールを起動し、適当なディレクトリをカレントディレクトリにします。このディレクトリに、Kubernetesクラスターのインストールスクリプト一式をダウンロードします。

    > mkdir cndjp
    > cd cndjp

インストールスクリプトには、Core OSの仮想マシン上で実行されるものが含まれます。これらは`git clone`時に改行コードが変換されてしまうと、Core OS上で正しく動作しない場合があります。これを避けるため、改行コードの自動変換を無効化するようにGitを設定しておきます。<br>
以下は、CLIのgitクライアントを利用してこの設定を行う例です。

    > git config --global core.autocrlf false
    > git config --global core.eol lf

[Kubernetesクラスターのインストールスクリプト](https://github.com/pires/kubernetes-vagrant-coreos-cluster)を、任意のGitクライアントを使ってクローンします。この時、タグ”1.7.10”を指定します。<br>
以下は、CLIのgitクライアントを利用する例です。

    > git clone -b 1.7.10 https://github.com/pires/kubernetes-vagrant-coreos-cluster.git

クローンして作成されたフォルダのトップを、カレントディレクトリにしておきます。

    > cd kubernetes-vagrant-coreos-cluster

### wgetのインストール（Macのみ）
wgetがインストールされていなければ、ここでインストールしておきます。

    > brew install wget

wgetはインストールスクリプトの実行時に、kubectlの取得のために利用されます。

### NFSのインストール（Linuxで該当する環境のみ）
今回作成するクラスターは、Kubernetesノードの共有ディスクとしてNFSを利用します。NFSがインストールされていなければ、新たにインストールする必要があります。この作業は、Windows/Macの場合は不要です。<br>
以下のコマンドは、debian/ubuntu系のLinuxでNFSをインストールする例です。

    > apt-get install nfs-kernel-server

WindowsではNFSと同等の機能を提供するVagrantプラグインが、自動的に利用されます。Macの場合は標準でNFSがインストールされています。

### Firewall無効化の確認
NFSへの通信がFirewallによって遮断されてしまうことが多いので、ここで無効化しておきます。

### インストールスクリプトの実行
インストールスクリプトを実行する前に、作成するクラスターの構成を決めます。デフォルトでは、以下のような構成のクラスターが構築されます。

- マスターノード:
    * CPU: 2 vCPUs
    * メモリ: 1024 MB
- メンバーノード:
    * CPU: 2 vCPUs
    * メモリ: 2048 MB
- メンバーノード数: 2

デフォルトの構成でインストールスクリプトを実行するには、以下のコマンドを実行します。

    > vagrant up

ご利用のPCが上記のスペックを賄うには非力な場合には、適宜パラメータを調整します。例えば、Macbook Air - Late 2011 4G Memでは、以下の構成でクラスターが稼働することを確認します。

- マスターノード:
    * CPU: 1 vCPUs
    * メモリ: 1024 MB
- メンバーノード:
    * CPU: 1 vCPUs
    * メモリ: 1024 MB
- メンバーノード数: 2

この構成でインストールスクリプトを実行するには、以下のコマンドを実行します。

    > NODES=1 MASTER_MEM=1024 MASTER_CPUS=1 NODE_MEM=1024 NODE_CPUS=1 vagrant up

パラメータの詳細は、[インストールスクリプトのREADME.md](https://github.com/pires/kubernetes-vagrant-coreos-cluster/blob/master/README.md)を参照してください。

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

### クラスターの停止/再起動
ハンズオン終了後に、クラスターを停止/再起動する際には、以下の操作を行ってください

- 停止

    > vagrant halt

- 起動

    > vagrant up

クラスターの仮想マシン削除するには、以下のクラスターの停止後に、以下のコマンドを実行してください。

    > vagrant destroy


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


 3 . 最初のアプリケーションを動かしてみる
---------------------------------------

### サンプルアプリケーションのデプロイ
Kubernetesクラスターに、サンプルアプリケーションをデプロイしてみます。以下のコマンドを実行します。[^2]

    > kubectl run kubernetes-bootcamp --image=docker.io/jocatalin/kubernetes-bootcamp:v1 --port=8080

[^2]: このアプリケーションは、Kubernetesのインタラクティブ・チュートリアルと同じものを利用しています。

自動的に、このアプリケーションに対応するデプロイメントオブジェクトが作成されています。デプロイメントオブジェクトの情報を確認するには、以下のコマンドを実行します。

・デプロイメントオブジェクトのリスト

    > kubectl get deployments

・デプロイメントオブジェクトの詳細情報

    > kubectl describe deployments/kubernetes-bootcamp

### プロキシの構成
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

### アプリケーションへのアクセス
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

