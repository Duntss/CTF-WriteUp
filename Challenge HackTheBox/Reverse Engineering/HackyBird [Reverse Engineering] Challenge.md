# HackyBird [Reverse Engineering] Challenge

Le challenge est un executable Windows, c’est une copie de flappy bird. 
Le chall est détecté comme un trojan je l’ai donc utilisé dans une VM.

Le programme étant un jeu j’ai pensé au cheat. Sur cheat engine il faut utilisé le VH debugger à activer dans les options. Il faut changer la valeur du score la valeur est de base 0 et se réinitialise à chaque nouveau lancement de partie il faut trouver la bonne valeur sinon nous obtenons une ligne ne voulant rien dire.

Si on test de plus en plus on peux trouver facilement la limite.

La limite est de 1000 donc pour avoir le flag mettre 999 et faire 1.