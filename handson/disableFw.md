Windows 10 / Windwos Defender利用時に必要なFirewallの設定変更
=============================================================

これは、CNDJP第1回勉強会のハンズオン（前半分）のチュートリアルで、Windows 10 / Windwos Defender利用の際に必要なFirewallの設定変更の手順を記したものです。<br>

ハンズオン・チュートリアルを実施中でなければ、[そちらの手順](./handson1.md)から始めてください。

設定変更の手順
--------------

Windowsのタスクバーから、Windows Defenderのアイコンをダブルクリックします。

![01_defender_in_tasktray](./images/01_defender_in_tasktray.png)


［Windows Devender セキュリティ センター］ウィンドウで、［ファイアウォールとネットワーク保護］をクリックします。

![02_select_firewall_settings](./images/02_select_firewall_settings.png)


［ファイアウォールによるアプリケーションの許可］をクリックします。

![03_select_application_settings](./images/03_select_application_settings.png)


［許可されたアプリ］ウィンドウで、［設定の変更］ボタンをクリックして、設定変更の権限を取得します。

![04_unlock_permission](./images/04_unlock_permission.png)


VagrantWinNFSdのプライベートネットワークのチェックを外し（プライベートネットワーク経由での通信を許可）、[OK]をクリックします。

![05_allow_vagrantWinNfsd](./images/05_allow_vagrantWinNfsd.png)


以上。
