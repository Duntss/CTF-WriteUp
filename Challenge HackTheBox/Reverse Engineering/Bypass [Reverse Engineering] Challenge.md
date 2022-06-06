# Bypass [Reverse Engineering] Challenge

**L’Application :**

![Untitled](Bypass%20%5BReverse%20Engineering%5D%20Challenge%200b3e0145b7ae4b83a661c10ae79c4594/Untitled.png)

Nous pouvons rentrer un username et un password.

Le logiciel est un .Net 32 bit exécutable.

![Untitled](Bypass%20%5BReverse%20Engineering%5D%20Challenge%200b3e0145b7ae4b83a661c10ae79c4594/Untitled%201.png)

Le problème dans les applications .Net est qu’il n’y a pas de fonction principal donc c’est inutiles de la cherché comme dans des exercices de reverse pour du C.

Analyse dans dnSpy :

L’application étant en .Net nous pouvons utiliser dnSpy ou ILSpy pour analyser l’application et changer son assembleur.

Après avoir analyser toutes les class de l’application nous devons comprendre comment fonctionne le programme. 

Dans la class 0 nous pouvons voir en plaçant un break-point à la ligne 26 que cette fonction s’occupe d’afficher et prendre des écritures. On peux remarquer quelque chose de bizarre c’est le retour de la fonction nous obtenons un booléen false. Nous pouvons changer le false pour un true. Clique droit > modifier les instruction IL. Changeons la ligne 11 ldc.i4.0 à ldc.i4.1 ce qui change le false à true. CTRL+Shift+S et nous relançons le programme avec le patch. Et maintenant nous pouvons rentrer n’importe quel données et nous recevrons un nouveaux retour.

![Untitled](Bypass%20%5BReverse%20Engineering%5D%20Challenge%200b3e0145b7ae4b83a661c10ae79c4594/Untitled%202.png)

Trouver la Secret Key :

Dans notre fonction 0 nous avons pu afficher le username et le password puis demander au code d’afficher l’input “Please Enter the secret Key:” en changeant le false à true dans le return. 

Le programme va ensuite appelé la méthode 2() en mettant un point d’arrêt  dans la méthode, on vois que c’est l’apparition de la clé secrète.

On voit dans la méthode une fonction booléen flag qui va nous sortir un nom vide si la ligne est écrite, nous pouvons changer le == en !=. On modifie les instructions IL et nous allons choisir la ligne 10 call pour changer l’ op_Equality à op_Inequality.

Le problème est que le programme se ferme des l’instant ou on affiche le flag.

Empêcher la fermeture du programme :

A la fin de la méthode vu précédemment nous avons un retour que nous pouvons voir dans l’assembleur IL un retour et un jump qui envoie à ce retour. On replace le return par un nop pour effacer la fonction jump et retour. On sauvegarde et nous ne n’avons plus de fermeture du programme et nous obtenons le flag.