# Cryptographie appliquée — CyberChef  

***

## 1. Objectifs

- Utiliser CyberChef pour appliquer différentes techniques de chiffrement, hachage et encodage  
- Visualiser le fonctionnement de la cryptographie symétrique et asymétrique  
- Comprendre les différences entre encodage, hachage et chiffrement  

***

## 2. Consignes

- Travail en binôme  
- Utilisation de l’application web **CyberChef** :  
  [https://gchq.github.io/CyberChef/](https://gchq.github.io/CyberChef/)  

### Pour chaque section :

- Réaliser les opérations demandées dans CyberChef  
- Appliquer les méthodes sur un message de votre choix puis transmettre ce message à votre binôme afin qu’il le décode/déchiffre  
- Répondre aux questions  

***

## 3. Contenu du TP

1. Chiffrement de César  
2. Encodage de Vigenère  
3. Chiffrement symétrique AES  
4. Chiffrement asymétrique RSA  
5. Hachage  
6. Encodage  

***

## 4. Tâches à réaliser

### I. Partie 1 : Chiffrement de César

Dans **CyberChef**, utilisez la recette **ROT13**.

1. Avec une *Box Height* de 13, chiffrer la phrase suivante :  
   **RENDEZ-VOUS À MIDI**  
   - Quel est le texte chiffré ?  
   - Déchiffrez ce texte pour vérifier le résultat

<img width="901" height="539" alt="Chiffrement_Cesar_ROT13" src="https://github.com/user-attachments/assets/7fe3dff4-bce8-45bf-b642-6f8d938c9698" />


2. Chiffrer le nom de votre film préféré avec une *Box Height* de votre choix  
   - Transmettre le texte chiffré à votre binôme sans lui communiquer la clé  
   - Au sein de votre binôme, essayer de retrouver le message en sens inverse  

<img width="794" height="597" alt="film_prefere_ROT47" src="https://github.com/user-attachments/assets/aba14bfd-df52-4d25-8588-d54f92436226" />

***

### II. Partie 2 : Vigenère

Dans **CyberChef**, utilisez la recette **Vigenère Encode**.

- Encodez le nom de votre plat préféré avec la clé **KEY**  
  - Quel est le texte chiffré ?  

<img width="867" height="667" alt="Plat_prefere_Vigenere" src="https://github.com/user-attachments/assets/cc818537-e5ec-4b6d-a382-3fd3d9791f36" />

- Transmettre le texte chiffré à votre binôme  
- Transmettre la clé à votre binôme par un autre canal  
  - Au sein de votre binôme, déchiffrez le message pour découvrir vos plats préférés respectifs  

***

### III. Partie 3 : Chiffrement symétrique AES

Dans **CyberChef**, utilisez les recettes **AES Encrypt** et **AES Decrypt**.

#### Découverte

- Chiffrez la chaîne `TESTSECRET1234567` avec les paramètres suivants :  
  - Key : `c34fa73d7c5f8901a23e4cd98e7f650d9a17d4e8f902fa0d3286d0beaad219b6`  
  - IV : (vide)  
  - Mode : ECB  
  - Input : Raw  
  - Output : Hex  

<img width="1113" height="626" alt="Chiffrement_AES_TESTSECRET" src="https://github.com/user-attachments/assets/b8eb6778-e692-48eb-957b-18a870ba1402" />


- Que constatez-vous si vous modifiez 1 caractère du texte initial ? 


- Déchiffrez le texte AES chiffré précédemment en adaptant les paramètres  
  - Vous devez retrouver le texte d'origine

<img width="1524" height="574" alt="Dechiffrement_AES_TESTSECRET" src="https://github.com/user-attachments/assets/d6c21aee-026c-48e3-bf4d-3f114d8ed2e3" />

    
#### Transmission d’un message chiffré à votre binôme

- Générer une clé adéquate  
- Chiffrez le nom de votre équipe de sport préférée avec les paramètres suivants :  
  - Key : « la clé que vous avez généreé »  
  - IV : (vide)  
  - Mode : ECB  
  - Input : Raw  
  - Output : Hex  

<img width="980" height="672" alt="Equipe_preferee_AES" src="https://github.com/user-attachments/assets/f2195aa3-90ad-4344-ac0a-f367ca1a66df" />


- Transmettre le texte chiffré à votre binôme  
- Transmettre la clé à votre binôme par un autre canal  
  - Au sein de votre binôme, déchiffrez le message pour découvrir vos équipes de sport préférées respectives  

***

### IV. Partie 4 : RSA

Dans **CyberChef**, utilisez les recettes **Generate RSA Key Pair**, **RSA Encrypt** et **RSA Decrypt**.

#### Génération d’une paire de clés RSA

- Utilisez **Generate RSA Key Pair** avec une taille de 1024 bits  
  - Que contiennent les clés générées ? (formats, longueur…)  

<img width="671" height="478" alt="Clé_RSA" src="https://github.com/user-attachments/assets/9e3d6b01-ac2d-403b-b70a-5385194012bb" />


#### Découverte

- Chiffrez le message suivant avec votre clé publique :  
  **LE MESSAGE EST SECRETSIMPLE**  
  - Quelle est la sortie chiffrée ?  

<img width="1515" height="551" alt="RSA_chiffrement" src="https://github.com/user-attachments/assets/436c64ef-c8c8-48ec-bf30-bd4ba9d4d601" />

puis ToBase64

<img width="1556" height="656" alt="RSA_chiffrement_pas_oublie_Tobase64 (2)" src="https://github.com/user-attachments/assets/60c4a883-056e-4f00-a019-d636e565fbe4" />



- Utilisez votre clé privée pour déchiffrer le message  
  - La sortie est-elle identique au message d’origine ?

Utilsation de TOBase64

<img width="1537" height="654" alt="dechiffrement_RSA" src="https://github.com/user-attachments/assets/0cbb27d2-6ebb-44fb-8ff9-172dba3ca22c" />


#### Transmission d’un message chiffré à votre binôme

- Récupérez la clé publique de votre binôme  
- Chiffrez votre réplique préférée avec les paramètres suivants :  
  - Key : « la clé publique de votre binôme »  
  - Encryption scheme : RSA‑OAEP  
  - Message Digest Algorithm : SHA‑1  

- Transmettre le texte chiffré à votre binôme  
  - Votre binôme doit déchiffrer le message à l’aide de sa clé privée pour découvrir votre réplique préférée
 
<img width="1522" height="563" alt="Capture d&#39;écran 2026-04-27 160308" src="https://github.com/user-attachments/assets/a5145b7b-21b6-4169-8fb8-e2d33874486c" />

  - Inversez ensuite les rôles pour que chacun connaisse la réplique préférée de son binôme

<img width="1537" height="603" alt="Message_recu_chiffre_dechiffrement" src="https://github.com/user-attachments/assets/c6808140-6773-4e36-aa43-740d002c2eb3" />

***

### V. Partie 5 : Hachage

- Utilisez différents algorithmes de hachage sur la chaîne **ADMIN123** :  
  - SHA‑1
 
<img width="923" height="538" alt="Hash_Sha1" src="https://github.com/user-attachments/assets/7ca5e373-e3aa-4924-acc5-387790caaadf" />

  - SHA‑2 : 256, 512

<img width="1157" height="537" alt="Hash_Sha2_256" src="https://github.com/user-attachments/assets/5f4794a7-dd4a-4089-a114-b8ab2bdcb0b8" />

<img width="1539" height="560" alt="Hash_Sha2_512" src="https://github.com/user-attachments/assets/16982bae-5865-499d-94dc-f58d924d5f2c" />

 
  - SHA‑3 : 256, 512

<img width="1076" height="487" alt="Hash_Sha3_256" src="https://github.com/user-attachments/assets/24e0f32c-293c-463e-8e73-af2db3d848a5" />


<img width="1529" height="588" alt="Hash_Sha3_512" src="https://github.com/user-attachments/assets/444c9392-505c-4d3e-9242-6d5c0fbb74b2" />

- Quelles sont les tailles des hashs produits ?  
  - Est‑il possible de retrouver le mot de passe à partir du hash ?  

- Essayez avec deux textes légèrement différents : **TEST** et **TESt**  
  - Que constatez-vous dans les résultats des hashs ?

<img width="1520" height="518" alt="Hash_TEST" src="https://github.com/user-attachments/assets/20573fe9-f6eb-464a-8414-da941a545b5a" />

<img width="1522" height="582" alt="Hash_TESt2" src="https://github.com/user-attachments/assets/9d42f193-88eb-4702-b5d7-71fa480ccc08" />


- Hacher le texte **`hello`** en SHA1 (80 rounds)
  - Crackez le hash sur [https://crackstation.net/](https://crackstation.net/)  
    - Le hash est cracké en quelques secondes, comment est‑ce possible ?  

<img width="908" height="492" alt="Hash_hello" src="https://github.com/user-attachments/assets/620f44f0-3e84-444b-a9a8-f9d34e4c2dc1" />

<img width="1451" height="503" alt="crack_hash_hello" src="https://github.com/user-attachments/assets/618886b6-4fe6-47c4-b06a-af18e234c4da" />



- Répéter avec SHA1 (50 rounds)  
  - Le hash est‑il cracké ? Pourquoi ?  

<img width="991" height="616" alt="hash_hello_50" src="https://github.com/user-attachments/assets/0b6ad901-c623-4f6b-9ed3-d84efcbe1fda" />

<img width="1079" height="393" alt="crack_hash_hello_50_KO" src="https://github.com/user-attachments/assets/03c7617e-d7eb-41a1-9c59-4e15a7944995" />

***

### VI. Partie 6 : Encodage

- Encodez le mot **`Bonjour`** en Base64  
  - Que représente le résultat ?  

<img width="787" height="549" alt="encodage_Bonjour_To_Base64" src="https://github.com/user-attachments/assets/24ea1031-a7cc-4f2d-a704-6fdb0d468c79" />

- Décoder le résultat obtenu précédemment  
  - Peut‑on confondre encodage et chiffrement ? Pourquoi ?  

<img width="814" height="538" alt="Decodage_Bonjour_depuis_Hash_From_Base64" src="https://github.com/user-attachments/assets/ed26d7ec-2749-440e-b0d7-a12dd81076af" />


***

### VII. Bonus

- Le diaporama contient un message caché, tentez de le découvrir !  
- **Indice :** plusieurs opérations utilisées dans le cadre de ce TP ont été utilisées pour cacher ce message…

<img width="1640" height="66" alt="flag_doc_Yanis_Crypto" src="https://github.com/user-attachments/assets/2c82cb81-d9b7-4255-825e-0f1c012ff28b" />


***

