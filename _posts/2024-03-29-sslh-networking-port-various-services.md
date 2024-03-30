---
title: Using a single networking port for various services with sslh
tags:  sslh, networking, port, linux, tcp, udp, openvpn, ssh, http, ssl
article_header:
  type: cover
  image:
    src: images/under-construction.png
---
# Using a single networking port for various services with sslh

Have you ever wondered if you could serve more than one service in one single networking port?.
If you googled this you may have got a no for an answer, but let me tell you one thing in between you and me, there is a way, there is always a way.
In the following lines I will be explaining you how to use `sslh` an ssl/ssh multiplexer that can allow you serve different applications through a single port.

## My usecase

I have started on this having a VPS which I SSH'ed to it, when I was at home everything fine, but once at Uni , or in the Library, or later on in my life in a co-working space, it wouldn't let me in.
You have probably changed the SSH port to 80 or 443 so you could get into, (maybe the firewall filter was more sophisticated and you needed to serve an obfuscated version of ssh)
For this first use case, we started swapping ports , having to use some services in a different purposed port for a legitimate reason. And by doing so you would have come across adding some more services, such as a later VPN, so you picked the remaining of 80 or 443 , and then you wanted to serve again a website and wait, `Address already in use` error.

And there is a way to bind a port to different applications, and is by using a 3rd application `sslh` that serves as a router, identifying the kind of application is doing the request and redirecting the request to a 2nd Port through the loopback interface in which the service is actually been served.

In this case you multiply your options , been able to serve now one instance of each Application per port , instead of one Appication per port.


For my current needs I am using `apache2` through `http` and `https` , `ssh` , two instances of `openvpn` and planing to add further services such as `shellinabox`

My new plan is the following

```
                                 ___________
                                |:8080      |
    ___________/--------------->|       http|
   |:80        |                 -----------
   |           |-------+         ___________
    -----------        |        |:8102      |
         |             +------->|    openvpn|
         |                       -----------
         |                       ___________
         +--------------------->|:22        |
                                |       sshd|
                                 -----------



                                 ___________
                                |:8443      |
       +----------------------> |      https|
       |                         -----------
    ___________                  ___________
   |:443       |                |:8101      |
   |           |  ------------> |    openvpn|
    -----------                  -----------
```

## Installing sslh

As for today the Debian Repositories have a really outdated version of `sslh` so I will be installing from source, although it may work perfectly fine the already built packages of Debian Bookworm


```
git clone https://github.com/yrutschle/sslh                               
cd sslh/
./configure                                                               
sudo apt install libconfig-dev                                            
sudo apt install libpcre2-dev                                             
sudo apt install libev-dev                                                
make                                                                      
sudo make install                                                         
```
Note: I had to install some libraries for building the package, you may need some more. If you get a compilation error use on the missing header the following to find that package

```
apt-file find <package.h>
```

## Changing the ports of your applications

In my case I have listed 3 applications , `sshd` , `apache2` and `openvpn` . You may need more you may need less, the important take is the following "Each application must serve its connection in other port" or in other words "no application must share a port" even if they are in fact sharing it. `sslh` is just routing that incoming traffic on `:80` and `:443` on a 2nd port, and this have to differ to this two , and they also differ each other.

Regarding to the firewall, just close all the ports excepts the ones you need to use them for WAN connectivity, in my case `:80` and `:443`. The rest will communicate through the loopback interface `127.0.0.1` which is not firewalled.

Also the final applications must be listening on the loopback interface only, for security purposes.

Thus the following commands in

### apache2

```
sudo vi /etc/apache2/ports.conf

Listen 127.0.0.1:8180         
                              
<IfModule ssl_module>         
        Listen 127.0.0.1:8443 
</IfModule>                   
                              
<IfModule mod_gnutls.c>       
        Listen 127.0.0.1:8443 
</IfModule>                   

sudo vi /etc/apache2/sites-available/000-default.conf

#change <VirtualHost *:80>
<VirtualHost *:8080>
#for
#change <VirtualHost *:443>
#for
<VirtualHost *:8443>

sudo systemctl restart apache2
```

### sshd

```
sudo vi /etc/ssh/sshd_config
ListenInterface 127.0.0.1
Port 22

sudo systemctl restart sshd
```

### openvpn

Note in here it may differ from you, but I have two servers running with different configurations
Note also if you moved your incoming connection to other port, to change the clients `.ovpn` accordingly.

```
sudo vi /etc/openvpn/server1.conf
local 127.0.0.1
port 8101

sudo systemctl restart openvpn@server1
```


## Creating sslh systemd units

Now you can use `sudo /sbin/sslh --help` to check all the options, even spawn this program in a shell to check if it works correctly, but appart of for troubleshooting purposes , you can just go straight an do systemd Unit files for persistant and daemonized usage.

For multiplexing port 80
```
sudo vi /etc/systemd/system/sslh@port80.service
sudo vi /etc/systemd/system/sslh@port80.service

[Unit]
Description=SSL/SSH multiplexer
After=network.target

[Service]
ExecStart=/sbin/sslh --foreground --listen 0.0.0.0:80 --ssh 127.0.0.1:22 --http 127.0.0.1:8180 --openvpn 127.0.0.1:8102 --tls 127.0.0.1:443 --pidfile /var/run/sslh-port80.pid
KillMode=process

[Install]
WantedBy=multi-user.target

sudo systemctl start sslh@port80
sudo systemctl enable sslh@port80
```

Analogously for port 443

```
sudo vi /etc/systemd/system/sslh@port443.service

[Unit]
Description=SSL/SSH multiplexer
After=network.target

[Service]
ExecStart=/sbin/sslh --foreground --listen 0.0.0.0:443 --tls 127.0.0.1:8443 --openvpn 127.0.0.1:8101 --pidfile /var/run/sslh-port443.pid
KillMode=process

[Install]
WantedBy=multi-user.target

sudo systemctl start sslh@port443
sudo systemctl enable sslh@port443
```

## Final notes

If you google configurations of `sslh` it has its own config files, which may help you achieve what I did instanciating different systemd units. I preferred my way because I couldn't achieve what I wanted, and this way is the only way it worked to me, but just note there may be an easier way out there.







```
