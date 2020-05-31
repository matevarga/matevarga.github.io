---
layout: post
title:  "Accessing ESP32-based devices under WSL2"
date:   2020-05-31 15:09:13 +0200
categories: esp32 m5stack esp-idf wsl2
---

### Use case
* you are using Windows + WSL2 on your desktop
* you want to develop and build your ESP32 (esp-idf) projects under WSL2

### Problem
* WSL2 does not have access to USB devices

### Solution

You'll need to forward USB serial port connection to WSL using TCP through the rfc2217 protocol. There are many ways to forward the raw socket through TCP (e.g. using `socat`), but flashing does not seem to work through those due to lack of port control functionality. rfc2217 allows port control through telnet.

### Steps:

* Connect your ESP32 device and find it in Windows Device Manager. It'll be located somewhere under `Ports (COM & LPT)`, and depending on the exact model you'll find a `USB Serial Port (COMX)` or `Silicon Labs CP210x USB to UART Bridge (COMX)`. You need to know the port number (e.g. `COM5`).
* Download [hub4com](https://sourceforge.net/projects/com0com/files/hub4com/2.1.0.0/hub4com-2.1.0.0-386.zip/download). Extract the contents (still under Windows).
* Navigate to the extracted files in a shell, and run `com2tcp-rfc2217.bat COM<number> <port number>`, e.g. `com2tcp-rfc2217.bat COM5 5555`. You should see something similar to

```
COM5 Open("COM5", baud=19200, data=8, parity=no, stop=1, octs=off, odsr=off, ox=off, ix=off, idsr=off, ito=0) - OK
Route data TCP(1) --> COM5(0)
Route data COM5(0) --> TCP(1)
Route flow control TCP(1) --> COM5(0)
Route flow control COM5(0) --> TCP(1)
Filters:
_______
       \->{telnet.IN}-------------------------------------------->
TCP(1) |     /
_______/<-----{telnet.OUT}<-{lsrmap.OUT}<-{pinmap.OUT}<-{lc.OUT}<-

________
        \->{parse.IN}------------------------------>
COM5(0) |     /
________/<-----{purge.OUT}<-{pinmap.OUT}<-{lc.OUT}<-

Started COM5(0)
Socket(0.0.0.0:5555) = 100
Listen(100) - OK
Started TCP(1)
```

* Under Linux, you can now monitor and flash your device with 

`idf.py flash -b <desired baud rate> -p rfc2217://<ip address of your Windows host>:<port number>`

e.g. 

`idf.py flash -b 1500000 -p rfc2217://192.168.1.120:5555`