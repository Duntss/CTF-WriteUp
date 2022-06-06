# anti_flag [Reverse Engineering] Challenge

Tout d’abord on analyse le fichier :

ltrace, file, strings. On apprend ce qu’on sait déjà c’est un binaire qui est exécuter les arguments ne changent pas le résultat.

On scan le binaire avec Ghidra → on va dans la fonction entry FUN_00101486 c’est une fonction à 6 arguments. ptrace permet de de surveiller processus si il est exécuter dans un débuguer.

Dans la mème fonction dans le code assembly on peux voir plus que dans le C on voit

`dword ptr [RBP + local_24] , 0x539`

On va prendre le registre RBP et l’ajouter à la variable local_24 et la comparer à 0x539.

En regardant les commentaires de Ghidra on peux voir beaucoup d’avertissement disant que certaine partie du code ne peuvent pas être atteinte. 

Edit → Options → Decompiler → Analysis → Décocher Eliminate Unreachable Code 

On peux maintenant voir ce code qui est important !

![Untitled](anti_flag%20%5BReverse%20Engineering%5D%20Challenge%203db488f66fc34f8d8a35a81ff4933818/Untitled.png)

Dans ce code il regarde si on l’exécute dans un décompiler ce qui est impossible a avoir car la valeur de retour a obtenir de lVar2 est -1 .

On passe forcément ua `else if` qui va exécuter la fonction FUN_001013ff avec 2 paramètres si false = true. Il est impossible que false = true le code va donc passer au else.

Le else lui nous retourne juste un message basique.

**Résoudre avec GDB-PwnDbg :**

Avec ghidra on a vu que :

1525 adresse à aller pour commencer la fonction cacher

14f4 le if à contourner

Après le starti nous n'avons pas dépassé la première adresse cible :

```bash
pwndbg> piebase
Calculated VA from /home/kali/Desktop/anti_flag = 0x555555554000
pwndbg> piebase 0x1525
Calculated VA from /home/kali/Desktop/anti_flag = 0x555555555525
```

break point au if que nous voulons contourner -> `breakrva 0x14f4`
continue avec `c`

Maintenant nous ne voulons pas aller au elseif car le programme va détecter que false != true et skip le programme on va aller directement à la fonction en dessous.

On connais l'adresse piebase de 0x1525 on peux jump directement avec => `jump *0x555555555525`

On obtient le flag !

Ce qu’on apprend avec cette méthode simple c’est qu’avec Piebase et dbg nous pouvons aller un peux partout dans le binaire rien qu’avec les adresses trouver dans ghidra.

**Résoudre avec Patch Ghidra :**

Pour patch dans ghidra nous avons besoin d’ajouter le srcipt [SavePatch.py](http://SavePatch.py) à ghidra 

![Untitled](anti_flag%20%5BReverse%20Engineering%5D%20Challenge%203db488f66fc34f8d8a35a81ff4933818/Untitled%201.png)

Cliquer sur ce boutton → ensuite créé un nouveau script python avec celui ci :

![Untitled](anti_flag%20%5BReverse%20Engineering%5D%20Challenge%203db488f66fc34f8d8a35a81ff4933818/Untitled%202.png)

Et ensuite collé le script de ce github de fou furieux :

[https://raw.githubusercontent.com/schlafwandler/ghidra_SavePatch/master/SavePatch.py](https://raw.githubusercontent.com/schlafwandler/ghidra_SavePatch/master/SavePatch.py)

Sauvegarder

L’objectif est de changer le programme pour mettre true à la place de false au `elseif`

On clique sur le false cela nous fais apparaitre dans la page assembleur à une instruction JZ qui est jump if equal zero on CTRL+SHIFT+G ou clique droit Patch Instruction on met 75 au lieu de 74 le nombre hexa pour JNZ jump if not equal zero. On sélectionne la ligne en entier pour qu’elle soit en surbriallance verte et on retourne dans la page des script pour exécuter PatchScript.py.

 

On sauvegarde ensuite le binaire patch et on l’exécute on obtient directement le flag !

On apprend avec cette méthode que nous pouvons patch des fichier avec ghidra ce qui peux être très utile