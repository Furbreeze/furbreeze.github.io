---
layout: post
title:  "Rooting the Sunbeam F1"
---

I am **not** an android guy. Before this I knew nothing about rooting android phones and I learned a lot along the way. I plan on putting out another blog post about the process to rooting a formerly undocumented phone, including all of the massive timewastes such as getting stuck without the adb popup and being forced to write my adb keys to the system image to allow adb. Watch out for it if you're interested.

## Steps assuming debian linux

### Unlock Bootloader and write adb keys to phone
* Grab and install [MTKClient](https://github.com/bkerler/mtkclient)
* Run MTKClient
* While holding Vol Down + Vol Up, plug the usb into your phone from computer
* Use MTKClient to unlock bootloader

### Secret commands to turn on dev mode + adb
* To open secret code menu dial `###2633###`
* To turn on dev mode type the following into the popup and press ok: `###3383567376633###`
* Reopen secret code menu by dialing `###2633###`
* To turn on adb type the following into the popup and press ok: `###864453232463###`
* Reboot the phone
* Open Home page and plug into usb
* Run `adb devices` and accept the popup on screen
* Run `adb shell` to get an adb shell on the phone

### Finishing up
From here you can install magisk and use it to get a root shell like normal. Have fun :)