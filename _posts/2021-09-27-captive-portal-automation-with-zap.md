---
title: Captive Portal Automation with Zap
tags:  tutorials, linux, Zap, http, captive portal
article_header:
  type: cover
  image:
    src: images/automating-captive-portal-zap.jpg
---

## Captive Portal Automation with Zap

This is just a quick guide to follow with the tutorial of the following [video](https://youtu.be/C_2w9mlxc-I).
After completing it you will be able to automate a captive Portal login process with the use of Zap

Requirements:
    - Linux (or any other major OS)
    - [Zap Proxy](https://github.com/zaproxy/zaproxy)
    - Firefox or Chrome

Technical knowledge requirements:
    - [HTTP protocol - Wikipedia](https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol)
    - [Curl - Manpage](https://linux.die.net/man/1/curl)
    - [Zaproxy - Getting Started](https://www.zaproxy.org/getting-started/)
    - [Zap - Deep Dive youtube playlist](https://www.youtube.com/watch?v=CxjHGWk4BCs&list=PLz_NN8o2uh8AQ7VyUEN1GCCnpzl5_FaJA)

#### Zaproxy in a nutshell

Briefly `Zaproxy` is a tool that allow us (among other things) to intercept and analize the communications in between our Browser and a Web Server.
It let us read register those HTTP requests, and their HTTP Responses received. 
We can Browse the Websites manually with the Proxied Browser or let `Zaproxy` spider capabilities analize it for us.

After registering those requests we can do many things with that, from scripting a certain process so automating it, to also use `Zaproxy` as a man in the middle and change Headers and Body of Responses accordingly. 

Those are among the utilities that I have seen myself in this software but there may be way more over there I haven't explored myself , feel free!!

This program is mainly based to be used with their GUI , so I will just paste below a quick guide on how to get graphicaly to the commands we need in order to automate a captive portal.


#### Brief explanation of a captive portal

A captive portal is just a web server that intercepts the HTTP request than you send from a newly connected AP, it responds you with a Web Form that you need to perform in order to authenticate yourself within the organization that owns that AP.

They are generally found in public spaces, or corporative as well.

They do keep track of :
    - MAC address of the Device connected
    - IP Address leased to it
    - Timestamp of the device connected
    - Geolocation (their own AP geolocation)
And will commonly ask for:
    - E-mail address
    - User Name
    - Login/ Password

(Depending on the type of network your are getting into they will need certain credentials from you)

#### Automating the portal

First lets reset our MAC address and reconnect to the Wifi in order to force a reauthentication to the portal.

```
interface=wlp1s0
sudo systemctl stop NetworkManager
sudo ifconfig $interface down
sudo macchanger -r $interface
sudo ifconfig $interface up
sudo systemctl start NetworkManager
```
Then lets open `Zaproxy`

```
zap.sh &
```

A little go through the functionalities you need to understand in order to perform the video tutorial

```
## analizing the interaction with certain portal
# 1st) Open zap GUI
# 2nd) Press in Explore Manually
# 3rd) Use Browser Accordingly
# 4th) Check the HTTP requests and responses that you are interested in.
    Tip: , a)copy as curl the requests (Right Click copy as curl command)
           b)For responses:
                - Press combined display for headers and body
                    Ctrl + A > Ctrl + C
                    $ xclip -o | pandoc -f html -t plain | xclip -i
                Then you can paste the response in plain text (with http headers) right after the curl command
https://github.com/zaproxy/zest

# create a basic zest script
    Scripts > Standalone (Right Click) > Create Script
    You can check the new script , is a JSON file (dont modify it manually)
        Right click on the script >
                        Add Request
                        Add Zest action                                
                        Add Zest CLient
                        (...)
    Add zest Action > Print "Hello Zest World"
    Drag the Action to the top

## creating a more complex zest script

> Press in the Robot Icon (Top Right) > Record Zest Script
        > Title "Zest bodgeit comment"
        > Standalone
        > Record Type: Server side script
        > Start Recording

    > Now go to the Zest Browser and perform an action on the website
            > Press again the Robot Icon
        > Goto Standalone > "Zest bodgeit comment" > Press Run

    Note that when you run one of this scripts the actions will be evaluated as Ticked (Sucessfull) or Crossed (Unsuccessfull)  
        That is based on Assertions , which states what should be the HTTP response of the HTTP requests you send via Zest Script

    Note: Zap circumvents anti csrf web policies
        Thanks to > Options > Anti-CSRF Token 

##    You can add actions done in the ZAP's history to a Zest Script by
    History > Hold Press Shift > Select Various Requests > Right Click > Add to Zest Script
##  You can change the parameters inputed on the script by
    Scripts (Window) > Double Click on POST request (within Zest Script)
                Headers , Body (can be altered)
## Note : When executing a Zest Script , it will create its own "History" Tab , "Zest Results"
    Batch registering loads of users
            > Right Click (Inside Script) > Add Zest Assignment        
                Assign Variable to Random Integer 
                    > Variable Name : rnd
                                        > OK 
                    (Drag and drop the assignment to the top in the script to get executed first) 
            > Add Zest Assignment   >  Add Variable to String
                                        > Variable Name : "name"
                                            String : test-(right CLick Zest Paste Variable > "rnd"
        Double Click POST mesage : Right click Text field , Paste Zest Variable {{name}}  so it is {"email":"{{name}}@test.com"...

Chaging the Assertion (sometimes are been assigned wrong)
    Delete the one previously > RighClick Add Assertion > Regex \Q"status":"success"\E
Listening to a response string and assign itto a variable
    Add Zest Assignment > Assign variable via String delimiters 
                    Variable name : id
                    Location : BODY
                    Prefix String : "id":
# from "id":25,"email"  remove the 25 , split the "id": (prefix) , the rest for (postfix)
                    Postfix String : ,"email"
now we can use that variable in other posts requests


## adding parameters to a Zest Script
    Double Click on the Script > Parameter Tab > Add > 
                        Name:  name
                        Value: 
    (Now anytime you run the script will prompt you for that parameter)

```

With this post/video just appreciating the effort the [Zapproxy](https://github.com/zaproxy/zaproxy) team has done to bring this tool to us. Keep it up!!
