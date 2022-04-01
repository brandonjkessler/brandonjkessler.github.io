---
title: Why I Converted to Unraid
Author: Brandon J. Kessler
tags: Homelab Self-Host SysAdmin
category: Homelab
published: true
---

<h1>{{ page.title }}</h1>

![Unraid Logo](/assets/img/unraid-logo.png)

## What is UnRAID

[UnRAID](https://unraid.net/) is a GNU/Linux-based server solution. UnRAID is based on [Slackware Linux](http://www.slackware.com/) as of this writing and implements data redundancy and parity in a slick way. It’s similar to using MergerFS and SnapRAID, but with a slight difference. It also supports Docker and KVM through a nice and easy-to-use web-ui. Oh, and it runs off of a flash drive and RAM.
<!--more-->
## RAID: What is RAID

At it’s very basics RAID ([Redundant Array of Inexpensive Disks or Redundant Array of Independent Disks](https://en.wikipedia.org/wiki/RAID)) is a method in which data can be stored on multiple storage devices. Generally speaking the most common RAID setups are RAID 0, RAID 1, and RAID 5 and RAID 10.

### RAID 0

RAID 0 requires at least two drives and it “stripes” data across those two drives. That means that the data is split and written or read from those two drives. It also takes the combined total of the drives and displays them as a single drive, i.e. two 1TB SSDs would apear as a single 2TB SSD to the operating system. It also has increased speed benefits as the data is being written and read by both drives. The drawback is that if Drive 0 or Drive 1 fails, all data is lost. RAID 0 is best if you want the highest throuput possible and the data is not important.

### RAID 1

RAID 1 “mirrors” the data on both drives. So whatever is written to Drive 0 is also written to Drive 1, so if Drive 0 fails Drive 1 can continue to operate with no loss of data. There are also write operation penalties for RAID 1 since the data must be written twice, however there are speed improvements for read operations because now the data can be read from two drives. The other drawback is that you lose half of your storage capacity, i.e. two 1TB SSDs will only present as a single 1TB SSD. RAID 1 is best if you need maximum protection and redundancy.

### RAID 5 and 6

RAID 5 uses “parity” to protect data. [Parity](https://en.wikipedia.org/wiki/Parity_bit#:~:text=In%20mathematics%2C%20parity%20refers%20to%20the%20evenness%20or,determined%20by%20the%20value%20of%20all%20the%20bits.) is when data is written to another drive or sections of drives to calculate when data is missing. RAID 5 dividies the parity data up across all drives in the array in small sections. RAID 5 requires a minimum of three drives and can suffer a single drive failure before data loss. If more than a single drive fails in a RAID 5 array then all data will be lost. RAID 5 allows for higher capacities and some drive failure protection, but there is higher overhead with RAID 5 over RAID 1 or RAID 0 because of the need to write write parity data. RAID 6 is just like RAID 5, but allows up to two drive failures. The other drawback to RAID 5 is that the total usable space of the array is the total amount of storage minus one drive, i.e. three 1TB SSDs would appear as a 2TB SSD.

### RAID 10 or RAID 1+0

[RAID 10](https://en.wikipedia.org/wiki/Nested_RAID_levels#RAID_10) (sometimes called RAID 1+0) is when an array stripes across a mirrored set. For example, you could have four 1TB SSDs. You would then mirror Drive 0 and Drive 1 as a single set and then Drive 2 and Drive 3 as a single set. Now you have two 1TB mirrored arrays. Then you stripe the data across those two mirrored arrays and achieving a 2TB SSD. The array can now suffer a single drive failure from each mirrored set. RAID 10 gives the greatest throughput and performance next to RAID 0 while also maintaining data redundancy.

### RAID Flexibility

UnRAID does not use any of these RAID levels, but instead uses an approach that allows for a lot more flexibility. UnRAID allows up to two Parity drives in an array. All Parity data is written to the parity drive(s) instead of splitting the parity data up across multiple drives. Even better, the array can be a mix of multiple drive sizes as long as they are as big or smaller than the parity drive(s). In all the RAID levels mentioned above the disks should be matching, but if they aren’t they are going to be limited to the size of the smallest drive. So if you had a 500GB drive and 1TB drive in a RAID 1 array, the usable space will be 500GB. If you had a 500GB drive, a 1TB drive, and a 2TB drive in a RAID 5 array, your usable space would be 1TB, essentially three 500GB drives in RAID 5. With UnRAID in the same configuration, your Parity drive would be the 2TB drive, and then your 1TB and 500GB drives would be in the array, giving you 1.5TB of storage. But, if you replaced the 500GB drive with another 2TB drive your usable storage would be 3TB of storage.

The other big drawback of the RAID setups above is that to “grow” the array usually involves backing up the current array, upgrading all the drives at once to a larger size, and building a new array and copying data back. With UnRAID you can grow the array just by replacing a disk and allowing the parity drive to rebuild the array. So for example, I had a Terramaster F4-220 NAS running [Open Media Vault](https://www.openmediavault.org/)(OMV) with four 3TB drives. I think OMV is a fantastic set of software and is great to create a home NAS out of. It’s performant and easy to use. However I ran in to the limitation of wanting to grow the array soon without building from scratch. Sadly there was no good way to do it without rebuilding anyway, so I chose to try UnRAID.

I’ve since backed up the data and installed UnRAID on the same machine and had no issues growing the array. I started off by building my UnRAID array using four 500GB HDDs with a single drive for Parity. Then, after transferring the most important data from my backup to the array I started upgrading the parity 500GB drive to one of the 3TB drives from my original RAID 5 array from OMV. After the parity re-synced I upgraded a 500GB drive to a 3TB drive and had 4TB of usable storage. I waited for Parity to resync to that drive and then transferred more data. I repeated this process for the other two drives and had my full 9TBs of usable storage.

You can also use a cache drive for faster performance. Cache drives aren’t part of the array and are not synced to parity. You can, however, create a cache pool of two or more drives using BTRFS, so there is some data protection there. It will also copy data over to the Array if you set it up to do so. The Cache pool is nice to use because it is more performant, so I tend to put my Docker containers and VMs on it.

### VM and Docker

The other benefit of UnRAID is that it has Docker and VM support built in. There are plenty of other distributions and solutions that also provide this support. I was using Proxmox on an older computer running a little Core i3-2120 and 8GB of RAM. It was enough to run my Plex Media Server, PiHole, and a few Minecraft servers on. They were all running as LXC linux containers so the overhead was quite small. I started migrating some of the LXC containers to the UnRAID server and using the [Community Apps](https://unraid.net/community/apps) plugin. After I was able to migrate everything over the Terramaster F4-220 just didn’t have the “oomph” that we needed for the family, so I frankensteined the Proxmox computer and the drives from the NAS. I had to get a new case to hold all the drives and I got a used motherboard that had enough SATA connections for all of them. Since UnRAID is not tied to the hardware but instead is tied to the Flash Drive it’s installed on, all I had to do is move the flash drive and all my config came over. Getting a VM or Docker setup is so easy in UnRAID, especially compared to Proxmox or rolling your own.

### Cost

UnRAID is not free nor open source. It costs as little as $60 for the “basic” which allows you to add 6 drives total. There’s another tier that allows 12 drives for $90, and for $130 you have unlimited drives. The best apps and docker containers come from the community. You can create all of that from scratch, but part of the appeal of UnRAID is ease-of-use.

## Conclusion

So I chose UnRAID because it has helped me simplify my self-hosted applications and server. I can grow the array easily as I need more storage without having to rebuild the whole thing. I can slowly upgrade the computer with little or no fuss. I’ve since bought another license for the Terramaster F4-220 and will be using it as a backup server. But I can start with the small 500GB drives and start upgrading them again with bigger and bigger drives. I think the ease of growing the array and the plethora of usable plugins makes it worth the $60 per server.