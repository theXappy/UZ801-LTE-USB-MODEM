# Web App
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

## Web Pentesting
What I notices is that many pages in the web app fetch information by sending requests to `http://192.168.100.1/ajax`.
A typical request contains a json like this:
```
{"funcNo":1022,"mode":"1"}: 
```
(Not sure what the last `:` are doing there, you can drop them and it still works)

I started enumerating the commands + visiting some pages and sniffing which commands they organically send.
(2nd one was important since some commands have arguments (like "mode" in the example above) that the dongle will refuse to answer to if they're missing.
Here's my current understanding of all commands:
<FFS>
