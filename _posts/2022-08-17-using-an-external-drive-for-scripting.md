---
title: Use an External Drive for Saving Scripts
Author: Brandon J. Kessler
tags: Software DevOps Productivity
categories: Technology Opinion
published: true
---

<h1>{{ page.title }}</h1>

For a while I've been using a second drive in my personal laptop (it supports 2 NVMe drives) to save any scripts, ISOs, Learning resources, etc. It's handy because if I need to re-image my Windows `C:` drive I still have all those resources there. Recently my main workstation at work ended up under a pipe that leaked heavily, ruining a lot of equipment. Luckily I tend to upload and commit most of my stuff to Github or the work repo, so no major data was lost, _but_ it took a lot longer to setup my laptop and pull down all the things I'd been working on.

<!--more-->

I picked up an [NVMe to USB 10GB/s external adapter](https://www.newegg.com/p/0VN-0003-001R5?Item=9SIA1DSBWX4083&Description=nvme%20to%20usb&cm_re=nvme_to%20usb-_-9SIA1DSBWX4083-_-Product&cm_sp=SP-_-1249774-_-0-_-2-_-9SIA1DSBWX4083-_-nvme%20to%20usb-_-nvme|to|usb-_-21) and dropped a 512GB NVMe drive into it. The access time is fast enough that it feels native to me, and it's a very small aluminum enclosure with decent heat dissapation. I added some extra thermal pads to help move heat even better. After I got it all put together I attached it to my laptop and formatted the drive to NTFS, and then Bitlocked it. That is something I feel is vital to this process is making sure that you encrypt the drive whether it's internal or external. You _especially_ need to encrypt it if it's external.

From here it's pretty simple. You make sure that you create a folder structure that makes sense for you. I have the following folders in mine: Dev, ISO, Scripts, WIM, WIMWitch. This covers 99% of my daily work, and I'm still comitting things to their repositories, but now if my laptop dies, I can get a few things setup on a new laptop and start working that much faster. I've already used it in this fashion to take a test script and directly copy it over to my test laptop. I didn't have to fiddle with network sharing, no need to commit yet because it's untested, etc. I highly recommend adding a second drive to your device, and even if it's a laptop USB-C speeds with an NVMe are so fast I doubt you'll realize it's an external drive.