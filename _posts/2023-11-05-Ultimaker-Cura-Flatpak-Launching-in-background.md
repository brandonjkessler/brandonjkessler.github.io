---
title: Ultimaker Cura 5.5 Flatpak Launching in the background on ElementaryOS
Author: Brandon J. Kessler
tags: Linux ElementaryOS "3D Printing" Cura
categories: Linux
published: true
---

<h1>{{ page.title }}</h1>
If you're having an issue with Ultimaker Cura 5.5 Flatpak launching in the background and you're unable to bring it to the foreground, I found that rolling it back to Cura 5.4 fixed the issue. To do so you'll need to follow the directions I found at [StackExchange](https://unix.stackexchange.com/questions/552688/is-it-possible-to-roll-back-a-flatpak-update).
<!--more-->
1. First you need to run `flatpak remote-info --log flathub com.ultimaker.cura`. This pulls up information that we'll need later.
![](/assets/screenshots/2023-11-05_Screenshot_Terminal_CuraFlatpak.png)
2. You'll want to look for the commit `ac4bdf52bd8ed4d1da279e4025693e41e89dec16441b2821b792cb0452ef37b2`
![](/assets/screenshots/2023-11-05_Screenshot_Terminal_Ultimaker.png)
3. Run `flatpak update --commit=ac4bdf52bd8ed4d1da279e4025693e41e89dec16441b2821b792cb0452ef37b2 com.ultimaker.cura` to rollback to Cura 5.4.0
4. Once Complete, run `flatpak mask com.ultimaker.cura` to stop it from updating automagically
    - NOTE: You'll need to run `flatpak mask --remove com.ultimaker.cura` to re-enable updating


That's it! Good luck!