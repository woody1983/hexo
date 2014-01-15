title: Mount USB-disk on Raspberry-Pi
date: 2014-01-10 15:30:10
tags: [Linux, RaspberryPi]
---

``` bash

pi@raspberrypi ~/github/hexo $ ls -lah /dev/sd*
brw-rw---T 1 root floppy 8, 0 Jan  9 03:28 /dev/sda
brw-rw---T 1 root floppy 8, 1 Jan  9 03:28 /dev/sda1

cd /home/pi
mkdir usbdisk
sudo mount -o rw /dev/sda1 /home/pi/usbdisk/
```
