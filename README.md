![image](https://github.com/user-attachments/assets/5dd36e2c-2999-4bda-86d7-e5496d736819)# Web App
First thing the manual tells you is to configure this stick using the web app.  
Once you insert the device to your (Windows) PC You'll see a new "RNDIS" network inerface:  
![{A93E26E2-7547-44E7-A486-CE7AEB96CE89}](https://github.com/user-attachments/assets/a52ea109-4e89-468e-85a3-aa4b71642219)  
Your PC will be assigned an IP in the `192.168.100.0/24` range and gateway set to `192.168.100.1`.

Browsing to `http://192.168.100.1/` will let you access to the dongle's web interface.  
![image](https://github.com/user-attachments/assets/97484a6e-c8c2-4e53-8850-b55d1c06e018)  
Default login: `admin`/`admin`  
  
![image](https://github.com/user-attachments/assets/9a070fc1-8726-48a1-8477-c1d900ede00c)

You'll likely need to do 2 things for the dongle to connect to the network:
1. Configure APN: Advanced -> APN Settings 
2. Click "Connect" in the Home page
In my personal experience, neither my PC (dongle connected via USB) nor my mobile (connected to the dongle's WiFi) could connect to the internet afterward.
But the dongle itself connected and I know that because later I ran `curl` directly on it.

None of the other menus are interesting and won't give you any clues of escalating to the inner stuff of the dongle.

## AJAX Enumeration (API Pentesting)
What I notices is that many pages in the web app fetch information by sending requests to `http://192.168.100.1/ajax`.
A typical request contains a json like this:
```json
{"funcNo":1022,"mode":"1"}
```

Below is a table with all implemented `funcNo` values and their meanings.  
I documented some by simply using the Web Interface and sniffing with Chrome.  
Others I just sent with Postman & blackbox'd the response.
The rest I figured by finding the handler in the web server APK* which I fetched from the device.   
(* had to figure out ADB first, which I randomally activated with command `2001`).

|funcNo|Description|Arguments         |Example Req|Example Resp| Related URL |
|------|-----------|------------------|-----------|------------|-------------|
|0-999 | Don't exist| -               | -         | -          | - |
| 1000 | Login     | "username", "password"                  |```{"funcNo":1000,"username":"admin","password":"admin"}```|```{"results":[{"net_mode":0,"fwversion":"UZ801-V3.4.3","conn_mode":"1","imei":"8666800660#####"}],"error_info":"none","flag":"1"}```| - |
| 1001 | Status     | -                                      |```{"funcNo":1001}```|```{"results":[{"oper":"","netstatus":"Disconnected","netmode":"UNKNOWN","rssi":0}],"error_info":"none","flag":"1"}```| - |
| 1002 | Get WiFi/DHCP Info | -                                   |```{"funcNo":1002}```|```{"results":[{"dns1":"8.8.8.8","ssid":"4G-UFI-1DE","dns2":"8.8.8.8","mask":"255.255.255.0","pwd":"1234567890","wlan_ip":"192.168.100.1","IP":"192.168.100.1"}],"error_info":"none","flag":"1"}```| - |
| 1003 | Get Clients Info | -                                |```{"funcNo":1003}```|```{"results":[{"up_bytes":0,"maxSta":10,"client_num":0,"down_bytes":0}],"error_info":"none","flag":"1"}```| - |
| 1004 | Connect/Disconnect | "conn_mode" (1=Connect)        |```{"funcNo":1004,"conn_mode":"1"}```|```{"error_info":"none","flag":"1"}```| - |
| 1005 | Set RAN?  | -                                       |```{"funcNo":1005}```| - |
| 1006 | Get WiFi Info | -                                   |```{"funcNo":1006}```|```{"results":[{"maxSta":10,"client_num":0,"ssid":"4G-UFI-1DE","ssid_flag":"1","mac":"a0:23:5b:3c:a1:de","wifi_status":"1","channel":6,"ip":"192.168.100.1","mode":"n"}],"error_info":"none","flag":"1"}```| - |
| 1007 | Set WiFi SSID/Max Stations | "ssid", "maxSta"       |```{"funcNo":1007,"ssid":"4G-UFI-1DE","maxSta":"10"}: ```|```{"error_info":"none","flag":"1"}```| - |
| 1008 | Restart AP| -                                       |```{"funcNo":1008}```|```{"error_info":"none","flag":"1"}```| - |
| 1009 | Get WiFi Enc Info | -                               |```{"funcNo":1009}```|```{"results":[{"pwd":"1234567890","encryp_type":4}],"error_info":"none","flag":"1"}```| - |
| 1010 | Set WiFi Enc Info | -                               |```{"funcNo":1010,"encryp_type":"4","pwd":"1234567890"}: ```|```{"error_info":"none","flag":"1"}```| - |
| 1011 | Get DHCP Info | -                                   |```{"funcNo":1011}```|```{"results":[{"device_arr":[],"dns1":"8.8.8.8","dns2":"8.8.8.8","range_low":"192.168.100.100","range_high":"192.168.100.200","ip":"192.168.100.1"}],"error_info":"none","flag":"1"}```| - |
| 1012 | Set DHCP Info | "dns1", "dns2", more?               |```{"funcNo":1012,"dns1":"8.8.8.8","dns2":"8.8.8.8"}```|```{"error_info":"none","flag":"1"}```| - |
| 1013 | Reboot Device | -                                   |```{"funcNo":1013}```|```NOTHING! Device Reboots```| - |
| 1014 | Restore Device(?) | -                               |```{"funcNo":1014}```|```{"error_info":"none","flag":"1"}```| - |
| 1015 | Get SIM Info | -                                    |```{"funcNo":1015}```|```{"results":[{"sim_status":"Absent"}],"error_info":"none","flag":"1"}```| - |
| 1016 | Get APNs Info | -                                   |```{"funcNo":1016}```|```{"results":[{"info_arr":[],"profile_num":0}],"error_info":"none","flag":"1"}```| - |
| 1017 | Set APN Info | -                                    |```{"funcNo":1017,"no":"1","name":"a","apn":"b","user":"c","pwd":"d","auth":"0"}: ```|```{"error_info":"none","flag":"1"}```| - |
| 1018 | Execute APN | -                                     |```{"funcNo":1018,"profile_num":"1"}```|```{"error_info":"none","flag":"1"}```| - |
| 1019 | Not Implemented? | -                                | - | - | - |
| 1020 | Change Web Password | -                             |```{"funcNo":1020,"oldpwd":"admin","newpwd":"abc"}: ```|```{"error_info":"none","flag":"1"}```| - |
| 1021 | Get Debug Mode Info | -                             |```{"funcNo":1021}```|```{"results":[{"mode":"0"}],"error_info":"none","flag":"1"}```| - |
| 1022 | Set Debug Mode | "mode" (1=Debug, 0=Normal)         |```{"funcNo":1022,"mode":"1"}```|```{"error_info":"none","flag":"1"}``` | http://192.168.100.1/ms.html |
| 1023 | Get Traffic Counters | -                            |```{"funcNo":1023}```|```{"results":[{"up_bytes":0,"down_bytes":0}],"error_info":"none","flag":"1"}```| - |
| 1024 | Reboot Device | -                                   |```{"funcNo":1013}```|```NOTHING! Device Reboots```| - |
| 1025 | Get WiFi Enc Info (old?) | -                        | - | - | - |
| 1026 | Get DHCP Info (old?) | -                            | - | - | - |
| 1027 | Get WiFi Clients IP Info | -                        |```{"funcNo":1027}```|```{"range_low":"192.168.100.100","range_high":"192.168.100.200","flag":"1","results":[{}],"error_info":"none"}```| - |
| 1028 | Get WiFi Clients List | -                           |```{"funcNo":1028}```|```{"results":[{"device_arr":[]}],"error_info":"none","flag":"1"}```| - |
| 1029 | Get Device Information | -                          |```{"funcNo":1029}```|```{"results":[{"manufacture":"Qualcomm Technology","dbm":" -67 dBm","fwversion":"V3.4.3","imei":"866680066017304"}],"error_info":"none","flag":"1"}```| http://192.168.100.1/deviceInformation.html |
| 1030 | Get Language | -                                    |```{"funcNo":1030}```|```{"results":[{"Language":"en"}],"error_info":"none","flag":"1"}```| - |
| 1031 | Set Language | ?                                    | - | - | - |
| 1032-1051 | Don't Exist | -                                | - | - | - |
| 1052 | Something MAC Filtering ? | ?                       | - | - | - |
| 1053 | Something MAC Filtering ? | ?                       | - | - | - |
| 1054 | Something MAC Filtering ? | ?                       | - | - | - |
| 1055 | Commit MAC Filtering ? | ?                          | - | - | - |
| 1056-1999 | Don't Exist | -                                | - | - | - |
| 2000 | Reboot Bootloader | -                               |```{"funcNo":2000}```|```NOTHING! Device Reboots```| http://192.168.100.1/reboot-bootloader.html |
| 2001 | Enable ADB | -                                      |```{"funcNo":2001}```| ? | http://192.168.100.1/usbdebug.html |
| 2002 | Get SIM Slot ? | -                                  |```{"funcNo":2002}```|```{"results":[{"simslot":"1"}],"error_info":"none","flag":"1"}``` |
| 2003 | Set SIM Slot ? | -                                  |```{"funcNo":2003,"simslot":"1","password":"abc"}```|```{"error_info":"password error!","flag":"0"}``` | Password is hard-coded to `admin8888`, does nothing regardless.|
| 2004 | Change IMEI | -                                     |```{"funcNo":2004, "imei":"123456789012345"}```|```{"error_info":"none","flag":"1"}``` |

