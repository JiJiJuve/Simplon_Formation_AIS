# TP – Cracking de mots de passe avec John the Ripper

Ce TP montre comment utiliser **John the Ripper** pour casser différents types de mots de passe (hash simples, NTLM, shadow Linux, ZIP, RAR, clé SSH), en s’appuyant sur des outils complémentaires comme **hashid**, `unshadow`, `zip2john`, `rar2john`, `ssh2john.py`.

Toutes les commandes sont adaptées à un environnement utilisateur `jiji` avec les fichiers de TP sous `~/TP4-RockYouJohn`.

***

## Sommaire

- [0. Préparation de l’environnement](#0-préparation-de-lenvironnement)  
- [1. Hash simples (MD5, SHA1, SHA256, Whirlpool)](#1-hash-simples-md5-sha1-sha256-whirlpool)  
- [2. Hashes Windows NTLM (NT Hash)](#2-hashes-windows-ntlm-nt-hash)  
- [3. Hashes Linux /etc/passwd + /etc/shadow](#3-hashes-linux-etcpasswd--etcshadow)  
- [4. Archive ZIP protégée (securezip)](#4-archive-zip-protégée-securezip)  
- [5. Archive RAR protégée (securerar)](#5-archive-rar-protégée-securerar)  
- [6. Clé SSH privée protégée (idrsa)](#6-clé-ssh-privée-protégée-id_rsa)  
- [7. Rôle de hashid dans le workflow](#7-rôle-de-hashid-dans-le-workflow)  
- [8. Vue d’ensemble : pattern commun](#8-vue-densemble--pattern-commun)  

***

## 0. Préparation de l’environnement

### 0.1. Vérifier John et les outils associés

```bash
which john
ls /usr/share/john
```

- `john` est l’outil principal de Decryptage.  
- `/usr/share/john` contient des scripts comme `ssh2john.py`.

### 0.2. Préparer la wordlist `rockyou.txt`

Sur Kali, `rockyou.txt` se trouve en général dans `/usr/share/wordlists/`. Si seule la version compressée existe :

```bash
ls /usr/share/wordlists
sudo gzip -d /usr/share/wordlists/rockyou.txt.gz
```

Puis copie dans le home de l’utilisateur pour éviter les problèmes de droits :

```bash
sudo cp /usr/share/wordlists/rockyou.txt /home/jiji/rockyou.txt
sudo chown jiji:jiji /home/jiji/rockyou.txt
ls -l /home/jiji/rockyou.txt
```

Désormais, toutes les commandes John utiliseront :

```bash
--wordlist=/home/jiji/rockyou.txt
```

***

## 1. Hash simples (MD5, SHA1, SHA256, Whirlpool)

### Objectif

Casser des hashes “simples” (un hash par ligne), en choisissant le bon format `--format` selon l’algorithme (MD5, SHA1, SHA256, Whirlpool).

### 1.1. Identification avec `hashid`

Dans le dossier des hashes basiques :

```bash
cd "/home/jiji/TP4-RockYouJohn/1-Basic Hashes"
ls
# hash1.txt, hash2.txt, hash3.txt, hash4.txt, etc.
```

hashid hash1.txt

<img width="766" height="502" alt="Basic_hash1" src="https://github.com/user-attachments/assets/eba9033d-a444-4d59-83e2-3fd49d97dbbb" />

<BVR><BVR>

<img width="1280" height="800" alt="Basic_hash1_decrypte" src="https://github.com/user-attachments/assets/7c1b0f1b-9bdb-46ef-b387-a9f5d47588ff" />


hashid hash2.txt

<img width="758" height="416" alt="Basic_hash2" src="https://github.com/user-attachments/assets/51feaf07-677b-498e-8ddb-1af421edc5fe" />

<BVR><BVR>

<img width="756" height="281" alt="Basic_hash2_decrypte" src="https://github.com/user-attachments/assets/887eb828-18a5-4937-9bf8-56e9fe4f9e7f" />

hashid hash3.txt

<img width="637" height="405" alt="Basic_hash3" src="https://github.com/user-attachments/assets/c3edf10e-4abf-43e3-8098-7d5a9bad5e4b" />

<BVR><BVR>

<img width="763" height="278" alt="Basic_hash3_decrypte" src="https://github.com/user-attachments/assets/2c504944-3070-4681-9d5f-fe26e71a0d64" />

hashid hash4.txt

<img width="1098" height="401" alt="Basic_hash4" src="https://github.com/user-attachments/assets/4fffa262-e9ec-42e6-9aa4-bd36c7dfa60f" />

<BVR><BVR>

<img width="1280" height="800" alt="Basic_hash4_decrypte" src="https://github.com/user-attachments/assets/ee177708-0cc9-4bd2-87ea-f4971f39ef1c" />

hashid hash7.txt

<img width="1280" height="106" alt="hash7" src="https://github.com/user-attachments/assets/2262a114-25ce-47e9-b405-cd19fee96162" />

<BVR><BVR>

<img width="438" height="171" alt="hash7" src="https://github.com/user-attachments/assets/836575b3-6954-407c-8f12-47997a007c49" />

<BVR><BVR>

<img width="1280" height="346" alt="hash7_decrypte" src="https://github.com/user-attachments/assets/7e39eb16-130a-455e-8040-ebbd766e4406" />



***

`hashid` se base sur la longueur et les caractères pour proposer des algorithmes possibles (MD5, SHA1, SHA256, Whirlpool, NTLM, etc.).

> Note : un même hash peut correspondre à plusieurs algos possibles (ex : MD5 vs NTLM) ; le **contexte** (Windows, Linux, TP) est essentiel pour choisir.

### 1.2. Decryptage avec John

Exemples typiques (formats à adapter à ce que donne le TP) :

```bash
# MD5
john --format=raw-md5 --wordlist=/home/jiji/rockyou.txt hash1.txt
john --show   --format=raw-md5 hash1.txt

# SHA1
john --format=raw-sha1 --wordlist=/home/jiji/rockyou.txt hash2.txt
john --show   --format=raw-sha1 hash2.txt

# SHA256
john --format=raw-sha256 --wordlist=/home/jiji/rockyou.txt hash3.txt
john --show   --format=raw-sha256 hash3.txt

# Whirlpool
john --format=whirlpool --wordlist=/home/jiji/rockyou.txt --rules hash4.txt
john --show   --format=whirlpool hash4.txt
```

**Pourquoi ça marche :**

- John recalcule le hash de chaque mot de la wordlist avec l’algorithme choisi (`--format`) et compare au hash cible.  
- Whirlpool est un algorithme de hachage 512 bits, différent de SHA‑512, mais supporté par John via `--format=whirlpool`.

***

## 2. Hashes Windows NTLM (NT Hash)

### Objectif

Decrypter un hash de type **NTLM** (NT hash) provenant de Windows.

### Contexte

Dossier :

```bash
cd "/home/jiji/TP4-RockYouJohn/2-Windows Authentication Hashes"
ls
cat ntlm.txt
# 32 caractères hexadécimaux
```

Un NT hash fait 128 bits, soit 32 hex, comme MD5 : `hashid` propose donc souvent MD5 **et** NTLM.

### Commandes

```bash
hashid ntlm.txt     # propose MD5 / NTLM / etc.

john --format=nt --wordlist=/home/jiji/rockyou.txt ntlm.txt
john --show --format=nt ntlm.txt
```

**Pourquoi `--format=nt` :**

- Dans un TP “Windows Authentication Hashes” avec un fichier nommé `ntlm.txt`, le bon choix est NTLM (NT hash) et non MD5.  
- Le format `NT` de John implémente l’algorithme de NTLM (MD4 sur le mot de passe Unicode).

<img width="1280" height="235" alt="Basic_Windows_Authentication_Hash_NTLM" src="https://github.com/user-attachments/assets/0cb46b54-8b4e-40cf-b593-157f8cb30d03" />

<BVR><BVR>

<img width="1280" height="277" alt="Basic_Windows_Authentication_Hash_NTLM_Decrypte" src="https://github.com/user-attachments/assets/1193d6e7-2496-42ef-a22b-2d1421a36e0d" />

***

## 3. Hashes Linux `/etc/passwd` + `/etc/shadow`

### Objectif

Casser des mots de passe Linux en combinant les fichiers de type `/etc/passwd` et `/etc/shadow` via `unshadow`.

### Contexte

```bash
cd "/home/jiji/TP4-RockYouJohn/3-Shadow Hashes"
ls
# local_passwd, local_shadow, etc_hashes.txt
```

- `local_passwd` : similaire à `/etc/passwd` (login, UID, etc.).  
- `local_shadow` : similaire à `/etc/shadow` (hash de mot de passe).

### Étape 1 – Fusion avec `unshadow`

```bash
unshadow local_passwd local_shadow > unshadowed.txt
cat unshadowed.txt
```

On obtient des lignes du type :

```text
root:$6$...$...:0:0:root:/root:/bin/bash
user1:$6$...$...:1000:1000:...:/home/user1:/bin/bash
```

### Étape 2 – Decryptage avec John

```bash
john --wordlist=/home/jiji/rockyou.txt unshadowed.txt
john --show unshadowed.txt
```

**Pourquoi ça marche :**

- `unshadow` reconstitue un format que John comprend (login + hash sur une seule ligne).  
- Les hash `$6$` correspondent souvent à `sha512crypt`, que John détecte automatiquement sans `--format`.

<img width="1280" height="317" alt="Shadow1" src="https://github.com/user-attachments/assets/a1e10c42-4e52-4296-9985-48867c7bc8a2" />

<BVR><BVR>

<img width="1070" height="406" alt="Shadow1_decrypte" src="https://github.com/user-attachments/assets/4c4f1ffc-3236-453c-b7ac-7e5bd0c46990" />


***

## 4. Archive ZIP protégée (`secure.zip`)

### Objectif

Retrouver le mot de passe d’un ZIP via `zip2john` + John.

### Contexte

```bash
cd "/home/jiji/TP4-RockYouJohn/5-Zip File"
ls
# secure.zip
```

### Étape 1 – Extraction du hash ZIP

```bash
zip2john secure.zip > zip.hash
chmod 644 zip.hash
head zip.hash
```

`zip.hash` contient un “pseudo-hash” spécifique au format ZIP que John sait attaquer.

### Étape 2 – Decryptage avec John

```bash
john --wordlist=/home/jiji/rockyou.txt zip.hash
john --show zip.hash
```

Dans le TP, le mot de passe trouvé est :

```text
pass123
```

**Pourquoi `zip2john` :**

- John ne travaille pas directement sur le `.zip`, mais sur une représentation intermédiaire extraite par `zip2john`, un utilitaire fourni avec John.

<img width="1165" height="800" alt="SecureZip_decrypte" src="https://github.com/user-attachments/assets/73995c7c-b6c0-4257-9b7c-29e77107b57c" />


***

## 5. Archive RAR protégée (`secure.rar`)

### Objectif

Même principe que pour ZIP, mais pour une archive RAR avec `rar2john`.

### Contexte

```bash
cd "/home/jiji/TP4-RockYouJohn/6-RAR File"
ls
# secure.rar
```

### Étape 1 – Extraction du hash RAR

```bash
rar2john secure.rar > rar.hash
chmod 644 rar.hash
head rar.hash
```

Le fichier contient une ligne avec `$RAR3$` ou `$RAR5$`.

### Étape 2 – Decryptage avec John

```bash
john --wordlist=/home/jiji/rockyou.txt rar.hash
john --show rar.hash
```

Dans le TP, le mot de passe trouvé est :

```text
password
```

**Pourquoi `rar2john` :**

- Comme `zip2john`, `rar2john` convertit l’archive RAR en un format que John peut exploiter (structure interne + sel, etc.).

<img width="731" height="401" alt="SecureRar_decrypte" src="https://github.com/user-attachments/assets/f559d64f-8743-4025-b778-a6621d0697e4" />


***

## 6. Clé SSH privée protégée (`id_rsa`)

### Objectif

Retrouver la **passphrase** d’une clé privée SSH avec `ssh2john.py` + John.

### Contexte

```bash
cd "/home/jiji/TP4-RockYouJohn/7-SSH Key"
ls
# id_rsa
```

`id_rsa` est une clé privée protégée par une passphrase.

### Étape 1 – Extraction du hash avec `ssh2john.py`

Vérifier la présence du script :

```bash
ls /usr/share/john/ssh2john.py
```

Puis :

```bash
cd "/home/jiji/TP4-RockYouJohn/7-SSH Key"
python3 /usr/share/john/ssh2john.py id_rsa > ssh.hash
head ssh.hash
```

`ssh.hash` contient une représentation du secret de la clé au format John.

### Étape 2 – Decryptage de la passphrase avec John

```bash
john --wordlist=/home/jiji/rockyou.txt ssh.hash
john --show ssh.hash
```

Dans le TP, la passphrase trouvée est :

```text
mango
```

**Pourquoi `ssh2john.py` :**

- Une clé SSH chiffrée ne contient pas directement le mot de passe, mais une structure chiffrée. `ssh2john.py` extrait de cette structure un hash que John peut attaquer.

<img width="1280" height="739" alt="SSh_decrypte" src="https://github.com/user-attachments/assets/6bc716f9-fdc9-4532-9a99-7c5074b708cf" />


***

## 7. Rôle de `hashid` dans le workflow

### 7.1. Identification du type de hash

`hashid` sert d’étape **d’analyse** avant le cassage :

```bash
hashid hash.txt            # vue générale
hashid -j hash.txt         # suggère le format John (si possible)
hashid -m hash.txt         # suggère le mode Hashcat (optionnel)
```

### 7.2. Limites

- `hashid` se base uniquement sur la **forme** (longueur, alphabet), il peut proposer **plusieurs** algos possibles (ex : MD5 / NTLM).  
- C’est le **contexte** (Windows, Linux, archive, etc.) qui permet de choisir le bon format `--format` dans John.

***

## 8. Vue d’ensemble : pattern commun

Pour tous les exercices, le schéma est le même :

1. **Identifier** le type de secret :
   - Hash simple → `hashid` + doc John.  
   - Hash NTLM → contexte Windows + format `NT`.  
   - Hash shadow → `unshadow`.  
   - Zip/RAR → `zip2john` / `rar2john`.  
   - Clé SSH → `ssh2john.py`.

2. **Convertir** si besoin avec un outil `*2john` ou `unshadow`.

3. **Decrypter** avec John :

```bash
john --wordlist=/home/jiji/rockyou.txt [fichier_hash]
```

4. **Afficher** les résultats :

```bash
john --show [fichier_hash]
```

Ce README couvre toute la logique du TP et peut servir de base de référence pour d’autres labs de cracking de mots de passe.
