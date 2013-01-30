---
layout: post
title: "Creating a Debian rescue usb drive in Mac OS X"
date: 2013-01-29 09:53
comments: true
categories: 
---

This guide will get you a USB stick running debian that you can use to boot into rescue mode. Well, you could also use it to install from, but that's not what I care about.

1. Do the following, while modifying the version string to whatever the latest release is in the second url (both places!):
        cd ~
        wget ftp://ftp.debian.org/debian/dists/stable/main/installer-amd64/current/images/hd-media/boot.img.gz
        wget http://cdimage.debian.org/debian-cd/6.0.6/amd64/iso-cd/debian-6.0.6-amd64-businesscard.iso

2. Procure a USB stick, at least 1GB in size

3. Plug it in. Run the following to see what device name the stick was assigned.:

        $ ls -l /dev/disk*
        brw-r----- 1 root operator 1, 0 Jan 19 11:10 /dev/disk0
        brw-r----- 1 root operator 1, 1 Jan 19 11:10 /dev/disk0s1
        brw-r----- 1 root operator 1, 3 Jan 19 11:10 /dev/disk0s2
        brw-r----- 1 root operator 1, 2 Jan 19 11:10 /dev/disk0s3
        brw-r----- 1 root operator 1, 4 Jan 19 11:10 /dev/disk1
        brw-r----- 1 root operator 1, 0 Jan 30 12:15 /dev/disk3
        brw-r----- 1 root operator 1, 1 Jan 30 12:15 /dev/disk3s1

4. Find the device that has a creation time of "just now". Obviously, which device you use may be different than mine. Run the following:

        diskutil unmountVolume /dev/disk3

5. Now you copy the boot.img onto the drive. *Please note* that I am using rdisk not disk in the device name, but the number is the same. Also note that you can destroy your entire computer if you put the wrong rdisk device name here, so uh... don't do that:

        gunzip boot.img.gz
        sudo dd if=boot.img of=/dev/rdisk3

6. Remove your USB stick, and then plug it right back in.

7. Copy the iso file onto the drive. Just drop it right in the root of the drive. No, it doesn't matter if you use the finder or the cp command. Sheesh.

8. You're done! Plug that little guy in, and select "advanced options" on the first menu, then "rescue mode". Follow the prompts, blah blah. Do whatever you need to.
