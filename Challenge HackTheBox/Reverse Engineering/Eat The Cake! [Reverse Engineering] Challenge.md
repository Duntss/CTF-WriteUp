# Eat The Cake! [Reverse Engineering] Challenge

Les Informations du fichier :

J’ai utilisé le programme setdllcharacteristics.exe trouvable ici : [https://gist.github.com/Riketta/04e8dc2fa08163463ab7](https://gist.github.com/Riketta/04e8dc2fa08163463ab7)

```bash
┌──(root💀kali)-[/home/kali/Desktop]
└─# ./setdllcharacteristics.exe cake.exe                                              127 ⨯
Original DLLCHARACTERISTICS = 0x8140
 DYNAMIC_BASE    = 1
 NX_COMPAT       = 1
 FORCE_INTEGRITY = 0
                                                                                            
┌──(root💀kali)-[/home/kali/Desktop]
└─# ./setdllcharacteristics.exe -d -n cake.exe
Original DLLCHARACTERISTICS = 0x8140
 DYNAMIC_BASE    = 1
 NX_COMPAT       = 1
 FORCE_INTEGRITY = 0
Updated  DLLCHARACTERISTICS = 0x8000
	DYNAMIC_BASE    = 0
 NX_COMPAT       = 0
 FORCE_INTEGRITY = 0
```

Avec ces trouvaille nous devons déterminer qu’en lançant l’exe sur Ida nous voyons absolument rien a part la fonction “entry”/start tous ceci est le signe d’un UPX modifié.

Ouvrons IDA : 

Nous cherchons les strings et les fonctions et nous avons quasiment rien juste start et quelque strings inutiles dans le premier block nous avons “pusha”

[https://modoocode.com/en/inst/pusha-pushad](https://modoocode.com/en/inst/pusha-pushad)

pusha va donc poussé le contenu des registres à usage général sur la pile.

Cette fonction pusha va nous permettre d’unpack l’UPX de l’exe (liens de la vidéo en ressource)

Suffit de prendre l’adresse de l’offset dans ce cas si c’est 40800.

![Untitled](Eat%20The%20Cake!%20%5BReverse%20Engineering%5D%20Challenge%209d9ba1aaf1f64ce08bff7ba4004621a3/Untitled.png)

de le soustraire à la valeur donné en dessous avec la valeur  du pointeur donné ici 7000.

![Untitled](Eat%20The%20Cake!%20%5BReverse%20Engineering%5D%20Challenge%209d9ba1aaf1f64ce08bff7ba4004621a3/Untitled%201.png)

En python `> hex(0X408000-0x7000)`

                  `> ‘0x401000’`

On va aller dans cette nouvelle adresse `0x401000` avec la fonctionalité Jump de IDA appuyer sur `G` ou `Jump > Jump to adresse`

En y allant on trouve une ligne commenté de `CODE XREF: start+1B4→j` double clique on ajoute un breakpoint au dessus sur le jmp

![Untitled](Eat%20The%20Cake!%20%5BReverse%20Engineering%5D%20Challenge%209d9ba1aaf1f64ce08bff7ba4004621a3/Untitled%202.png)

On lance le .exe et on démarre le programme normalement avec IDA Après avoir passé le breakpoint Nous pouvons aller dans `View > Open Subviews > Strings` ou ‘`Shift+F12`’ et maintenant nous avons accès à tous les strings que nous ne pouvions voir au part avant. 

![Untitled](Eat%20The%20Cake!%20%5BReverse%20Engineering%5D%20Challenge%209d9ba1aaf1f64ce08bff7ba4004621a3/Untitled%203.png)

Maintenant nous pouvons accéder à tous le code allons regarder la fonction qui affiche le  message “Congratulations! Now go validate your flag!”.

![Untitled](Eat%20The%20Cake!%20%5BReverse%20Engineering%5D%20Challenge%209d9ba1aaf1f64ce08bff7ba4004621a3/Untitled%204.png)

Cette fonction prend 2 argument pour être validé F5 pour afficher cette fonction en pseudo code.

Nous pouvons voir que le programme s’occupe juste de vérifier les caractère qui sont  écrit dans le code directement :

![Untitled](Eat%20The%20Cake!%20%5BReverse%20Engineering%5D%20Challenge%209d9ba1aaf1f64ce08bff7ba4004621a3/Untitled%205.png)

Il va faire des check de caractère en plusieurs temps pour nous ressortir après sa réponse.

Voici une copie des variables :

```cpp
char v1; // dl
  char v2; // al
  char v3; // cl
  char *v4; // edx
  char v6; // [esp+Bh] [ebp-431h]
  _DWORD v7[4]; // [esp+Ch] [ebp-430h] BYREF
  int v8; // [esp+1Ch] [ebp-420h]
  unsigned int v9; // [esp+20h] [ebp-41Ch]
  int v10; = @
  char v11;  = h
  char v12; = a
  char v13; = r
  char v14; = a
  char v15; = d
  char v16; = $
  char v17; = E
  char v18; // [esp+423h] [ebp-19h]
  int v19; // [esp+438h] [ebp-4h]
```

Avec cette fonction nous avons déjà 12 caractères. Dans le code nous avons un appel à une autre fonction en cliquant dessus nous avons 3 autres caractère l p t

Autres :

On peux voir aussi dans Ida quand nous avons retrouver le string les lettres du mot de passe :

k  a h a h r d @ E c $

0 1 2 3 4 5 6 7 8 9 a b c e f

HTB{h@ckth3parad1$E}

**Ressources :** 

****Vidéo trop dark mais qui apprend à unpack un UPX avec IDA :
****[https://www.youtube.com/watch?v=vvk_ISkKOAE](https://www.youtube.com/watch?v=vvk_ISkKOAE)