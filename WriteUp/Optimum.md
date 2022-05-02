nmap
```
Starting Nmap 7.91 ( https://nmap.org ) at 2022-05-02 15:05 JST
Stats: 0:00:29 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 6.42% done; ETC: 15:12 (0:06:34 remaining)
Nmap scan report for 10.10.10.8
Host is up (0.29s latency).
Not shown: 19999 filtered ports
PORT   STATE SERVICE VERSION
80/tcp open  http    HttpFileServer httpd 2.3
|_http-server-header: HFS 2.3
|_http-title: HFS /
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 273.83 seconds
```

searchsploit
```
┌──(root💀DESKTOP-IUAFKPR)-[~]
└─# searchsploit HttpFileServer                                                                                                                              
--------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                             |  Path
--------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Rejetto HttpFileServer 2.3.x - Remote Command Execution (3)                                                                | windows/webapps/49125.py
--------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results

```



