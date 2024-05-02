---
title: Hello World
date: 2024-05-02 01:22:33 +0800
categories: test, blog
tags: hack pentest     # TAG names should always be lowercase
---



# [HTB - Devvortex](HTB - Devvortex "wikilink")

------------------------------------------------------------------------

## Summary :

We found 2 open ports(22, 80). We fuzz and found other subdomain which lead to directory of Joomla CMS Login Page that is vulnerable and allow us to extract DB user and password that is also used to login to the CMS. The CMS has template that allow us to inject PHP code, where we get reverse shell. We login to mysql using found credential from Joomla vulnerability and crack other user to escalate our privilege. To escalate to root, we can run apport-cli as sudo where there is vulnerability on the version where you can execute command as root when viewing the report crash since it use LESS.

First thing first, we run the machine to receive our target IP
![](/assets/devvortex/htb-devvortex-box.png)

I first run rustscan to quickly scan for open port and as we can see we have 2 open ports which is port 22(SSH) and port 80(http)
![](/assets/devvortex/htb-devvortex-box-rustscan.png)

I then run nmap to scan the version and run default script. We can see that it redirect to devvortex.htb.
![](/assets/devvortex/htb-devvortex-box-nmap.png)

As we can see we can't access the machine since we the DNS can't resolve it.

Let's put the IP into /etc/hosts first.
![](/assets/devvortex/htb-devvortex-box-etchosts.png)

And now we can access it normally. Let's dig in. But before that I ran enumerate directory using automate tools.
![](/assets/devvortex/htb-devvortex-box-web1.png)

My tools of choice is dirsearch as well as gobuster. I ran both because sometimes dirsearch miss something, vice versa.
![](/assets/devvortex/htb-devvortex-box-dirsearch.png)
![](/assets/devvortex/htb-devvortex-box-gobuster.png)

