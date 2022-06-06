# Impossible_password [Reverse Engineering] Challenge

Analyse du fichier :

On commence par analyser le fichier avec la commande `file` c’est un ELF Linux basique.

Quand on le lance dans le terminal nous avons 1 entré à mettre  :

![Untitled](Impossible_password%20%5BReverse%20Engineering%5D%20Challeng%205e02deb391354d87bd9e592d5f65963d/Untitled.png)

`ltrace ./impossible_password.bin` On peux rentrer ce qu’on veux en entré :

![Untitled](Impossible_password%20%5BReverse%20Engineering%5D%20Challeng%205e02deb391354d87bd9e592d5f65963d/Untitled%201.png)

ltrace nous montre qu’il y à une comparaison entre notre entrée et le string “SuperSeKretKey”.
On test de rentré le string en tant que mot de passe et ça marche.

Nouveau problème :
La première entré à fonctionné avec SuperSeKretKey le Nouveau problème est qu’il nous demande un deuxième mot de passe.

![Untitled](Impossible_password%20%5BReverse%20Engineering%5D%20Challeng%205e02deb391354d87bd9e592d5f65963d/Untitled%202.png)

En faisant le même procédé que le dernier mot de passe avec ltrace nous avons ceci en retours

![Untitled](Impossible_password%20%5BReverse%20Engineering%5D%20Challeng%205e02deb391354d87bd9e592d5f65963d/Untitled%203.png)

On nous valide toujours la première entré mais la deuxième est comparé à une valeurs aléatoire on sait que cette valeur est choisi aléatoirement à cause des fonction rand en retour.
Nous pouvons refaire le procédés plusieurs fois nous aurons un nouveau mot de passe différent.

Trouver le flag avec radare2 :

Nous devons dé bugger l’application je vais utiliser le logiciel radare2 

`r2 -d -A ./impossible_password`

`afl`
`pdf @main`
`s main`
Le deuxième strcmp compare le deuxième string entré qui est créé aléatoirement.

L’application va run cette fonction :

![Untitled](Impossible_password%20%5BReverse%20Engineering%5D%20Challeng%205e02deb391354d87bd9e592d5f65963d/Untitled%204.png)

`pdf 0x40078d`

Cette fonction la est pour générer un string aléatoire.

Il y a ensuite la comparaison entre notre mot de passe et celui généré aléatoirement.
ensuite le test va nous faire finir le programme avec un jump sur un retour. Une fonction n’est donc pas jouer car le jump va la dévié c’est la fonction 0x400978. On va changer le jump en nop.

On peux le faire de deux manières avec la commande wx ou avec le mode visuel de radare2.

`oo+` pour passer en mode écriture

On peux appuyer sur `P` pour changer de vu.

Appuyer sur `C` et allons jusqu’au jump de l’adresse 00400968

![Untitled](Impossible_password%20%5BReverse%20Engineering%5D%20Challeng%205e02deb391354d87bd9e592d5f65963d/Untitled%205.png)

Maintenant nous allons appuyer sur `i` pour écrire. 90 en hex cella est égale à l’instruction nop. Maintenant un dernier problème si nous changeons juste le saut non égal en 1 nop donc 750C à 90.  Nous changeons 4 caractère en seulement 2. La solution est de mettre 9090 pour avoir deux ligne nop. `esc` pour exit. Nous rentrons le même string pour le premier mot de passe et n’importe pour le deuxième.

HTB{40b949f92b86b18}