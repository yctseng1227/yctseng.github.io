---
title: 從零開始的 Raspberry Pi 3...玩具？
date: 2020-06-31 11:52:10
categories: 
- 
tag:
- 
---


<!--more-->

[[ Raspberry Pi ] 透過 MacOS 安裝 Raspbian](https://oranwind.org/-raspberry-pi-tou-guo-macos-an-zhuang-raspbian/)

diskutil list
diskutil eraseDisk FAT32 RPI /dev/disk3
diskutil unmountDisk /dev/disk3
sudo dd bs=1m if=2020-05-27-raspios-buster-lite-armhf.img of=/dev/rdisk3
diskutil unmountDisk /dev/disk3

(new a ssh empty file)


[Raspberry Pi 的基礎 - 系統設定的調教](http://blog.itist.tw/2014/05/raspberry-pi-setup.html)
sudo apt-get install vim
sudo raspi-config #edit some setting
sudo vim /etc/apt/sources.list
#deb http://free.nchc.org.tw/raspbian/raspbian/ testing main contrib non-free rpi

sudo reboot


sudo apt update
sudo apt full-upgrade

[使用 Raspberry Pi 架設自己的 AP](https://dotblogs.com.tw/sideprogrammer/2019/02/17/raspberry-pi-set-ap)