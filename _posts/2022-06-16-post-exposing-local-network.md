---
title: "Exposing local networks to the Internet."
date: 2022-06-16T15:34:30-04:00
categories:
  - blog
tags:
  - IP_Forwarding
  - localhost
  - ssh
  - internet
---

We'll talk about how to make your local network services public today (internet). There are several methods for doing so, including **port forwarding**, router configuration, and so on. In this blog, we'll use the **SSH** (secure shell) service and the [localhost.run](https://localhost.run/) service.

**localhost.run** is a client-less tool to instantly make a locally running application available on an internet accessible URL.

_To know more_ ➡ [localhost.run](https://localhost.run/docs/)

## To make your local server available on the Internet, follow these steps:
- Make sure SSH is installed in your device. To install for [Windows](https://docs.microsoft.com/en-us/windows-server/administration/openssh/openssh_install_firstuse). To install for [Linux](https://ubuntu.com/server/docs/service-openssh).
- Start you [Apache](https://ubuntu.com/tutorials/install-and-configure-apache#1-overview) or any other operator’s server to setup HTTP server.

#### Terminal Command:
- ```ssh-keygen```

The command ***ssh-keygen*** will generate a RSA key so that **localhost.run’s** server can identify you.

While generating RSA key you will be asked to enter a **passphrase** it is recommended to use or you may leave it by entering it empty.

#### Terminal Command:
- ```ssh -R 80:localhost:80 localhost.run```

Here we are going to forward any server on our PORT 80 to localhost.run’s PORT 80 to make it available on internet.

Now you can get your public IP provide by localhost.run in your terminal. The IP address will be in the form of: **(some strings).localhost.run**

Now anyone who search for this IP from any part of the world will be forwarded to your local network and therefore can access your server.

#### This strategy can be applied to a wide variety of attack vectors. Example:
- **Credential Harvesting**
- **Downloading backdoors on victim’s machine.**
- **Rubber ducky attacks**, etc.


*[SSH]: Secure Shell