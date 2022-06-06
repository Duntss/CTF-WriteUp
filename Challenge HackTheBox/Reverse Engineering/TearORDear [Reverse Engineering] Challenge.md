# TearORDear [Reverse Engineering] Challenge

Sur Linux commande `file` nous savons maintenant que c’est une application en C# nous pouvons dans la décompiler avec dnSpy.

![Untitled](TearORDear%20%5BReverse%20Engineering%5D%20Challenge%20b945dab81d8b40b2bd9e5dbd06067266/Untitled.png)

Lançons l’application sur Windows, elle est extrêmement basique juste 2 boite de texte qui vont être vérifié chacun indépendamment. La vérification des deux boite seront lancé quand l’utilisateur appuie sur le bouton.

Dans dnSpy la partie qui va nous intéressé le plus est  Login Form. 

En regardant les  déclaration de variable nous pouvons voir `oura = “p00f123#”;`

![Untitled](TearORDear%20%5BReverse%20Engineering%5D%20Challenge%20b945dab81d8b40b2bd9e5dbd06067266/Untitled%201.png)

et `pepper = 10;`

![Untitled](TearORDear%20%5BReverse%20Engineering%5D%20Challenge%20b945dab81d8b40b2bd9e5dbd06067266/Untitled%202.png)

En fonction utilisé nous pouvons voir plusieurs fonctions avec le nom “check” ou “encrypté”.

Dans la fonction button1_Click nous avons un appel de la fonction check1 selon l’username qui va nous ressortir si le username envoyé est le bon ou non. 

![Untitled](TearORDear%20%5BReverse%20Engineering%5D%20Challenge%20b945dab81d8b40b2bd9e5dbd06067266/Untitled%203.png)

Avec dnSpy nous pouvons mettre un point d’arrêt et au lieu de reverse en static nous allons pouvoir regarder ce qui est stocké en mémoire.

En rentrant des login au hasard nous allons trigger le break-point que nous avons mis précédemment. J’ai mis password = abc et username = 012 

Nous pouvons voir en même temps les variables local du programmes grâce à la fenêtre du bas du programme.

![Untitled](TearORDear%20%5BReverse%20Engineering%5D%20Challenge%20b945dab81d8b40b2bd9e5dbd06067266/Untitled%204.png)

Sur la partie if du programme mise avant nous pouvons voir des `this.` on défile donc les variables local this de la fenêtres basse.

Le programme a bien mis mon username dans la variable local `this.username`

![Untitled](TearORDear%20%5BReverse%20Engineering%5D%20Challenge%20b945dab81d8b40b2bd9e5dbd06067266/Untitled%205.png)

`username= 012`

Ayant mis en username sur le programme `abc` et en password `012` la boite password stock dans la variable password le if check donc le password que nous rentrons. 

Une fonctionnalité intéressante de  dnSpy est que nous pouvons survéiller une variable. Nous allons l’utilisé pour this.username. Clic droit + Add Watch.

![Untitled](TearORDear%20%5BReverse%20Engineering%5D%20Challenge%20b945dab81d8b40b2bd9e5dbd06067266/Untitled%206.png)

Dans les variables local du programme nous pouvons trouver 2 autres variables intéressantes `this.o` et `s` 

![Untitled](TearORDear%20%5BReverse%20Engineering%5D%20Challenge%20b945dab81d8b40b2bd9e5dbd06067266/Untitled%207.png)

Comme nous avons déjà vu `this.o` est le mot de passe nous allons changer la valeurs de la variable `this.uername` directement dans dnSpy pour `roiw!@#` . Dans la ligne du if nous avons une dérnière vérification du mot de passe qui regarde si notre mot de passe est égal à `this.check1(s)` allons donc dans cette fonction et mettons un breakpoint à la ligne ou le string est calculé. Puis nous pouvons appuyé sur Continuer.

![Untitled](TearORDear%20%5BReverse%20Engineering%5D%20Challenge%20b945dab81d8b40b2bd9e5dbd06067266/Untitled%208.png)

Cette ligne est calculé mais nous n’avons pas la valeur en retour nous pouvons step over jusqu’au return (F10) et dans nos variables local nous trouvons `s2 = “roiw “` .

Nous faisons de la même manière le check2 pour obtenir s3.

Ce que nous pouvons voir c’est que les 4 check sont exactement les même donc nous n’avons pas besoin de refaire la même démarche pour tous car cela ne changera pas le résultat de la variable nous pouvons donc avancer jusqu’au dernier check.

On vois directement que la ligne return de la fonction check() à changer c’est bien le dernier check de notre variable.

Comme d’habitude break-point et le return de cette fonction est de regarder si notre username est égal à `this.aa` . On va regarder dans les variables local la valeur de la variable et c’est bon nous pouvons tester notre username et notre mot de passe dans l’application. On a en retour le message correct nous avons donc tous ce qu’il nous faut pour écrire le flag.