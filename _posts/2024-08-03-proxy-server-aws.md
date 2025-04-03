---
title: "How to Use an AWS EC2 Server as a Proxy with SSH Tunneling"
date: 2024-06-03T15:34:30-04:00
categories:
  - blog
tags:
  - proxy
  - aws
  - ec2
  - internet
---
>Using an AWS EC2 server as a proxy can be a powerful tool for various  tasks such as accessing restricted content, enhancing security, or managing multiple accounts. In this tutorial, we'll guide you through the process of creating an EC2 instance, configuring it as a proxy server, and connecting to it using SSH tunneling.

---

># Table of Contents
1. [Introduction](https://aryantiwary.online/blog/proxy-server-aws/#1-introduction)
2. [Prerequisites](https://aryantiwary.online/blog/proxy-server-aws/#2-prerequisites)
3. - [Step 1](https://aryantiwary.online/blog/proxy-server-aws/#step-1-create-an-aws-ec2-instance): Create an AWS EC2 Instance
   - [Step 2](https://aryantiwary.online/blog/proxy-server-aws/#step-2-configure-security-group-for-ssh-access): Configure Security Group for SSH Access
   - [Step 3](https://aryantiwary.online/blog/proxy-server-aws/#step-3-connect-to-your-ec2-instance): Connect to Your EC2 Instance
   - [Step 4](https://aryantiwary.online/blog/proxy-server-aws/#step-4-install-and-configure-squid-proxy-server): Install and Configure Squid Proxy Server
   - [Step 5](https://aryantiwary.online/blog/proxy-server-aws/#step-5-configure-ssh-tunneling): Configure SSH Tunneling
   - [Step 6](https://aryantiwary.online/blog/proxy-server-aws/#step-6-verify-the-proxy-setup): Verify the Proxy Setup
4. [Conclusion](https://aryantiwary.online/blog/proxy-server-aws/#conclusion)
5. [FAQs](https://aryantiwary.online/blog/proxy-server-aws/#faqs)

---

># 1. Introduction
Setting up an AWS EC2 server as a proxy involves creating an EC2 instance, configuring a proxy server on it, and establishing an SSH tunnel from your local machine. This setup enhances your privacy and security while browsing the web.

---

># 2. Prerequisites
- An AWS account
- Basic knowledge of AWS EC2
- SSH client (e.g., OpenSSH, PuTTY)

>#### Step 1: Create an AWS EC2 Instance
1. Log in to your AWS Management Console.
2. Navigate to the EC2 Dashboard and click on "Launch Instance".
3. Choose an Amazon Machine Image (AMI). For this tutorial, we will use the Amazon Linux 2 AMI.
4. Select an Instance Type. The t2.micro instance type is sufficient for this tutorial.
5. Configure Instance Details. Use the default settings and click "Next: Add Storage".
6. Add Storage. The default settings are usually sufficient. Click "Next: Add Tags".
7. Add Tags (Optional). Tags help you organize your resources.
8. Configure Security Group.
    - Create a new security group.
    - Add a rule to allow SSH (port 22) from your IP.
9. Review and Launch the instance.
10. Download the Key Pair and save it securely. You will need this to connect to your instance.

>#### Step 2: Configure Security Group for SSH Access
*To ensure you can connect to your instance, you need to configure the security group:*
1. Navigate to the EC2 Dashboard.
2. Select your instance, then click on the Security Groups associated with it.
3. Edit inbound rules to allow SSH access:
```
    Type: SSH
    Protocol: TCP
    Port Range: 22
    Source: My IP
```

>#### Step 3: Connect to Your EC2 Instance
*Use the downloaded key pair to connect to your instance via SSH:*
1. Open your terminal (or PuTTY on Windows).
2. Change the permissions of your key pair file:
    ```
    chmod 400 your-key-pair.pem
    ```
   Using GUI:
   - Go to Key file's properties > security > Advanced
   - Change the owner of the file to yourself.
   - Disable inheritance.
   - Remove other accounts from principal.
   - Double click on you account from principal tab and give "Full      Control" permission.
3. Connect to your instance:
```
ssh -i "your-key-pair.pem" ec2-user@your-ec2-public-dns
```

>#### Step 4: Install and Configure Squid Proxy Server
1. Update the package list and install Squid:
```
sudo apt update
sudo apt install squid
```
2. Configure Squid by editing the Squid configuration file:
```
sudo nano /etc/squid/squid.conf
```
3. Add the following lines to allow access:
```
acl localnet src 0.0.0.0/0
http_access allow localnet
```
4. Start the Squid service:
```
sudo systemctl start squid
sudo systemctl enable squid
```

>#### Step 5: Configure SSH Tunneling
1. Create an SSH tunnel from your local machine:
```
ssh -i "your-key-pair.pem" -L 3128:localhost:3128 ec2-user@your-ec2-public-dns
```
**Note: You can use -D tag instead of -L to dynamically port forward. [Dynamic vs Local Port Forwarding](https://blog.jakuba.net/ssh-tunnel---local-remote-and-dynamic-port-forwarding/)**
This command forwards your local port 3128 to the Squid server on the EC2 instance.

>#### Step 6: Verify the Proxy Setup
1. Configure your web browser to use the proxy:
    - Set the proxy server to localhost and the port to 3128.
2. Verify your IP address by visiting a site like [WhatIsMyIP](https://www.whatismyip.com/). You should see the IP address of your EC2 instance.

---

># Conclusion
You have successfully set up an AWS EC2 server as a proxy and connected to it using SSH tunneling. This setup enhances your browsing privacy and security. For any questions or further assistance, feel free to refer to the FAQs below.
**Advantages of using a proxy server:**
1. Security
    Proxy servers can encrypt web requests, hide IP addresses, and filter content, which can improve network security and online privacy.
2. Access
    Proxy servers can allow users to access content that is blocked by companies or governments, or that is geo-restricted or censored.
3. Speed and bandwidth
    Proxy servers can improve internet speeds and save bandwidth by caching frequently accessed web pages and files, compressing data, and filtering unwanted content.
4. Control
    Proxy servers can be used to control and monitor internet usage, such as preventing employees from browsing inappropriate or distracting sites. 
---

># FAQs
1. Can I use a different proxy server instead of Squid?
    Yes, there are several other proxy servers like Tinyproxy, Privoxy, etc. The setup process will be similar.
2. How can I automate the SSH tunneling process?
    You can create a script to automate the SSH tunneling process. Simply save the SSH command in a shell script and execute it when needed.
3. What if I want to allow other users to use my proxy?
    You can modify the Squid configuration to specify the allowed IP ranges. Be cautious about the security implications.
4. Is it possible to use this setup for secure browsing over public Wi-Fi?
    Yes, using an SSH tunnel over public Wi-Fi encrypts your traffic, providing additional security.
5. How do I stop the Squid service if I no longer need the proxy?
    You can stop the Squid service using the following command:
    ```
    sudo systemctl stop squid
    ```