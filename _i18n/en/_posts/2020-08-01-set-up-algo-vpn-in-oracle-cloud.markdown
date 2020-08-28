---
layout: post
title:  "Setting up Algo VPN in Oracle cloud"
description: 'Using VPN abroad sometimes has advantages but paying a VPN service subscription could cost more than it worth.
There seems to be some free VPS hosting nowadays so you can use as a VPN server.
WireGuard app helps you to configure the client like iPhone, Android, iPad, Windows, Mac and more!
Here I share the walk through how you get a free VPN service :)'
date:   2020-08-01 20:41:01 +1000
tags:
  - algovpn
  - algo
  - wireguard
  - vpn
  - ipad
  - phone
  - android
  - oracle
  - oracle-cloud
categories: IT
---

Using VPN abroad sometimes has advantages but paying a VPN service subscription could cost more than it worth.

However, what if you already have a VPS aboard? There seems to be a free VPS hosting nowadays.

As I told in the previous post,
I tried to set Algo VPN server in Oracle Cloud.

Hope this dump to help you.

### Disclaimer

Please use your VPN server in your own responsibility.
I am not responsible any of the consequences of using Oracle Cloud or VPN.

### Create an instance

1. Check [Oracle Cloud](https://www.oracle.com/au/cloud/free/){:target="_blank"} and Sign up.
    - Choose your home region where you intend to be.
1. Sign into the web console and click `Create a VM instance`.
  ![Insttance1](/assets/images/2020/vpn/instance1.jpg)
    - Choose ubuntu minimal and upload your public key
        - Check [how to create ssh key][howto-ssh]{:target="_blank"} if you don't have one.
1. Click create and you will see above details.
  ![Insttance2](/assets/images/2020/vpn/instance2.jpg){:width="80%"}
    - Memo the public IP.

Now You can connect into the instance using ssh client.

### Install Algo VPN server

Connect to the instance created above and follow the following instructions to install the VPN server.

#### Apt upgrade and install dependencies

```sh
$ sudo apt update && sudo apt upgrade -y
$ sudo apt install -y python3-virtualenv unzip vim
```

Install your editor as well above like vim or nano.


#### Download Algo

```sh
$ wget https://github.com/trailofbits/algo/archive/master.zip
$ unzip master.zip
```

#### Set virtualenv

```sh
$ cd algo-master
$ python3 -m virtualenv --python="$(command -v python3)" .env &&
  source .env/bin/activate &&
  python3 -m pip install -U pip virtualenv &&
  python3 -m pip install -r requirements.txt

```

#### Config users

Edit `config.cfg` to add users

```sh
$ vi config.cfg
```

I recommend you to use like the following since sharing accounts is told to cause some issues

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

#### Start deployment

It's an interactive script.
You need to choose options and the script will get the job done.

```sh
$ ./algo
```

The first question is the most important
Put 11 since you are using the localhost which you are using for the installation

```sh
[Cloud prompt]
What provider would you like to use?
--> 11. Install to existing Ubuntu 18.04 or 20.04 server (for more advanced users)
```

Most of the following would be okay for their default (just enter)
but you will need the followings two answered differently

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



And it will start to install the VPN server and then it show the server details.

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

If something went wrong, it's recommended to restart the whole procedure from the top.
Terminate the instance and recreate the instance.

#### Copy credentials from the Algo server

Now it's okay to disconnect the ssh connection to the VPN server.

From your local env (eg. laptop), run scp to copy files from the server
Replace `vpnserver` to your VPN server IP.

```sh
$ scp -r ubuntu@vpnserver:algo-master/configs .
```

Now you can use credentials for the VPN server in the `configs` directory.

### Open the ports for the VPN

VPN server itself is ready now but there is still one more step left.
The [ports][algo-vpn-ports]{:target="_blank"} are usually closed so it should be allowed manually.

#### Add Security List for VPN

Sign into the web console.

1. Go to Networking > Virtual Cloud Networks
1. Click the VCN name in the right plan to see the details
1. Click the Security Lists in the Resources in the Left pane
1. Click `Create Security List`
1. Put details in the popup
    - Name: VPN
    - Additional Ingress Rule: Ingress Rule 1
        - Source CIDR: 0.0.0.0/0
        - Destination Port Range: 4160
    - Additional Ingress Rule: Ingress Rule 2
        - Source CIDR: 0.0.0.0/0
        - IP Protocol: UDP
        - Destination Port Range: 51820,500,4500
1. Click `Create Security List` to confirm

![vpn ports](/assets/images/2020/vpn/vpn.ports.jpg)

#### Bind the Security List to the Public Subnet

1. Go to the Instace Details (Compute > Instaces > Instance Details)
1. Click `Subnet` of Primary VNIC
1. Click `Add Security List`
1. Select the `VPN` in the security list pull down in the popup
1. Click `Add Security List` to confirm

### Client settings

Let me share how I connect my iPad

1. Get the WireGuard in Appstore. (iOS12 or higher is required)
1. Open the PNG QR code file of the user such as `configs/localhost/wireguard/tablet.png`
  `configs` directory is the one you copied in the previous step
1. Launch WireGuard and touch `+` button to Create from QR code
1. Take the QR code and name it like `tablet`
1. turn on the connection

That's it. 

Go to the browser and search for `my ip` to confirm your ip is of the one of the VPN server.

WireGuard app is available for Mac and Windows10 as well so you can do similar

### Troubleshooting

When the server is not available, WireGuard app silently fails so network is not available.

Turn off the connection switch in the WireGuard and resolve the server issue and retry.

### Further readings

- [Algo VPN](https://github.com/trailofbits/algo#deploy-the-algo-server){: target="_blank" }

[algo-vpn-ports]: https://github.com/trailofbits/algo/blob/master/docs/firewalls.md
[howto-ssh]: https://docs.github.com/en/github/authenticating-to-github/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent
