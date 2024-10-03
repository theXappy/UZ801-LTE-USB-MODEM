# Setting up Modem Access
After enabling ADB on the device, we exepect to see 4 different 'devices' in Device Manager.  
Notable, 2 of them won't be recognized (at least on my Windows 11). They'll show up as "Other devices -> Android"

![device_manager_in_debug_mode_unk_devices](https://github.com/user-attachments/assets/1adbc8de-89f4-4ad4-9175-30a7123cdab4)

Those 2 devices are missing drivers to operate. Here are they're full identifiers:
```
USB\VID_05C6&PID_90B6&MI_02
USB\VID_05C6&PID_90B6&MI_03
```
Googling for those will bring up shady drivers websites.  
Lucky for us drivers are signed so if you find ones that fit the IDs you can be, to some extent, sure that the drivers are legit.  
I've added to this repo both drivers but since I'm not sure what the `MI_03` one does, so let's talk about the `MI_02` one:  
[qualcomm-hs-usb-modem-90b6-1826440.zip](drivers/qualcomm-hs-usb-modem-90b6-1826440.zip)  

After installing this driver your known device will turn into a functional "Modem" device:  
![{483B7A24-B4CC-4BF0-9557-C711D500250F}](https://github.com/user-attachments/assets/dc985b48-74c8-447f-b1ca-454634799db6)  

Double-click it to bring up the device info (took like 30 seconds on my machine, be patient).
Click the "Modem" tab to see which COM port was assigned:  
![image](https://github.com/user-attachments/assets/f503b0e1-1d37-4e41-80f8-2c2e4741700c)  

# AT Communication
Install PuTTY and connect to the port:  
![image](https://github.com/user-attachments/assets/a599585b-2316-48ff-9b63-b4d60760f8d5)  
On connection those commands/responses show up without doing anything. This is a good sign that you've reached the modem:  
![{00010D2F-C9E0-42B0-9D4A-0323840B85AC}](https://github.com/user-attachments/assets/8b996014-c321-4140-aa37-afbedc62b061)  

If you're not familiar with AT communication via PuTTY, you should know that the default configuration of both PuTTY and most modems is set to "no echo".  
This means that everything you write is sent to the modem but not written on the screen (unlike normal command line prompts).
You can fix this with 2 options:
1. Via the modem: Immediately send "ATE1" (AT command 'E' for ECHO with value 1, which means ON).
   - This makes the modem send back to the PC every character that you sent to it.
2. Via PuTTY's configuration: Re-open putty and go the "Terminal" menu. Enable both "Local echo" and "Local line editiong" to "Force On".
   - PuTTY will buffer the commands and show them to you. Only ENTER presses will actually send data to the modem.

Now try sending the `ATI` command, you should see both the command and the resoibse which contains several of the modem's Identifiers.  
![image](https://github.com/user-attachments/assets/b694a243-7555-409a-9814-a7bda1c8cb51)
