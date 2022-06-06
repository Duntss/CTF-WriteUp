# The Art of Reversing [Reverse Engineering] Challenge

[https://app.hackthebox.com/challenges/the-art-of-reversing](https://app.hackthebox.com/challenges/the-art-of-reversing)

Sur Linux nous faisons la commande `file` cela nous ressort que le .exe est une application faites en .NET nous pouvons utilisé dnSpy pour analyser l’application et c’est différentes fonctions.

![Untitled](The%20Art%20of%20Reversing%20%5BReverse%20Engineering%5D%20Challen%20477cf423d6924c57a358f849defb0c95/Untitled.png)

En rentrant des Strings au hasard sur l’application on peux voir que les 2 partie sont calculé l’une sans l’autre. En changeant la date tout le temps nous aurons toujours la même partie au début.

![Untitled](The%20Art%20of%20Reversing%20%5BReverse%20Engineering%5D%20Challen%20477cf423d6924c57a358f849defb0c95/Untitled%201.png)

J’ai eu de la chance j’ai trouvé par hasard 365 en jours ce qui nous donne la bonne suite sur la clé.

Pour trouver la première partie du String j’ai créé une clé avec un ordre logique pour voir comment il été réassemblé puis j’ai remis dans l’ordre pour recréé le username

![Untitled](The%20Art%20of%20Reversing%20%5BReverse%20Engineering%5D%20Challen%20477cf423d6924c57a358f849defb0c95/Untitled%202.png)

Si nous voulons trouver sans bruteforce ou par chance nous pouvons revenir sur dnSpy :

En cherchant dans la classe From1 on peux voir la fonction ToR() qui va nous retourner la valeur jours de la clé. Sauf que dans la fonction nous avons en retour que des chiffres romain pas de w et y. Si nous allons dans la fonctions DoR nous avons une conversion de notre int a un tableau puis une boucle selon la taille de notre int qui va retourné notre et qui va rajouté 1 a tous les caractère et nous le retourné en lowerCase.