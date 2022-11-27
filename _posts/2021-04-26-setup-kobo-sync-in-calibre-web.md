---
title: Setup Kobo Sync in Calibre-Web
Author: Brandon J. Kessler
tags: Self-Host Kobo eBook Software
categories: Technology
published: true
---

<h1>{{ page.title }}</h1>

## Calibre-Web

_This guide assumes you’ve already setup [Calibre-Web](https://github.com/janeczku/calibre-web) on a server of your choice._
<!--more-->
-   Navigate to the “Admin” section of your server.

![](/assets/screenshots/msedge_sIXDsR3doT-1.png)

-   Scroll down to the “Configuration” section

![](/assets/screenshots/msedge_xAuLuJIrrQ.png)

-   Select the “Edit Basic Configuration” button

![](/assets/screenshots/msedge_OLiI9HIxZg.png)

-   Scroll down to “Feature Configuration” and select the “+” icon to expand

![](/assets/screenshots/msedge_Ui96An4ll2.png)

![](/assets/screenshots/msedge_CDB8Q9UgTQ.png)

-   Select the checkbox next to “Enable Kobo Sync”

![](/assets/screenshots/msedge_tmUARuEXsZ.png)

-   Now Select your user name in the upper-right corner

![](/assets/screenshots/msedge_97dKor2r0E.png)

-   Scroll down until you see the “Kobo Sync Toke” section

![](/assets/screenshots/msedge_uckSaeoz4Z.png)

-   Select the “Create/View” button to create a token

![](/assets/screenshots/msedge_AsBFUGZpQX.png)

-   Copy the `api_endpoint=http://IPADDRESS:PORT/kobo/TOKEN` line

![](/assets/screenshots/msedge_C1AS8CPFUy.png)

## On the Kobo

-   Connect the Kobo to your computer and tap the “Connect” button on the Kobo

-   In File Explorer Navigate to the Kobo mounted as storage, i.e. `D:KOBOReader`

![](/assets/screenshots/explorer_5LiLIGlUrH-1.png)

-   Locate the folder “.kobo” and enter it.

![](/assets/screenshots/explorer_PUrK2BQkxi.png)

-   Locate the folder “Kobo” in that folder and enter it.

![](/assets/screenshots/explorer_OGwfWgZibE.png)

-   Locate the “Kobo eReader.conf” file and edit it with either Notepad, or any other plaintext editor. I prefer Visual Studio Code.

![](/assets/screenshots/explorer_5WAga41Nlt.png)

![](/assets/screenshots/explorer_V9ViypzsOU.png)

![](/assets/screenshots/explorer_CNN7cjwN6m.png)

![](/assets/screenshots/explorer_aZsQg4KKnw.png)

-   Locate the line in the file `api_endpoint=https://storeapi.kobo.com` and replace it with the line you copied earlier

![](/assets/screenshots/Code_aeldyUYhLW.png)

![](/assets/screenshots/Code_7KMjf5aPvu.png)

-   Save the file and safely eject your Kobo

-   Sync your Kobo after ejecting it and it should now pull all your books from your Calibre-Web instance.

## Caveat

Note that this only works on your local network. If you want it to work everywhere you’d need to expose your Calibre-Web instance to the internet, and most likely require a Dynamic DNS provider or a static Public IP address. This is the absolute most basic configuration.