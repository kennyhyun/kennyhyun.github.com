---
layout: post
title:  "Setting up Windows for developers with WSL2"
description: 'Many developers use Mac. That would be for building iOS apps. But there are more competitive laptops with Windows. If you do not need to build iOS apps, Windows is actually a great option for developing thanks to WSL2. Following this article, you can have a professional develop environment without a hitch.'
date:   2020-09-10 23:29:21 +1000
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

Many developers use Mac. That would be for building iOS apps. But there are more competetive laptops with Windows. If you do not need to build iOS apps, Windows is actually a great option for developing, thanks to WSL2. Following this article, you can have a professional development environment without a hitch.

This includes...

- Git, Git bash
- Windows Terminal
- WSL2
  - Using Linux GUI apps
  - Ssh to WSL
- Docker
- VSCode
- Defender Antivirus Exclusion

## Windows version and build

![winver](/assets/images/2020/wsl2/winver.jpg){:width="70%" style="margin: 0"}

`Wnd`+`R`, type `winver`, `enter` will give you the version info like above. If you have already version 2004 (OS build 19041), you are ready to go.

May 2020 update (version 2004 or build 19041) will support WSL2 and Docker will use WSL backend as a docker engine by default. Windows Home edition is also okay since you can now use Docker.

If your Windows does not show 2004 update for some reason, try [this link](https://www.microsoft.com/en-au/software-download/windows10){:target="_blank"} to update.

## Developer mode

Symlinks needs Administrator privilege or developer mode.

A git repo with symlinks will needs this


![Developer mode](/assets/images/2020/wsl2/developer-mode.jpg){:width="70%" style="margin: 0"}

Find more about [using Symlinks in Windows](https://github.com/git-for-windows/git/wiki/Symbolic-Links#allowing-non-administrators-to-create-symbolic-links){:target="_blank"}

## Git and Git-bash

Now git is a must have when you develop.

### Install Git with Git bash

Let's [download here](https://git-scm.com/downloads){:target="_blank"} and install or update if you have already

```
C:\>git update-git-for-windows
```

If you don't mind overriding DOS command, you can use bash command in command window

![Unix tools from Command Prompt](/assets/images/2020/wsl2/git-1.jpg){:width="70%" style="margin: 0"}


Choose autocrlf as `input`

![Autocrlf](/assets/images/2020/wsl2/git-2.jpg){:width="70%" style="margin: 0"}

and enable symlinks while installing

![Enable symlinks](/assets/images/2020/wsl2/git-3.jpg){:width="70%" style="margin: 0"}

** symlinks need developer mode or administrator



Don't worry if you missed the symlink setting. You can set anytime later

```
C:\>git config --global core.autocrlf input
C:\>git config --global core.symlinks true
```

Don't forget to set your info

```
C:\>git config --global user.name "Your Name"
C:\>git config --global user.email "your-email-addr@your.domain"
```


### About autoclrf

Should be **input**.

Converting LF to CRLF when checking out might seem to be nice but most files can be used in docker when you develop. And it will complain about CRLF.

Don't worry about LF only files unless you are using Notepad.


### Setting up RSA key

Github, Gitlab or Bitbucket (or more) use RSA key for authentication.

If you have your public key registered to the git service,
you don't have to provide username and password.

```cmd
C:\>ssh-keygen -t RSA -b 2048
```

and enter a few times to answer using default value

```cmd
C:\>type %USERPROFILE%\.ssh\id_rsa.pub
```

and copy paste the public key to your git service.

This is not only for Windows but you will need in in WSL as well.
After install WSL, do that again in WSL side.

## Windows Terminal

In MS Store, you can install [Windows Terminal](https://aka.ms/terminal){:target="_blank"}


![Windows Terminal](/assets/images/2020/wsl2/terminal.jpg){:width="70%" style="margin: 0"}

### CamingoCode font

I like [CamingoCode](https://www.fontsquirrel.com/fonts/camingocode){:target="_blank"}. It is quite slick and clear monospaced font and provides distinctive `I`, `l` and `1` glyphs.
Download TTF files from the link above and install by right clicking on each TTF files.

You can add this to the profile of the Windows Terminal setting.

```sh
	      	      "fontFace": "CamingoCode",
	      	      "fontSize": "9",
```

### Using Git-bash

Add following to `profiles.list` array in the Terminal setting.

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

Windows Subsystem for Linux has been recently added support for version 2.
It provides better file I/O performance and an actual Linux kernel.
You cannot use the host disk for WSL2 local files like in WSL1.

Using WSL2 is not quite different to using Hyper-V VM, but you can share the host (Windows) memory with the guest (Linux) so you can optimize memory usage as you are using only one OS.
(Oracle Virtualbox also does this as well but you cannot use it with Docker at the same time :/)

And more importantly, Docker supports WSL2 now and it routes all the exported ports from the container to your host network.

** Windows 10 Home edition also supports WSL2.

** You still need the virtualisation support in your BIOS or UEFI setting.

### Installing WSL

You can refer [the official document](https://docs.microsoft.com/en-us/windows/wsl/install-win10){:target="_blank"} for details but could just follow my sequence.

Open the elevated powershell. Right click the start button and choose `Windows PowerShell (Admin)`

1. Enable WSL
```sh
C:\> dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
```
1. Enable the 'Virtual Machine Platform' optional component
```sh
C:\> dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```
1. Install kernel update\
Download [the WSL2 kernel](https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi){:target="_blank"} and install. Refer to [the official document](https://docs.microsoft.com/en-gb/windows/wsl/wsl2-kernel){:target="_blank"} for details.

1. Install Ubuntu\
You can use another Linux distro but I prefer ubuntu. Using [MS store](https://www.microsoft.com/store/apps/9n6svws3rx71){:target="_blank"}, install the distro

1. Set WSL 2 as your default version
```sh
C:\> wsl --set-default-version 2
```

1. Set default username\
Running Ubuntu from the start menu will need you to input the username and password for the first time once

### Accessing WSL directories

By default, WSL drive is shared by `\\wsl$\<distro name>` like `\\wsl$\Ubuntu-20.04`.
`Cmd`+`R`, type `\\wsl$` and enter will show the Linux root directories.

You can also map the drive by right click the directory.

![map drive](/assets/images/2020/wsl2/mapdrive.jpg){:width="70%" style="margin: 0"}


### Sudo without password

You can use sudo without password in your WSL

```sh
$ sudo visudo
```

will show the editor and you can add 

``` ini
<username> ALL=(ALL) NOPASSWD:ALL
```

at last.

Replace `<username>` with yours and `^x`, `y`, `enter` to save and quit Nano.

### Managing WSL memory

WSL2 shares your host memory but without a limit, it could drain your host memory.

It is required to set the maximum at the half of your entire memory. It will not use the maximum memory unless required.

#### Limiting maximum memory usage

Create `%USERPROFILE%\.wslconfig` with the following content

```ini
[wsl2]
memory=4GB
```

#### Reclaiming WSL2 memory

On [an MS blog article](https://devblogs.microsoft.com/commandline/memory-reclaim-in-the-windows-subsystem-for-linux-2/){:target="_blank"}, it says

> In order to return as much memory to the host as possible, we periodically compact memory to ensure free memory is available in contiguous blocks.

WSL kernel tries to return memory as much as possible but Linux still caches memory and WSL does not try to clear since it would affect the performance.

If you want to give more memory to host, try this command on WSL,

```sh
$ echo 1 | sudo tee /proc/sys/vm/drop_caches
```

or on Windows, you can do

```dos
C:\>bash -c "echo 1 | sudo tee /proc/sys/vm/drop_caches"
```

### SSH to WSL

Since WSL2 is a linux VM, you can install sshd in it. But to connect to the machine you will need the IP address.

One of the issue on Hyper-v/WSL2 is you cannot make the VM's IP address static. When the network connection has been changed, the host virtual network interface address are changed so the guest address is also changed. This would not happen so often but if it changed, you need to get the new address again.

Recently I have found [a nice article](https://www.hanselman.com/blog/THEEASYWAYHowToSSHIntoBashAndWSL2OnWindows10FromAnExternalMachine.aspx){:target="_blank"} to solve this issue.

You can install Openssh server in Windows side and set the default shell as bash(WSL2).

Following powershell commands will install Openssh server

```powershell
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
Set-Service -Name sshd -StartupType 'Automatic'
Start-Service sshd
New-ItemProperty -Path "HKLM:\SOFTWARE\OpenSSH" -Name DefaultShell -Value "C:\WINDOWS\System32\bash.exe" -PropertyType String -Force
```

And try 

```cmd
C:\>ssh localhost
```
It will show ssh prompt to login with your Windows login password.

If you have got used to [Putty](https://putty.org/){:target="_blank"} or [Kitty](http://www.9bis.net/kitty/#!index.md){:target="_blank"}, now you can use it as a terminal.

![TMUX](/assets/images/2020/wsl2/tmux.jpg){:width="70%" style="margin: 0"}

Tmux is a command line session manager. Visit the [Official Wiki](https://github.com/tmux/tmux/wiki){:target="_blank"} if you are interested.

#### ssh without the password

If you register your public key (id_rsa.pub usually) to `.ssh/authorized_keys` file, you can connect without the password.

But if you belongs to the Administrators group, you should add the public key to `%programdata%/ssh/administrators_authorized_keys`.

### Using Linux GUI Apps

GUI is not supported in WSL. But if you can use XServer, Linux GUI apps can be used in Windows side. Try VCXSrv project. It’s an opensource X11 server. [Download it here](https://sourceforge.net/projects/vcxsrv/){:target="_blank"} and install.

#### Launching X server

![VCXSrv Xlaunch](/assets/images/2020/wsl2/vcxsrv-access-control.jpg){:width="70%" style="margin: 0"}

Run XLaunch and it will ask some settings. Don't forget to turn on `Disable access control`. If you save the setting on the Desktop, you can use the config file to launch the server without setting it again.

![VCXSrv Save](/assets/images/2020/wsl2/vcxsrv-save.jpg){:width="70%" style="margin: 0"}

When you launch vcxsrv for the first time, Windows Security Alert will ask if you allow that but by default, private access is only allowed but you actually need both private and public.

![VCXSrv Save](/assets/images/2020/wsl2/vcxsrv-security-alert.jpg){:width="70%" style="margin: 0"}

If you have just allowed only private, or not sure, check Windows firewall.

![firewall](/assets/images/2020/wsl2/firewall.jpg){:width="70%" style="margin: 0"}

And allow all the 4 items in the Windows Defender Firewall with Advanced Security if you haven't yet.

![firewall](/assets/images/2020/wsl2/vcsxrv-icon.jpg){:width="30%" style="margin: 0"}

You will have this icon in the system tray.

#### Setting DISPLAY env variable

You will also need to set DISPLAY env var in the WSL side. Because the host network address is not fixed, you might need to run the following before you use. Save the following to like `~/setDisplay` and run `source ~/setDisplay` or jusr `. ~/setDisplay`.

```sh
$ export DISPLAY=$(awk '/nameserver / {print $2; exit}' /etc/resolv.conf 2>/dev/null):0
$ export LIBGL_ALWAYS_INDIRECT=1
```

#### Trying a Linux app

```
$ sudo apt install stacer
$ export DISPLAY=$(awk '/nameserver / {print $2; exit}' /etc/resolv.conf 2>/dev/null):0
$ export LIBGL_ALWAYS_INDIRECT=1
$ stacer
```

Those commands will install Stacer and launch in Windows side


![firewall](/assets/images/2020/wsl2/stacer.jpg){:width="70%" style="margin: 0"}


## Docker Desktop

Installing Docker on Windows is simple.
Just [download it here](https://hub.docker.com/editions/community/docker-ce-desktop-windows/){:target="_blank"} and install.

Since it provide a predefined service like an app, many projects utilise Docker in their project. But Using docker in Windows could be challenging due to the difference of the file system.

There is [a nice blog article from Docker](https://www.docker.com/blog/docker-desktop-wsl-2-best-practices/){:target="_blank"}.

Even if your project does not require Docker, it would be beneficial.

For example, if you are not a ruby developer but if you want to use ruby for like writing a jekyll blog.
You can use Docker to do that without installing all the ruby tools.

Moreover, you can install apps and Docker can start it always. How about getting a local music player for local files, [Airsonic](https://github.com/airsonic/airsonic){:target="_blank"} ([Docker hub](https://hub.docker.com/r/airsonic/airsonic){:target="_blank"})

### Limiting container memory

Using docker-compose is easier to manage the containers.

You can manage multiple containers
and save all the options in the file and creating, start, stop and removing are quite handy.
Conatainers can be named using `container_name` BTW.

If you have not used it yet, please leran more [about docker-compose](https://docs.docker.com/compose/){:target="_blank"}

Use docker-compose version 2 (2.4 is latest at the point of writing this article) if you are not using Swarm.

`mem_limit: 512m` will limit the memory usage to 512M

** This can also be done by `--memory=512m` using docker

## VSCode

vscode seems like being a de facto standard. If you're not a big fan of the other fancy editors, you can use this.

There is [a nice document from MS](https://code.visualstudio.com/docs/remote/wsl){:target="_blank"}

### Where should the project files be placed

If your project is using docker and watching your source code and try to rebuild, it [will not be working in Windows](https://github.com/microsoft/WSL/issues/216){:target="_blank"}. This could be challenging to know the reason why.

If you are using windows dev tools you can keep source files in Windows side.

If you want to avoid hassles, just stay only in the WSL side.

## Defender Antivirus Exclusion

You can put the all repo, project files in one directory in your home directory such as `Projects`. Usually it could create many automatically generated files in it.

For example, `node_modules`, `.bundle`, `vendor`, `build` or so on.

You can speed up by excluding Antivirus scanning.

If you feel some building process is quite slow, you can turn on the task manager and locate `Antimalware Service Executable` and if that consumes CPU over 10%, consider adding some more exclusion. 


![Antivirus Exclusion](/assets/images/2020/wsl2/exclusion.jpg){:width="70%" style="margin: 0"}

## Optionals

### Chocolatey, package manager

It has so less hassles to install all the tools manually. It is like using Homebrew (brew) in mac.

[Chocolatey Official Site](https://chocolatey.org/){:target="_blank"}

### Using Node.js in Windows side

If you are a frontend developer or nodejs developer, you would have already Node.js installed.
The latest Node.js installer actually includes build tools and package manager for Windows, Chocolatey. Don't forget to install necessary tools. Unless you would have some error while building binary packages.

![Node.js Choco](/assets/images/2020/wsl2/install-choco.jpg){:width="70%" style="margin: 0"}

When it is used in Gitbash, it is better. Even if your projects uses bash scripts which is usually not supported in Windows, you can still run it with git-bash.

You will need to exclude npm/yarn cache from Defender Antivirus setting.

```cmd
%APPDATA%\npm-cache
%LOCALAPPDATA%\Yarn
```

## Say Hello to Windows

Thanks to WSL2, Windows machine is also have been developer friendly. Docker performance it not worse than OSX and you can use it on Home Edition too. It actually has a better multiple display experience than OSX and Cleartype will give you a better font rendering in full HD display (might be a personal preference though). Edge browser is now chromium based.

I still don't like too many rebooting but I would choose next dev machine as a Windows laptop :)
