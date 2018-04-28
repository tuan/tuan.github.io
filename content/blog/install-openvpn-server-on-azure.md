---
title: "Install Openvpn Server on Azure"
date: 2018-04-27T19:50:34-07:00
draft: false
---

We're going to install OpenVPN Server on a Ubuntu Server VM running on Azure.
Steps:

1.  Go to portal.azure.com > Marketplace and search for Ubuntu Server.

2.  Select a Ubuntu Server image that we want and follow the steps on Azure Portal to create a new VM.

    * I use a cheapest VM size that I can find in the list which is A0. It costs about $4/month.
    * I want the VPN server to be on all the time, so I disable Auto Shutdown option.

3.  After the VM is ready, `ssh` to the server using `your-user-name@vm-public-ipaddress`

4.  Execute this command

    ```
    wget https://git.io/vpn -O openvpn-install.sh && sudo bash ./openvpn-install.sh
    ```

    * Use the default IPv4 address of the network insterface.
    * Use UDP
    * Use port 1194
    * Use current system resolver for DNS option
    * Your VM is behind a NAT, so specify a external IP address. This is the Public Ip Address that you find in Overview page for your VM. You can also use a public host name, e.g. `your-custom-name.westus.cloudapp.azure.com`, here instead. This public host name is listed under DNS category in the Overview page, if not specified, you have the option to configure it.

5.  Copy the output .ovpn file to use on the client later to connect to this vpn server.
6.  Go to Network Interface resource for your VPN and add an Inbound connection rule to whitelist port 1194 (the port that you chose in step 4)

That's it. Now you can test your vpn server by openning the ovpn file in step 5 in a OpenVPN client. I use OpenVPN Connect app on my Android to test. In OpenVPN Connect app, you have an option to import a .ovpn file from SD card which makes it very easy to setup the connection to your server.
