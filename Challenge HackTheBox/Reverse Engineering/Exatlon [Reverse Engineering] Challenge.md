# Exatlon [Reverse Engineering] Challenge

File type :

C’est un exécutable Linux ELF basique. Une bannière s’affiche peut à peut avec le nom Exalton et on peut rentrer un mot de passe. 

En regardant avec IDA le programme Exatlon on trouve que très peux de fonction et aucun print ce qui nous donne tous les signaux d’une modification upx.

`upx -d exalton -o exalton_clear`

Analyse :

Maintenant nous pouvons retourner sur IDA est nous obtenons un nombre énorme de fonction et de string. On vas les trier par nom et trouver la fonction principal main.

On trouve directement un string “Looks Good”

![Untitled](Exatlon%20%5BReverse%20Engineering%5D%20Challenge%20c0fa6ea7e7944bbf9016e9740f0aa0bc/Untitled.png)

Plus haut du string nous avons un autre string chiffré on met un breakpoint a la ligne en dessous et regardons dans les registre le retour du string complète.

"1152 1344 1056 1968 1728 816 1648 784 1584 816 1728 1520 1840 16”

En cliquant dessus et en regardant toutes les lignes en dessous 

“4 784 1632 1856 1520 1728 816 1632 1856 1520 784 1760 1840 1824 8”

“16 1584 1856 784 1776 1760 528 528 2000”

En recollant les numéro et en divisant par 16 chaque suite en changeant les chiffres par des caractères avec la table ascii.

 

```cpp
1152 1344 1056 1968 1728 816 1648 784 1584 816 1728 1520 1840 1664 784 1632 1856 1520 1728 816 1632 1856 1520 784 1760 1840 1824 8

1152 = 72 = H
1344 = 84 = T
1056 = 66 = B
1968 = 123 = {
1728 = 108 = l
816 = 51 = 3
1648 = 103 = g
784 = 49 = 1
1584 = 99 = c
816 = 51 = 3
1728 = 108 = l
1520 = 95 = _
1840 = 115 = s
1664 = 104 = h
784 = 49 = 1
1632 = 102 = f
1856 = 116 = t
1520 = 95 = _
1728 = 108 = l
816 = 51 = 3
1632 = 102 = f
1856 = 116 = t
1520 = 95 = _
784 = 49 = 1
1760 = 110 = n
1840 = 115 = s
1824 = 114 = r
816 = 51 = 3
1584 = 99 = c
1856 = 116 = t
784 = 49 = 1
1776 = 111 = o
1760 = 110 = n
528 = 33 = !
528 = 33 = !
2000 = 125 = }

HTB{l3g1c3l_sh1ft_l3ft_1nsr3ct1on!!}
```

Ce que j’ai appris :
Regarder plus les instruction de transformation de variable et apprendre a rentrer des valeurs dans ida pendant l’execution