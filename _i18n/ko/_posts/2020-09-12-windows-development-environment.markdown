---
layout: post
title:  "Setting up Windows for developers with WSL2"
description: '요즘 개발자 사이에서 맥이 인기입니다. 하지만 최근 추가된 WSL 2 덕분에 윈도 10 홈 에디션도 리눅스와 도커를 거의 완벽하게 사용할 수 있게 되었습니다. 신제품의 경우는 맥보다 윈도 랩탑들이 더 가성비가 좋고 더 새로운 CPU를 더 일찍 만나 볼 수 도 있습니다. 혹시 남는 윈도 랩탑이 있다면 SSD를 장착하고 최신 윈도로 업데이트 해 보세요. 다음을 따라서 설치하게 되면 꽤 쾌적하고 전문적인 개발 환경을 손에 넣으실 수 있습니다.'
date:   2020-09-12 20:32:24 +1000
tags:
  - windows
  - windows-10
  - home-edition
  - wsl
  - wsl2
  - git
  - git-bash
  - windows-terminal
  - docker
  - docker-desktop
  - wsl-backend
  - camingo-code
  - tmux
  - vscode
  - vcxsrv
  - windows-defender-exclusion
  - antivirus-exclusion
  - ssh-wsl
categories: IT
---

요즘 개발자 분들 맥 많이들 사용하시죠. 개발 툴들 지원도 잘 되고 설치와 사용이 편해서 많이들 사용하시는 것 같습니다. 저도 한때 즐겨 사용하기도 했습니다만 버전업 되면서 마음에 안 드는 점들이 거슬리기도 하네요. 최근 저는 윈도 랩탑을 사용하고 있습니다만, 여러 개발 툴 사용이 좀 불편한 관계로 개발에는 오라클 버추얼박스를 이용해 리눅스를 사용하고 있습니다.

맥과 윈도는 단순히 OS만의 차이가 아니라 자판 배열이나 단축키 설정이 많이 달라집니다. 같은 크롬, VSCode 일지라도 단축키는 꽤 달라집니다. 이미 윈도를 사용하고 계신다면 맥에 적응하기 위해 단축키를 다시 익혀야 하고 또다시 되돌아오기도 그만큼 힘들어지게 됩니다. 저는 그런 경험을 거친 뒤 차라리 VIM을 메인으로 사용하기로 마음먹고 현재는 어떤 환경이든 동일하게 사용하고 있습니다.

최근 2020 5월 윈도 10 업데이트를 기준으로 WSL2가 정식 배포가 되었죠. 이제 윈도 홈 에디션에서도 도커를 사용할 수 있게 되었습니다.
저는 윈도 프로 버전에서 도커 데스크톱도 써 보았고, 인사이더 프리뷰를 통해 WSL2도 미리 접해보고 시행착오를 겪었는데요, Hyper-v는 버추얼박스와 동시에 동작하지 않습니다. Hyper-v를 버철머신으로 사용하는 것도 가능은 하지만 메모리 관리나 네트웍 사용이 아무래도 불편해서 버추얼 박스를 놓지 못하고 있네요. WSL2도 역시 Hyper-v 기반이기 때문에 버추얼 박스와 공존하지 못하고 불편함이 여전했습니다. 그래서 WSL2를 멀리하고 있었는데 최근 도커가 WSL2를 지원하는 것을 알고 다시 사용해 보게 되었습니다.

결과는 꽤 만족스럽습니다. 윈도 터미널, WSL2, 도커를 사용하면 설정하기에 따라 쾌적한 개발환경을 손에 넣을 수 있습니다. 다음을 따라 설정을 완료하면 어떠한 개발에도 손색이 없는 윈도/리눅스 개발환경을 구축 할 수 있습니다. 개발에 친숙한 유닉스 환경을 윈도에서도 손쉽게 도입하고 그 자유를 즐겨 보세요.

개요

- Git, Git bash
- Windows Terminal
- WSL2
  - Using Linux GUI apps
  - Ssh to WSL
- Docker
- VSCode
- Defender Antivirus Exclusion

## Windows 버전과 빌드 넘버

![winver](/assets/images/2020/wsl2/winver.jpg){:width="70%" style="margin: 0"}

`Wnd`+`R` 후 `winver` 를 입력하고 `enter` 하면 위와같이 버전정보를 볼 수 있습니다. 이미 2004 버전 (OS build 19041)를 가지고 있다면 바로 WSL2를 사용할 수 있습니다.

May 2020 업데이트 (version 2004 or build 19041) 는 WSL2를 지원하므로 Docker는 WSL2 백엔드를 기본 도커엔진으로 사용합니다. 때문에 이제는 윈도 홈 에디션도 개발에 무리가 없습니다.

만일 윈도 업데이트 화면에 2004버전이 나타나지 않는다면 다음의 [윈도 업데이트 툴](https://www.microsoft.com/en-au/software-download/windows10){:target="_blank"}을 이용해 업데이트 할 수 도 있습니다.

## 개발자 모드

윈도에서도 심링크를 사용할 수 는 있지만 Administrator 권한이나 개발자 모드가 필요합니다.
심링크가 있는 Git 리포지토리는 이것이 필요하게 됩니다.

![Developer mode](/assets/images/2020/wsl2/developer-mode.jpg){:width="70%" style="margin: 0"}

자세한 정보는 다음 위키 문서를 참고하세요 [using Symlinks in Windows](https://github.com/git-for-windows/git/wiki/Symbolic-Links#allowing-non-administrators-to-create-symbolic-links){:target="_blank"}

## Git과 Git-bash

깃은 말할 필요도 없이 이제 개발에는 필수죠

### 설치

다음 [다운로드 링크](https://git-scm.com/downloads){:target="_blank"} 에서 다운받고 설치합니다.

```
C:\>git update-git-for-windows
```

윈도 커맨드 라인에서 도스명령어를 자주 사용하지 않으신다면 유닉스 명령을 사용하시는 것도 좋습니다. 동일한 이름의 명령어는 유닉스 명령이 우선이 됩니다.

![Unix tools from Command Prompt](/assets/images/2020/wsl2/git-1.jpg){:width="70%" style="margin: 0"}

autocrlf 옵션은 `input`을 선택합니다.

![Autocrlf](/assets/images/2020/wsl2/git-2.jpg){:width="70%" style="margin: 0"}

그리고 심링크를 활성화 합니다.

![Enable symlinks](/assets/images/2020/wsl2/git-3.jpg){:width="70%" style="margin: 0"}

** 심링크는 개발자 모드나 administrator 권한을 필요로 합니다.

만약 설치시에 지나쳤다 해도 나중에 설정이 가능합니다.

```
C:\>git config --global core.autocrlf input
C:\>git config --global core.symlinks true
```

그리고 다음과 같이 유저 정보를 입력 합니다.

```
C:\>git config --global user.name "Your Name"
C:\>git config --global user.email "your-email-addr@your.domain"
```

### autoclrf 에 대해

반드시 **input** 으로 설정하세요.

체크아웃시 자동으로 LF 을 CRLF 로 변환하는 게 멋져 보일지도 모르지만, 사실 모든 파일은 도커에 바인드되어 유닉스에서 사용될 수 있기 떄문에 도커에서 알기 어려운 에러를 만나고 싶지 않다면 LF만으로 통일하는 것이 좋습니다. 

윈도 Notepad를 사용할 때가 아니라면 LF only 파일에 대해서 걱정할 필요는 없습니다.


### RSA key 설정하기

Github, Gitlab 또는 Bitbucket 등등 RSA key는 인증으로 널리 사용되고 있습니다.

생성된 퍼블릭 키를 깃 서비스에 등록하게 되면 사용자 이름과 패스워드를 입력하지 않아도 됩니다.

```cmd
C:\>ssh-keygen -t RSA -b 2048
```

그리고 엔터를 수차례 입력해서 디폴트 설정으로 넘어갑니다.

```cmd
C:\>type %USERPROFILE%\.ssh\id_rsa.pub
```

그럼 퍼블릭 키 내용을 깃 서비스에 복사, 붙여넣기를 합니다.

윈도 쪽 뿐 아니라 WSL 쪽 에서도 동일하게 생성하고 등록해야 합니다. 뒤에 WSL 설치 후에 위 작업을 반복하세요.

## Windows Terminal

MS 스토어 에서 설치 할 수 있습니다. [Windows Terminal](https://aka.ms/terminal){:target="_blank"}

![Windows Terminal](/assets/images/2020/wsl2/terminal.jpg){:width="70%" style="margin: 0"}

### CamingoCode 폰트

코딩할 때 좋은 무료 폰트를 하나 추천합니다 [CamingoCode](https://www.fontsquirrel.com/fonts/camingocode){:target="_blank"}. 상당히 컴팩트 하고 미려한 고정폭 글꼴로, `I`, `l` and `1` 등의 비슷한 글꼴이 확연히 구분되서 즐겨 사용합니다.
위의 TTF 파일을 다운 받고 오른클릭으로 각각의 TTF를 설치하세요.

설치 후 윈도 터미널에도 등록하세요. 설정을 눌러 profile란에 다음을 추가하세요.

```sh
	      	      "fontFace": "CamingoCode",
	      	      "fontSize": "9",
```

### Git-bash 사용하기

다음을 윈도 터미널 설정에서 `profiles.list` 어레이에 추가합니다

```json
            {
                "guid": "{abc00000-0000-0000-0000-000000000000}",
                "name": "Git-Bash",
                "commandline": "%PROGRAMFILES%\\Git\\bin\\bash.exe",
                "icon": "%PROGRAMFILES%\\Git\\mingw64\\share\\git\\git-for-windows.ico",
                "fontFace": "CamingoCode",
                "fontSize": "9",
                "startingDirectory" : "~"
            },
```

## WSL2

Windows Subsystem for Linux 는 최근에 버전 2를 지원 하게 되었습니다.
실제 리눅스 커널을 지원하고 너 나은 디스크 접근 성능을 제공합니다. WSL1 처럼 호스트 디스크를 리눅스 로컬 파일로 사용할 수 는 없습니다.

WSL2는 Hyper-v 환경이랑 크게 다르지 않습니다만, 호스트(윈도) 메모리를 게스트(리눅스)와 공유합니다. 그래서 마치 하나의 OS을 사용하고 있는것 처럼 메모리 사용을 최적화 할 수 있습니다. (오라클 버추얼 박스도 이와 같이 사용 할 수 있습니다만, 안타깝게도 Hyper-v와 공존할 수 없습니다 :/)

더 중요한 것은 도커에서 라우팅을 지원하므로 WSL쪽의 포트를 호스트 로컬 네트웍 장치를 통해 접근 할 수 있습니다.

** Windows 10 홈 에디션 에서도 WSL2을 사용할 수 있습니다.

** WSL2를 사용하기 위해선 BIOS 나 UEFI 설정에서 virtualisation support를 활성화 해야 합니다.

### WSL 설치하기

다음 [공식 문서](https://docs.microsoft.com/en-us/windows/wsl/install-win10){:target="_blank"} 에서도 확인할 수 있지만 아래 요약을 보고 진행하시는 것이 더 좋습니다.

먼저 어드민 권한으로 파워셸을 엽니다. 시작버튼은 오른클릭하고 `Windows PowerShell (Admin)`을 선택합니다.

1. WSL 활성화
```sh
C:\> dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
```
1. 'Virtual Machine Platform' 추가 컴포넌트 활성화
```sh
C:\> dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```
1. 커널 업데이트 설치\
[WSL2 kernel](https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi){:target="_blank"}을 다운로드하고 설치합니다. 자세한 내용은 [공식문서](https://docs.microsoft.com/en-gb/windows/wsl/wsl2-kernel){:target="_blank"}를 참고하세요.

1. Ubuntu 설치\
선호하는 아무 Linux 배포판이나 설치가 가능합니다만 저는 우분투를 선호합니다. [MS 스토어 링크](https://www.microsoft.com/store/apps/9n6svws3rx71){:target="_blank"}를 사용해 설치하세요

1. 기본 WSL 버전 설정
```sh
C:\> wsl --set-default-version 2
```

1. 기본 username 설정\
시작메뉴에서 Ubuntu 20.04를 선택하면 우분투가 시작되며 최초 실행시에는 username과 password를 입력하게 됩니다.

### WSL 폴더 접근

기본으로 WSL 디스크는 `\\wsl$\` 에 공유되어 있습니다. `\\wsl$\Ubuntu-20.04`에서 우분투 루트 디렉토리를 확인 할 수 있습니다.
`Cmd`+`R`, type `\\wsl$` and enter will show the Linux root directories.

해당 폴더를 오른클릭하여 네트웍 드라이브로 등록하면 편리합니다.

![map drive](/assets/images/2020/wsl2/mapdrive.jpg){:width="70%" style="margin: 0"}


### Sudo 비번 생략하기

WSL에서 리눅스는 가상머신이고 윈도 파이어 월로 보호되며, 주로 도커를 사용할 것이므로 딱히 높은 보안을 유지할 필요는 없다고 생각합니다. 그보다는 편리하게 루트 권한을 사용하는 것이 좋습니다.

WSL쪽에서 다음과 같이 visudo 명령을 입력합니다.

```sh
$ sudo visudo
```

그러면 나노 에디터가 실행되며 다음을 마지막에 추가합니다.

``` ini
<username> ALL=(ALL) NOPASSWD:ALL
```

`<username>`은 위에 입력한 username으로 대체해 줍니다. `^x`, `y`, `enter`를 차례로 눌러 저장하고 종료합니다.

### WSL memory 관리

WSL2 는 호스트 메모리를 제약 없이 공유하기 때문에 기본 설정으로는 호스트 메모리를 남겨놓지 않고 필요 이상으로 사용 할 수 도 있습니다.

최대 메모리는 전체 메모리의 절반 정도가 적당 합니다. WSL쪽에서 사용하지 않으면 호스트는 물로 남은 메모리를 모두 사용 할 수 있습니다.

#### 최대 memory 제한하기

`%USERPROFILE%\.wslconfig` 파일을 다음의 내용으로 생성합니다.

```ini
[wsl2]
memory=4GB
```

#### WSL2 메모리 반환하기

어느 [MS 블로그 기사](https://devblogs.microsoft.com/commandline/memory-reclaim-in-the-windows-subsystem-for-linux-2/){:target="_blank"}, 에서 다음과 같이 언급하고 있습니다.

> 호스트에 가능한 한 많은 메모리를 반환하기 위해, 우리는 주기적으로 메모리 단편화를 제거 합니다.

WSL 커널은 메모리 사용을 최적화해 최대한 메모리를 반환하려고 노력하지만, 여전히 캐시된 메모리를 유지합니다. 이것을 무조건 반환하게 되면 성능에 영향을 주므로 필요하지 않다면 캐시를 유지하는 것이 좋습니다.

하지만 때때로 더 많은 호스트 메모리를 사용하고 싶다고 생각할 때에는 수동으로 캐시메모리를 반환 시키는 것이 필요 할 때 도 있을 것입니다. 그럴때엔 WSL쪽에서 다음 명령어를 사용해 보세요.

```sh
$ echo 1 | sudo tee /proc/sys/vm/drop_caches
```

아니면 윈도 쪽 에서도 다음과 같이 할 수 있습니다.

```dos
C:\>bash -c "echo 1 | sudo tee /proc/sys/vm/drop_caches"
```

### WSL로 SSH 접속하기

WSL2 는 linux VM이므로 ssh 데몬을 설치할 수도 있습니다. 하지만 접속하기가 쉽지만은 않습니다. IP어드레스가 매번 변경되기 때문이죠.

Hyper-v/WSL2의 이슈중 하나로, IP 어드레스를 고정할 수 없는 점이 있습니다. 와이파이 네트웍이 재접속 되거나 변경되면 호스트 네트웍 서브넷이 변경되고 따라서 VM의 어드레스가 변경됩니다. 이것이 그렇게 자주 일어나지는 않지만 사용하다 보면 매번 새 주소를 찾아내 변경하는 게 꽤 불편하게 느껴 집니다.

최근 제가 찾게 된 [블로그 기사](https://www.hanselman.com/blog/THEEASYWAYHowToSSHIntoBashAndWSL2OnWindows10FromAnExternalMachine.aspx){:target="_blank"}에 이에 대한 멋진 해결책이 있었습니다.

바로 리눅스쪽이 아닌 윈도쪽에 Openssh 서버를 설치하는 것입니다. 그리고 기본 shell 을 bash (WSL2) 로 설정 하는 것이죠.

어드민 파워셸을 열고 다음과 같이 Openssh server를 설치합니다.

```powershell
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
Set-Service -Name sshd -StartupType 'Automatic'
Start-Service sshd
New-ItemProperty -Path "HKLM:\SOFTWARE\OpenSSH" -Name DefaultShell -Value "C:\WINDOWS\System32\bash.exe" -PropertyType String -Force
```

그리고 나면 

```cmd
C:\>ssh localhost
```

윈도 로긴 패스워드로 접속이 가능해집니다.

[Putty](https://putty.org/){:target="_blank"} 나 [Kitty](http://www.9bis.net/kitty/#!index.md){:target="_blank"} 가 익숙해서 계속 터미널로 사용하고 싶으시면 위와 같은 방법으로 가능합니다.

![TMUX](/assets/images/2020/wsl2/tmux.jpg){:width="70%" style="margin: 0"}

위 스크린샷의 tmux 는 커맨드라인 세션 매니저입니다. 자세한 정보는 [공식 Wiki](https://github.com/tmux/tmux/wiki){:target="_blank"}에서 확인하세요.

#### ssh 비번 생략하기

일반적으로 퍼블릭 키 (보통 id_rsa.pub)를 ssh 서버단의 `.ssh/authorized_keys`파일에 등록하면, 비번없이 로긴 할 수 있습니다.

하지만 보통 윈도 유저는 Administrators 그룹에 속해 있는데요, 그럴 때엔 꼭 `%programdata%/ssh/administrators_authorized_keys` 파일에 등록을 해줘야 합니다.

### Linux GUI 앱 사용하기

WSL은 커맨드 라인만 제공하기 때문에 기본적으로 GUI 앱을 사용할 수 없습니다만 방법이 없지는 않죠. 윈도 쪽에 X11 서버를 설치하고, 원격 디스플레이를 쓰면 Windows 쪽에서 GUI 앱이 사용 가능합니다. 오픈소스 X11 서버인 VCXSrv를 사용해 보세요. [다운로드는 여기](https://sourceforge.net/projects/vcxsrv/){:target="_blank"}에서 하고 설치합니다.

#### X server 시작하기

![VCXSrv Xlaunch](/assets/images/2020/wsl2/vcxsrv-access-control.jpg){:width="70%" style="margin: 0"}

시작메뉴에서 XLaunch를 선택하면 몇 가지 설정을 물어봅니다. `Disable access control` 이것은 디폴트로 꺼져있지만 켜줘야 합니다. 그리고 저장 버튼을 눌러 설정을 저장합니다. 바탕화면에 저장하면 더블클릭해서 매번 같은 설정으로 시작 할 수 있습니다.

![VCXSrv Save](/assets/images/2020/wsl2/vcxsrv-save.jpg){:width="70%" style="margin: 0"}

vcxsrv 서버를 최초 실행하게 되면 Windows Security Alert가 파이어월 설정을 물어봅니다. 디폴트로는 프라이빗만 활성화되어 있기 때문에 꼭 둘 다 켜 주세요. 

![VCXSrv Save](/assets/images/2020/wsl2/vcxsrv-security-alert.jpg){:width="70%" style="margin: 0"}

만약 이미 프라이빗만 허용해 버렸다면 Windows firewall 설정을 확인하시고 아래와 같은 항목을 찾아 네 개 모두 허용시킵니다.

![firewall](/assets/images/2020/wsl2/firewall.jpg){:width="70%" style="margin: 0"}

![firewall](/assets/images/2020/wsl2/vcsxrv-icon.jpg){:width="30%" style="margin: 0"}

시작되면 시스템 트레이에 아이콘을 볼 수 있습니다.

#### DISPLAY 변수 설정

WSL 쪽에서 GUI 앱을 쓰기 위해 DISPLAY 변수를 설정합니다. 호스트 주소가 매번 바뀌기 때문에 아래 명령은 사용할 때 마다 실행하는 것이 좋습니다. `~/setDisplay` 등과 같이 저장하고 `source ~/setDisplay` 또는 `. ~/setDisplay` 와 같이 실행시킬 수 있습니다.


```sh
$ export DISPLAY=$(awk '/nameserver / {print $2; exit}' /etc/resolv.conf 2>/dev/null):0
$ export LIBGL_ALWAYS_INDIRECT=1
```

#### 리눅스 앱 실행하기 예제

```
$ sudo apt install stacer
$ export DISPLAY=$(awk '/nameserver / {print $2; exit}' /etc/resolv.conf 2>/dev/null):0
$ export LIBGL_ALWAYS_INDIRECT=1
$ stacer
```

상기 커맨드는 Stacer를 설치하고 실행합니다.

![firewall](/assets/images/2020/wsl2/stacer.jpg){:width="70%" style="margin: 0"}


## Docker Desktop

Docker 설치는 Windows에서도 간단합니다. [이곳에서 다운로드](https://hub.docker.com/editions/community/docker-ce-desktop-windows/){:target="_blank"} 하시고 설치하세요.

도커는 간단하게 미리 설정된 서비스를 설치 할 수 있기 때문에 많은 프로젝트들이 도커를 활용합니다. 하지만 윈도 쪽에서 도커를 사용할 때 판이한 파일시스템 때문에 약간의 문제가 있을 수 있습니다.

이쪽에 [도커사의 블로그 기사](https://www.docker.com/blog/docker-desktop-wsl-2-best-practices/){:target="_blank"}를 보시면 WSL을 사용할 때의 일반적인 주의점을 확인 할 수 있습니다.

만약 업무에 활용하는 것이 아니더라도 도커를 사용하는 것은 매력적입니다.

예를 들면 루비 개발자가 아니지만, 지킬 블로그를 사용하고 싶을 때에, 시스템에 복잡하게 루비 런타임을 설치하지 않고도 도커를 이용해 블로깅을 할 수 있게 됩니다.

또한 restart 옵션을 통해 특정 서비스를 재부팅 후 일지라도 항상 동작시킬 수 있습니다. 로컬 음악 파일을 위한 플레이어를 사용할 수도 있답니다. [Airsonic](https://github.com/airsonic/airsonic){:target="_blank"} ([Docker hub](https://hub.docker.com/r/airsonic/airsonic){:target="_blank"}) 은 웹 플레이어와 모바일 앱을 제공합니다.

### 컨테이너 메모리 제한하기

도커 컴포즈를 사용하고 계신가요? 로컬 시스템의 컨테이너들을 일회성이 아닌 파일을 통해 일정하게 관리 할 수 있습니다.

여러 개의 컨테이너를 조합해서 사용할 수도 있고요. 긴 옵션들을 모두 파일에 넣어서 짧은 명령으로 시작/정지/삭제 등이 가능합니다.
컨테이너 이름이 좀 길어지긴 하지만 `container_name` 옵션을 쓰면 간결하게 유지할 수도 있죠. 만약 아직 사용하고 있지 않다면 사용해 보시길 추천 드립니다. 자세한 정보는 [이쪽에서](https://docs.docker.com/compose/){:target="_blank"} 확인해 보세요.

도커 컴포즈에서의 메모리 제한은 스웜을 사용하지 않는다면 버전 2 (현재 최신은 2.4) 에서만 가능합니다.

`mem_limit: 512m` 옵션으로 메모리 사용을 512M 이하로 제한 할 수 있습니다.

** docker 명령을 사용할 때는 `--memory=512m` 로 사용 가능합니다.

## VSCode

아마도 가장 많은 개발자가 사용하는 에디터가 VSCode 일 듯합니다.
딱히 다른 에디터를 사용해야 할 이유가 없다면 나쁘지 않은 선택입니다.

역시 WSL에서 사용할 때의 주의점은 [MS사의 블로그 기사](https://code.visualstudio.com/docs/remote/wsl){:target="_blank"}
를 확인해 보세요

### project 소스파일은 어디에?

윈도에 설치된 빌드 툴을 사용하는 게 아니라면, 프로젝트 파일은 WSL 쪽에 두세요. 도커 환경을 사용하여 소스가 변경될 때 마다 자동빌드를 하게 된다면 윈도쪽의 소스코드에서는 [자동 빌드가 동작하지 않기가 쉽습니다](https://github.com/microsoft/WSL/issues/216){:target="_blank"}. 이런 종류의 에러는 보통 이유를 알기가 쉽지 않죠.

원치 않은 오동작을 피하고 싶다면 WSL 쪽에 소스를 두시길 추천드립니다.

### Defender Antivirus 예외 등록

윈도 빌드 툴을 사용하시고 윈도 쪽에 소스를 두실 경우는 이 설정을 확인하세요. 그렇지 않으면 빌드 성능이 크게 저하될 수 있습니다.
소스 코드는 `Projects`와 같은 이름으로 한곳에 모아 두시고, 안티바이러스 예외등록을 해 두세요.
보통 많은 빌드 툴이 프로젝트 폴더에 `node_modules`, `.bundle`, `vendor`, `build` 와 같은 폴더 안에 많은 임시 파일을 생성합니다.

소스 디렉토리를 예외등록하게 되면 빌드 성능의 저하를 막을 수 있습니다.

만약 빌드 성능이 너무 느리다 싶을 때엔 타스크 매니저를 띄우고 혹시 `Antimalware Service Executable` 프로세스가 CPU 자원을 높게 사용하고 있지 않은지 확인해 보세요 CPU over 10%~20%정도 사용하고 있다면, 예외등록 추가를 고려해 보세요. 빌드 툴에 따라 %APPDATA% 등에 파일을 추가할 수도 있습니다.

![Antivirus Exclusion](/assets/images/2020/wsl2/exclusion.jpg){:width="70%" style="margin: 0"}

## 선택사항

### Chocolatey, 패키지 매니저

여러 툴을 설치하는 것은 때로 번거럽고 아리송할 때가 있습니다. choco를 사용하면 쉽고 간단하게 프로그램 설치가 가능합니다. 맥에서 많이들 사용하시는 Homebrew (brew)와 비슷합니다.

[Chocolatey Official Site](https://chocolatey.org/){:target="_blank"}

### 윈도 쪽 Node.js 사용하기

프론트엔드 개발자나 노드 개발자는 Node.js가 필수죠. 윈도 쪽에 node를 설치하셨다면 빌드툴은 가지고 있지 않을 가능성이 높습니다.
최근 Node.js 인스톨러는 빌드 툴 설치 옵션이 있습니다. 디폴트로는 꺼져 있지만 단지 재인스톨 만으로 고칠 수 있습니다. 이미 설치되었다면 변경을 선택해서 다시 설치 가능 합니다. 아래 옵션을 켜고 설치하시면 설치 종료 후 자동으로 Chocolatey와 함께 빌드툴을 설치해 줍니다. 만약 바이너리 패키지 설치 시 자주 실패 로그를 볼 수 있다면 빌드툴을 설치 해 보시길 권합니다.

![Node.js Choco](/assets/images/2020/wsl2/install-choco.jpg){:width="70%" style="margin: 0"}

또 윈도 쪽에서 node 프로젝트를 사용하시더라도 Git Bash를 사용해 보세요. 사용중인 프로젝트에 bash 스크립트가 사용되더라도 실패없이 사용 할 수 있습니다.

Defender Antivirus 예외설정에 Yarn/Npm 캐시를 꼭 등록하세요. 설치과정이 눈에 띄게 빨라질 것입니다.

```cmd
%APPDATA%\npm-cache
%LOCALAPPDATA%\Yarn
```

## 다시 Windows의 세계로

WSL2 덕분에 윈도에서도 마침내 개발자 친화적이 환경이 완성 되었습니다. 도커 성능도 맥에 비해 떨어지지도 않고 홈 에디션에서도 완벽하게 동작합니다. 사실 멀티 디스플레이 지원은 윈도가 한 수 위인 듯합니다. 호불호가 갈리긴 하지만 클리어 타입으로 FullHD급의 디스플레이 에서도 좀 더 나은 글꼴 렌더링을 해 줍니다. 최근 에지 브라우저는 개발자들에게 친숙한 크롬 기반으로 변경되었죠.

여전히 너무 잦은 재부팅은 별로 내키지 않지만 저는 다음 랩탑을 사게 되면 윈도를 선택할 것입니다 :)