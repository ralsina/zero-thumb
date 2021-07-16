# Zero-Thumb

What is this? It's a short explanation on how to turn a Raspberry Pi Zero into
a Internet-enabled thumb drive.

## What is an Internet-enabled thumb drive?

Imagine I gave you a thumb drive, and told you:

> Every time you want me to send you something, ask for it, and then
> plug this thumb drive into your computer.
> After a while, the files will be there.

Or imagine if I told you:

> Plug this USB drive into a TV. Once a week a new video will appear in it.

Neat, uh?

SanDisk sells something like it, but this is nicer because it's 

* Cheap
* Homemade
* I am giving you a free computer instead of a stupid thumb drive

This is heavily based on [this article](https://magpi.raspberrypi.org/articles/pi-zero-w-smart-usb-flash-drive) by Russell Barnes

## Basic idea

Spoiler: it's not a thumb drive, it's a Raspberry Pi Zero W.

Spoiler: but it's a thumb drive. I am using one of [these](https://www.banggood.com/USB-Dongle-With-Acrylic-Shield-for-Raspberry-Pi-Zero-or-Zero-W-p-1432397.html?gmcCountry=US&cur_warehouse=CN&createTmp=1&utm_source=google&utm_medium=cpc_ods&utm_campaign=nancy-content-sdsrm-jewelry-nancy-content&utm_content=nancy&gclid=Cj0KCQjwub-HBhCyARIsAPctr7xhj-ZatxXm-5wK3RPS8mBrAbdUVOqarSjvxbvF1Jjt_-fH2SXAO_oaAn1mEALw_wcB) but you can just use a plain old micro USB cable. There are versions of these "dongle converters" for a dollar or two if you look around.

* It's a Raspberry Pi Zero W configured as a USB mass storage device
* It will appear to be a read-only thumb drive of whatever size you want when you
  plug it into a computer
* It uses [SyncThing](https://syncthing.net/) to keep that "thumb drive" in sync
  with a folder in another computer.
* Put stuff in that computer's special folder ... it pops up (eventually) in the
  computer where the "thumb drive" is plugged

## Setup

No, I am not going to give every detail

1. Install Raspberry Pi OS into a micro SD card
2. Put it in your Raspberry Pi Zero
3. Finish configuring it so:
   a. It has WiFi
   b. It has ssh access
4. Create an empty file of the size you want the "thumb drive" to be (2048 is 2GB):

```
sudo dd bs=1M if=/dev/zero of=/piusb.bin count=2048
```
5. Format the file as vfat:
```
sudo mkdosfs /piusb.bin -F 32 -I
```
6. Mount it as ~pi/Sync by adding this in `/etc/fstab`:
```
/piusb.bin /home/pi/Sync vfat users,umask=000 0 2
```
7. Add at the bottom of `/etc/modules`
```
dwc2
```
8. Add at the bottom of `/boot/config.txt`
```
dtoverlay=dwc2
```
9. Reboot
10. Install Syncthing in the raspberry
11. Install Syncthing on your own computer
12. Sync ~/Sync between the two
13. In the raspberry, make syncthing start on boot for the pi user:
```
systemctl enable syncthing@pi
```
14. Install python3-watchdog
15. Copy `usb_share.py` from this repo as `/usr/local/bin/usb_share.py`
16. Copy `usbshare.service` from this repo as `/usr/lib/systemd/system/usbshare.service`
17. Enable usbshare:
```
systemctl enable usbshare
```
18. reboot

That's it.

The `usbshare.py` script disconnects and reconnects the computer from the "thumb drive" when new files appear, otherwise it will not see them. Should be seamless in Windows and OSX, not
so much in Linux (works fine if you open the device in a file manager without mounting it)

Now, as long as the Zero is in a network it can connect to, you can just send it files 
by dropping them in your `~/Sync` and any computer it's plugged into will see them
(eventually).

## Notes

I have had *very* bad performance with large files, probably because the version of
Syncthing in Raspberry OS is ancient. See if you can update it.

You can see the status of sync between devices using your computer's syncthing UI:

![image](https://user-images.githubusercontent.com/1579/125876367-ff127982-3267-4f84-ba3e-0253298b93c7.png)
