---
layout: post
title:  "무료 VPS Oracle cloud에 Algo VPN사용하기"
date:   2020-08-01 20:41:01 +1000
categories: IT
---

가끔 해외 VPN서버를 사용하면 좋을 때가 있지만 꼭 그걸위해 매달 비용을 지불해가며 쓰고 싶진 않기도 하죠.

하지만 가상서버가 해외에 이미 있다면 어떨까요? 요즘엔 공짜 서버도 좀 있는것 같군요.

이전 포스트에서 예고한 대로, Oracle Cloud에 Algo VPN서버를 설치해 봤습니다.

여러분에게 이 정보가 도움이 되길 바랍니다.

### 면책

VPN사용에 관해선 각자의 책임하에 사용하시길 바랍니다.
저는 Oracle Cloud나 VPN사용에 대해 어떠한 책임도 지지 않습니다.

### 인스턴스 생성

1. 이곳 [Oracle Cloud](https://www.oracle.com/au/cloud/free/){:target="_blank"} 에서 계정을 생성합니다.
    - 홈 리전은 VPN 사용을 원하는 곳으로 설정 합니다.
1. 웹 콘솔에 접속하여 `Create a VM instance` 버튼을 클릭합니다
  ![Insttance1](/assets/images/2020/vpn/instance1.jpg)
    - ubuntu minimal을 고르고 public key를 업로드 합니다
        - ssh 키가 없다면 다음을 참고해 생성합니다 [how to create ssh key][howto-ssh]{:target="_blank"}
1. create 를 클릭하면 다음과 같이 상세를 확인 할 수 있습니다.
  ![Insttance2](/assets/images/2020/vpn/instance2.jpg){:width="80%"}
    - public IP 를 메모해 둡니다

이제 생성이 완료되면 ssh를 통해 인스턴스로 접속 할 수 있습니다.


### Algo VPN 설치

위에서 생성한 인스턴스에 접속하고 다음과 같이 VPN서버를 설치합니다

#### Apt 패키지 설치

```sh
$ sudo apt update && sudo apt upgrade -y
$ sudo apt install -y python3-virtualenv unzip vim
```

원하는 에디터도 잊지않고 추가합니다. vim이 익숙치 않으시면 대신 nano를 사용하세요

#### Algo 다운로드

```sh
$ wget https://github.com/trailofbits/algo/archive/master.zip
$ unzip master.zip
```

#### virtualenv 설정

```sh
$ cd algo-master
$ python3 -m virtualenv --python="$(command -v python3)" .env &&
  source .env/bin/activate &&
  python3 -m pip install -U pip virtualenv &&
  python3 -m pip install -r requirements.txt
```

#### 사용자 추가

`config.cfg` 를 편집하여 사용자를 추가합니다.

```sh
$ vi config.cfg
```

사용자를 공유하는 것에는 가끔 문제가 있다고 하므로 다음과 같이 넉넉히 추가하는것을 추천합니다.

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

#### 배포스크립트 실행

다음은 대화형 스크립트 입니다.
몇가지 선택사항에 대해 응답하고 나면 스크립트가 설치를 완료합니다.

```sh
$ ./algo
```

```sh
[Cloud prompt]
What provider would you like to use?
--> 11. Install to existing Ubuntu 18.04 or 20.04 server (for more advanced users)
```

첫 질문이 가장 중요합니다.
현재 접속한 서버에 설치할 것 이므로 11번을 선택합니다.

나머지 대부분은 기본 설정으로 엔터만 눌러 진행하면 되지만 다음 두 질문들에는 답변을 따로 해 줍니다.

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

그러면 다음과 같이 출력이되고 설치가 완료 됩니다.

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

뭔가 잘못되면 처음부터 다시 설치하는 것이 권장됩니다. 현재 인스턴스를 폐기하고 새로운 인스턴스를 재생성 합니다.

#### 접속정보 복사

이제 VPN 서버의 ssh접속은 종료해도 됩니다.

지금 사용하는 로컬 환경에서 scp명령으로 접속정보를 복사합니다.
아래의 `vpnserver`은 VPN서버의 공인 IP로 대체합니다.

```sh
$ scp -r ubuntu@vpnserver:algo-master/configs .
```

이제 `configs` 디렉토리에 있는 접속정보를 사용 할 수 있습니다.

### VPN 서버 포트 열기

VPN 서버 자체는 준비가 되었지만 아직 한가지 스텝이 남아 있습니다.

VPN접속에 필요한 [포트들][algo-vpn-ports]{:target="_blank"}은 보통 닫혀있으므로 수동으로 열어줘야 합니다.

#### VPN 접속에 사용할 Security List 생성 

Oracle Cloud 웹 콘솔에 접속합니다.

1. Networking > Virtual Cloud Networks 로 갑니다
1. 오른쪽의 VCN name을 클릭합니다
1. 왼쪽의 `Security Lists`를 클릭합니다
1. `Create Security List`를 클립합니다
1. 팝업창에 다음과 같이 입력합니다
    - Name: VPN
    - Additional Ingress Rule: Ingress Rule 1
        - Source CIDR: 0.0.0.0/0
        - Destination Port Range: 4160
    - Additional Ingress Rule: Ingress Rule 2
        - Source CIDR: 0.0.0.0/0
        - IP Protocol: UDP
        - Destination Port Range: 51820,500,4500
1. `Create Security List`을 클릭하여 확정합니다

![vpn ports](/assets/images/2020/vpn/vpn.ports.jpg)

#### Security List를 Public Subnet에 연결하기

1. Instace Details (Compute > Instaces > Instance Details)로 갑니다
1. Primary VNIC의 `Subnet`을 클릭합니다
1. `Add Security List`을 클릭합니다
1. 팝업창에서 security list 풀다운에 `VPN`을 선택합니다
1. `Add Security List`를 클릭하여 확정합니다

### Client 설정하기

일단 iPad를 기준으로 설정방법에 대해 설명하겠습니다

1. Appstore에서 WireGuard 를 설치합니다 (iOS12 또는 상위버전이 필요합니다)
1. `configs/localhost/wireguard/tablet.png`와 같이 사용자에 맞는 PNG QR 코드를 엽니다
  `configs` 디렉토리는 이전단계에서 복사해 둔 것입니다
1. WireGuard 를 실행해 `+` 버튼을 터치하고 `Create from QR code`를 선택합니다
1. QR code를 찍어서 `tablet`과 같이 이름을 붙입니다.
1. 생성된 해당 연결을 켭니다

이것으로 VPN에 연결되었습니다

브라우저를 열어 `my ip`를 검색하면 VPN server의 주소가 표시될 것입니다

WireGuard 앱은 Mac and Windows10 에서도 사용 가능하므로 비슷한 방법으로 설정 가능 합니다.

### 문제해결

서버에 연결할 수 없어도 WireGuard 앱은 조용히 실패하고 인터넷이 불통이 됩니다
인터넷 연결에 문제가 있으면 VPN 커넥션을 종료하고 서버 문제를 해결한 다음에 다시 시도 하세요

### 더 읽어보기

- [Algo VPN](https://github.com/trailofbits/algo#deploy-the-algo-server){: target="_blank" }

[algo-vpn-ports]: https://github.com/trailofbits/algo/blob/master/docs/firewalls.md
[howto-ssh]: https://docs.github.com/en/github/authenticating-to-github/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent
