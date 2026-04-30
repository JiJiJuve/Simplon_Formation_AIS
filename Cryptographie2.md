# TP2 – Chiffrement symétrique (AES) et asymétrique (RSA) avec OpenSSL

## Sommaire

- [1. Objectifs](#1-objectifs)  
- [I. Chiffrement symétrique AES](#i-chiffrement-symétrique-aes)  
  - [A. Découverte avec `TESTSECRET1234567`](#a-découverte-avec-testsecret1234567)  
  - [B. Voitures préférées – échange binôme](#b-voitures-préférées--échange-binôme)  
  - [C. Chansons – clé et IV explicites](#c-chansons--clé-et-iv-explicites)  
- [II. Chiffrement asymétrique RSA](#ii-chiffrement-asymétrique-rsa)  
  - [A. Génération des clés](#a-génération-des-clés)  
  - [B. J’envoie “Maldives” → Hicham](#b-jenvoie-maldives--hicham)  
  - [C. Agustin m’envoie un message](#c-agustin-menvoie-un-message)  
- [Conclusion](#conclusion)  

***

## 1. Objectifs

- Comprendre le **chiffrement symétrique** avec **AES-256-CBC** (une seule clé partagée).  
- Comprendre le **chiffrement asymétrique** avec **RSA** (paire clé publique / clé privée).  
- Mettre en pratique avec des **échanges en binôme** (voitures, chansons, destinations touristiques) en ligne de commande avec OpenSSL.

***

## I. Chiffrement symétrique AES

### A. Découverte avec `TESTSECRET1234567`

#### 1. Chiffrement de base

On chiffre une petite chaîne pour visualiser le fonctionnement :

```bash
echo -n "TESTSECRET1234567" | openssl enc -aes-256-cbc -salt -a -pass pass:123456789
```

- `-aes-256-cbc` : algorithme AES en 256 bits, mode CBC.  
- `-salt` : ajoute un sel de 8 octets pour que le résultat change même avec la même passphrase.  
- `-a` : encode le résultat en Base64 pour qu’il soit copiable/collable (mail, chat, etc.). 

**Sortie :** une chaîne du type `U2FsdGVkX1+...` (débute souvent par `U2FsdGVkX1` à cause du préfixe `Salted__`).

  
<img width="1280" height="363" alt="Chiffrement_Dechiffrement_TESTSECRET1234567" src="https://github.com/user-attachments/assets/f1d4d8fc-03c5-425b-af45-936be4f81dcd" />


#### 2. Voir la clé réelle et l’IV

Pour voir la **clé** et l’**IV** dérivés de la passphrase et du sel :

```bash
openssl enc -aes-256-cbc -P -pass pass:123456789
```

- `-P` : n’effectue pas le chiffrement, affiche la clé (`key`) et le vecteur d’initialisation (`iv`). 

On obtient quelque chose comme :

- Clé : `84CA006878BF3BAA...` (32 octets pour AES-256).  
- IV : `589E3710...` (16 octets pour le mode CBC).
  
#### 3. Déchiffrement

En reprenant le texte chiffré (Base64) et la même passphrase, on peut déchiffrer :

```bash
echo "U2FsdGVkX1+..." | openssl enc -d -aes-256-cbc -salt -a -pass pass:123456789
```

On récupère `TESTSECRET1234567` → démonstration du chiffrement symétrique : **même secret** pour chiffrer et déchiffrer. 

***

### B. Voitures préférées – échange binôme

Ici, le but est d’échanger sa voiture préférée avec son binôme en utilisant **la même passphrase**.

#### 1. Moi → Binôme

```bash
echo -n "Mercedes CLA ou GLE" | openssl enc -aes-256-cbc -salt -a -pass pass:PassphraseVoiture123
```

On obtient un bloc Base64 que l’on peut envoyer au binôme.

  
<img width="1229" height="138" alt="Voiture_Preferee_chiffrement" src="https://github.com/user-attachments/assets/1650fec6-18b0-44aa-8293-b589d14f0170" />


#### 2. Binôme → Moi

Le binôme fait la même chose de son côté (avec **la même passphrase** partagée oralement), puis on échange les blocs chiffrés par mail / chat.  

Pour déchiffrer :

```bash
echo "BlocBase64Reçu..." | openssl enc -d -aes-256-cbc -salt -a -pass pass:PassphraseVoiture123
```

  
<img width="1222" height="145" alt="Voiture_Preferee_Dechiffrement_Elif" src="https://github.com/user-attachments/assets/7a9e9071-4989-420c-be59-224ea45d800e" />


Résultat : on découvre la voiture du binôme, mais uniquement si on connaît la **passphrase partagée** → c’est bien du chiffrement **symétrique**.

***

### C. Chansons – clé et IV explicites

Ici, on pousse un peu plus loin : on regarde explicitement la **clé** et l’**IV** utilisés, puis on s’en sert pour chiffrer/déchiffrer des chansons / destinations.

#### 1. Génération de la clé + IV

```bash
openssl enc -aes-256-cbc -P -pass pass:123456789
```

  
<img width="791" height="285" alt="creation_key_chiffrement_Musique_Preferee2" src="https://github.com/user-attachments/assets/e6e0da39-78b0-4264-9b37-b5961f7cb758" />


On garde ces valeurs sous les yeux (clé + IV).

#### 2. Ma musique chiffrée

Avec la clé et l’IV (ou en continuant à utiliser `-pass` si le TP ne détaille pas `-K` / `-iv`), on chiffre le nom de sa musique préférée :

  
<img width="1227" height="259" alt="creation_key_chiffrement_Musique_Preferee" src="https://github.com/user-attachments/assets/7a32e09f-206d-4536-9f87-ff1472495467" />


On obtient un bloc chiffré que l’on peut envoyer.

#### 3. Musique du binôme déchiffrée

Avec la même clé/IV ou la même passphrase, on déchiffre la musique du binôme et sa destination touristique :

  
<img width="1920" height="132" alt="dechiffrement_clé_IV_Musique_Desti" src="https://github.com/user-attachments/assets/6e7edada-a719-4a1e-9326-7763187a1f9c" />


Là encore, on illustre le principe : même secret partagé = on peut lire les infos.

***

## II. Chiffrement asymétrique RSA

Passons à RSA : au lieu d’une seule clé partagée, chaque personne a **une paire** :

- Clé **privée** : à garder pour soi.  
- Clé **publique** : à distribuer aux autres.

On chiffre avec la **clé publique du destinataire** et seul le détenteur de sa **clé privée** peut déchiffrer. 

### A. Génération des clés

```bash
openssl genrsa -out ma_cle_privee.pem 2048 
openssl rsa -in ma_cle_privee.pem -pubout -out ma_cle_publique.pem
```

- `ma_cle_privee.pem` : clé privée RSA 2048 bits.  
- `ma_cle_publique.pem` : clé publique correspondante.  

  
<img width="755" height="800" alt="key_privee_publique" src="https://github.com/user-attachments/assets/c2545533-1611-435d-990f-ba58f05a1eb9" />

  
<img width="652" height="790" alt="key_privee_publique2" src="https://github.com/user-attachments/assets/f05b047a-bf30-4f48-aae3-45bb81e1471c" />


### B. J’envoie “Maldives” → Hicham

Objectif : envoyer “Maldives” à Hicham de manière confidentielle.

1. Récupérer la **clé publique de Hicham** et la stocker dans :

```bash
nano cle_publique_hicham.pem    # Coller SA clé publique
```

2. Préparer le message :

```bash
echo "Maldives" > maldives.txt
```

3. Chiffrer avec la **clé publique de Hicham** :

```bash
openssl pkeyutl -encrypt -inkey cle_publique_hicham.pem -pubin \
  -in maldives.txt -out maldives_pour_hicham.enc
```

- `-encrypt` : chiffrement.  
- `-inkey ... -pubin` : on indique qu’on utilise une **clé publique**.  
- `-in` / `-out` : fichier en clair → fichier chiffré. 

  
<img width="881" height="269" alt="fichier_chiffrer_RSA_a_envoyer_Desti" src="https://github.com/user-attachments/assets/86c55866-7eaa-42f8-808d-efe8a8719e83" />


On envoie `maldives_pour_hicham.enc` à Hicham.  
**Seule sa clé privée** lui permet de déchiffrer ce fichier.

### C. Agustin m’envoie un message

Agustin a récupéré **ma clé publique** et l’a utilisée pour chiffrer un message, qu’il m’envoie sous forme de fichier `message_pour_jiji.enc`.

Pour le déchiffrer :

```bash
openssl pkeyutl -decrypt -inkey ma_cle_privee.pem -in message_pour_jiji.enc
```

- `-decrypt` : déchiffrement.  
- `-inkey ma_cle_privee.pem` : j’utilise **ma clé privée** pour lire le message.

Résultat obtenu :

> ✅ `La Playa de los Hippies Cordoba Argentina`

  
<img width="1224" height="187" alt="fichier_Dechiffrer_RSA_recu_Agustin" src="https://github.com/user-attachments/assets/6bb72579-12fe-464e-9d86-d1d7b6e6f700" />


Ce scénario illustre bien :  
- Agustin chiffre avec **ma clé publique**,  
- je suis le seul à pouvoir déchiffrer avec **ma clé privée**.

***

## Conclusion

Dans ce TP, tu as :

- Manipulé **AES-256-CBC** en ligne de commande avec OpenSSL pour du chiffrement **symétrique** : même secret pour chiffrer et déchiffrer, pratique pour des échanges entre deux personnes qui se sont mis d’accord sur une passphrase. 
- Utilisé **RSA** avec `openssl genrsa` et `pkeyutl` pour du chiffrement **asymétrique**, où chaque personne a une paire clé publique / clé privée. 
- Mis en pratique via des exemples concrets : voitures préférées, chansons, destinations touristiques, qui rendent les concepts plus parlants que de simples “hello world”.
