---
title: Kali Proxmark Quick Setup
date: 2019-08-23
draft: false
description: How to quickly get up and running with your proxmark on kali linux
---
This massive one-liner should work to install all the source.

`sudo apt install -y p7zip git build-essential libreadline5 libreadline-dev libusb-0.1-4 libusb-dev libqt4-dev perl pkg-config wget libncurses5-dev gcc-arm-none-eabi libpcsclite-dev && git clone https://github.com/Proxmark/proxmark3.git && cd proxmark3 && make clean && make all`

Now install the bootloader.

`./client/flasher /dev/ttyACM0 -b bootrom/obj/bootrom.elf`

Now install the full image. I encountered issues flashing the full image wherein my Proxmark3 RD4 had both A & C lights lit red, and it would not reconnect. Try this command to flash the bootloader, and if you have the same issue I did follow the next steps.

`./client/flasher /dev/ttyACM0 armsrc/obj/fullimage.elf`

If you have issues like I did, stop the Modem Manager service, unplug the device, hold the white button, plug it back in, and reflash the full image. Once flashing has completed, you can release the button.

`systemctl stop ModemManager.service`

Finally, connect to your Proxmark 🙂

`sudo ./client/proxmark3 /dev/ttyACM0`

Credit to [Alex Dib](https://web.archive.org/web/20220520135811/https://scund00r.com/all/rfid/2018/06/05/proxmark-cheatsheet.html)