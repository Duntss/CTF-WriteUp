# APKrypt [Mobile] Challenge

[https://app.hackthebox.com/challenges/apkrypt](https://app.hackthebox.com/challenges/apkrypt)

Nous avons une application en .apk android écrite en Java.

En lançant l’application dans un émulateur nous avons 1 boîte de dialogue nous demandant un VIP code

Quand nous regardons le code dans jadx-gui la class la plus intéressantes du programme est Main Activity.

![Untitled](APKrypt%20%5BMobile%5D%20Challenge%205b63b004654f4ffbbfcc7002b2162079/Untitled.png)

Dans cette class nous voyons un if else statement. 

```java
if (MainActivity.md5(MainActivity.this.ed1.getText().toString()).equals("735c3628699822c4c1c09219f317a8e9")) {
                        Toast.makeText(MainActivity.this.getApplicationContext(), MainActivity.decrypt("k+RLD5J86JRYnluaZLF3Zs/yJrVdVfGo1CQy5k0+tCZDJZTozBWPn2lExQYDHH1l"), 1).show();
                    } else {
                        Toast.makeText(MainActivity.this.getApplicationContext(), "Wrong VIP code!", 0).show();
										}
```

Dans ces lignes il est demandé une comparaison avec un string qui à été hash en MD5 avec le string “735c3628699822c4c1c09219f317a8e9” .Nous savons seulement que le String est changé en md5 la première chose a tenté est de prendre le string écrit en vert si-dessus et d’essayer de le déhash. Après tentative ça ne marche pas c’est normal car c’est un challenge Mobile. Nous avons 3 possibilité pour faire la suite de ce challenge. BruteForce le hash ça prendra un temps indéterminé donc il vaut mieux ne pas partir la dessus. La deuxième est de changé la logique du if else par exemple au lieux de regarder si les 2 chaines sont égales pour afficher le supposé flag il faudrait changé ce bout de code pour vérifier si elle sont non égal pour afficher le flag. Cette deuxième solution serrait très utilisé dans un environnement de fichier binaire C. La troisième solution et la plus simple est de changé le md5 comparé par un hash ou nous connaissons la valeurs à l’avance, pour synthétiser nous allons changer le mot de passe par le notre.

J’ai choisi la solution 2 et 3 :

Nous allons donc changer le code et pour cela nous devons désassembler l’apk avec apktool 

[https://ibotpeaches.github.io/Apktool/install/](https://ibotpeaches.github.io/Apktool/install/)

Et j’utiliserai vscode mais vous pouvez modifier le code avec l’ide que vous voulez.

Je vais ajouter aussi l’extension APKLab pour coder en smali

Le String que nous voulons modifier est dans MainActivity$1.smali à la ligne 59 

Je l’ai replacé par le hash md5 de admin :

```python
echo -n "admin" | md5sum                                 
21232f297a57a5a743894a0e4a801fc3  -
```

et nous devons changer la ligne 67 `if-eqz` à `if-nez` ce qui va nous enlever la vérification de l’application sur la chaine md5

On rebuild l’application avec `apktool b APKrypt/ -o modified.apk`

b pour build et -o pour le fichier output

Maintenant nous devons juste mettre notre clé certificat sur l’apk

`keytool -genkey -v -keystore my-release-key.keystore -alias modified_apk -keyalg RSA -keysize 2048 -validity 10000`

Puis on utilise la clé sur l’apk 

`jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 -keystore my-release-key.keystore modified.apk modified_apk`

On lance notre apk et nous pouvons mettre le mot de passe que nous voulons pour avoir le flag.

**Ressources :**

WriteUp avec vidéo de CryptoCat :

[https://www.youtube.com/watch?v=s-yK85t4_EY](https://www.youtube.com/watch?v=s-yK85t4_EY)

Article de comment reverse un jeux et le modifier :

[https://medium.com/swlh/reverse-engineering-and-modifying-an-android-game-apk-ctf-c617151b874c](https://medium.com/swlh/reverse-engineering-and-modifying-an-android-game-apk-ctf-c617151b874c)