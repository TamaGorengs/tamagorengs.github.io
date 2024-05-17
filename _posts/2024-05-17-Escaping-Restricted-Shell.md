---
title: Escaping Restricted Shell
date: 2024-05-17 09:00:00 +0800
categories: offsec
tags: privesc
image: /assets/general/escape-shell.png
---

Good day everyone, been quite some times since I last post anything in this blog. Gonna try be more consistent after this. Wish me luck.

Today I want to share about How to escape restricted shell. But first what is Restricted Shell ?

## Restricted Shell

Basically a way to restrict user from executing command other than what is being allowed. Take example below is using `rbash`. We try to use `clear` and `cat` command but the shell does not even found the command.

<figure>
<img
src="/assets/etc/98e5cf463dcb735ada51fc3fd2117ea8129dbf6c.png"
 alt="offsec-dc2-restrictedshell.png" />
</figure>

Before we continue I need to explain what is `PATH` in linux.

## What is PATH in Linux ?

`PATH` is an environment variable that specifies a list of directories where the system looks for executable files. In easy words, if we were to execute command, for example `ls`, the system will check through all the PATH to see if there's `ls`. If there is, then the command can be executed, but if none, it will prompt `command not found` like images before.

We continue.

To check what command is allowed we can check the `PATH` by using `export -p` to show the environment.

<figure>
<img
src="/assets/etc/b5a81dd2b04001547cbc0946e3fdf2cee072b185.png"
 alt="offsec-dc2-exportp.png" />
</figure>

Then we `-ls $PATH`. This show what command that we are allow to execute.

<figure>
<img
src="/assets/etc/8e5e8350af61b3485e087c34bff7fee46cd9410b.png"
 alt="offsec-dc2-lsexport.png" />
</figure>

To escape this restriction shell, we can use any of the above command and check on GTFOBins and check if there's function `shell` on the allowed command. Here we can see `vi` does have Shell function.

<figure>
<img
src="/assets/etc/23479e5ae4d5a165d8163178fddbc58892e5b9c7.png"
 alt="offsec-dc2-gtfobinvi.png" />
</figure>

We can try any of these 2. I tried both but the second one works. We only need to copy all the command and paste it on the restricted shell that we have.

<figure>
<img
src="/assets/etc/6b321fe1cba27f6efd17a10e784676436a8b7dff.png"
 alt="offsec-dc2-gtfobinvi2.png" />
</figure>

When prompt below, just press enter.

<figure>
<img
src="/assets/etc/8615b99284b336e6f6a3c26b9f0bf938fce4fe2e.png"
 alt="offsec-dc2-gtfobinvishell.png" />
</figure>

Then we can export the default path for linux, since this is linux box. Then we now have the ability to execute command like normal again since the PATH for the linux is not restricted to only `/home/tom/usr/bin` anymore.

<figure>
<img
src="/assets/etc/b1ae5b8e00bf8c2a48f6309432fab8b0d3f8c9dd.png"
 alt="offsec-dc2-exportpath.png" />
</figure>

There's other way you can experiment on how to escape restricted shell. You play around this box on [https://portal.offsec.com/labs/play](https://portal.offsec.com/labs/play), Room: DC-2.

Thanks for reading !
