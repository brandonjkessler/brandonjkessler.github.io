---
title: 
Author: Brandon J. Kessler
tags: linux ubuntu 
categories: technology review
published: false
---

<h1>{{ page.title }}</h1>

Recently my used MacPro 2013 (TrashMacPro) died on me. It refuses to load MacOS and I've tried multiple SSDs, different known good RAM sticks, etc. Looks like it's toast. I absolutely refuse to go back to Windows as a consumer, and I can't really justify buying another computer at the moment, so I went in search of a linux operating system that I can throw on a spare laptop to use as my daily driver. I started on SUSE back in the day, and then moved to Ubuntu 8.10 I think. From there I've distro-hopped between SUSE and some form of Debian derivatives. I tried Kubuntu but HATE Snaps. They aren't compelling enough to switch to and aren't actual as portable as Flatpaks because of some [security settings that are currently Ubuntu only](https://forum.snapcraft.io/t/is-it-less-secure-to-run-snaps-in-opensuse/44601/3). This led me to [AnduinOS](https://www.anduinos.com), an Ubuntu derivative that looks and feels similar to Windows 11.

<!--more-->

## What is AnduinOS
AnduinOS is the brainchild of Anduin Xue, an ex-Microsoft Software Engineer, that they developed in their spare time. In their post "[Story behind AnduinOS - A letter from Anduin](https://news.anduinos.com/post/2025/5/6/story-behind-anduinos-a-letter-from-anduin)" they write "It was simply a toy and practice exercise I put together during a leisurely afternoon." They mention the use of Windows at work, and they prefer the interaction logic of Windows, but not the actual shell and adds. In their spare time they use Linux exclusively, preferring Arch or NixOS. The gist of the letter is that they created AnduinOS as a passion project in such a way that requires very little maintenance. It also appears to be more of a showcase for the build system they hope that AnduinOS will be "...user-friendly experience for decades but also to become a versatile customization tool and builder for Linux distributions," according to their post in [AnduinOS Future Development and Roadmap](https://news.anduinos.com/post/2025/5/21/anduinos-future-development-and-roadmap)

## Installing
This is a pretty short section since if you've ever installed Ubuntu nothing here will seem drastically different. You'll walk through the basic system setup, and it has a few nice customizations, but it's nothing crazy, which to be frank, is nice. A simple and intuitive installer built on something that Ubuntu has refined is not a terrible idea. And it also symbolizes what AnduinOS is going to be overall. It's Ubuntu, but tweaked in some interesting ways.

![](../assets/screenshots/anduinos_review/anduinos_install_1.png)
![](../assets/screenshots/anduinos_review/anduinos_install_2.png)
![](../assets/screenshots/anduinos_review/anduinos_install_3.png)
![](../assets/screenshots/anduinos_review/anduinos_install_4.png)
![](../assets/screenshots/anduinos_review/anduinos_install_5.png)
![](../assets/screenshots/anduinos_review/anduinos_install_6.png)
![](../assets/screenshots/anduinos_review/anduinos_install_7.png)
![](../assets/screenshots/anduinos_review/anduinos_install_8.png)
![](../assets/screenshots/anduinos_review/anduinos_install_9.png)
![](../assets/screenshots/anduinos_review/anduinos_install_10.png)
![](../assets/screenshots/anduinos_review/anduinos_install_11.png)
![](../assets/screenshots/anduinos_review/anduinos_install_12.png)
![](../assets/screenshots/anduinos_review/anduinos_install_13.png)
![](../assets/screenshots/anduinos_review/anduinos_install_14.png)

## Basic Layout
The basic layout of AnduinOS is heavily inspired by Windows 11, with the taskbar at the bottom, the notification center is on the bottom right along with the date and clock, and the start menu is to the left of the icons on the taskbar. There's even the weather widget that sits to the far left. There's a customized icon set, Fluent, that helps unify a lot of the system, and little touches to blur the shell. Pressing the "start" button reveals the start menu with an "All Apps" arrow up top, your pinned apps, and below that a frequent apps. On the very bottom of the window is your username, which will take you to settings for your user account, and then Files, Console, and Settings shortcuts, along with shutdown shortcuts.

![](../assets/screenshots/anduinos_review/anduinos_desktopcustomized_1.png)
![](../assets/screenshots/anduinos_review/anduinos_startmenucustomized_1.png)
![](../assets/screenshots/anduinos_review/anduinos_startmenu_allapps_1.png)

Even though it looks very similar to Windows it is not like Winux or Lindows or Wubuntu or whatever in which it tries to emulate the entire OS visually, too. AnduinOS strikes a good balance of emulating the layout and workflow of Windows, without being exactly like Windows. I also use Windows daily at work, and when I was using MacOS as my daily driver, had to context switch from work to personal or personal to work. With AnduinOS, however, the general layout is similar enough that there's not much context switching, but it's also different enough that I don't mistakenly try to run Windows commands or use it like I would windows. I think this, like ZorinOS, is the better way to do it. Keeping the layout but reminding users it's _not_ Windows will lead to fewer frustrations.

That being said, if you don't like the windows layout, you probably won't like this one either. I might be a heretic, but I actually like the taskbar icons and start button in the center, especially with dual monitors or ultra-wides. I'm not mousing as far to get to where I need to be. 

## Using Day-to-Day
#-- TODO: discuss the day-to-day usage

## Customizations
#-- TODO: Discuss the fact that this is essentially Ubuntu Gnome, without snaps, and some cool extensions

## Conclusion
#-- TODO: Discuss the pros and cons

[Home]({% link index.md %})
