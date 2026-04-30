# TP1 – Cryptographie appliquée avec CyberChef

## Sommaire
- [1. Objectifs](#1-objectifs)  
- [2. Consignes](#2-consignes)  
- [3. Contenu du TP](#3-contenu-du-tp)  
- [4. Tâches à réaliser](#4-tâches-à-réaliser)  
  - [I. Partie 1 : Chiffrement de César](#i-partie-1--chiffrement-de-césar)  
  - [II. Partie 2 : Vigenère](#ii-partie-2--vigenère)  
  - [III. Partie 3 : Chiffrement symétrique AES](#iii-partie-3--chiffrement-symétrique-aes)  
  - [IV. Partie 4 : RSA](#iv-partie-4--rsa)  
  - [V. Partie 5 : Hachage](#v-partie-5--hachage)  
  - [VI. Partie 6 : Encodage](#vi-partie-6--encodage)  
  - [VII. Bonus](#vii-bonus)  

***
## 1. Objectifs
- Utiliser CyberChef pour appliquer différentes techniques de chiffrement, hachage et encodage. 
- Visualiser le fonctionnement de la cryptographie symétrique et asymétrique.  
- Comprendre les différences entre **encodage**, **hachage** et **chiffrement** (réversibilité, usages, objectifs).  

***
## 2. Consignes
- Travail en **binôme**.  
- Utilisation de l’application web **CyberChef** :  
 

Dans CyberChef, une “recette” est simplement une **suite d’opérations** (algorithmes, encodages, transformations) appliquées à une entrée. 
Pour chaque section :

- Réaliser les opérations demandées dans CyberChef.  
- Appliquer les méthodes sur un message de votre choix, puis transmettre ce message à votre binôme pour qu’il le décode/déchiffre.  
- Répondre aux questions pour faire le lien avec les concepts vus en cours.  

***
## 3. Contenu du TP
1. Chiffrement de César (ROT)  
2. Chiffrement de Vigenère  
3. Chiffrement symétrique AES  
4. Chiffrement asymétrique RSA  
5. Hachage (SHA‑1 / SHA‑2 / SHA‑3)  
6. Encodage (Base64)  

***
## 4. Tâches à réaliser
### I. Partie 1 – Chiffrement de César
Dans **CyberChef**, utilisez la recette **ROT13**.

1. Avec une *Rotation* (box height) de 13, chiffrer la phrase suivante :  
   **RENDEZ-VOUS À MIDI**  

   - Quel est le texte chiffré ?  
   - Déchiffrez ce texte pour vérifier le résultat.

  
<img width="901" height="539" alt="Chiffrement_Cesar_ROT13" src="https://github.com/user-attachments/assets/7fe3dff4-bce8-45bf-b642-6f8d938c9698" />


2. Chiffrer le nom de votre film préféré avec une rotation de votre choix.  

   - Transmettre le texte chiffré à votre binôme sans lui communiquer la clé.  
   - Au sein de votre binôme, essayer de retrouver le message en sens inverse (brute force ou en devinant la rotation).  

  
<img width="794" height="597" alt="film_prefere_ROT47" src="https://github.com/user-attachments/assets/aba14bfd-df52-4d25-8588-d54f92436226" />


> Idée clé : César / ROT sont des **chiffrements très simples**, facilement cassables par essai/erreur ou analyse de fréquence. On s’en sert ici pour illustrer le principe de base du chiffrement, pas pour faire de la sécurité réelle.

***
### II. Partie 2 – Vigenère
Dans **CyberChef**, utilisez la recette **Vigenère Encode**.

- Chiffrer le nom de votre plat préféré avec la clé **KEY**.  
  - Quel est le texte chiffré ?  

  
<img width="867" height="667" alt="Plat_prefere_Vigenere" src="https://github.com/user-attachments/assets/cc818537-e5ec-4b6d-a382-3fd3d9791f36" />


- Transmettez le texte chiffré à votre binôme.  
- Transmettez la clé à votre binôme par un **autre canal** (oralement, message séparé, etc.).  
  - Au sein de votre binôme, déchiffrez le message pour découvrir vos plats préférés respectifs.  

> Idée clé : Vigenère est un **chiffrement polyalphabétique** (plus solide que César, mais cassé depuis longtemps). On introduit la notion de **clé partagée** et de **canal sécurisé** pour la transmettre.

***
### III. Partie 3 – Chiffrement symétrique AES
Dans **CyberChef**, utilisez les recettes **AES Encrypt** et **AES Decrypt**. 

#### Découverte

- Chiffrez la chaîne `TESTSECRET1234567` avec les paramètres suivants :  
  - Key : `c34fa73d7c5f8901a23e4cd98e7f650d9a17d4e8f902fa0d3286d0beaad219b6`  
  - IV : (vide)  
  - Mode : `ECB`  
  - Input : `Raw`  
  - Output : `Hex`  

  
<img width="1113" height="626" alt="Chiffrement_AES_TESTSECRET" src="https://github.com/user-attachments/assets/b8eb6778-e692-48eb-957b-18a870ba1402" />


- Modifiez 1 caractère du texte initial (par exemple `TESTSECRET1234567` → `TESTSECRET2234567`).  
  - Que constatez-vous sur la sortie chiffrée ?  

> Idée clé : même une **petite modification** du texte clair entraîne un changement **massif** du texte chiffré → c’est l’**avalanche effect**, une propriété recherchée en cryptographie moderne. 

- Déchiffrez le texte AES chiffré précédemment en adaptant les paramètres (même clé, même mode) :
  - Vous devez retrouver le texte d’origine.

  
<img width="1524" height="574" alt="Dechiffrement_AES_TESTSECRET" src="https://github.com/user-attachments/assets/d6c21aee-026c-48e3-bf4d-3f114d8ed2e3" />


> Idée clé : en chiffrement **symétrique**, la même clé (ici la key hex) sert à **chiffrer et déchiffrer**. 

#### Transmission d’un message chiffré à votre binôme

- Générez une clé adéquate (clé aléatoire de bonne longueur pour AES).  
- Chiffrez le nom de votre équipe de sport préférée avec les paramètres suivants :  
  - Key : votre clé générée.  
  - IV : (vide)  
  - Mode : ECB.  
  - Input : Raw.  
  - Output : Hex.  

  
<img width="980" height="672" alt="Equipe_preferee_AES" src="https://github.com/user-attachments/assets/f2195aa3-90ad-4344-ac0a-f367ca1a66df" />


- Transmettez le texte chiffré à votre binôme.  
- Transmettez la clé à votre binôme par un autre canal.  
  - Au sein de votre binôme, déchiffrez le message pour découvrir vos équipes de sport préférées respectives.  

> Note : dans un contexte réel, on évite ECB et on utilise un mode comme **CBC** ou **GCM**, mais pour un TP de découverte, ECB permet de simplifier les paramètres.

***
### IV. Partie 4 – RSA
Dans **CyberChef**, utilisez les recettes **Generate RSA Key Pair**, **RSA Encrypt** et **RSA Decrypt**. 

#### Génération d’une paire de clés RSA

- Utilisez **Generate RSA Key Pair** avec une taille de **1024 bits**.  
  - Que contiennent les clés générées ? (formats, longueur…)  

  
<img width="671" height="478" alt="Clé_RSA" src="https://github.com/user-attachments/assets/9e3d6b01-ac2d-403b-b70a-5385194012bb" />


> Idée clé :  
> - La **clé publique** peut être distribuée à tout le monde.  
> - La **clé privée** doit rester secrète.  
> RSA sert à chiffrer des petits messages ou des clés symétriques, pas des gros fichiers en direct.

#### Découverte

- Chiffrez le message suivant avec votre clé publique :  
  **LE MESSAGE EST SECRETSIMPLE**  
  - Quelle est la sortie chiffrée ?

  
<img width="1515" height="551" alt="RSA_chiffrement" src="https://github.com/user-attachments/assets/436c64ef-c8c8-48ec-bf30-bd4ba9d4d601" />


- Convertissez le résultat en Base64 (recette **To Base64**) pour obtenir un texte facilement partageable (mail / chat).

  
<img width="1556" height="656" alt="RSA_chiffrement_pas_oublie_Tobase64 (2)" src="https://github.com/user-attachments/assets/60c4a883-056e-4f00-a019-d636e565fbe4" />


- Utilisez votre clé privée pour déchiffrer le message chiffré.  
  - La sortie est-elle identique au message d’origine ?

  
<img width="1537" height="654" alt="dechiffrement_RSA" src="https://github.com/user-attachments/assets/0cbb27d2-6ebb-44fb-8ff9-172dba3ca22c" />


> Idée clé : on chiffre avec la **clé publique** du destinataire, et **seule sa clé privée** permet de lire le message. C’est le principe de la cryptographie **asymétrique**.

#### Transmission d’un message chiffré à votre binôme

- Récupérez la **clé publique** de votre binôme.  
- Chiffrez votre réplique (phrase) préférée avec les paramètres suivants :  
  - Key : clé publique de votre binôme.  
  - Encryption scheme : `RSA‑OAEP`.  
  - Message Digest Algorithm : `SHA‑1`.  

- Transmettez le texte chiffré à votre binôme.  
  - Votre binôme doit déchiffrer le message à l’aide de sa clé privée pour découvrir votre réplique préférée.  

  
<img width="1522" height="563" alt="Capture d&#39;écran 2026-04-27 160308" src="https://github.com/user-attachments/assets/a5145b7b-21b6-4169-8fb8-e2d33874486c" />


- Inversez ensuite les rôles pour que chacun connaisse la réplique préférée de son binôme.

  
<img width="1537" height="603" alt="Message_recu_chiffre_dechiffrement" src="https://github.com/user-attachments/assets/c6808140-6773-4e36-aa43-740d002c2eb3" />


***
### V. Partie 5 – Hachage
- Utilisez différents algorithmes de hachage sur la chaîne **ADMIN123** :  

  - SHA‑1  

  
<img width="923" height="538" alt="Hash_Sha1" src="https://github.com/user-attachments/assets/7ca5e373-e3aa-4924-acc5-387790caaadf" />


  - SHA‑2 : 256, 512  

  
<img width="1157" height="537" alt="Hash_Sha2_256" src="https://github.com/user-attachments/assets/5f4794a7-dd4a-4089-a114-b8ab2bdcb0b8" />

  
<img width="1539" height="560" alt="Hash_Sha2_512" src="https://github.com/user-attachments/assets/16982bae-5865-499d-94dc-f58d924d5f2c" />


  - SHA‑3 : 256, 512  

  
<img width="1076" height="487" alt="Hash_Sha3_256" src="https://github.com/user-attachments/assets/24e0f32c-293c-463e-8e73-af2db3d848a5" />

  
<img width="1529" height="588" alt="Hash_Sha3_512" src="https://github.com/user-attachments/assets/444c9392-505c-4d3e-9242-6d5c0fbb74b2" />


- Questions :  
  - Quelles sont les **tailles** des hashs produits (en bits / en caractères hex) ?  
  - Est‑il possible de retrouver le mot de passe à partir du hash ?  

- Essayez avec deux textes légèrement différents : **TEST** et **TESt**.  
  - Que constatez-vous dans les résultats des hashs ?

  
<img width="1520" height="518" alt="Hash_TEST" src="https://github.com/user-attachments/assets/20573fe9-f6eb-464a-8414-da941a545b5a" />

  
<img width="1522" height="582" alt="Hash_TESt2" src="https://github.com/user-attachments/assets/9d42f193-88eb-4702-b5d7-71fa480ccc08" />


> Idée clé : un **hachage cryptographique** est :  
> - unidirectionnel (on ne “déchiffre” pas un hash),  
> - très sensible aux petites modifications de l’entrée (avalanche effect également),  
> - utilisé pour stocker des mots de passe, vérifier l’intégrité, etc.

- Hachez le texte **`hello`** en SHA‑1 (80 rounds).  
  - Crackez le hash sur [https://crackstation.net/](https://crackstation.net/).  
    - Le hash est cracké en quelques secondes, comment est‑ce possible ?

  
<img width="908" height="492" alt="Hash_hello" src="https://github.com/user-attachments/assets/620f44f0-3e84-444b-a9a8-f9d34e4c2dc1" />

  
<img width="1451" height="503" alt="crack_hash_hello" src="https://github.com/user-attachments/assets/618886b6-4fe6-47c4-b06a-af18e234c4da" />


> CrackStation utilise de **gigantesques tables pré‑calculées** (wordlists + tables arc‑en‑ciel) qui contiennent déjà les hashs de millions de mots de passe courants. 
> Le service ne “inverse” pas le hash, il cherche simplement s’il existe dans sa base, ce qui est très rapide pour des mots de passe simples.

- Répétez avec SHA‑1 (50 rounds).  
  - Le hash est‑il cracké ? Pourquoi ?

  
<img width="991" height="616" alt="hash_hello_50" src="https://github.com/user-attachments/assets/0b6ad901-c623-4f6b-9ed3-d84efcbe1fda" />

  
<img width="1079" height="393" alt="crack_hash_hello_50_KO" src="https://github.com/user-attachments/assets/03c7617e-d7eb-41a1-9c59-4e15a7944995" />


> Idée clé : plus on **ralentit** le hachage (rounds, dérivation de clé type PBKDF2/BCrypt/Argon2), plus c’est difficile de pré‑calculer des tables ou de tester des milliards de mots de passe par seconde.

***

### VI. Partie 6 – Encodage
- Encodez le mot **`Bonjour`** en Base64.  

  - Que représente le résultat ?

  
<img width="787" height="549" alt="encodage_Bonjour_To_Base64" src="https://github.com/user-attachments/assets/24ea1031-a7cc-4f2d-a704-6fdb0d468c79" />


- Décodez le résultat obtenu précédemment (recette **From Base64**).  
  - Peut‑on confondre **encodage** et **chiffrement** ? Pourquoi ?

  
<img width="814" height="538" alt="Decodage_Bonjour_depuis_Hash_From_Base64" src="https://github.com/user-attachments/assets/ed26d7ec-2749-440e-b0d7-a12dd81076af" />


> Idée clé :  
> - L’**encodage** (Base64, hex, etc.) est **réversible** sans secret.  
> - Il sert à transporter des données binaires sous forme de texte, pas à les protéger.  
> - Le **chiffrement**, lui, nécessite une clé et vise la **confidentialité**.

***
### VII. Bonus
- Le diaporama associé au TP contient un message caché : tentez de le découvrir !  
- **Indice :** plusieurs opérations utilisées dans ce TP (hachage, Base64, chiffrement…) ont été enchaînées pour le cacher.

  
<img width="1640" height="66" alt="flag_doc_Yanis_Crypto" src="https://github.com/user-attachments/assets/2c82cb81-d9b7-4255-825e-0f1c012ff28b" />

  
<img width="357" height="147" alt="flag_doc_Yanis_Crypto2" src="https://github.com/user-attachments/assets/499a02ff-4a84-4178-836e-bc62ea92d814" />

  
<img width="1543" height="620" alt="flag_doc_Yanis_Crypto3" src="https://github.com/user-attachments/assets/fb218c7d-fe07-4a6b-b598-12ab6c127cb9" />


> Objectif bonus : montrer qu’en combinant plusieurs **recettes CyberChef**, on peut faire de la petite stéganographie / des challenges style CTF, en réutilisant les briques vues tout au long du TP.

