Voici une version **unique, propre et complète** de ton TP, avec :

- Ton cas réel Windows → Ubuntu.  
- La variante `ssh-copy-id` depuis Linux (mise en opposition).  
- Les explications sur la conf `sshd_config` et le fonctionnement.  

Tu peux la coller telle quelle dans ton README.

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
- Connaître la méthode équivalente depuis un **client Linux** (`ssh-copy-id`). [ubuntu](https://ubuntu.com/tutorials/ssh-keygen-on-windows)

***

## I. Génération de la clé SSH (Windows 11 → `id_rsa`)

### 1. Prérequis côté Windows 11

Sur Windows 11, le client OpenSSH est intégré par défaut. Tu peux vérifier dans PowerShell :

```powershell
ssh -V
```

La clé sera stockée dans le dossier `C:\Users\<ton_user>\.ssh`. [ubuntu](https://ubuntu.com/tutorials/ssh-keygen-on-windows)

### 2. Commande utilisée

Dans **PowerShell** (ou Windows Terminal) en tant qu’utilisateur normal :

```powershell
ssh-keygen -t rsa -b 4096 -f $env:USERPROFILE\.ssh\id_rsa
```

- Cela crée :
  - `C:\Users\<user>\.ssh\id_rsa` → clé **privée** (à ne jamais partager).  
  - `C:\Users\<user>\.ssh\id_rsa.pub` → clé **publique** (à copier sur le serveur Ubuntu).  
- Tu peux laisser le nom par défaut et définir une **passphrase** pour protéger la clé. [learn.microsoft](https://learn.microsoft.com/en-us/windows-server/administration/openssh/openssh_keymanagement)

### 3. Vérification des fichiers générés

Après la génération, on vérifie la présence des clés dans le dossier `.ssh` :

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

> Remarque : la clé privée `id_rsa` ne doit jamais être publiée ni envoyée à quelqu’un. [concurrency](https://concurrency.com/blog/key-based-authentication-for-openssh-on-windows/)

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

Le service doit être `active (running)`. [digitalocean](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-on-ubuntu-22-04)

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

À ce stade, la connexion utilise encore le **mot de passe** de `jiji`.

***

## III. Dépôt de la clé publique sur le serveur Ubuntu

### III.A. Depuis Windows 11 (méthode manuelle – utilisée dans ce TP)

#### 1. Récupérer la clé publique sur Windows

Dans PowerShell :

```powershell
type $env:USERPROFILE\.ssh\id_rsa.pub
```

Copier **toute la ligne** qui commence par `ssh-rsa` et se termine par `jiji@JiJi`. [reddit](https://www.reddit.com/r/linuxadmin/comments/1j0ziba/ssh_keys_between_windows_10_and_linux/)

#### 2. Se connecter à la VM (avec mot de passe une dernière fois)

Depuis Windows :

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

La clé publique du client Windows est maintenant autorisée pour l’utilisateur `jiji` sur le serveur Ubuntu. [digitalocean](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-on-ubuntu-22-04)

***

### III.B. Depuis un client Linux (avec `ssh-copy-id` – en opposition)

Si le **client** n’est pas Windows mais un Linux (Kali, Ubuntu, WSL, etc.), la même opération peut être automatisée avec `ssh-copy-id`. [man7](https://man7.org/linux/man-pages/man1/ssh-copy-id.1.html)

Sur le client Linux :

1. Générer la clé :

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
- Met les permissions correctes (`700` sur `~/.ssh`, `600` sur `authorized_keys`). [ssh](https://www.ssh.com/academy/ssh/copy-id)

> En résumé :  
> - **Windows 11** → méthode manuelle (copier/coller avec `nano`).  
> - **Client Linux** → méthode automatisée avec `ssh-copy-id`.

***

## IV. Configuration du serveur Ubuntu pour forcer l’authentification par clé

Sur la VM Ubuntu, éditer la configuration SSH :

```bash
sudo nano /etc/ssh/sshd_config
```

Ajouter ou vérifier les lignes suivantes :

```text
PubkeyAuthentication yes
PasswordAuthentication no
KbdInteractiveAuthentication no
UsePAM yes
AcceptEnv LANG LC_*
Subsystem       sftp    /usr/lib/openssh/sftp-server
```

- `PubkeyAuthentication yes` active l’authentification par clé.  
- `PasswordAuthentication no` désactive le mot de passe classique.  
- `KbdInteractiveAuthentication no` évite d’autres prompts interactifs liés au mot de passe.  
- `UsePAM yes` laisse PAM actif pour d’autres mécanismes si besoin.  
- `AcceptEnv` et `Subsystem sftp` sont les valeurs par défaut d’Ubuntu. [docs.rackspace](https://docs.rackspace.com/docs/enable-ssh-public-key-authentication)

Sauvegarder puis redémarrer le service :

```bash
sudo systemctl restart ssh
```

Le serveur accepte maintenant uniquement les connexions avec clé publique pour l’utilisateur `jiji`.

***

## V. Validation du fonctionnement (Windows 11 → Ubuntu VM)

### 1. Test de connexion depuis Windows

Dans PowerShell (mode normal) :

```powershell
ssh jiji@10.225.123.61
```

La connexion doit se faire :

- soit directement, si la clé n’a pas de passphrase,  
- soit en demandant la **passphrase de la clé** (et non plus le mot de passe Linux).

Exemple en mode verbeux :

```powershell
ssh -v jiji@10.225.123.61
```

Extraits importants :

```text
Authentications that can continue: publickey
Next authentication method: publickey
Offering public key: C:\\Users\\JiJi/.ssh/id_rsa ...
Server accepts key: C:\\Users\\JiJi/.ssh/id_rsa ...
Enter passphrase for key 'C:\\Users\\JiJi/.ssh/id_rsa':
Authenticated to 10.225.123.61 ([10.225.123.61]:22) using "publickey".
```

Ces lignes confirment que :

- le serveur n’autorise plus que `publickey`,  
- ta clé `id_rsa` est bien proposée et acceptée,  
- l’authentification est réussie **via la clé**. [linuxize](https://linuxize.com/post/ssh-command-in-linux/)

### 2. Cas particuliers

**Si la clé privée `id_rsa` est supprimée sur Windows :**

- Tu ne peux plus prouver que tu es le propriétaire de la clé publique dans `authorized_keys`.  
- Avec `PasswordAuthentication no`, tu ne peux plus te connecter depuis cette machine tant que :
  - tu ne remets pas une copie valide de `id_rsa`,  
  - ou tu ne réactives pas temporairement le mot de passe sur le serveur. [docs.rackspace](https://docs.rackspace.com/docs/enable-ssh-public-key-authentication)

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

`ssh-agent` garde ta clé privée déverrouillée en mémoire après que tu as saisi sa passphrase une fois, ce qui évite de la retaper à chaque `ssh` tout en gardant la clé protégée. [scivision](https://www.scivision.dev/ssh-agent-config/)

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

La passphrase de la clé ne sera plus redemandée tant que `ssh-agent` tourne.

### 3. Alternatives (PuTTY / MobaXterm)

- **PuTTY + Pageant** : convertir la clé en `.ppk` avec PuTTYgen, charger la clé dans Pageant, et utiliser PuTTY.  
- **MobaXterm** : charger la clé dans l’agent SSH intégré et utiliser une session SSH classique ; MobaXterm utilisera la clé automatiquement. [concurrency](https://concurrency.com/blog/key-based-authentication-for-openssh-on-windows/)

***

## VII. Résumé du workflow (Windows 11 → Ubuntu VM)

1. **Sur Windows 11 :**
   - Générer la paire de clés : `ssh-keygen -t rsa -b 4096`.  
   - Vérifier `id_rsa` et `id_rsa.pub` dans `C:\Users\<user>\.ssh`.

2. **Sur Ubuntu VM :**
   - Installer et démarrer `openssh-server`.  
   - Copier la clé publique dans `~/.ssh/authorized_keys` :
     - manuellement depuis Windows (copier/coller),  
     - ou automatiquement depuis un client Linux avec `ssh-copy-id`.  
   - Configurer `/etc/ssh/sshd_config` avec :
     - `PubkeyAuthentication yes`,  
     - `PasswordAuthentication no`.  
   - Redémarrer le service SSH.

3. **Sur Windows 11 :**
   - Tester : `ssh jiji@10.225.123.61`.  
   - Optionnel : activer `ssh-agent` et ajouter la clé avec `ssh-add`.

***

## Conclusion

Avec ce TP, tu as :

- Mis en place une authentification par clé SSH entre un **client Windows 11** et un **serveur Ubuntu en VM**.  
- Sécurisé l’accès en désactivant l’authentification par mot de passe dans `sshd_config`.  
- Vu comment `ssh-agent` améliore le confort d’utilisation des clés sous Windows.  
- Comparé deux méthodes de déploiement de la clé publique : **manuelle** (Windows) et **automatisée** (`ssh-copy-id` côté Linux). [ssh](https://www.ssh.com/academy/ssh/copy-id)

Si tu veux, on pourra ensuite ajouter une section “Captures d’écran” avec les commandes clés (génération, `ls .ssh`, `nano authorized_keys`, `ssh -v ... using "publickey"`).
