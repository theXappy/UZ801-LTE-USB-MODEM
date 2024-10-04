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
### 🌐 Web App & ADB
1. Blackbox Web App UI + AP [[Link](Web_Interface.md)]
2. Whitebox Web Server via ADB exploration [[Link](Web_Server_ADB_exploration.md)]
3. Replacing the Web Server (or any APK) but abusing test-keys [[Part 1](https://www.blinkenlights.ch/ccms/posts/aliexpress-lte-1/)] [[Part 2](https://www.blinkenlights.ch/ccms/posts/aliexpress-lte-2/)] (by adrian-bl)
   - Their device doesn't look the same (+ I had root on by default) by mine also deployed test-keys so the same logic applies.

### ☎️ Modem
1. Modem drivers for Windows + AT Commands communication [[Link](Modem_AT_Commands.md)]
2. Sniffing 3G/4G Traffic via QCSuper & Wireshark [[Link](https://github.com/P1sec/QCSuper)] (by P1sec).
   - Just run the `./qcsuper.py --adb --wireshark-live` afte enabling ADB (which enables DIAG as well, I think)

### 📱Screen Control
1. Screenshots + Disabling Screen Timeout [[Link](https://github.com/u0d7i/uz801)] (by u0d7i)
   - Don't use the screenshots manually, use the next link (adbcontrol) for 2-way interactions.
   - The device is running Android KitKat (4.4.4, SDK 19) so neither scrcpy nor Vysor work.
2. View & Control Device "Display" via adbcontrol [[Link](https://github.com/AlienWolfX/UZ801-USB_MODEM?tab=readme-ov-file#view-device-display)] (by AlienWolfX)
3. Change UI Language to English [[Link](https://www.youtube.com/watch?v=8krFZxOXuiE)]
   - I tried u0d7i's way via the shell and it didn't work for me. Using the Settings app via adbcontrol did.

### 🪄 Firmware Backup/Flashing
1. SuperSU, EDL, FW Dump/Restore, Installing OpenWRT/Debian [[Link](https://github.com/AlienWolfX/UZ801-USB_MODEM?tab=readme-ov-file#firmware-dump-and-restore)] (by AlienWolfX)
2. Another EDL Guide, Lots of hardware/software documentations [[Link](https://github.com/u0d7i/uz801)] (by u0d7i)

