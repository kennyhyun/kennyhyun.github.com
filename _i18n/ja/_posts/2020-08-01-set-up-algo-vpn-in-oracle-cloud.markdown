---
layout: post
title:  "無料VPS、Oracle cloud使ったAlgo VPNの紹介"
date:   2020-08-01 20:41:01 +1000
categories: IT
---

たまに海外のVPNサーバーがいる時がありますが、そのためにわざわざ月額払ってまで使いたくはないですよね。

しかし、既に仮想サーバーが海外に既にある場合はどうでしょうか？最近は無料のサーバーも結構あるようです。

以前の記事で予告したように、Oracle CloudにAlgo VPNサーバーをインストールしてみました。

皆さんにこの情報が役立てればと思います。

### 免責

VPNを使用に関しては各自の責任で御使用願います。私はOracle CloudやVPNの使用について一切の責任を負いません。

### インスタンス生成

1. ここ[Oracle Cloud](https://www.oracle.com/au/cloud/free/){:target="_blank"}でアカウントを作成します
    - ホームリージョンは、VPNを使用したい場所に設定します
1. Webコンソールにアクセスして「Create a VM instance」ボタンをクリックします
  ![Insttance1](/assets/images/2020/vpn/instance1.jpg)
    - Ubuntu minimalを選んでpublic keyをアップロードします
        - sshキーがない場合は、次を参考に作成しますhow to create ssh key
1. 「create」をクリックすると、次のように詳細を確認することができます
  ![Insttance2](/assets/images/2020/vpn/instance2.jpg){:width="80%"}
    - 公認のIPアドレスをメモっておきます

さて、生成が完了するとssh経由でインスタンスに接続できます。

### Algo VPNのインストール

上記の生成したインスタンスに接続して、次のようにVPNサーバーをインストールします。

#### Aptパッケージのインストール

```sh
$ sudo apt update && sudo apt upgrade -y
$ sudo apt install -y python3-virtualenv unzip vim
```
希望のエディタも忘れずに追加します。vimが慣れてない方は代わりnanoを使用しましょう。

#### Algoのダウンロード

```sh
$ wget https://github.com/trailofbits/algo/archive/master.zip
$ unzip master.zip
```

#### virtualenvの設定

```sh
$ cd algo-master
$ python3 -m virtualenv --python="$(command -v python3)" .env &&
  source .env/bin/activate &&
  python3 -m pip install -U pip virtualenv &&
  python3 -m pip install -r requirements.txt
```

#### ユーザーの追加

`config.cfg`を編集して、ユーザーを追加します。

```sh
$ vi config.cfg
```

ユーザーを共有すると時々問題があるそうなので、次のように十分に追加しておくことをお勧めします。

```yml
users:
  - phone
  - phone2
  - tablet
  - tablet2
  - laptop
  - laptop2
  - desktop
  - desktop2
  - server
  - server2
  - guest
  - guest2
```

#### 配布スクリプトの実行

次は対話型のスクリプトです。いくつかの選択肢に答えると、スクリプトがインストールを行います。

```sh
$ ./algo
```

```sh
[Cloud prompt]
What provider would you like to use?
--> 11. Install to existing Ubuntu 18.04 or 20.04 server (for more advanced users)
```

最初の質問が最も重要です。現在接続したサーバーにインストールするので11番を選択します。

残りのほとんどはデフォルトの設定でエンターを押すだけで良いですが、次の２つの質問に違う答えを入れましょう。

```sh
[Retain the PKI prompt]
Do you want to retain the keys (PKI)? (required to add users in the future, but less secure)
[y/N]
--> Y
```

```sh
[local : pause]
Enter the public IP address or domain name of your server: (IMPORTANT! This is used to verify the certificate)
[localhost]
---> [Put the public IP adress of your instance]
```

これにより、次のように出力され、インストールが終了します。

```python
'#                       Congratulations!                         #'
'#                  Your Algo server is running.                  #'
'# Config files and certificates are in the ./configs/ directory. #'
'#           Go to https://whoer.net/ after connecting            #'
'#     and ensure that all your traffic passes through the VPN.   #'
'#                  Local DNS resolver 172.29.**.**               #'
'#     The p12 and SSH keys password for new users is *********   #'
'#     The CA key password is ****************                    #'
```

何かが間違ったら、最初から再インストールすることが良いと言われます。現在のインスタンスを破棄し、新しいインスタンスを再作成します。

#### 接続情報のコピー

次はVPNサーバーのssh接続は終了してもいいです。

今使用しているローカル環境でscpコマンドで接続情報をコピーします。次の「vpnserver」はVPNサーバーのパブリックIPに置き換えます。

```sh
$ scp -r ubuntu@vpnserver:algo-master/configs .
```

それでは「configs」ディレクトリにある接続情報が使えます。

### VPNサーバーのポートの開放

VPNサーバー自体は用意できてますが、あとひとステップが残っています。

VPN接続に必要な[ポート][algo-vpn-ports]{:target="_blank"}は、通常閉じてあるので、手動で開けなければなりません。

#### VPN接続に使用するSecurity Listの作成

Oracle Cloudのウェブコンソールに接続します。

1. Networking > Virtual Cloud Networksを開きます
1. 右のVCN nameをクリックします
1. 左の「Security Lists」をクリックします
1. 「Create Security List」をクリックします
1. ポップアップには、次のように入力します
    - Name: VPN
    - Additional Ingress Rule: Ingress Rule 1
        - Source CIDR: 0.0.0.0/0
        - Destination Port Range: 4160
    - Additional Ingress Rule: Ingress Rule 2
        - Source CIDR: 0.0.0.0/0
        - IP Protocol: UDP
        - Destination Port Range: 51820,500,4500
1. `Create Security List`をクリックして確定します

![vpn ports](/assets/images/2020/vpn/vpn.ports.jpg)

#### Security ListをPublic Subnetに繋ぐ

1. Instace Details (Compute > Instaces > Instance Details)を開きます
1. Primary VNICのSubnetをクリックします
1. 「Add Security List」をクリックします
1. ポップアップのsecurity listプルダウンのVPNを選択します
1. 「Add Security List」をクリックして確定します

### Clientの設定

取りあえずアイパッドにての設定法について説明します。

1. AppstoreでWireGuardをインストールします（iOS 12以降が必要です）
1. `configs/localhost/wireguard/tablet.png`のようにユーザーに合ったPNGのQRコードを開きます
  「configs」ディレクトリは、前の手順でコピーしておいたものです
1. WireGuardを実行し、「+」ボタンをタッチして「Create from QR code」を選択します
1. QR code撮ってtabletのように名前を付けときます
1. その生成されたtabletの接続をオンにします

これでVPNに接続されます。

ブラウザを開いて「my ip」を検索するとVPN serverのアドレスが表示されるか確認します。

WireGuardアプリはMac and Windows 10にても使用可能なので、似たような方法で設定でます。

### トラブルシューティング

サーバーに接続することができなくてもWireGuardアプリは黙ってインターネットが不通になります。
インターネット接続に問題がある場合VPN接続を終了し、サーバーの問題を解決した後、もう一度お試しください。

### もっと読む

- [Algo VPN](https://github.com/trailofbits/algo#deploy-the-algo-server){: target="_blank" }

[algo-vpn-ports]: https://github.com/trailofbits/algo/blob/master/docs/firewalls.md
[howto-ssh]: https://docs.github.com/en/github/authenticating-to-github/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent
