<br /> <p align="center"><a href="https://github.com/theXappy/UZ801-LTE-USB-MODEM" target="_blank"><img src="img/4g_lte.png" width="200"></a></p>

<p align="center"><img src="https://img.shields.io/packagist/l/laravel/framework" alt="License"></a>
</p>

## Intro
I bought this "4G LTE USB Dongle WiFi" from AliExpress around 2023.  
After playing with it for a while, I decied to document all the quirks/tricks I've learned.  
I'm not the first one to research this thing, so some of the link below are to other GH repos by other, smarter people.  
SoC seems to be: `Qualcomm Snapdragon 410 (MSM8916)`  
USB Hardware IDs: `VID_05C6 PID_90B6`

## Table of Contents
1. [Blackbox Web App UI + API](Web_Interface.md)
2. Whitebox Web Server via ADB exploration ( TODO )
3. AT Commands via Modem Device ( TODO )
4. [Sniffing 3G/4G Traffic via QCSuper](https://github.com/P1sec/QCSuper) (by P1sec).
   - Just run the `./qcsuper.py --adb --wireshark-live` afte enabling ADB (which enables DIAG as well, I think)
5. [Screenshots + Disabling Screen Timeout](https://github.com/u0d7i/uz801) (by u0d7i)
   - Don't use the screenshots manually, use the next link (adbcontrol) for 2-way interactions.
   - The device is running Android KitKat (4.4.4, SDK 19) so neither scrcpy nor Vysor work.
7. [View & Control Device "Display" via adbcontrol](https://github.com/AlienWolfX/UZ801-USB_MODEM?tab=readme-ov-file#view-device-display) (by AlienWolfX)
8. [Change UI Language to English](https://www.youtube.com/watch?v=8krFZxOXuiE)
   - I tried u0d7i's way and it didn't work for me. Using the Settings app did.
10. [SuperSU, EDL, FW Dump/Restore, Installing OpenWRT/Debian](https://github.com/AlienWolfX/UZ801-USB_MODEM?tab=readme-ov-file#firmware-dump-and-restore) (by AlienWolfX)
11. [Another EDL Guide, Lots of hardware/software documentations](https://github.com/u0d7i/uz801) (by u0d7i)
