# Jerry [Windows] Easy

[https://app.hackthebox.com/tracks/Beginner-Track](https://app.hackthebox.com/tracks/Beginner-Track)

— 1 . nmap —

`nmap -sS -sV $IP`

On trouve un site héberger avec Tomcat Apache sur le port 8080

— 2 . gobuster —

`gobuster -dir -u http://$IP:8080 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt`

on trouve 

/docs                 (Status: 302) [Size: 0] [--> /docs/]
/examples             (Status: 302) [Size: 0] [--> /examples/]
/manager              (Status: 302) [Size: 0] [--> /manager/]

— 3 . Connexion en tant que admin du serveur Tomcat —

IP:8080/manager 

on test les logs par défaut `tomcat:s3cret`

— 4. MetaSploit —

Pour y accéder plus rapidement on va utiliser le script metaSploit `multi/http/tomcat_mgr_upload`

On exploit et il n’y a plus qu’a recherché le Flag car nous avons directement un shell root.