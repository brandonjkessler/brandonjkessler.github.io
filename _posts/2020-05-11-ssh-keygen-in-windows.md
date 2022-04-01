---
title: SSH Keygen in Windows
Author: Brandon J. Kessler
tags: DevOps SysAdmin Windows Scripting
category: Tip
published: true
---
<h1>{{ page.title }}</h1>

## TL;DR

```
ssh-keygen -t rsa | ecdsa | ed25519
```
<!--more-->
## Overview

Like most homelab-ers I use GNU/Linux for my servers/container. I am also lazy, so I try to use PuTTy and OpenSSH as often as possible. I also try to use a unique key pair for all of my servers for security, but doing so in the past has usually required me busting out my trusty Elementary OS laptop and creating those key pairs (I don’t like how PuTTy generates the keys). Imagine my joy when I found out that you can generate key pairs in Windows 10!

You can do this command in both Windows Command Prompt and PowerShell. Running `ssh-keygen /?` will bring up all the myriad of options. That’s it, that’s all there is to it.