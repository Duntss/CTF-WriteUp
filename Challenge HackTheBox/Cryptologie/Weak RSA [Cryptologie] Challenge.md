# Weak RSA [Cryptologie] Challenge

`unzip weak-rsa.zip`

On a donc un fichier chiffrer avec une clé privé inconnu et la clé public de ce fichier.

Il faut donc craquer la clé à partir de la clé public, un tool existe pour faire cela automatiquement.

[https://github.com/Ganapati/RsaCtfTool](https://github.com/Ganapati/RsaCtfTool)

`./RsaCtfTool.py --publickey ./key.pub --private`

`openssl rsautl -decrypt -in $ENCRYPTED -out $PLAINTEXT -inkey keys/privkey.pem`