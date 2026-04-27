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
  https://gchq.github.io/CyberChef/  

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

   ![Chiffrement César ROT13](https://github.com/user-attachments/assets/eb7acf0e-7884-460c-9a82-b5e5ef124624)



2. Chiffrer le nom de votre film préféré avec une *Box Height* de votre choix  
   - Transmettre le texte chiffré à votre binôme sans lui communiquer la clé  
   - Au sein de votre binôme, essayer de retrouver le message en sens inverse  

***

### II. Partie 2 : Vigenère

Dans **CyberChef**, utilisez la recette **Vigenère Encode**.

- Encodez le nom de votre plat préféré avec la clé **KEY**  
  - Quel est le texte chiffré ?  

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

- Que constatez-vous si vous modifiez 1 caractère du texte initial ?  

- Déchiffrez le texte AES chiffré précédemment en adaptant les paramètres  
  - Vous devez retrouver le texte d'origine  

#### Transmission

- Générer une clé adéquate  
- Chiffrez le nom de votre équipe de sport préférée avec les paramètres suivants :  
  - Key : votre clé générée  
  - IV : (vide)  
  - Mode : ECB  
  - Input : Raw  
  - Output : Hex  

- Transmettre le texte chiffré à votre binôme  
- Transmettre la clé par un autre canal  
  - Déchiffrez les messages pour découvrir vos équipes préférées  

***

### IV. Partie 4 : RSA

Dans **CyberChef**, utilisez :  
**Generate RSA Key Pair**, **RSA Encrypt**, **RSA Decrypt**

#### Génération

- Générer une paire de clés RSA (1024 bits)  
  - Que contiennent les clés générées ? (format, longueur…)  

#### Découverte

- Chiffrez le message :  
  **LE MESSAGE EST SECRETSIMPLE**  
  avec votre clé publique  
  - Quelle est la sortie chiffrée ?  

- Déchiffrez avec votre clé privée  
  - Le message est-il identique ?  

#### Transmission

- Récupérer la clé publique de votre binôme  
- Chiffrer votre réplique préférée avec :  
  - Key : clé publique du binôme  
  - Scheme : RSA-OAEP  
  - Digest : SHA-1  

- Envoyer le message chiffré  
  - Votre binôme doit le déchiffrer avec sa clé privée  

- Inversez les rôles  

***

### V. Partie 5 : Hachage

- Appliquer les algorithmes suivants sur `ADMIN123` :  
  - SHA-1  
  - SHA-2 : 256, 512  
  - SHA-3 : 256, 512  

- Questions :  
  - Quelles sont les tailles des hashs ?  
  - Peut-on retrouver le mot de passe à partir du hash ?  

- Tester avec `TEST` et `TESt`  
  - Que constatez-vous ?  

- Hacher `hello` en SHA1 (80 rounds)  
  - Cracker sur : https://crackstation.net/  
  - Pourquoi est-ce rapide ?  

- Répéter avec SHA1 (50 rounds)  
  - Le hash est-il cracké ? Pourquoi ?  

***

### VI. Partie 6 : Encodage

- Encoder `Bonjour` en Base64  
  - Que représente le résultat ?  

- Décoder le résultat  
  - Peut-on confondre encodage et chiffrement ? Pourquoi ?  

***

### VII. Bonus

- Le diaporama contient un message caché  
  - Essayez de le découvrir  

**Indice :** plusieurs techniques vues dans ce TP ont été utilisées  

***


