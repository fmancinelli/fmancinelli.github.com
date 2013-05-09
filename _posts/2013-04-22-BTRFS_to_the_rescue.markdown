---
layout: post
title: BTRFS to the rescue!
---

In a [previous post](http://blog.fabio.mancinelli.me/2012/12/28/Arch_Linux_on_BTRFS.html) I described how I installed Arch Linux on BTRFS and how I decided to use a particular layout that would have come handy in case of problems.

So far I haven't had any problem during my almost-daily upgrades (yes, Arch is very good to this respect), so the snapshot-before-upgrade technique described in that post didn't help too much.

But today a huge upgrade was available: it was a combo Kernel + Gnome 3.8 + LibreOffice. I ran my script for taking a snapshot of my ROOT partition and started a `pacman -Syu` as usual.

However, this time things went wrong. Packages installed correctly but at reboot my laptop froze. I forced a power-off by pressing the on/off button, and when it restarted got a Kernel Panic on... BTRFS! 

Things were pretty messed up. However with `btrfs utils` in the ram disk I was able to do a quick check. Data seemed to be there and a `btrfsck` was enough to make the Kernel Panic go away. However the laptop continued to freeze at boot, when `GDM` was started.

This was the perfect situation for reverting things back. I just needed to retrieve the name of the snapshot to restore. Since I name snapshots as `ROOT@YYYYMMDD-HHMMSS` I couldn't remember the exact name, and since I wasn't able to use my laptop I had to boot using an USB key.

Once retrieved the snapshot name, I just edited on the fly the Grub boot command and... Arch booting as usual, as if the upgrade didn't take place! 

Today I have been very glad of having spent some time trying to devise the setup that allowed me to operate in this way. There was only a missing thing... Having a snapshot with an easy-to-remember name. 

So I have improved the script I usually run before any upgrade. Now it takes two snapshots of the current state: one whose name has the timestamp, and one that is always called `LATEST`. In this way booting using the previous state would be straightforward.

If you are interested, here it is the version 2.0 of the script I mentioned before:

{% highlight bash %}
#!/bin/bash

DATE=`date "+%Y%m%d-%H%M%S"`
read -p "Comment: " COMMENT

echo "$DATE
$COMMENT" > /SNAPSHOT

btrfs subvolume snapshot -r /run/btrfs-root/__current/ROOT /run/btrfs-root/__snapshot/ROOT@$DATE
btrfs subvolume delete /run/btrfs-root/__snapshot/LATEST
btrfs subvolume snapshot -r /run/btrfs-root/__snapshot/ROOT@$DATE /run/btrfs-root/__snapshot/LATEST

rm /SNAPSHOT
{% endhighlight %}

If you think of changing distribution, or re-installing the current one, I highly recommend you to switch to BTRFS. It could really save you the day!
