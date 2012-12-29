---
layout: post
title: Arch Linux on BTRFS
---

During the holidays, at the end of the year, I usually spend some time cleaning up my system and experimenting with new technologies. This often boils down to starting from scratch with a complete reinstallation of my laptop.

Last year I switched to [Arch Linux](http://www.archlinux.org) (which turned out to be one of the best Linux distribution I have ever used); this year I got interested in advanced file systems like [ZFS](http://en.wikipedia.org/wiki/ZFS) (and its [Linux incarnation](http://zfsonlinux.org/)).

So I opted for reinstalling Arch Linux using one of those filesystems.

After several unsuccessful attempts to install Arch Linux on ZFS (there is a [nice article](https://wiki.archlinux.org/index.php/Installing_Arch_Linux_on_ZFS) on the Arch Linux wiki) I managed to finally boot it, using ZFS as the root filesystem.

However, since the ZFS kernel modules are not (yet) part of the standard Linux kernel, it was quite painful to finish such an installation... I was experimenting in a [QEmu](http://wiki.qemu.org/Main_Page) virtual machine, and re-doing the same steps on my actual system would have been even more painful.

Finally, Kernel upgrades would have been too tricky because the corresponding ZFS modules must be correctly recompiled at each upgrade in order to make the system boot... Too risky.

So I turned to another *advanced* filesystem that is gaining momentum, has several features comparable to ZFS and is already included in the standard Kernel: [BTRFS](https://btrfs.wiki.kernel.org/index.php/Main_Page)

After several installation attempts, I finally had a fine-tuned Arch Linux installation with the root filesystem completely on BTRFS.

In the following sections I will describe how I did it and some tricks I implemented in order to have a nice way of keeping track of system upgrades and make it possible to easily roll back to the previous system state in the case of a broken upgrade.

# BTRFS

The BTRFS home page describes it in this way:

> *"a new copy on write (CoW) filesystem for Linux aimed at implementing advanced features while focusing on fault tolerance, repair and easy administration."*

You can have a look at a detailed list of all its features on the [BTRFS home page](https://btrfs.wiki.kernel.org/index.php/Main_Page). However what captured my interest was the following:

* The possibility of creating root filesystems (subvolumes) on the fly.

* The possibility of creating (COW) snapshots.

* The possibility of manipulating snapshots in a very easy way.

Just to give you a little more details about the previous items, here it is a quick overview of some of the BTRFS concepts.

A BTRFS-formatted partition contains by default a single subvolume where users can store files. A subvolume can be thought as an independent filesystem that is attached to some parent (another subvolume).

The interesting thing is that subvolumes can be mounted as root filesystems. When this is the case, users can only "see" what is inside the subvolume and not what is contained in the parent subvolume.

Only by mounting the toplevel subvolume you can have access to *all* the data stored in a BTRFS filesystem.

Subvolumes appear as directories in the filesystem hierarchy. You can distinguish between them and "vanilla" directories by using the `btrfs subvolume list` command.

Another interesting thing about subvolumes is that they can be *snapshotted*. A snapshot is an exact copy of a subvolume that has an independent life. A snapshot uses a copy-on-write policy which makes snapshots quite space-efficient. The initial snapshot, in fact, doesn't take any additional space. Only what is modified is then copied and starts to occupy actual disk space.

Snapshots can be writable and are actually subvolumes that can be used as such.

Snapshots and subvolumes can be renamed and moved using the standard `mv` command. They can also be deleted using the `rm -rf` command (like if they were standard directories), but a more efficient way to do it is by using the `btrfs subvolume delete` command.

When mounting a subvolume, all the child-subvolumes are recursively mounted as well. However snapshotting is not recursive. So when you take a snapshot of a subvolume `X` that has a child `Y`, only `X` is snapshotted.

BTRFS also has a ton of advanced features such as adding other devices to the filesystem, handling RAID*n* setup, resizing volumes, etc. Many of them can be done on-the-fly which means that no downtime/reboot is needed.

# The setup

My goal was to have the whole SSD allocated to BTRFS, and having a setup that allowed me to take *snapshots* of the relevant part of the system.

This would allow to do the following operations:

* Rolling back to the previous state of the system

* Being able to understand what has changed (useful also for security reasons)

The problem was to identify what were the the *relevant* parts of the system. I ended up with considering relevant everything except `/home`, `/opt` and `/var` which are basically the three subdirectories that you usually mount on separate filesystems.

With this setup, by snapshotting `/` I would end up with a subvolume containing everything that was needed for booting a complete system with all its functionalities and their configurations.

Since I have 8GB of memory I didn't create a swap partition, and even if I would have needed it I could have used a swap file instead. If you want to create also a swap partition you should create it as a separate partition (and not as a BTRFS subvolume).

Also, I didn't create a separate `/boot` partition because, otherwise, I would not have had the kernel images in a BTRFS subvolume and, hence, in the snapshots. Rolling back to a previous system state would have been more difficult.

# Installation

The following is the annotated step-by-step transcription of what I did to install Arch Linux on BTRFS.

The parameters written there may not completely match your system (e.g., `/dev/sda`), so change them to as you see fit.

### 0. Download and boot the installation medium

We need the ISO image of the Arch Linux installation medium. You can find it [here](https://wiki.archlinux.org/index.php/Beginners'_Guide#Burn_or_write_the_latest_installation_medium). The version I used is *2012.12.01*.

Follow the instructions in the [Beginners' guide](https://wiki.archlinux.org/index.php/Beginners'_Guide) up to section 2.2 in order to have a suitable environment.

In my case, I just set up the WiFi by doing

{% highlight bash %}
$ ip link set wlan0 up
$ wifi-menu wlan0
{% endhighlight %}

### 1. Partition and format the disk

{% highlight bash %}
$ fdisk /dev/sda
{% endhighlight %}

Create a partition as big as the whole disk and save. For performance reasons, make sure that it is correctly aligned if you have a 4Kb-sector disk. `fdisk` should give you default values that take into account partition alignment.

I used `fdisk` to setup a plain-old [MBR](http://en.wikipedia.org/wiki/Master_boot_record) because my system (a Dell XPS L502x) doesn't seem to support booting from a  [GPT (UEFI standard)](http://en.wikipedia.org/wiki/GUID_Partition_Table). I tried, but I had an *"Operation system not found"* error when I rebooted (yes, *operation* was actually mispelled in the error message :))

If your computer supports booting from a GPT you might consider to [partition the disk](https://wiki.archlinux.org/index.php/Beginners'_Guide#Prepare_the_storage_drive) using it.

However, be careful... UEFI, GTP, booting, supporting multiple operating systems are quite tricky to handle so make sure to read the documentation before proceeding.

Then we can format the partition using BTRFS:

{% highlight bash %}
$ mkfs.btrf -L "Arch Linux" /dev/sda1
{% endhighlight %}

And finally mount the partition:

{% highlight bash %}
$ mkdir /mnt/btrfs-root
$ mount -o defaults,relatime,discard,ssd /dev/sda1 /mnt/btrfs-root
{% endhighlight %}

The mount options are used to speed-up BTRFS:

* `relatime` updates access timestamps only if if the previous access time was earlier than the current one. `noatime` could also be used to disable access timestamps update but this could break some programs that rely on that.

* `discard` and `ssd` are optimizations for SSD drives that make BTRFS to send [discard/TRIM](http://en.wikipedia.org/wiki/TRIM) commands to the underlying block device when blocks are freed.

### 2. Create subvolumes

In this step we are going to create subvolumes for our filesystems. We will create 4 subvolumes:

* `ROOT` which will be mounted on `/`
* `home` which will contain user data
* `opt` which will contain optional programs and data
* `var` which will contain runtime information such as logs and spool files.

This is a [fairly standard way](https://wiki.archlinux.org/index.php/Security#Partitions) to partition the system.

We will mount `home`, `opt`, `var` in the corresponding directories of the `ROOT` subvolume.

But there's a twist: in fact, the `var` subvolume will then contain the `lib` directory where `pacman`, the Arch Linux package manager, store information about the installed packages.  In order to have a complete *snapshot* of the system we must include this directory. So we have to make sure that the `var/lib` directory (and only this) *is* on the `ROOT` subvolume. We will achieve this by binding the `var/lib` directory on the `ROOT` subvolume to the one on the `var` subvolume:

{% highlight bash %}
mkdir -p /mnt/btrfs/__snapshot
mkdir -p /mnt/btrfs/__current
btrfs subvolume create /mnt/btrfs-root/__current/ROOT
btrfs subvolume create /mnt/btrfs-root/__current/home
btrfs subvolume create /mnt/btrfs-root/__current/opt
btrfs subvolume create /mnt/btrfs-root/__current/var
{% endhighlight %}

The `__snapshot` and `__current` directories are created in the top-level subvolume of the BTRFS partition, and are used to distinguish between the subvolumes that are snapshots and those that are currently used as active subvolumes.

You can see the newly created subvolumes with the following command:

{% highlight bash %}
$ btrfs subvolume list -p /mnt/btrfs-root/
ID 256 gen 5 parent 5 top level 5 path __current/ROOT
ID 259 gen 5 parent 5 top level 5 path __current/home
ID 260 gen 5 parent 5 top level 5 path __current/opt
ID 261 gen 5 parent 5 top level 5 path __current/var
{% endhighlight %}

### 3. Mount subvolumes

In this step we will mount the `__current/ROOT`, in a given location so that we can install the base system on it. We will also mount `__current/home`, `__current/opt`, and `__current/var` on the corresponding directories in the `ROOT` subvolume.

First let's create a directory where to mount `__current/ROOT`:

{% highlight bash %}
$ mkdir -p /mnt/btrfs-current
{% endhighlight %}

Then we mount `__current/ROOT` and create the mount points for mounting the other subvolumes:

{% highlight bash %}
$ mount -o defaults,relatime,discard,ssd,nodev,subvol=__current/ROOT /dev/sda1 /mnt/btrfs-current
$ mkdir -p /mnt/btrfs-current/home
$ mkdir -p /mnt/btrfs-current/opt
$ mkdir -p /mnt/btrfs-current/var/lib
{% endhighlight %}

Finally we mount the other subvolumes on the corresponding mount points:

{% highlight bash %}
$ mount -o defaults,relatime,discard,ssd,nodev,nosuid,subvol=__current/home /dev/sda1 /mnt/btrfs-current/home
$ mount -o defaults,relatime,discard,ssd,nodev,nosuid,subvol=__current/opt /dev/sda1 /mnt/btrfs-current/opt
$ mount -o defaults,relatime,discard,ssd,nodev,nosuid,noexec,subvol=__current/var /dev/sda1 /mnt/btrfs-current/var
{% endhighlight %}

At this point we have all the filesystem mounted. However the `/var/lib` directory resides on the `__current/var` subvolume, while we would like to use the `var/lib` directory on `__current/ROOT`.

In order to do so, we do the following:

{% highlight bash %}
$ mkdir -p /mnt/btrfs-current/var/lib
$ mount --bind /mnt/btrfs-root/__current/ROOT/var/lib /mnt/btrfs-current/var/lib
{% endhighlight %}

Now, the `/var/lib` on the `__current/var` subvolume will be bound to the `/var/lib` in the `__current/ROOT` subvolume, and whatever is written there will end up in the right location on the `__current/ROOT` subvolume.

### 4. Install the base system

After having chosen the mirror to be used, we can bootstrap the base system:

{% highlight bash %}
$ nano /etc/pacman.d/mirrorlist 
$ pacstrap /mnt/btrfs-current base base-devel
{% endhighlight %}

Before continuing we need to generate the `/etc/fstab` for the installed system, based on the previously created subvolumes:

{% highlight bash %}
$ genfstab -U -p /mnt/btrfs-current >> /mnt/btrfs-current/etc/fstab
{% endhighlight %}

At this point we have an initial `/etc/fstab` but we have to edit it because the bound `/var/lib` is not recognized correctly by `genfstab`. We should end-up with the following:

{% highlight bash %}
tmpfs						/tmp		tmpfs		rw,nodev,nosuid 				0 0
tmpfs						/dev/shm	tmpfs		rw,nodev,nosuid,noexec 				0 0

# /dev/sda1 LABEL=Arch Linux
UUID=...	/         	btrfs     	rw,nodev,relatime,ssd,discard,space_cache,subvol=__current/ROOT			0 0
UUID=...	/home     	btrfs     	rw,nodev,nosuid,relatime,ssd,discard,space_cache,subvol=__current/home		0 0
UUID=...	/opt      	btrfs     	rw,nodev,nosuid,relatime,ssd,discard,space_cache,subvol=__current/opt		0 0
UUID=...	/var      	btrfs     	rw,nodev,nosuid,noexec,relatime,ssd,discard,space_cache,subvol=__current/var	0 0
UUID=...	/run/btrfs-root	btrfs     	rw,nodev,nosuid,noexec,relatime,ssd,discard,space_cache				0 0
/run/btrfs-root/__current/ROOT/var/lib		/var/lib	none		bind 						0 0
{% endhighlight %}


* `__current/ROOT` is mounted on `/`, this is specified by using the `subvol` option.

* `__current/home` is mounted on `/home`

* `__current/opt` is mounted on `/opt`

* `__current/var` is mounted on `/var`

* The whole BTRFS filesystem is mounted on `/run/btrfs-root` (no `subvol` option will mount the whole filesystem). We need this because we have to bind the `/var/lib` directory to the one on the `ROOT` subvolume, and we need a way to access to this subvolume.

* The `/var/lib` on the `__current/ROOT` subvolume  (accessible via the `/run/btrfs-root/__current/ROOT/var/lib` directory) will be bound to the `/var/lib` directory.

The options specified for all the BTRFS subvolumes contain the parameters we mentioned before. We also add `nodev`, `nosuid` and `noexec` in order to further secure our installation, and we do the same for the temporary filesystems. This practice is described with more details [here](https://wiki.archlinux.org/index.php/Security#Relevant_mount_options)

### 5. Configure the system

At this point we have a base system installed. The steps to be performed here are basically those described in the [Beginners' Guide](https://wiki.archlinux.org/index.php/Beginners'_Guide#Chroot_and_configure_the_base_system) starting from section 2.8.

You need to start by chrooting to `/mnt/btrfs-current` and not to `/mnt` as written in the guide, since the filesystem where to install Arch Linux (i.e., the `__current/ROOT` subvolume) is mounted there.

When you generate the initial ramdisk environment, make sure to edit the `/etc/mkinitcpio.conf` and to do the following:

* Remove `fsck` from the `HOOKS` line. BTRFS doesn't have a `fsck` program, and leaving it in the `HOOKS` will only generate a warning.

* Add `btrfs` to the `HOOKS` line.

Then do, as usual:

{% highlight bash %}
$ mkinitcpio -p linux
{% endhighlight %}

### 6. Installing the bootloader

In order to boot the system I used the GRUB bootloader. It detects the `/boot` directory on a BTRFS subvolume and correctly boots the system. I didn't test *Syslinux*. Probably with this setup it would work as well.

You might want to edit the `/etc/default/grub` file before generating the `grub.cfg` file using `grub-mkconfig`:

{% highlight bash %}
pacman -S grub-bios
grub-install --target=i386-pc --recheck /dev/sda
grub-mkconfig -o /boot/grub/grub.cfg
{% endhighlight %}

### 7. Unmount and reboot

At this point we have a ready-to-boot Arch Linux installation. We will first exit from the chroot environment, unmount all the filesystem and then reboot into our newly installed Arch Linux.

{% highlight bash %}
$ exit

$ umount /mnt/btrfs-current/home
$ umount /mnt/btrfs-current/opt
$ umount /mnt/btrfs-current/var/lib
$ umount /mnt/btrfs-current/var
$ umount /mnt/btrfs-current
$ umount /mnt/btrfs-root

$reboot
{% endhighlight %}

# Snapshotting

Once the system is rebooted we will have `__current/ROOT` mounted as `/` and the whole BTRFS filesystem accessible from `/run/btrfs-root`.

We can take a snapshot of this initial state and keep it for reference. We could use it, for example, for restoring configuration files or checking what has changed in subsequent upgrades.

In order to take a snapshot we will first create a `SNAPSHOT` file in the subvolume root directory containing the current time and a comment. This file will become part of the snapshot and will help us to identify what this snapshot refers to. We will remove it from `/` after the snapshot has been created:

{% highlight bash %}
echo `date "+%Y%m%d-%H%M%S"` > /run/btrfs-root/__current/ROOT/SNAPSHOT
echo "Fresh install" >> /run/btrfs-root/__current/ROOT/SNAPSHOT

btrfs subvolume snapshot -r /run/btrfs-root/__current/ROOT
                            /run/btrfs-root/__snapshot/ROOT@`head -n 1 /run/btrfs-root/__current/ROOT/SNAPSHOT`

rm /run/btrfs-root/__current/ROOT/SNAPSHOT
{% endhighlight %}

The `-r` parameter will take a read-only snapshot, otherwise the snapshot will be writable as the original `__current/ROOT` subvolume.

We can see that the created snapshot is actually listed as a subvolume:

{% highlight bash %}
$ btrfs subvolume list -p /run/btrfs-root/
ID 256 gen 1372 parent 5 top level 5 path __current/ROOT
ID 259 gen 1373 parent 5 top level 5 path __current/home
ID 260 gen 7 parent 5 top level 5 path __current/opt
ID 261 gen 1373 parent 5 top level 5 path __current/var
ID 263 gen 68 parent 5 top level 5 path __snapshot/ROOT@20121227-163413
{% endhighlight %}

### Rolling back to a previous system state

Let's suppose that we started to install some packages and that at some point we did something wrong and we want to start over. We can do that by rebooting the system after changing the `__current/ROOT` subvolume.
To do so, we will rename the `__current/ROOT` subvolume, and take a writable snapshot of the fresh-install snapshot we took earlier.

{% highlight bash %}
$ mv /run/btrfs-root/__current/ROOT /run/btrfs-root/__current/ROOT.old
$ btrfs subvolume snapshot /run/btrf-root/__snapshot/ROOT@20121227-163413 /run/btrfs-root/__current/ROOT
$ reboot
{% endhighlight %}

When the system reboots, it will be like if it was just installed. Of course the `/home`, `/opt` and `/var` (except `/var/lib`) might contain additional stuff because they were not part of the snapshot. But the system will be in its pristine state because all the relevant part and configurations were snapshotted.

Now, if we want, we can also remove the `__current/ROOT.old` subvolume.

### Upgrading the system

A snapshot of `__current/ROOT` could be taken just before every system upgrade. If the upgrade succeeds, then we can remove it otherwise we can roll back to the previous system state and retry the upgrade, maybe after having fixed what made the upgrade fail.

This process could be also automated with a simple script which basically does something similar to what we described in the previous section.

### Using snapshots for improving system security

Snapshots can be very useful to understand what changed in a system and whether our system has been compromised. For example, if we have a "trusted" snapshot, like the one taken just after the system has been installed, we can compare the current state of the system against that *previous* state and check if binaries, configuration files or libraries have changed when they shouldn't.

<div style="text-align: center; width: 100%">
  <div style="margin: 0px auto 0px"> 
    <div style="float: left; width: 410px; padding: 5px">
	<a href="/images/Arch_Linux_on_BTRFS/compare1.png"><img style="display: inline" src="/images/Arch_Linux_on_BTRFS/compare1_small.png"/></a>
	<div style="width: 100%; text-align: center; font-size: 0.7em; line-height: 1.1em">Comparing directories</div>
    </div>
    <div style="float: left; width: 410px; padding: 5px">
	<a href="/images/Arch_Linux_on_BTRFS/compare2.png"><img style="display: inline" src="/images/Arch_Linux_on_BTRFS/compare2_small.png"/></a>
	<div style="width: 100%; text-align: center; font-size: 0.7em; line-height: 1.1em">Comparing files</div>
    </div>
    <br style="clear: both"/>
  </div>
</div>

In the previous pictures (click on them for a larger version), we used a diff tool like [Meld](https://www.archlinux.org/packages/extra/any/meld/) to compare directories and files. We can see that many things changed in `/etc/`, including `groups`.

This could be quite alarming because usually this file should not be touched. Looking at the actual changes we see that groups for `gdm`, `avahi`, etc. were added: that's OK if we have installed GNOME. So everything is fine.

This diff-check can also be automated because we have all the metadata associated with the installed packages. So a script could check which packages have been installed, which files have been affected and report only the files that weren't touched by any package installation but that still have changes. In this case, spotting anomalies would be easier.

### User created subvolumes

With BTRFS users can also create subvolumes (though only `root` can delete them). This is very useful if we want to take advantage of the snapshotting mechanism for user data.

Usually revision control systems like [Git](http://git-scm.com/) are more suitable for doing this kind of tracking, but if we have to manage a lot of binary data BTRFS snapshotting could be a valid alternative.

Think for example to video editing: a user might want to track the state of her work, so she can want to put all the files related to a given project in a separate subvolume and take regular snapshots as her work progresses.

Snapshotting can also be useful when dealing with virtual machines. Online cloud services like Amazon EC2 already has this kind of feature for incrementally building virtual machine images by taking subsequent snapshots of it. With BTRFS a user can apply the same principle by putting the virtual machine image in a separate subvolume and incrementally install the system by taking intermediate snapshots.

# Conclusion

BTRFS is a very nice filesystem and as far as I see it performs quite well. It is not yet marked as completely stable, so you might want to do frequent backups of your data if you decide to use it. There is also a lot of criticism about its implementation, but it is evolving fast and it is going to be adopted as the main filesystem in many mainstream distributions. So many shortcomings should be addressed and solved.

For your information, [this](/data/arch_linux_on_btrfs_installation.txt) is the complete step-by-step sequence I used to setup my system. It contains also some extra stuff about power management, firewall and TCP/IP configuration. It's mostly a *"note to self"*, but you can use it if you want to install Arch Linux on your own.
