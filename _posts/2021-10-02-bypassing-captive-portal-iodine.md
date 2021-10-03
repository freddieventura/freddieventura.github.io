---
title: Bypassing a captive portal with iodine
tags:  tutorials, linux, hacking, networking, iodine, dns tunnel, dns, captive portal
article_header:
  type: cover
  image:
    src: images/bypassing-captive-portal-iodine.jpg
---

## Bypassing a captive portal with iodine

This is just a quick guide to follow with the tutorial of the following [video](https://youtu.be/huJoRzWVedI).
After completing it you will be able to get access to internet in a retricted Access Point which requires of authentication via Captive Portal
Note: that the following procedure if done in a public Network with no consentment of the Administrators is ilegal in many countries, so either get consentment to do it or try it in your own environment.
The objective of these tutorials is always educative, being aware of the weaknesses of the system will make you capable of providing safer environments to your customers, organization or household

Requirements:
    - Linux (or any other major OS)
    - VPS
    - Purchased Domain Name (1 Pound a year in [namecheap](https://www.namecheap.com) )
    - [iodine](https://github.com/yarrick/iodine/)
    - Medium knowledge of Networking environments.

### Iodine in a nutshell

**Iodine** is a tool that allow us to create *DNS Tunnels* across networks, these *DNS Tunnels* create a virtual link in between the Host and your VPS by establishing *Virtual NIC* in each side of the connection. This connection uses the **DNS** protocol [Domain Name System](https://en.wikipedia.org/wiki/Domain_Name_System) in a unconventional way to transfer data through it.

As many of the Restricted Networks don't block the communications on **UDP 53** , (because of the very nature of this protocol or the *DNS Redirection* the cannot) we will be able to get access to the internet despite having authanticated on the **Captive Portal**.

Becaue of the way the data is been transfered the main caveat is that the connection bandwidth is really reduced, as well as the latency is quite high, so it's practical usage is really restricted.

### Tutorial

First and obvious disclaimer , you need to have full internet access first in order to make the arrangements, don't intend to make magic.

##### 1st) Purchase a domain in a domain registar
    https://www.namecheap.com/
    yourdomainname.xyz (1.18 Dollars a Year first year)

##### 2nd) Add the DNS records
```
    > You Account > Dashboard > Domain List > Manage > Advanced DNS > Add record
```
Add 2 records:

```
-  A record : dnsa.yourdomainname.xyz → your VPS IP address
- NS record : t.yourdomainname.xyz → dnsa.yourdomainname.xyz 
```
Wait for it propagate (1h to 24h)

##### 3th) Install iodine in both sever and client

Both in Server and client do
```
sudo apt install zlib1g-dev
git clone https://github.com/yarrick/iodine
cd iodine
sudo make
sudo make install
```

Only on Server allow the UDP Protocol in through the firewall

```
sudo ufw allow in from any to any port 53
```

##### 4th) Leave the server listening

Server side
Note: it will prompt you for a password, set up a password
Note2: Remember to run it via tmux session so the daemon persist when you disconnect from your server, if not you can do a systemd unit for it
```
sudo /usr/local/sbin/iodined -f 10.0.0.1 yourdomainname.xyz
```

##### 5th) Establish the connection with the Client

Once the previous steps were done you already have the insfrastructure ready to go to this restricted environment and attempt the operation
You should connect to that restricted AP , and check that in the Browser it will prompt you to go to the captive portal. Use iodine instead

You can ping any host in the internet , (as long as you know in advanced they are pineable) , and check that you don't get reply, althoug the domain name has been resolved into ip
```
ping bbc.com
dig bbc.com
```
If you are not able to ping, but you are able to resolve the domain names then you are good to go.

```
sudo iodine -f -r $yourVpsIp yourdomainname.xyz
```

It should output connection established in both sides of the link.
See if there are any new **NIC** 

```
ip addr
```
You should see one named dns0 

##### 5th) Change the routes on the client

We need to change the Default Gateway which is established to be the one of the Network, you need to use that route only for the DNS tunnel specifically (so to your VPS) , the rest is gonna go through the DNS tunnel link connection established

```
sudo ip route add $yourVpsIp via $networkDefaultGatewayIp dev $yourWanNicName
## in my case 
## sudo ip route add 192.100.200.45 via 10.100.96.1 dev wlp1s0
sudo ip route del default via 10.0.0.1 dev dns0
```

You should now be able to ping any outside domain

```
ping bbc.com
```

Pacience it will be slow, you can use the browser but it will take some time to load a fully featured website.


Thumbs up to the developers who did [iodine](https://github.com/yarrick/iodine/) for making us aware of such a vulnerability. 

See you in the next tutorials.
