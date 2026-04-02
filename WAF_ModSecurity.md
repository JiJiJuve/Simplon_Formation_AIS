# 🛡️ Déploiement d'un WAF ModSecurity de niveau production avec Apache sur Ubuntu 24.04

## Le guide de laboratoire définitif pour le déploiement de ModSecurity et de l'OWASP CRS**

Ce guide fournit une méthodologie complète et éprouvée pour la mise en place d'un pare-feu applicatif web (**WAF**) robuste basé sur **ModSecurity** et l'**OWASP Core Rule Set (CRS)** sur un serveur web Apache fonctionnant sous Ubuntu 24.04. Chaque étape est conçue pour construire une couche de défense sécurisée, maintenable et prête pour la production.


## Pré-requis / contexte

- Distribution : Debian / Ubuntu. 
- Service : Apache 2. 

**But :** partir d’un serveur qui fonctionne déjà, puis le sécuriser.

## Étape 1 – Mettre à jour le système + installation du service Apache2

**But :** ne pas travailler sur une version vulnérable, appliquer les patchs de sécurité.
1. **Mettre l’index des paquets à jour et à appliquer toutes les mises à jour :**


   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

<img width="1920" height="421" alt="MàJ_apache" src="https://github.com/user-attachments/assets/69a40883-971b-4ef0-8455-bf751c798a78" />

💡 Bonne pratique : Pour adhérer au principe du moindre privilège, toutes les opérations suivantes doivent être effectuées par un utilisateur non-root disposant de privilèges sudo.

2.**Installation service Apache2**

```bash
sudo apt install apache2 -y
```

<img width="1920" height="407" alt="installation_service_apache2" src="https://github.com/user-attachments/assets/0c82b5d2-d111-4a39-9590-234b0f31e583" />

Une fois l'installation terminée, vérifiez la version et l'état du service.

* **Vérifier la version d'Apache**


    ```bash
    apache2 -v
    ```
    
* **Vérifier l'état du service Apache**

    ```bash
    sudo systemctl status apache2
    ```

* **Configurez Apache pour qu'il démarre automatiquement au démarrage du système.**

   ```bash
   sudo systemctl enable apache2
   ```
   
<img width="958" height="584" alt="verif_version_etat_redemarrage_auto_apache2_" src="https://github.com/user-attachments/assets/77b43a4e-3118-4b6f-9a24-1161286540aa" />


3.**Mise en place du pare-feu UFW (host firewall)**

**Objectif :** limiter la surface d’attaque réseau du serveur en n’autorisant que les ports nécessaires (SSH + HTTP/HTTPS).

Commandes :

* **Lister les profils applicatifs disponibles (dont Apache, OpenSSH, etc.)**
```bash
sudo ufw app list
```

* **Autoriser le trafic web (HTTP + HTTPS)**
```
sudo ufw allow 'Apache Full'
```
<img width="958" height="225" alt="liste_profils_applic" src="https://github.com/user-attachments/assets/5f964bb1-e497-4fe6-ad06-f90acc5e7447" />


* **Installation OpennSSH** 
```
sudo apt install openssh-server -y
```

<img width="950" height="334" alt="installation_openssh_server" src="https://github.com/user-attachments/assets/bb40342f-3ace-4d94-82e4-3e26c547115c" />


* **Autoriser l’accès SSH pour administrer le serveur à distance**
```
sudo ufw allow 'OpenSSH'
```

<img width="958" height="262" alt="verif_liste_app_openssh_server_autori_parefeu_ufw" src="https://github.com/user-attachments/assets/034312d9-48fd-43b3-8447-801adbc1860e" />


* **Activer UFW**
```
sudo ufw enable
```

* **Vérifier les règles actives**
```
sudo ufw status
```

<img width="946" height="298" alt="activation_verif_status_ufw" src="https://github.com/user-attachments/assets/09c0a34f-529f-46b5-8c58-f479adddffa9" />


**Résultat attendu :**

- Les ports 22 (SSH), 80 (HTTP) et 443 (HTTPS) sont en `ALLOW`.
- Tous les autres ports entrants sont bloqués par défaut.

**Rôle dans l’architecture :**

- UFW agit comme un **pare-feu local** sur le serveur (niveau réseau).
- Combiné à Apache + ModSecurity (niveau applicatif), cela ajoute une couche de défense supplémentaire.

## Étape 2 – Installation et configuration du moteur ModSecurity

## Qu’est-ce que ModSecurity ?

* ModSecurity est un **pare-feu applicatif Web (WAF)** qui s’intègre à Apache.

Concrètement :
- Apache sert les pages web (PHP, HTML, etc.).
- ModSecurity analyse chaque requête HTTP/HTTPS qui arrive sur Apache.
- Avec un jeu de règles (OWASP Core Rule Set), il peut **détecter et bloquer** des attaques comme :
  - injection SQL (SQLi),
  - Cross-Site Scripting (XSS),
  - inclusion de fichiers (LFI/RFI),
  - injections de commandes, etc.

On peut le voir comme une **couche de sécurité en plus**, placée entre l’internaute et l’application web :
il inspecte le trafic et stoppe les requêtes suspectes avant qu’elles n’atteignent le code PHP ou la base de données.

* Installation de ModSecurity sur Apache :

   * Déploiement du module libapache2-mod-security2

 ```
sudo apt install libapache2-mod-security2 -y
```

<img width="934" height="261" alt="deploiement_liapache2_mod_security2" src="https://github.com/user-attachments/assets/f9c71231-4aa8-495e-8f68-b6d4acf141ed" />


   * Activer le module dans Apache

```
sudo a2enmod security2
```

<img width="948" height="118" alt="activation_module_security2" src="https://github.com/user-attachments/assets/508f46cc-4398-4cca-89d0-26f9b5121527" />

   * Redémarrage d'Apache est requis pour charger le nouveau module

```
sudo systemctl restart apache2
```

   * Vérifier que le module est bien chargé

```
   apache2ctl -M | grep security2
```

<img width="958" height="140" alt="verif_module_security2" src="https://github.com/user-attachments/assets/79093ec8-a55f-4d75-a704-ee458017e159" />



## Étape 3 - Intégration de l'OWASP Core Rule Set (CRS)

## Qu’est-ce que l’OWASP Core Rule Set (CRS) ?

ModSecurity est le **moteur**, mais il a besoin de règles pour savoir quoi bloquer.
C’est le rôle de l’**OWASP Core Rule Set (CRS)**.

L’OWASP CRS est un **ensemble de règles génériques** qui couvrent les principales attaques Web
(issues notamment de l’OWASP Top 10) :
- injections SQL,
- XSS,
- LFI/RFI,
- injections de commandes,
- attaques sur les sessions, scanners/bots, etc.

En résumé :
- ModSecurity = le WAF (moteur d’inspection).
- OWASP CRS = le **pack de règles** qui décrit les patterns d’attaque à détecter et comment réagir.

* **Activer le moteur ModSecurity (SecRuleEngine)**

But : dire à ModSecurity s’il doit juste loguer ou aussi **bloquer**. 

1. Aller dans le dossier de conf :


   ```bash
   cd /etc/modsecurity
   ls
   ```
   Tu devrais voir `modsecurity.conf-recommended`. 

2. Copier le fichier recommandé vers la conf active :


   ```bash
   sudo cp modsecurity.conf-recommended modsecurity.conf
   ```

Le fichier /etc/modsecurity/modsecurity.conf devient le fichier de configuration principal du moteur ModSecurity.

3. Examen approfondi des directives essentielles**

Plusieurs directives dans `modsecurity.conf` sont d'une importance capitale :

  * **`SecRuleEngine`**: C'est l'interrupteur principal du WAF.

      * `On` : Les règles sont actives et bloquent les menaces.
      * `Off` : Le module est complètement désactivé.
      * `DetectionOnly` : Les règles journalisent les menaces mais ne bloquent rien.
        ⚠️ **Crucial** : Pour une nouvelle installation, commencez toujours en mode **`DetectionOnly`** pour identifier les faux positifs sans impacter les utilisateurs.

  * **`SecAuditEngine`**: Contrôle la journalisation d'audit. La valeur `On` est recommandée au début pour enregistrer toutes les transactions.

  * **`SecAuditLog`**: Définit le chemin du fichier journal d'audit, généralement `/var/log/apache2/modsec_audit.log`.

  * **`SecRequestBodyAccess On`**: Directive **critique**. Elle ordonne à ModSecurity d'inspecter le corps des requêtes (données de formulaire POST, JSON, etc.), où se trouvent la plupart des attaques applicatives.

  * **`SecRequestBodyLimit`**: Une mesure de défense contre les attaques par déni de service (**DoS**) qui limite la taille maximale du corps d'une requête.

---

### Étape 3.1 – Téléchargement et installation de l’OWASP Core Rule Set (CRS)

**Objectif :** récupérer une version récente du pack de règles OWASP CRS et l’installer sur le serveur, afin d’alimenter ModSecurity en signatures d’attaques (SQLi, XSS, LFI, etc.).

1. Télécharger l’archive de la version souhaitée (ici `v4.25.0`, comme dans le cours) depuis GitHub :

```bash
cd /tmp
wget "https://github.com/coreruleset/coreruleset/archive/refs/tags/v4.25.0.tar.gz"
```

2. Extraire l’archive et déplacer le dossier du CRS dans l’arborescence d’Apache :

```bash
tar -xzvf v4.25.0.tar.gz
sudo mv coreruleset-4.25.0 /etc/apache2/modsecurity-crs
```

À partir de maintenant, la configuration et les règles CRS se trouveront dans `/etc/apache2/modsecurity-crs`.

---

### Étape 3.2 – Activation du fichier `crs-setup.conf` et configuration de base

Le CRS fournit un fichier de configuration modèle qu’il faut activer :

```bash
cd /etc/apache2/modsecurity-crs
sudo cp crs-setup.conf.example crs-setup.conf
```

Le fichier `crs-setup.conf` permet notamment de définir le **Paranoia Level (PL)**, c’est-à-dire le niveau de “strictness” des règles :

- **PL1** : protection de base contre les attaques les plus courantes, peu de faux positifs (niveau recommandé pour commencer).
- PL2 à PL4 : niveaux plus stricts, adaptés à des applis très sensibles, mais plus de faux positifs.

Pour ce lab, on reste sur **PL1**, via la directive qui fixe `tx.blocking_paranoia_level=1` dans `crs-setup.conf`.

On vérifie également dans ce fichier que :

```text
SecDefaultAction "phase:1,log,auditlog,deny,status:403"
SecDefaultAction "phase:2,log,auditlog,deny,status:403"
```

et que la version CRS (`tx.crs_setup_version`) est correctement renseignée.

---

### Étape 3.3 – Charger le CRS dans Apache

Il faut maintenant dire à Apache de charger la configuration CRS et toutes les règles.

On édite la configuration du module `security2` :

```bash
sudo nano /etc/apache2/mods-enabled/security2.conf
```

On ajoute (ou ajuste) le bloc suivant :

```apache
<IfModule security2_module>
    # Répertoire pour les données persistantes de ModSecurity
    SecDataDir /var/cache/modsecurity

    # Charger d’abord la configuration CRS
    IncludeOptional /etc/apache2/modsecurity-crs/crs-setup.conf
    # Charger toutes les règles CRS
    IncludeOptional /etc/apache2/modsecurity-crs/rules/*.conf

    # Charger ensuite les fichiers de configuration ModSecurity locaux
    IncludeOptional /etc/modsecurity/*.conf

    # Ne pas charger d’anciens profils CRS système si présents
    #IncludeOptional /usr/share/modsecurity-crs/*.load
</IfModule>
```

On vérifie la configuration et on redémarre Apache :

```bash
sudo apache2ctl configtest
sudo systemctl restart apache2
```

À ce stade :
- ModSecurity est installé et configuré.
- L’OWASP CRS est en place et ses règles sont chargées.
- Le WAF est prêt à analyser et à bloquer des requêtes malveillantes.
