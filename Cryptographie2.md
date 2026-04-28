# AES et RSA avec OpenSSL

***

## 1. Objectifs

- Manipuler les algorithmes AES et RSA avec OpenSSL.
- Visualiser le fonctionnement de la cryptographie symétrique et asymétrique.

## 2. Consignes

- Travail en binôme.
- Utilisation d’OpenSSL sur une machine Linux.
- Pour chaque section, vous devrez :
  - Réaliser les opérations demandées.
  - Appliquer les méthodes sur un message de votre choix puis transmettre ce message à votre binôme afin qu’il le décode / déchiffre.

## 3. Contenu du TP

1. Chiffrement symétrique AES.
2. Chiffrement asymétrique RSA.

**ASR / AIS / M1 SI**  
**CRYPTOGRAPHIE**  
**Numéro : 2**

## 4. Tâches à réaliser

### I. Chiffrement symétrique AES

#### A. Découverte

- Chiffrez la chaîne `TESTSECRET1234567` avec les paramètres suivants :
  - Mode de chiffrement AES 256 bits en CBC.
  - Sortie en base64.
  - Ajouter un sel (`salt`) pour sécuriser la dérivation de clé.
  - Fournir une passphrase pour dériver la clé (123456789).

- Quelle est la clé réelle utilisée et comment est-elle générée ?

- Déchiffrez le texte AES chiffré précédemment en adaptant les paramètres :
  - Vous devez retrouver le texte d’origine.

<img width="1280" height="363" alt="Chiffrement_Dechiffrement_TESTSECRET1234567" src="https://github.com/user-attachments/assets/da69e17a-67af-490f-9fc2-5112b3177149" />


#### B. Transmission d’un message chiffré à votre binôme

**Passphrase**

- Chiffrez le nom de votre voiture préférée avec les paramètres suivants :
  - Mode de chiffrement AES 256 bits en CBC.
  - Sortie en base64.
  - Ne pas ajouter de sel (`salt`) pour sécuriser la dérivation de clé.
  - Fournir une passphrase pour dériver la clé.

- Transmettre le texte chiffré à votre binôme.
- Transmettre la passphrase à votre binôme par un autre canal.
- Au sein de votre binôme, déchiffrez le message pour découvrir vos voitures préférées respectives.

**Transmission d’un message chiffré à votre binôme**

- Générez une clé de chiffrement et un vecteur d’initialisation (`IV`) à partir d’une passphrase sans sel.
- Notez les valeurs renvoyées.
- Chiffrez le nom de votre chanson préférée en utilisant la clé et l’IV générés précédemment.
- Transmettre le texte chiffré à votre binôme.
- Transmettre la clé et l’IV à votre binôme par un autre canal.
- Au sein de votre binôme, déchiffrez le message pour découvrir votre chanson préférée respective.

### II. RSA

En vous inspirant de l’exercice AES, réalisez l’équivalent avec RSA :

#### A. Génération de la paire de clés RSA

- Générez une paire de clés RSA de 2048 bits.

#### B. Chiffrement d’un message

- Écrivez un court message, par exemple le nom de votre destination de tourisme préférée.
- Chiffrez-le avec la clé publique de votre binôme.

#### C. Échange et déchiffrement

- Transmettez le fichier chiffré `message_rsa.bin` à votre binôme.
- Votre binôme doit déchiffrer le message avec sa clé privée.

### III. Bonus

- Utilisez le chiffrement hybride pour transmettre à votre binôme les paroles de votre chanson préférée.
