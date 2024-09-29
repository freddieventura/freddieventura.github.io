---
title: Setting Up a Free Signed TLS Certificate
tags:  tls, encryption, webserver, linux, dns
article_header:
  type: cover
  image:
    src: images/under-construction.png
---

# Setting Up a Free Signed TLS Certificate

This is going to be a brief tutorial, on acquiring a free TLS certificate and setting it up to your webserver, so following the modern standards required for trusteability on websites that are to be shown to the general public.
Your use-case may vary in some aspects , but taking it to a [minimal, reproducible example](https://stackoverflow.com/help/minimal-reproducible-example) will help you through the process.

Assuming we will be using the following:
    - A VPS using Linux Debian
    - Apache2 Webserver
    - https//letsencrypt.org , for CA
    - ACME protocol for automated signed certificates retrieval

## Tutorial Index

    We are going to set up a signed TLS certificate using the Acmetool on letsencrypt.org
    - First purchase a Domain name (for instance https://namecheap.com )
    - Set up the DNS records
    - Using Acmetool
    - Configuring your webserver to use TLS
    - Setting up a cronjob
    - Article Sources

## Purchase a Domain Name on https://www.namecheap.com/
    - Sign up
    - Check an available Domain
    > Dashboard > Search for your next domain > Fill in
                > Select one option > Add chart > Give registration and billing details

## Set up the DNS records

We will need to add two basic records, the one pointing to all the subdomains within the domain, and the specific one for the www. subdomain. ^[1]

    Namecheap.com > (Login) > Dashboard > (Select Domain name) > Manage > Advanced DNS (hidden on a burguer dropdown menu)
    Host Records > + Add New Record (on red)
    
    Record Type > A Record
                Host > @   Ip Address > (yourVpsIp)   TTL > Automatic

    Host Records > + Add New Record (on red)

    Record Type > A Record
                Host > www   Ip Address > (yourVpsIp)   TTL > Automatic
            
                
In your VPS you should have a webserver such as `apache2` , just start it and test it on your broswer first by IP , then test it by domain and subdomain on your browser.
Troubleshoot WAN access to your webserver such as designated ports for the service, firewalls etc..
For DNS records you may have to wait some minutes (up to 30 minutes) , so be patient.

```
sudo apt install apache2
sudo systemctl start apache2

# On local machines browser try
http://<yourVpsIp>
# On local machines browser try
http://<yourDomain>.<domainAuthority>
http://thepepe.xyz
# On local machines browser try
http://www.<yourDomain>.<domainAuthority>
http://www.thepepe.xyz
```


When you get connectivity to your `apache2` server you will be ready for the next step


## Using Acmetool

In order to get a free SSL signed certificate we will use the **ACME** protocol on a free Trusted CA (Certificate Authority) which is trusted by the browsers (for that Green padlock) https://letsencrypt.org/ . ^[2] ^[3]

Install (steps below for Debian)

```
sudo apt update
sudo apt install acmetool
```

Using ACME tool ^[4]
Note , for the following steps we are asuming you have got the basic webserver (the one that comes with apache2 with just the login page), on port 80.

```
sudo acmetool quickstart 
1) Lets Encrypt Live Server
1) Webroot

sudo acmetool want <mySite>.<domainAuthority>
sudo acmetool want thepepe.xyz
```

Note: The last step will call for `www.thepepe.xyz` for some reason.
If you didn't get any error the signed certificates should have been generated on `/var/lib/acme/live/www.thepepe.xyz/`


## Configuring your webserver to use TLS

Here we need some modifications to our webserver in order to parse that SSL certificate by its side.
We will be using `apache2` as we did before

```
sudo a2enmod ssl

sudo vi /etc/apache2/sites-available/000-default.conf

<VirtualHost *:443>                                                        
    ServerName www.thepepe.xyz                                      
    ServerAlias thepepe.xyz                                         
                                                                            
    DocumentRoot /var/www/html                                              
                                                                            
    SSLEngine on                                                            
    SSLCertificateFile /var/lib/acme/live/www.thepepe.xyz/fullchain 
    SSLCertificateKeyFile /var/lib/acme/live/www.thepepe.xyz/privkey
    SSLCACertificateFile /var/lib/acme/live/www.thepepe.xyz/chain   
                                                                            
    # Other SSL configurations (optional)                                   
</VirtualHost>                                                              


sudo systemctl restart apache2
```

Now you should be able to prove your website

```
## On your browser
https://www.thepepe.xyz
```

Remember to prove it to the Domain Name , otherwise it will have trust SSL issues , such as if you do

```
https://<myServerIp>.xyz
```


## Setting up a cronjob

Using free letsencrypt.org TLS certificates come with a caveat, they expire every 90 days, so you need to be repeating the `acmetool` process every 90 days.
For this matter just set up a `cronjob`

```
0 0 1 * * /usr/local/bin/acmetool reconcile --batch; service nginx restart
```

This will renew the certificates every 30 days at each 1st of each month



## Reference sources

A series of references on the form of ^[1] , ^[2] , ... , are placed throughout the article to the following references.
1. [How do I set up host records for a domain?](https://www.namecheap.com/support/knowledgebase/article.aspx/434/2237/how-do-i-set-up-host-records-for-a-domain/)
2. [Securing Your Website with Let’s Encrypt: A Step-by-Step Guide to the ACME Protocol](https://medium.com/@saichander17/securing-your-website-with-lets-encrypt-a-step-by-step-guide-to-the-acme-protocol-f6a1c2d3a8e2)
3. [Getting Started - Let's Encrypt](https://letsencrypt.org/getting-started/)
4. [Protect your site for free with Let’s Encrypt SSL certificates and acmetool](https://griggheo.medium.com/protect-your-site-for-free-with-let-s-encrypt-ssl-certificates-and-acmetool-3139dd5af5d0)


