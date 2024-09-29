# ADB Exploration
When first connecting the modem to your PC, you'll likely only see the "Remote NDIS" (RNDIS) device show up.  
The RNDIS device is mostly noticable as a new network interface, but you can also see it in the Device Manager.  

To get access to the ADB interface, we need to enter "debug mode".  
The easiest way to do it is to send the following POST request to `http://192.168.100.1/ajax`:
```json
{"funcNo":2001}
```

curl command:
```cmd
C:\>curl -X POST http://192.168.100.1/ajax -H "Content-Type: application/json" -d "{\"funcNo\":2001}"
curl: (56) Recv failure: Connection was reset
```
The "Connection was reset" is an expected response since the device re-starts after changing modes.  

Afterwards you'll see a total of 4 different devices in the Device Manger owned by the LTE stick:
![device_manager_in_debug_mode_no_drivers](https://github.com/user-attachments/assets/3064098a-fd6d-4f97-8e56-961041f9e88f)

Drivers are missing for `MI_02` and `MI_03` but we'll deal with it in another guide.  
We're mostly interested in the `ADB Interface` one.  
If you see it, you should also see a new device when running `adb devices -l`:
```
C:\>adb devices -l
List of devices attached
0123456789ABCDEF       device product:msm8916_32_512 model:UZ801 device:msm8916_32_512 transport_id:17
```

From here, you should be able to `adb shell` into the device. The default user is `root` (uid=0) so I didn't encounter any permissions problems when exploring.  

## Finding the Web Server Pages
I was curious about possible "hidden" pages in the web interface/hidden AJAX commands so I went looking for the webserver's dir.  
From the URLs of the pages I could navigate, it seems like I'm looking for a directory with `main.html`, `status.html`, `wifiSettings.html` etc.  
The title of the web pages also say "4G MiFi" so once I `ls`'d the `/data/data` dir I quickly noticed something suspicious:
```
root@msm8916_32_512:/data/data # ls -l
... many lines ...
drwxr-x--x u0_a23   u0_a23            1970-01-02 02:00 com.gd.mobicore.pa
drwxr-x--x radio    radio             1970-01-02 02:00 com.iwprivservice.hello
drwxr-x--x system   system            1970-01-02 02:01 com.mifiservice.hello <--------- Says "MiFi"!
drwxr-x--x u0_a11   u0_a11            1970-01-02 02:00 com.qti.cbwidget
drwxr-x--x system   system            1970-01-02 02:00 com.qualcomm.agent
drwxr-x--x system   system            1970-01-02 02:00 com.qualcomm.agpstestmode
drwxr-x--x system   system            1970-01-02 02:00 com.qualcomm.atfwd
... more lines ...
```
A few directories in and I found exactly what I expected:
```
root@msm8916_32_512:/data/data/com.mifiservice.hello/files/jetty2 # ls -l
-rw------- system   system       2935 2024-09-28 19:33 D_HCP.html
-rw------- system   system       3456 2024-09-28 19:33 MacFilter.html
-rw------- system   system    3084265 2024-09-28 19:33 USER_Manual_en.pdf
-rw------- system   system    7833495 2024-09-28 19:33 USER_Manual_zh.pdf
-rw------- system   system       1259 2024-09-28 19:33 chat.html
drwx------ system   system            1970-01-02 02:01 css
-rw------- system   system       2013 2024-09-28 19:33 deviceInformation.html
-rw------- system   system       2412 2024-09-28 19:33 deviceOperation.html
-rw------- system   system       3480 2024-09-28 19:33 help.html
drwx------ system   system            1970-01-02 02:01 img
-rw------- system   system       4789 2024-09-28 19:33 index.html
drwx------ system   system            1970-01-02 02:01 js
drwx------ system   system            1970-01-02 02:01 json
-rw------- system   system       8345 2024-09-28 19:33 main.html
-rw------- system   system       2099 2024-09-28 19:33 modifyPwd.html
-rw------- system   system       2388 2024-09-28 19:33 ms.html
-rw------- system   system       2124 2024-09-28 19:33 netsel.html
-rw------- system   system       1268 2024-09-28 19:33 reboot-bootloader.html
-rw------- system   system       2703 2024-09-28 19:33 sim.html
-rw------- system   system        244 2024-09-28 19:33 static.html
-rw------- system   system       4165 2024-09-28 19:33 status.html
-rw------- system   system       1241 2024-09-28 19:33 usbdebug.html
-rw------- system   system       2995 2024-09-28 19:33 wifiSecurity.html
-rw------- system   system       5238 2024-09-28 19:33 wifiSetting.html
-rw------- system   system       7091 2024-09-28 19:33 wwanConfig.html
```
I believe at least `ms.html`, `usbdebug.html` and `chat.html` were not accessible in any way via the regular UI.  
Both `ms.html`, `usbdebug.html` are related to switching into debug mode (a bit late to discover this now, but as reassurance it's nice):
* `usbdebug.html` just loads `usbdebug.js` which invokes `funcNo=2001`.
* `ms.html` has a tiny form with 1 radio button + normal button mentioning a switch to "Debug Mode(DIAG+AT+MODEM)". When clicked `ms.js` invokes `funcNo=1022`.

This was nice but I still didn't see where the `funcNo` ajax requests are handled.
I did have a direction to look at: the package ID of the web server seems to be `com.mifiservice.hello`.

## Finding the Web Server's App
Knowing the package ID, I invoked `pm` to find it's APK path:
```
root@msm8916_32_512:/ # pm list packages -f | grep mifi
package:/system/priv-app/MifiService.apk=com.mifiservice.hello
```
Then a simple `adb pull /system/priv-app/MifiService.apk` and we're ready to decompile.  
The apk opens nicely in JADX and no tricks/obfuscations in sight. 
![{37CD89B5-D495-49FF-912C-3212142D3DF3}](https://github.com/user-attachments/assets/eaf9704e-eb33-47ee-82fb-651da0e2df85)

Finding the AJAX commands handler was quite simple. Searching for any of the non-trivial `funcNo`s (meaning, no round numbers like `1000`) would take you directly there:
![{A49F442A-3B69-43BC-95C4-03DDB2D81833}](https://github.com/user-attachments/assets/133d73ca-d582-4bd3-a803-019db55996f9)
Eventaully this function helped compose the complete `funcNo` list found in [Web_Interface.md](Web_Interface.md#ajax-enumeration-api-pentesting)

