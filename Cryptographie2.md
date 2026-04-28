# AES et RSA avec OpenSSL 

---

## **1. Objectifs**
- ✅ **AES** : Chiffrement symétrique (1 clé partagée)
- ✅ **RSA** : Chiffrement asymétrique (pub/priv)  
- ✅ **Échanges binôme** : Voitures, chansons, destinations touristiques

---

## **I. Chiffrement symétrique AES**

### **A. Découverte - `TESTSECRET1234567`**

#### **1. Chiffrement**
```bash
echo -n "TESTSECRET1234567" | openssl enc -aes-256-cbc -salt -a -pass pass:123456789
```
**✅ Sortie :** `U2FsdGVkX1+...` 

**Explication :** AES 256 + sel + base64 = texte **email-compatible**

<img width="1280" height="363" alt="Chiffrement_Dechiffrement_TESTSECRET1234567" src="https://github.com/user-attachments/assets/f1d4d8fc-03c5-425b-af45-936be4f81dcd" />


#### **2. Clé réelle**
```bash
openssl enc -aes-256-cbc -P -pass pass:123456789
```
**Clé :** `84CA006878BF3BAA...` (32 octets) + **IV :** `589E3710...` (16 octets)

#### **3. Déchiffrement**
**✅ `TESTSECRET1234567` récupéré !**

---

### **B. Voitures préférées - Échange binôme**

#### **Moi → Binôme**

```bash
echo -n "Mercedes CLA ou GLE" | openssl enc -aes-256-cbc -a 
```

<img width="1229" height="138" alt="Voiture_Preferee_chiffrement" src="https://github.com/user-attachments/assets/1650fec6-18b0-44aa-8293-b589d14f0170" />


#### **Binôme → Moi**
**Passphrase verbale + déchiffrement → Sa voiture découverte !**

<img width="1222" height="145" alt="Voiture_Preferee_Dechiffrement_Elif" src="https://github.com/user-attachments/assets/7a9e9071-4989-420c-be59-224ea45d800e" />

---

### **C. Chansons - Clé+IV explicites**

#### **1. Génération**
```bash
openssl enc -aes-256-cbc -P -pass pass:123456789
```
<img width="791" height="285" alt="creation_key_chiffrement_Musique_Preferee2" src="https://github.com/user-attachments/assets/e6e0da39-78b0-4264-9b37-b5961f7cb758" />

#### **2. Ma musisque chiffrement**

<img width="1227" height="259" alt="creation_key_chiffrement_Musique_Preferee" src="https://github.com/user-attachments/assets/7a32e09f-206d-4536-9f87-ff1472495467" />


#### **3. Musique Binôme déchiffrée**

<img width="1920" height="132" alt="dechiffrement_clé_IV_Musique_Desti" src="https://github.com/user-attachments/assets/6e7edada-a719-4a1e-9326-7763187a1f9c" />


---

## **II. RSA Asymétrique**

### **A. Génération clés**
```bash
openssl genrsa -out ma_cle_privee.pem 2048 
openssl rsa -in ma_cle_privee.pem -pubout -out ma_cle_publique.pem
```

<img width="755" height="800" alt="key_privee_publique" src="https://github.com/user-attachments/assets/c2545533-1611-435d-990f-ba58f05a1eb9" />

<img width="652" height="790" alt="key_privee_publique2" src="https://github.com/user-attachments/assets/bc469ae8-16ef-4497-ac55-c37655602343" />


### **B. J'envoie "Maldives" → Hicham**
```bash
nano cle_publique_hicham.pem    # Colle SA clé publique
echo "Maldives" > maldives.txt
openssl pkeyutl -encrypt -inkey cle_publique_hicham.pem -pubin -in tuvaskiffer.txt -out tuvaskiffer_pour_hicham.enc
```

<img width="881" height="269" alt="fichier_chiffrer_RSA_a_envoyer_Desti" src="https://github.com/user-attachments/assets/86c55866-7eaa-42f8-808d-efe8a8719e83" />


### **C. Agustin m'envoie "message_pur_jimmy"**
```bash
openssl pkeyutl -decrypt -inkey ma_cle_privee.pem -in message_pour_jiji.enc
```
**✅ "La Playa de los Hippies Cordoba Argentina"**

<img width="1224" height="187" alt="fichier_Dechiffrer_RSA_recu_Agustin" src="https://github.com/user-attachments/assets/6bb72579-12fe-464e-9d86-d1d7b6e6f700" />


