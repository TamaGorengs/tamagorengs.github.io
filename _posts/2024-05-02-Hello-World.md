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

While waiting for the tools to finish running, we explore the website. We found a contact page where we can input text. I tried inputting random strings but it just refreshed the page. Try inputting other payload like sqlinjection and XSS but none works. Move on.
![](htb-devvortex-box-contact.png "wikilink")

<figure>
<img src="htb-devvortex-box-network.png" title="wikilink"
alt="htb-devvortex-box-network.png" />
<figcaption
aria-hidden="true">htb-devvortex-box-network.png</figcaption>
</figure>

We found email convention that maybe we could use for any further brute forcing.
![](htb-devvortex-box-footer.png "wikilink")

After looking around and the result from dirsearch and gobuster doesn't seem useful. I then try fuzzing for subdomains and we found a subdomain dev.devvortex.htb. Nice.
![](htb-devvortex-box-ffuf.png "wikilink")

We try visiting the website but like before, we have to put into /etc/hosts first.
![](htb-devvortex-box-dev.png "wikilink")

After putting into /etc/host. We now found a new webpage to explore. And also like before, we run our directory enumeration tools before further exploring the page.
![](htb-devvortex-box-devhome.png "wikilink")

We run gobuster and dirsearch again.
![](htb-devvortex-box-gobuster2.png "wikilink")

We found another email but with same email naming convention. Other than that, nothing interesting that could help further. Let's check our dirsearch.
![](htb-devvortex-box-devcontact.png "wikilink")

And we found directory /administrator which is super interesting. Let's have a look.
![](htb-devvortex-box-dev-dirsearch.png "wikilink")

It's a Joomla Admin login page. Joomla is a Content Management System, just like WordPress.
We try default credential, but it doesn't work.
![](htb-devvortex-box-joomladefault.png "wikilink")

Let's google for any exploit or vulnerability in Joomla.
![](htb-devvortex-box-hacktricks-joomla.png "wikilink")

Here it's show how we can get the version of the joomla. We can just copy the path, or run the script.
![](htb-devvortex-box-joomlaversion.png "wikilink")

Here we found the version of the Joomla which is 4.2.6
![](htb-devvortex-box-joomlavers.png "wikilink")

Let's check for any possible exploit regarding this version of Joomla.
![](htb-devvortex-box-joomla-cve.png "wikilink")

Let's try the POC by Acceis on CVE-2023-23752.
![](htb-devvortex-box-acceis.png "wikilink")

The exploit occur because of improper access check within the application, enabling unauthorized access to critical webservice endpoints.

Here we can find the user and password.
![](htb-devvortex-box-joomlacred.png "wikilink")

Here we found the id, group, name and email of the users
![](htb-devvortex-box-joomlacred2.png "wikilink")

We can just run the script and it gave us the same information.
![](htb-devvortex-box-exploit.png "wikilink")

Let's try login into the CMS using the found credential.
![](htb-devvortex-box-adminlogin.png "wikilink")

And we are in !
![](htb-devvortex-box-joomlacms.png "wikilink")

Just like in wordpress, normally we could run a php code somewhere in the template where we could probably get a reverse shell. After googling for reverse shell in Joomla, I came across this blog. https://www.hackingarticles.in/joomla-reverse-shell/. Basically we could inject code in the template.

We go to Systems \> Site Templates \> And click the link of the template.
![](htb-devvortex-box-joomlathemes.png "wikilink")

In the code section, all we need to do is get a PHP reverse shell, and insert into the code. Make sure to change the IP and the Port. I chose error.php file.
![](htb-devvortex-box-phprevinject.png "wikilink")

Run our netcat listener on the port, and visit the file.
![](htb-devvortex-box-error.png "wikilink")
And voila, we now have A reverse shell. But it's not interactive. Let's use python tty.
![](htb-devvortex-box-revshell.png "wikilink")

Much better. We are user www-data. There's /logan directory in /home. We can get into it, but we cannot user flag because we don't have permission.

<figure>
<img src="htb-devvortex-box-initial.png" title="wikilink"
alt="htb-devvortex-box-initial.png" />
<figcaption
aria-hidden="true">htb-devvortex-box-initial.png</figcaption>
</figure>

Checking /etc/passwd. We know we have user logan. I also notice we have mysql in it. From the exploit on Joomla, we know that it extract the databases user and password. So we try login into mysql using those credential.

<figure>
<img src="htb-devvortex-box-etc-passwd.png" title="wikilink"
alt="htb-devvortex-box-etc-passwd.png" />
<figcaption
aria-hidden="true">htb-devvortex-box-etc-passwd.png</figcaption>
</figure>

And looks like we can, now we have access to mysql databases. Let's see what's in here.
![](htb-devvortex-box-mysql.png "wikilink")

We found Joomla Database.
![](htb-devvortex-box-databases.png "wikilink")

Many tables in the databases but none interesting. Yet.
![](htb-devvortex-box-tables1.png "wikilink")

And at the bottom, we found table users which is obviously interesting. Let's see what's inside.

<figure>
<img src="htb-devvortex-box-tableuser.png" title="wikilink"
alt="htb-devvortex-box-tableuser.png" />
<figcaption
aria-hidden="true">htb-devvortex-box-tableuser.png</figcaption>
</figure>

And looks like we found user and password hash for lewis and logan.
![](htb-devvortex-box-hashes.png "wikilink")

I only crack user logan since we know from /etc/passwd, that there's no user lewis in this machine.
![](htb-devvortex-box-hashcatlogan.png "wikilink")

Hashcat manage to crack it and found the password.
![](htb-devvortex-box-crackedlogan.png "wikilink")

SU into the user, and we are now logged in as user logan.
![](htb-devvortex-box-sulogan.png "wikilink")

As always, check low hanging fruit. We ran sudo -l to check for any sudo permission
![](htb-devvortex-box-sudol.png "wikilink")

We found the version of the apport-cli
![](htb-devvortex-box-apportver.png "wikilink")

Googling for exploit for this version lead me to this 2 post.

https://bugs.launchpad.net/ubuntu/+source/apport/+bug/2016023
https://github.com/diego-tella/CVE-2023-1326-PoC

> If a system is specially configured to allow unprivileged users to run sudo apport-cli, less is configured as the pager, and the terminal size can be set: a local attacker can escalate privilege.

After playing with the apport-cli, -f is where we can report for any bug or problem. We can try choose any until it lead to the next option.

<figure>
<img src="htb-devvortex-box-apport1.png" title="wikilink"
alt="htb-devvortex-box-apport1.png" />
<figcaption
aria-hidden="true">htb-devvortex-box-apport1.png</figcaption>
</figure>

From here, type `V` which bring us to the new pager.
![](htb-devvortex-box-apport2.png "wikilink")

This pager uses LESS, and just like VIM, we put `!` to execute command. We run `!id` which show that we can execute as root !

![](htb-devvortex-box-rootid.png "wikilink")
![](htb-devvortex-box-execroot.png "wikilink")

Now we try execute `!/bin/bash` to get bash shell as root.
![](htb-devvortex-box-binbash.png "wikilink")

And lo and behold, we are now root. All we need to do now is grab the root flag and we have completed Devvortex.
![](htb-devvortex-box-root.png "wikilink")

------------------------------------------------------------------------

## Resources:

| Hyperlink                                     | Info          |
|-----------------------------------------------|---------------|
| https://app.hackthebox.com/machines/Devvortex | HTB Devvortex |
