---
layout: post
title: "How I Lost a Month's Work and Data"
date: 2013-10-04 00:27
comments: true
categories: 
---

While this story is still fresh in my mind, I figured I'll share it. Little under a week ago, I made the mistake of
installing Linux on my primary computer, without thinking about what I'm doing, and more importantly-- without making 
any backups. Somehow the Linux bootloader, GRUB, killed my MBR bootloader for Windows (and with it, all my NTFS partitions),
but not before killing itself too. Not only did I lose my Windows OS and all my data, I lost Linux too; I had *no OS*.


### It started out as a typical Sunday evening

The time was about midnight and I was finishing my lecture notes and my 
homework for the night. Since I was done with my assignments for the weekend, I figured it would be a good time to install
that Linux distro that I've always wanted for software development. Since I was doing more project-based classes this semester,
it would be nice to have a Terminal at hand rather than having to SSH into the school lab machines every time I want to run
some code. I wasn't ready to give up Windows just yet-- I still need it for video games and audio/photo editing tasks, so dual
booting would be a good compromised.

So I headed over to the **[Linux Mint](http://www.linuxmint.com/)** website to grab the latest copy of my favorite Linux distro, and "burned"
it on to an USB flash drive. Now on my desktop, I have one large SSD installed with three partitions: one for Windows and 
programs, one for games, and one for my data. Since my game and data partitions were nearly full, I decided to shrink my 
primary partition (the one with Windows) and create a new partition for Mint. I also created a small 100MB partition to install
bootloader information (which Mint complained about not having one). The installation went smoothly and within a few minutes
I was on the Mint desktop. 

Mint complained that it was running in "software rendering mode", and I thought duh, no video card drivers installed. So I headed
off to NVIDIA's site to download the latest Linux drivers. I wasn't able to install the drivers; the installer complained that
**[I was running an X server](http://askubuntu.com/questions/149206/how-to-install-nvidia-run)** and it must be closed before proceeding. 
After some research I learned how to TTY into my own installation with `ctl+alt+F1` and tried to kill the X server, 
which didn't work so I just ran it with the `--no-x-check` flag which worked. I rebooted for good measure.

One thing I immediately noticed is that upon boot up, I didn't get any OS selection menu from the bootloader; it just booted directly
into Linux Mint. I just ignored that for now. Upon booting into the Mint desktop, I faced the 
**[Cinnamon just crashed. You are currently running in Fallback Mode](http://unix.stackexchange.com/questions/86336/cinnamon-crashed-after-nvidia-drivers-installation)**
error message. After spending a good half-hour trying to fix this problem by reinstalling various NVIDIA drivers, I just gave up; 
it was getting late at night and I wanted finish some things off before calling it a day. I wanted to switch back to Windows, except,
I couldn't. I forgot that there wasn't any bootloader menu for OS selection, great. After some research, I discovered the 
**[Boot-Repair](https://help.ubuntu.com/community/Boot-Repair#Using_Boot-Repair)** tool that's supposed to automatically fix any GRUB
issues:
```
sudo add-apt-repository ppa:yannubuntu/boot-repair && sudo apt-get update
sudo apt-get install -y boot-repair && (boot-repair &)
```
I ran the "Recommended repair" option and upon reboot, GRUB 2 OS selection was available! I booted into Windows and I was a happy
camper.


### I wish I had stopped right here. After this is when things went *sour*

Since I was in a good mood after fixing the bootloader issue, I thought Mint was beyond repair, and probably reinstalling
the distro would be better at this point. So I did, opting to "Replace the old Mint installation" during the install process instead
of messing with the partitions again. After booting into the desktop, I installed the NVIDIA drivers (properly this time) by formally
killing the X server `sudo service mdm stop`. Upon reboot I once again noticed that the GRUB OS selector was gone, but more 
importantly, the horrocious "Cinnamon just crashed" error was back. I wasn't going to try to mess with the drivers any 
more and decided to just fix GRUB before calling it a day. This time running the Boot-Repair tool, the Windows option didn't come back.
I tried to add the Windows partition into GRUB manually:
```
$ sudo update-grub
Generating grub.cfg ...
Found linux image: /boot/vmlinuz-2.6.35-22-generic
Found initrd image: /boot/initrd.img-2.6.35-22-generic
Found memtest86+ image: /boot/memtest86+/memtest.bin
  No volume groups found
done

$ fdisk -l /dev/sda
Disk /dev/sda: 80.0 GB, 80026361856 bytes
255 heads, 63 sectors/track, 9729 cylinders, total 156301488 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x000c33e1

   Device Boot      Start         End      Blocks   Id  System
/dev/sda1            0         145407       71680   83  Linux
/dev/sda2   *      145408    25319423    12587008   83  Linux
/dev/sda5        25319424    29511679     2096128   82  Linux swap / Solaris
```

Updating GRUB didn't work, and fdisk only showed the Linux partitions! Somehow my Windows NTFS partitions dissapeared. I then figured Windows 
would be able to fix its own problems so I let Windows fix the MBR by running a Windows rescue disk. No luck; it couldn't find any boot errors.  
So I figured it was a problem  with a corrupt MBR and I ran Boot-Repair again, but this time with the manual option to "Restore MBR" 
as well as "Reinstall GRUB". Still no go. In fact, it got worse: GRUB wouldn't load at all!
```
Error 1: filename must either be an absolute pathname or blocklist 
Press any key to continue... 

grub>_ 
```

At this point I was getting impatient and desperate. I didn't care about fixing the boot issues; I just wanted to back up my data and start
freshly new. If neither Mint, GRUB, nor Windows can see my NTFS partitions, then the only thing left is the all-almighty **[GParted Live CD](http://gparted.sourceforge.net/)**. 
It has never failed me in the past, being able to find the most hidden partitions with ease. After firing
it up, and doing a quick scan, I noticed once again only three partitions: boot, Mint and swap. As a final resort, I ran a deep scan overnight
to search for hidden/lost partitions. No luck; no new partitions found. At this point, I called it a total loss. 

***
What probably happened was that when I installed Mint for the second time, the installer wrote its bootloader over the MBR partition instead 
of using the partition I specifically made for Mint in the first installation, thinking that it was the only OS installed on the disk.
In any case, I wiped the entire disk, created brand new partitions, and installed Windows. Fortunately, I had a backup from end of August, so 
not all was lost. But everything I've worked on since August (and isn't on GitHub) was pretty much gone. 

**Valuable lessons learned**:
	1. Think before doing (especially installing new OS's).
	2. Always back up data before working with partitions.
	3. Don't be messing with OS's late at night.
