# TP Ansible – Mes premiers pas (Debian 13)

Ce fichier fait partie de ma formation **AIS (Administrateur d’Infrastructures Sécurisées)** chez Simplon.  
Il décrit mon premier TP d’automatisation avec **Ansible** sur Debian 13 : partir d’une VM “vierge” et, étape par étape, la transformer en serveur à jour, sécurisé en SSH et capable de servir un site web statique.

---

## 1. Contexte et objectif

- Mettre en place un **contrôleur Ansible (C2)** sur Debian.
- Administrer une **cible Debian** à distance via SSH.
- Automatiser 3 grands types de tâches avec des playbooks :
  1. Mise à jour du système.
  2. Gestion des comptes / sudo / SSH (sécurité).
  3. Déploiement d’un service (Nginx) + site web statique.

L’idée est de passer d’une administration “manuelle” en SSH avec mot de passe à une **administration automatisée** et sécurisée avec Ansible et des clés SSH.

---

## 2. Phase 0 – Première connexion sans clé SSH

Au tout début du TP :

- J’ai 2 VMs Debian 13 en mode **bridge** :
  - 1 VM contrôleur (C2) avec Ansible installé.
  - 1 VM cible joignable en IP (ex : `192.168.1.17`).
- Je me connecte à la VM cible en SSH avec un **mot de passe**, car aucune clé SSH n’est encore configurée.

Exemple :

```bash
ssh jiji@192.168.1.17
# puis saisie du mot de passe de jiji
```

Cette première phase me permet de vérifier que la connectivité SSH fonctionne, avant de mettre en place l’authentification par clé.

---

## 3. Phase 1 – Premier inventory et playbook avec mot de passe

Sur le contrôleur, je crée un premier petit projet Ansible, en utilisant encore le mot de passe pour SSH.

### 3.1. Fichiers de base

Dans un dossier de travail (par exemple `~/ansible-c2`) :

- `inventory.ini` (version mot de passe) :

  ```ini
  [targets]
  server1 ansible_host=192.168.1.17 ansible_user=jiji
  ```

À ce stade, je n’utilise pas encore de clé SSH dans l’inventaire.  
Quand je lance un playbook, j’utilise l’option `--ask-pass` pour qu’Ansible me demande le mot de passe SSH.

### 3.2. Premier test de connexion

```bash
ansible -i inventory.ini targets -m ping --ask-pass
```

Cette commande vérifie qu’Ansible arrive à se connecter en SSH avec le mot de passe et à exécuter le module `ping` sur la cible.

---

## 4. Phase 2 – Génération et utilisation d’une clé SSH

Pour ne plus taper le mot de passe à chaque fois et sécuriser les connexions, je passe à une **clé SSH**.

### 4.1. Génération de la clé sur le C2

Sur le contrôleur (C2), connecté avec mon utilisateur :

```bash
ssh-keygen -t ed25519 -f ~/.ssh/jiji_key
```

Je protège la clé avec une passphrase, puis j’utilise `ssh-agent` pour ne pas la retaper à chaque fois :

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/jiji_key
```

### 4.2. Copie de la clé publique sur la cible

Je copie la clé publique sur la VM cible (initialement encore en mot de passe) :

```bash
ssh-copy-id -i ~/.ssh/jiji_key.pub jiji@192.168.1.17
```

Après cette étape, je peux me connecter en SSH sans mot de passe grâce à la clé :

```bash
ssh -i ~/.ssh/jiji_key jiji@192.168.1.17
```

### 4.3. Mise à jour de l’inventaire

Je mets à jour `inventory.ini` pour utiliser la clé :

```ini
[targets]
server1 ansible_host=192.168.1.17 ansible_user=jiji ansible_ssh_private_key_file=~/.ssh/jiji_key

[all:vars]
ansible_python_interpreter=/usr/bin/python3
```

Désormais, je n’ai plus besoin de `--ask-pass` pour SSH, seulement éventuellement de `--ask-become-pass` si `sudo` demande un mot de passe.

---

## 5. Playbook 1 – Mise à jour du système

Fichier : `1-update-os.yml`

Ce playbook :

- Met à jour le cache APT.
- Fait une mise à niveau des paquets (`upgrade: dist`) sur la cible.
- Utilise `become: true` pour exécuter les tâches avec les droits admin (équivalent de `sudo`).

Commande d’exécution :

```bash
ansible-playbook -i inventory.ini 1-update-os.yml --ask-become-pass
```

**But :** automatiser la gestion des mises à jour système sur la VM cible.

---

## 6. Playbook 2 – Utilisateur jiji + SSH durci

Fichier : `2-create-user.yml`

Ce playbook automatise la partie comptes et sécurité SSH :

- Vérifie ou crée l’utilisateur **`jiji`** avec :
  - un home,
  - le shell `/bin/bash`,
  - l’appartenance au groupe `sudo`.
- Ajoute la clé publique SSH générée sur le C2 dans `~jiji/.ssh/authorized_keys`.
- Durcit la configuration SSH :
  - `PermitRootLogin prohibit-password`,
  - `PasswordAuthentication no`.
- Redémarre le service SSH pour appliquer la configuration.

Commande :

```bash
ansible-playbook -i inventory.ini 2-create-user.yml --ask-become-pass
```

**But :** ne plus gérer la machine en root ou par mot de passe, mais via un utilisateur dédié (`jiji`) et une clé SSH, avec un SSHd durci.

---

## 7. Playbook 3 – Nginx + site web statique

Fichier : `3-deploy-website.yml`

### 7.1. Préparation de la page sur le C2

Sur le contrôleur, je prépare le dossier et la page :

```bash
mkdir -p site
nano site/index.html
```

Exemple de contenu :

```html
<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="UTF-8">
  <title>TP Ansible C2</title>
</head>
<body>
  <h1>TP Ansible - Déploiement Nginx</h1>
  <p>Bonjour les Amis, Vive Ansible.</p>
</body>
</html>
```

### 7.2. Rôle du playbook

Le playbook :

- Installe **Nginx** sur la cible (module `apt`).
- Copie `site/index.html` vers `/var/www/html/index.html` (module `copy`).
- S’assure que Nginx est démarré et activé au boot (module `service`).

Commande :

```bash
ansible-playbook -i inventory.ini 3-deploy-website.yml --ask-become-pass
```

Test depuis le C2 ou un navigateur :

```bash
curl http://192.168.1.17
```

**But :** déployer automatiquement un serveur web et un site statique à partir du contrôleur.

---

## 8. Idempotence et rejouer les playbooks

- Si je relance `1-update-os.yml`, la machine est déjà à jour, donc il y a peu ou pas de changements.
- Si je relance `2-create-user.yml`, `jiji` existe déjà, la clé SSH est en place, SSH est déjà durci, les tâches passent en `ok` sans casser la configuration.
- Si je relance `3-deploy-website.yml`, Nginx est déjà installé, `index.html` est recopié si je l’ai modifié côté contrôleur.

L’idée est qu’Ansible décrit un état cible et remet la machine dans cet état à chaque exécution, de façon répétable.
