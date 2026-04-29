RockYou John !

---

## 1. Objectifs

- Cracker des hash de différents types :
  - Hash basiques.
  - Hash d’authentification Windows.
  - Hash issus de `/etc/shadow`.
  - Fichiers ZIP protégés par mot de passe.
  - Fichiers RAR protégés par mot de passe.
  - Clés SSH protégées par passphrase.

L’outil utilisé est **John The Ripper**, un logiciel de cassage de mots de passe supportant de nombreux formats de hash, en mode dictionnaire avec la wordlist `rockyou.txt` [web:125][web:137][web:138].

---

## 2. Consignes

- Vous devez **cracker les différents fichiers** mis à votre disposition (hash ou fichiers chiffrés).
- Vous devez **utiliser John The Ripper** en combinaison avec la wordlist **« rockyou.txt »**.
- Pour chaque type de hash / fichier, conservez une trace des commandes utilisées et des mots de passe trouvés.

---

## 3. Contenu du TP

Les exercices portent sur les cas suivants :

1. Hash basiques (ex. MD5, SHA…).
2. Hash d’authentification Windows (NTLM).
3. Hash issus de `/etc/shadow` (Linux).
4. Fichiers ZIP protégés par mot de passe.
5. Fichiers RAR protégés par mot de passe.
6. Clés SSH chiffrées (protégées par une passphrase).

Pour chacun, l’objectif est d’extraire ou de préparer les hash dans un format compatible John, puis de lancer une attaque par dictionnaire avec `rockyou.txt` [web:123][web:126][web:129][web:130].

---

## 4. Outils et ressources

- **John The Ripper (version jumbo de préférence)** pour supporter un maximum de formats (NTLM, ZIP, RAR, SSH, etc.) [web:129][web:130].
- **Wordlist `rockyou.txt`**, généralement disponible dans :
  - `/usr/share/wordlists/rockyou.txt` sur Kali/Ubuntu orienté sécu.
- Utilitaires associés à John :
  - `unshadow` pour fusionner `/etc/passwd` et `/etc/shadow`.
  - `zip2john` pour les fichiers ZIP.
  - `rar2john` pour les fichiers RAR.
  - `ssh2john` pour les clés SSH.

---

## 5. Exemple de commande générale

Pour un fichier de hash générique `hashes.txt` et la wordlist rockyou :

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt
```

Pour afficher les mots de passe trouvés :

```bash
john --show hashes.txt
```

Ces modèles seront adaptés à chaque format (option `--format` si nécessaire, et outils `zip2john`, `rar2john`, `ssh2john`, etc.) conformément à la documentation et aux cours [web:125][web:137][web:138].
