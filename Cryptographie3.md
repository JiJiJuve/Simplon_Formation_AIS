Tu as raison, c’est intéressant de garder les **deux méthodes** (Windows vs Linux) pour montrer que tu maîtrises les deux scénarios. Je te refais la fiche complète avec :

- Ton cas réel : **client Windows 11** (méthode manuelle).  
- En parallèle : la méthode **`ssh-copy-id` depuis un client Linux** (mise en opposition, comme tu dis).

Voici le README complet prêt à coller.

***

# TP3 – Authentification par clé SSH (Client Windows 11 → Serveur Ubuntu VM)

## Sommaire

- [Objectifs](#objectifs)  
- [I. Génération de la clé SSH (Windows 11 → id_rsa)](#i-génération-de-la-clé-ssh-windows-11--id_rsa)  
- [II. Préparation du serveur Ubuntu (activer SSH)](#ii-préparation-du-serveur-ubuntu-activer-ssh)  
- [III. Dépôt de la clé publique sur le serveur Ubuntu](#iii-dépôt-de-la-clé-publique-sur-le-serveur-ubuntu)  
  - [III.A. Depuis Windows 11 (méthode manuelle)](#iiia-depuis-windows-11-méthode-manuelle)  
  - [III.B. Depuis un client Linux (avec ssh-copy-id)](#iiib-depuis-un-client-linux-avec-ssh-copy-id)  
- [IV. Configuration du serveur Ubuntu pour forcer l’authentification par clé](#iv-configuration-du-serveur-ubuntu-pour-forcer-lauthentification-par-clé)  
- [V. Validation du fonctionnement (Windows 11 → Ubuntu VM)](#v-validation-du-fonctionnement-windows-11--ubuntu-vm)  
- [VI. Bonus : Utilisation de ssh-agent sur Windows 11](#vi-bonus--utilisation-de-ssh-agent-sur-windows-11)  
- [VII. Résumé du workflow (Windows 11 → Ubuntu VM)](#vii-résumé-du-workflow-windows-11--ubuntu-vm)  
- [Conclusion](#conclusion)  

***

## Objectifs

- Générer une paire de clés SSH (publique/privée) sur un poste **Windows 11**.  
- Utiliser le client SSH natif (PowerShell, Windows Terminal).  
- Déployer la clé publique sur un serveur **Ubuntu** virtualisé (VM).  
- Forcer l’authentification par clé sur le serveur Ubuntu.  
- Comprendre et utiliser `ssh-agent` sous Windows pour ne pas retaper la passphrase en boucle.  
- Connaître la méthode équivalente depuis un **client Linux** (`ssh-copy-id`).

***

## I. Génération de la clé SSH (Windows 11 → `id_rsa`)

### 1. Prérequis côté Windows 11

Sur Windows 11, le client SSH est intégré par défaut. Vérification dans PowerShell :

```powershell
ssh -V
```

La clé sera stockée dans le dossier `C:\Users\<ton_user>\.ssh`.

### 2. Commande utilisée

Dans **PowerShell** (ou Windows Terminal) en tant qu’utilisateur normal :

```powershell
ssh-keygen -t rsa -b 4096 -f $env:USERPROFILE\.ssh\id_rsa
```

- Cela crée :
  - `C:\Users\<user>\.ssh\id_rsa` → clé **privée** (à ne jamais partager).  
  - `C:\Users\<user>\.ssh\id_rsa.pub` → clé **publique** (à copier sur le serveur Ubuntu).  
- Tu peux laisser le nom par défaut et définir une **passphrase** pour protéger la clé.

### 3. Vérification des fichiers générés

Après la génération, je vérifie la présence des clés dans mon dossier `.ssh` :

```powershell
PS C:\WINDOWS\system32> cd $env:USERPROFILE\.ssh
PS C:\Users\JiJi\.ssh> ls


    Répertoire : C:\Users\JiJi\.ssh

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----        30/04/2026     10:52           3414 id_rsa
-a----        30/04/2026     10:52            736 id_rsa.pub
-a----        20/03/2026     10:18           2494 known_hosts
-a----        18/03/2026     19:50           2110 known_hosts.old
```

Afficher la clé **publique** (celle à copier sur le serveur) :

```powershell
PS C:\Users\JiJi\.ssh> type $env:USERPROFILE\.ssh\id_rsa.pub
ssh-rsa AAAA[...]Q== jiji@JiJi
```

Vérifier l’existence de la clé **privée** (sans la publier) :

```powershell
PS C:\Users\JiJi\.ssh> type $env:USERPROFILE\.ssh\id_rsa
-----BEGIN OPENSSH PRIVATE KEY-----
[contenu masqué volontairement]
-----END OPENSSH PRIVATE KEY-----
```

> Remarque : la clé privée `id_rsa` ne doit jamais être publiée ni envoyée à quelqu’un.

***

## II. Préparation du serveur Ubuntu (activer SSH)

### 1. Vérifier l’adresse IP de la VM

Sur la VM Ubuntu :

```bash
ip a
```

Identifier l’adresse de la carte réseau (par exemple) :

```text
10.225.123.61
```

Cette IP sera utilisée côté Windows :

```powershell
ssh jiji@10.225.123.61
```

### 2. Installer et démarrer le serveur SSH

Sur la VM Ubuntu :

```bash
sudo apt update
sudo apt install openssh-server
```

Puis activer et vérifier le service :

```bash
sudo systemctl enable --now ssh
sudo systemctl status ssh
```

Le service doit être `active (running)`.

### 3. Premier test de connexion depuis Windows (avec mot de passe)

Depuis PowerShell :

```powershell
ssh jiji@10.225.123.61
```

Exemple de première connexion :

```text
The authenticity of host '10.225.123.61 (10.225.123.61)' can't be established.
ED25519 key fingerprint is SHA256:...
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.225.123.61' (ED25519) to the list of known hosts.
jiji@10.225.123.61's password:
Welcome to Ubuntu 24.04.4 LTS ...
```

À ce stade, la connexion utilise encore le **mot de passe** de `jiji`. La suite va basculer vers l’authentification par clé.

***

## III. Dépôt de la clé publique sur le serveur Ubuntu

### III.A. Depuis Windows 11 (méthode manuelle – utilisée dans ce TP)

#### 1. Récupérer la clé publique sur Windows

Dans PowerShell :

```powershell
type $env:USERPROFILE\.ssh\id_rsa.pub
```

Copier **toute la ligne** qui commence par `ssh-rsa` et se termine par `jiji@JiJi`.

#### 2. Se connecter à la VM (avec mot de passe une dernière fois)

```powershell
ssh jiji@10.225.123.61
```

Entrer le mot de passe du compte `jiji`.

#### 3. Créer `~/.ssh` et `authorized_keys` sur Ubuntu

Sur la VM Ubuntu :

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh

nano ~/.ssh/authorized_keys
```

Dans `nano` :

- Coller la ligne de la clé publique copiée depuis Windows.  
- Enregistrer (`Ctrl+O`, Entrée) puis quitter (`Ctrl+X`).

Mettre les bonnes permissions :

```bash
chmod 600 ~/.ssh/authorized_keys
```

La clé publique du client Windows est maintenant autorisée pour l’utilisateur `jiji` sur le serveur Ubuntu.

***

### III.B. Depuis un client Linux (avec `ssh-copy-id` – en opposition)

Si le **client** n’est pas Windows mais un Linux (Kali, Ubuntu, WSL, etc.), la même opération peut être automatisée avec `ssh-copy-id`.

Sur le client Linux :

1. Générer la clé (équivalent Unix) :

```bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa
```

2. Copier automatiquement la clé publique sur le serveur Ubuntu :

```bash
ssh-copy-id -i ~/.ssh/id_rsa.pub jiji@10.225.123.61
```

Cette commande :

- Se connecte une fois en **mot de passe** à `jiji@10.225.123.61`.  
- Crée `~/.ssh` sur la VM si besoin.  
- Ajoute automatiquement la clé publique dans `~/.ssh/authorized_keys`.  
- Met les permissions correctes (`700` sur `~/.ssh`, `600` sur `authorized_keys`).

> En résumé :  
> - **Windows 11** → méthode manuelle (copier/coller avec `nano`).  
> - **Client Linux** → méthode automatisée avec `ssh-copy-id`.

***

## IV. Configuration du serveur Ubuntu pour forcer l’authentification par clé

Sur la VM Ubuntu, éditer la configuration SSH :

```bash
sudo nano /etc/ssh/sshd_config
```

Vérifier ou modifier :

```text
PubkeyAuthentication yes
PasswordAuthentication no
```

Enregistrer puis redémarrer le service :

```bash
sudo systemctl restart ssh
```

Le serveur acceptera alors uniquement les connexions avec clé publique (plus de mot de passe).

***

## V. Validation du fonctionnement (Windows 11 → Ubuntu VM)

### 1. Test de connexion depuis Windows

Dans PowerShell :

```powershell
ssh jiji@10.225.123.61
```

- Si la clé n’a pas de passphrase : connexion directe sans demande de mot de passe Linux.  
- Si la clé a une passphrase (et sans `ssh-agent`) : on te demande la **passphrase de la clé**, mais plus le mot de passe du compte Ubuntu.

### 2. Cas particuliers

**Si la clé privée `id_rsa` est supprimée sur Windows :**

- Tu ne peux plus prouver que tu es le propriétaire de la clé publique stockée dans `authorized_keys`.  
- Avec `PasswordAuthentication no`, aucune connexion possible depuis cette machine tant que :
  - tu ne remets pas une copie valide de `id_rsa`,  
  - ou tu ne réactives pas temporairement l’authentification par mot de passe sur le serveur.

**Réactiver le mot de passe en cas d’urgence :**

Sur la VM Ubuntu (console locale) :

```bash
sudo nano /etc/ssh/sshd_config
```

Modifier :

```text
PasswordAuthentication yes
```

Puis :

```bash
sudo systemctl restart ssh
```

Tu peux à nouveau te connecter par mot de passe.

***

## VI. Bonus : Utilisation de `ssh-agent` sur Windows 11

### 1. Rôle de `ssh-agent`

`ssh-agent` garde ta clé privée déverrouillée en mémoire après que tu as saisi sa passphrase une fois.  
Tu peux ensuite ouvrir plusieurs sessions SSH sans retaper la passphrase.

### 2. Activer et utiliser `ssh-agent` (client SSH natif Windows)

Dans PowerShell **en administrateur** :

```powershell
# Activer le service au démarrage
Get-Service ssh-agent | Set-Service -StartupType Automatic

# Démarrer le service maintenant
Start-Service ssh-agent
```

Puis, en tant qu’utilisateur :

```powershell
ssh-add $env:USERPROFILE\.ssh\id_rsa
```

Tu entres une fois la passphrase de la clé, puis :

```powershell
ssh jiji@10.225.123.61
```

La passphrase ne sera plus redemandée tant que l’agent tourne.

### 3. Alternatives (PuTTY / MobaXterm)

- **PuTTY + Pageant** : convertir la clé en `.ppk` avec PuTTYgen, charger la clé dans Pageant, et utiliser PuTTY.  
- **MobaXterm** : charger la clé dans l’agent SSH intégré et utiliser une session SSH classique ; MobaXterm utilisera la clé automatiquement.

***

## VII. Résumé du workflow (Windows 11 → Ubuntu VM)

1. **Sur Windows 11 :**
   - Générer la paire de clés : `ssh-keygen -t rsa -b 4096`.  
   - Vérifier `id_rsa` et `id_rsa.pub` dans `C:\Users\<user>\.ssh`.

2. **Sur Ubuntu VM :**
   - Installer et démarrer `openssh-server`.  
   - Vérifier que `ssh` est `active (running)`.  
   - Ajouter la clé publique dans `~/.ssh/authorized_keys` :
     - soit manuellement depuis Windows (copier/coller),  
     - soit automatiquement depuis un client Linux avec `ssh-copy-id`.  
   - Forcer l’authentification par clé (`PubkeyAuthentication yes`, `PasswordAuthentication no`).

3. **Sur Windows 11 :**
   - Tester : `ssh jiji@<IP_VM>`.  
   - Optionnel : activer `ssh-agent` et ajouter la clé avec `ssh-add`.

***

## Conclusion

Ce TP montre comment :

- Mettre en place une authentification par clé SSH entre un **client Windows 11** et un **serveur Ubuntu en VM**.  
- Sécuriser l’accès en désactivant l’authentification par mot de passe.  
- Améliorer le confort d’utilisation avec `ssh-agent`.  
- Utiliser **deux méthodes de déploiement** de la clé publique :
  - manuelle (cas Windows),  
  - automatisée (`ssh-copy-id` côté client Linux).

Tu peux réutiliser ce modèle pour l’administration de serveurs, l’accès Git (GitHub/GitLab), des déploiements automatisés ou la création de tunnels SSH.# TP3 – Authentification par clé SSH (Client Windows 11 → Serveur Ubuntu VM)

## Sommaire

- [Objectifs](#objectifs)  
- [I. Génération de la clé SSH (Windows 11 → id_rsa)](#i-génération-de-la-clé-ssh-windows-11--id_rsa)  
- [II. Dépôt de la clé publique sur le serveur Ubuntu (VM)](#ii-dépôt-de-la-clé-publique-sur-le-serveur-ubuntu-vm)  
- [III. Configuration du serveur Ubuntu pour forcer l’authentification par clé](#iii-configuration-du-serveur-ubuntu-pour-forcer-lauthentification-par-clé)  
- [IV. Validation du fonctionnement (Windows 11 → Ubuntu VM)](#iv-validation-du-fonctionnement-windows-11--ubuntu-vm)  
- [V. Bonus : Utilisation de ssh-agent sur Windows 11](#v-bonus--utilisation-de-ssh-agent-sur-windows-11)  
- [VI. Résumé du workflow (Windows 11 → Ubuntu VM)](#vi-résumé-du-workflow-windows-11--ubuntu-vm)  
- [Conclusion](#conclusion)  

***

## Objectifs

- Générer une paire de clés SSH (publique/privée) sur un poste **Windows 11**.  
- Utiliser le client SSH natif (PowerShell, Windows Terminal).  
- Déployer la clé publique sur un serveur **Ubuntu** virtualisé (VM).  
- Forcer l’authentification par clé sur le serveur Ubuntu.  
- Comprendre et utiliser `ssh-agent` sous Windows pour ne pas retaper la passphrase en boucle.

***

## I. Génération de la clé SSH (Windows 11 → `id_rsa`)

### 1. Prérequis côté Windows 11

Sur Windows 11, le client SSH est intégré par défaut. Vérification dans PowerShell :

```powershell
ssh -V
```

La clé sera stockée dans le dossier `C:\Users\<ton_user>\.ssh`.

### 2. Commande utilisée

Dans **PowerShell** (ou Windows Terminal) en tant qu’utilisateur normal :

```powershell
ssh-keygen -t rsa -b 4096 -f $env:USERPROFILE\.ssh\id_rsa
```

- Cela crée :
  - `C:\Users\<user>\.ssh\id_rsa` → clé **privée** (à ne jamais partager).
  - `C:\Users\<user>\.ssh\id_rsa.pub` → clé **publique** (à copier sur le serveur Ubuntu).
- Tu peux laisser le nom par défaut et définir une **passphrase** pour protéger la clé.

### 3. Vérification des fichiers générés

Après la génération, je vérifie la présence des clés dans mon dossier `.ssh` :

```powershell
PS C:\WINDOWS\system32> cd $env:USERPROFILE\.ssh
PS C:\Users\JiJi\.ssh> ls


    Répertoire : C:\Users\JiJi\.ssh

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----        30/04/2026     10:52           3414 id_rsa
-a----        30/04/2026     10:52            736 id_rsa.pub
-a----        20/03/2026     10:18           2494 known_hosts
-a----        18/03/2026     19:50           2110 known_hosts.old
```

Je peux afficher le contenu de la clé **publique** (celle que je vais copier sur le serveur Ubuntu) :

```powershell
PS C:\Users\JiJi\.ssh> type $env:USERPROFILE\.ssh\id_rsa.pub
ssh-rsa AAAA[...]Q== jiji@JiJi
```

Et vérifier que la clé **privée** existe bien (sans la partager) :

```powershell
PS C:\Users\JiJi\.ssh> type $env:USERPROFILE\.ssh\id_rsa
-----BEGIN OPENSSH PRIVATE KEY-----
[contenu masqué volontairement]
-----END OPENSSH PRIVATE KEY-----
```

> Remarque : la clé privée `id_rsa` ne doit jamais être publiée telle quelle dans un dépôt GitHub ou partagée avec quelqu’un.

***

## II. Dépôt de la clé publique sur le serveur Ubuntu (VM)

Supposons que ton serveur Ubuntu tourne en VM (VirtualBox, VMware, etc.) et qu’il est joignable en SSH sur :

```text
utilisateur = ubuntu
adresse IP = 192.168.X.Y    (IP de la VM)
```

### 1. Méthode recommandée : `ssh-copy-id` (si disponible)

Sous Windows 11, `ssh-copy-id` n’est pas fourni par défaut. Tu peux l’avoir via WSL/Kali/MobaXterm. Si ton environnement le propose, la commande serait :

```bash
ssh-copy-id -i ~/.ssh/id_rsa.pub ubuntu@192.168.X.Y
```

Cette commande :

- Crée `~/.ssh` sur la VM si besoin.  
- Ajoute la clé dans `~/.ssh/authorized_keys`.  
- Met les bonnes permissions.

### 2. Méthode manuelle (adaptée au TP)

Depuis Windows, affiche ta clé publique :

```powershell
type $env:USERPROFILE\.ssh\id_rsa.pub
```

Copie le contenu affiché.

Sur le serveur Ubuntu (connexion initiale **avec mot de passe**, une seule fois) :

```bash
ssh ubuntu@192.168.X.Y
```

Puis, sur la VM Ubuntu :

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh

# Éditer le fichier authorized_keys
nano ~/.ssh/authorized_keys
# Coller la ligne de ta clé publique Windows (id_rsa.pub)
# Enregistrer et quitter

chmod 600 ~/.ssh/authorized_keys
```

La clé publique du client Windows est maintenant autorisée sur le serveur Ubuntu.

***

## III. Configuration du serveur Ubuntu pour forcer l’authentification par clé

Sur la VM Ubuntu, édite la configuration SSH :

```bash
sudo nano /etc/ssh/sshd_config
```

Vérifie ou modifie les lignes suivantes :

```text
PubkeyAuthentication yes
PasswordAuthentication no
```

Enregistre, puis redémarre le service SSH :

```bash
sudo systemctl restart ssh
```

À partir de ce moment :

- Le serveur **accepte** l’authentification par clé publique.  
- L’authentification par mot de passe est **désactivée** (tu ne peux plus te connecter juste avec le mot de passe système).

***

## IV. Validation du fonctionnement (Windows 11 → Ubuntu VM)

### 1. Test de connexion depuis Windows

Toujours dans PowerShell :

```powershell
ssh ubuntu@192.168.X.Y
```

- Si `ssh-agent` n’est pas utilisé et que ta clé est protégée par une passphrase, on te la demandera une fois.  
- Si tout est correct, la connexion se fait **sans** demander le mot de passe de l’utilisateur Linux (`ubuntu`), uniquement grâce à la clé.

### 2. Questions fréquentes

**Que se passe-t-il si je supprime la clé privée sur Windows (`id_rsa`) ?**

- Tu perds la capacité de prouver que tu es le propriétaire de la clé publique installée sur le serveur.  
- Comme `PasswordAuthentication no`, tu ne peux plus te connecter via SSH depuis cette machine, sauf si :
  - tu remets une copie de la clé privée,  
  - ou tu réactives temporairement le mot de passe dans `sshd_config` depuis la console de la VM (ou un autre accès).

**Comment réactiver l’authentification par mot de passe en cas de problème ?**

Sur la VM Ubuntu, depuis une console locale :

```bash
sudo nano /etc/ssh/sshd_config
```

Remets :

```text
PasswordAuthentication yes
```

Puis :

```bash
sudo systemctl restart ssh
```

Tu pourras à nouveau te connecter par mot de passe.

***

## V. Bonus : Utilisation de `ssh-agent` sur Windows 11

### 1. Rôle de `ssh-agent`

`ssh-agent` est un service qui garde ta clé privée **déverrouillée en mémoire** après que tu as saisi la passphrase une première fois. Cela évite de retaper la passphrase à chaque nouvelle connexion SSH, tout en gardant la clé protégée.

### 2. Activer et utiliser `ssh-agent` (client SSH natif Windows)

Dans PowerShell **en administrateur** :

```powershell
# Activer le service au démarrage
Get-Service ssh-agent | Set-Service -StartupType Automatic

# Démarrer le service maintenant
Start-Service ssh-agent
```

Puis, en tant qu’utilisateur :

```powershell
# Ajouter ta clé dans l'agent
ssh-add $env:USERPROFILE\.ssh\id_rsa
```

Tu entres une fois la passphrase. Ensuite :

```powershell
ssh ubuntu@192.168.X.Y
```

La passphrase ne sera plus redemandée tant que `ssh-agent` tourne et que ta session est active.

### 3. Si tu utilises PuTTY / Pageant (optionnel)

Si tu préfères PuTTY :

- Convertis ta clé OpenSSH en `.ppk` avec PuTTYgen.  
- Lance **Pageant** et charge ta clé `.ppk`.  
- Configure PuTTY pour utiliser cette clé ; les connexions se feront sans redemander la passphrase à chaque fois (Pageant joue le rôle d’agent).

### 4. Si tu utilises MobaXterm (optionnel)

MobaXterm possède un agent SSH intégré :

- Charge ta clé privée dans l’agent SSH de MobaXterm.  
- Utilise une session SSH classique vers `ubuntu@192.168.X.Y`.  
- La clé stockée dans l’agent sera utilisée automatiquement pour l’authentification.

***

## VI. Résumé du workflow (Windows 11 → Ubuntu VM)

1. **Sur Windows 11 :**
   - Générer une paire de clés : `ssh-keygen -t rsa -b 4096`.  
   - Stocker la clé dans `C:\Users\<user>\.ssh\`.

2. **Sur Ubuntu VM :**
   - Copier la clé publique dans `~/.ssh/authorized_keys`.  
   - Configurer `/etc/ssh/sshd_config` avec `PubkeyAuthentication yes` et `PasswordAuthentication no`.  
   - Redémarrer le service SSH.

3. **Sur Windows 11 :**
   - (Optionnel) Activer `ssh-agent` et ajouter la clé avec `ssh-add`.  
   - Tester : `ssh ubuntu@192.168.X.Y`.

***

## Conclusion

Avec ce TP, tu as :

- Mis en place une authentification par clé SSH entre un **client Windows 11** et un **serveur Ubuntu en VM**.  
- Sécurisé l’accès en désactivant l’authentification par mot de passe.  
- Vu comment `ssh-agent` améliore le confort d’utilisation des clés, tout en gardant un bon niveau de sécurité.

Ce modèle est directement réutilisable en entreprise pour les accès à des serveurs Linux via SSH, mais aussi pour Git (GitHub, GitLab), des déploiements automatisés, des tunnels SSH, etc.

***

Quand tu voudras, je pourrai t’aider à intégrer les captures d’écran de ce TP comme on a fait pour celui sur John (structure `images/` + liens dans le README).# TP3 – Authentification par clé SSH (Client Windows 11 → Serveur Ubuntu VM)

## Objectifs

- Générer une paire de clés SSH (publique/privée) sur un poste **Windows 11**.  
- Installer/ utiliser le client SSH natif ou un terminal (PowerShell, Windows Terminal).  
- Déployer la clé publique sur un serveur **Ubuntu** virtualisé (VM).  
- Forcer l’authentification par clé sur le serveur Ubuntu.  
- Comprendre et utiliser `ssh-agent` sous Windows pour ne pas retaper la passphrase en boucle.

***

## I. Génération de la clé SSH (Windows 11 → `id_rsa`)

### 1. Prérequis côté Windows 11

Sur Windows 11, le client SSH est intégré par défaut. Tu peux vérifier dans PowerShell :

```powershell
ssh -V
```

La clé sera stockée dans le dossier `C:\Users\<ton_user>\.ssh`.

### 2. Commande utilisée

Dans **PowerShell** (ou Windows Terminal) en tant qu’utilisateur normal :

```powershell
ssh-keygen -t rsa -b 4096 -f $env:USERPROFILE\.ssh\id_rsa
```

- Cela crée :
  - `C:\Users\<user>\.ssh\id_rsa` → clé privée (à ne jamais partager).
  - `C:\Users\<user>\.ssh\id_rsa.pub` → clé publique (à copier sur le serveur Ubuntu).
- Tu peux laisser le nom par défaut et définir une **passphrase** pour protéger la clé.

***

## II. Dépôt de la clé publique sur le serveur Ubuntu (VM)

Supposons que ton serveur Ubuntu tourne en VM (VirtualBox, VMware, etc.) et qu’il est joignable en SSH sur :

```text
utilisateur = ubuntu
adresse IP = 192.168.X.Y    (IP de la VM)
```

### 1. Méthode recommandée : `ssh-copy-id` depuis Windows

Sous Windows 11 récent, `ssh-copy-id` n’est pas fourni par défaut. Tu as deux options :

- Soit tu disposes d’un environnement type WSL/Kali/MobaXterm qui inclut `ssh-copy-id`.  
- Soit tu fais la méthode **manuelle** (voir §2).

Si tu as un shell qui propose `ssh-copy-id`, la commande serait :

```bash
ssh-copy-id -i ~/.ssh/id_rsa.pub ubuntu@192.168.X.Y
```

Cette commande :

- Crée `~/.ssh` sur la VM si besoin.
- Ajoute la clé dans `~/.ssh/authorized_keys`.
- Met les bonnes permissions.

### 2. Méthode manuelle (adaptée à ton TP)

Depuis Windows, affiche ta clé publique :

```powershell
type $env:USERPROFILE\.ssh\id_rsa.pub
```

Copie le contenu affiché.

Sur le serveur Ubuntu (connexion initiale **avec mot de passe**, une seule fois) :

```bash
ssh ubuntu@192.168.X.Y
```

Puis, sur la VM Ubuntu :

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh

# Éditer le fichier authorized_keys
nano ~/.ssh/authorized_keys
# Coller la ligne de ta clé publique Windows (id_rsa.pub)
# Enregistrer et quitter

chmod 600 ~/.ssh/authorized_keys
```

La clé publique du client Windows est maintenant autorisée sur le serveur Ubuntu.

***

## III. Configuration du serveur Ubuntu pour forcer l’authentification par clé

Sur la VM Ubuntu, édite la configuration SSH :

```bash
sudo nano /etc/ssh/sshd_config
```

Vérifie ou modifie les lignes suivantes :

```text
PubkeyAuthentication yes
PasswordAuthentication no
```

Enregistre, puis redémarre le service SSH :

```bash
sudo systemctl restart ssh
```

À partir de ce moment :

- Le serveur **accepte** l’authentification par clé publique.
- L’authentification par mot de passe est **désactivée** (tu ne peux plus te connecter juste avec le mot de passe système).

***

## IV. Validation du fonctionnement (Windows 11 → Ubuntu VM)

### 1. Test de connexion depuis Windows

Toujours dans PowerShell :

```powershell
ssh ubuntu@192.168.X.Y
```

- Si `ssh-agent` n’est pas utilisé et que ta clé est protégée par une passphrase, on te la demandera une fois.
- Si tout est correct, la connexion se fait **sans** demander le mot de passe de l’utilisateur Linux (ubuntu), uniquement grâce à la clé.

### 2. Questions fréquentes (comportements à connaître)

**Que se passe-t-il si je supprime la clé privée sur Windows (`id_rsa`) ?**

- Tu perds la capacité de prouver que tu es le propriétaire de la clé publique installée sur le serveur.
- Comme `PasswordAuthentication no`, tu ne peux plus te connecter du tout via SSH depuis cette machine, sauf si :
  - tu remets une copie de la clé privée,
  - ou tu réactives temporairement le mot de passe dans `sshd_config` depuis la console de la VM (ou un autre accès).

**Comment réactiver l’authentification par mot de passe en cas de problème ?**

Sur la VM Ubuntu, depuis une console locale :

```bash
sudo nano /etc/ssh/sshd_config
```

Remets :

```text
PasswordAuthentication yes
```

Puis :

```bash
sudo systemctl restart ssh
```

Tu pourras à nouveau te connecter par mot de passe.

***

## V. Bonus : Utilisation de `ssh-agent` sur Windows 11

### 1. Rôle de `ssh-agent`

`ssh-agent` est un service qui garde ta clé privée **déverrouillée en mémoire** après que tu as saisi la passphrase une première fois.  
Cela évite de retaper la passphrase à chaque nouvelle connexion SSH, tout en gardant la clé protégée.

### 2. Activer et utiliser `ssh-agent` (client SSH natif Windows)

Dans PowerShell en administrateur :

```powershell
# Activer le service au démarrage
Get-Service ssh-agent | Set-Service -StartupType Automatic

# Démarrer le service maintenant
Start-Service ssh-agent
```

Puis, en tant qu’utilisateur :

```powershell
# Ajouter ta clé dans l'agent
ssh-add $env:USERPROFILE\.ssh\id_rsa
```

Tu entres une fois la passphrase. Ensuite :

```powershell
ssh ubuntu@192.168.X.Y
```

- La passphrase ne sera plus redemandée tant que `ssh-agent` tourne et que ta session est active.

### 3. Si tu utilises PuTTY / Pageant (optionnel)

Si tu préfères PuTTY :

- Convertis ta clé OpenSSH en `.ppk` avec PuTTYgen.  
- Lance **Pageant** et charge ta clé `.ppk`.  
- Configure PuTTY pour utiliser cette clé ; les connexions se feront sans redemander la passphrase à chaque fois (Pageant joue le rôle d’agent).

### 4. Si tu utilises MobaXterm (optionnel)

MobaXterm possède un agent SSH intégré :

- Charge ta clé privée dans l’agent SSH de MobaXterm.  
- Utilise une session SSH classique vers `ubuntu@192.168.X.Y`.  
- La clé stockée dans l’agent sera utilisée automatiquement pour l’authentification.

***

## VI. Résumé du workflow (Windows 11 → Ubuntu VM)

1. **Sur Windows 11 :**
   - Générer une paire de clés : `ssh-keygen -t rsa -b 4096`.
   - Stocker la clé dans `C:\Users\<user>\.ssh\`.

2. **Sur Ubuntu VM :**
   - Copier la clé publique dans `~/.ssh/authorized_keys`.
   - Configurer `/etc/ssh/sshd_config` avec `PubkeyAuthentication yes` et `PasswordAuthentication no`.
   - Redémarrer le service SSH.

3. **Sur Windows 11 :**
   - Optionnel : activer `ssh-agent` et ajouter la clé avec `ssh-add`.
   - Tester : `ssh ubuntu@192.168.X.Y`.

***

## Conclusion

Avec ce TP, tu as :

- Mis en place une authentification par clé SSH entre un **client Windows 11** et un **serveur Ubuntu en VM**.  
- Sécurisé l’accès en désactivant l’authentification par mot de passe.  
- Vu comment `ssh-agent` améliore le confort d’utilisation des clés, tout en gardant un bon niveau de sécurité.

Ce modèle est directement réutilisable en entreprise pour les accès à des serveurs Linux via SSH, mais aussi pour Git (GitHub, GitLab), des déploiements automatisés, des tunnels SSH, etc.
