# TP Ansible C2 – Debian 13.1.0

Ce fichier fait partie de ma formation **AIS (Administrateur d’Infrastructures Sécurisées)** chez Simplon.  
Il décrit mon premier TP d’automatisation avec **Ansible** sur Debian 13.1.0 : partir d’une VM “vierge” et, étape par étape, la transformer en serveur à jour, sécurisé en SSH et capable de servir un site web statique.

---

## 0. Objectif du TP

- Controller (C2) : Debian 13.1.0 avec Ansible.
- Cible : Debian 13.1.0.
- Utilisateur de service utilisé : `jiji`.
- Playbooks :
  - `1-update-os.yml`
  - `2-create-user.yml`
  - `3-deploy-website.yml`

L’idée est de passer d’une administration “manuelle” en SSH avec mot de passe à une **administration automatisée** et sécurisée avec Ansible et des clés SSH.

---

## 1. Pré‑requis sur la VM cible (Debian)

Sur la **cible** (console ou SSH) :

```bash
# 1. Installer le serveur SSH
sudo apt update
sudo apt install -y openssh-server

# 2. Activer et démarrer le service SSH
sudo systemctl enable ssh
sudo systemctl start ssh
sudo systemctl status ssh
```

Si besoin d’installer `sudo` et donner les droits à `jiji` :

```bash
su -
apt install -y sudo
adduser jiji sudo
```

Déconnexion / reconnexion en `jiji`, puis vérification :

```bash
groups          # doit contenir sudo
sudo whoami     # doit répondre "root"
```

---

## 2. Installer Ansible sur le controller (C2)

Sur le **controller** :

<img width="1063" height="913" alt="Maj_server" src="https://github.com/user-attachments/assets/8b74d1df-adeb-4d69-8a0f-b1c099120b8c" />


```bash
sudo apt update && sudo apt install -y ansible
mkdir -p ~/ansible-c2
cd ~/ansible-c2
```

---

## 3. Créer l’inventaire `inventory.ini`

```bash
cd ~/ansible-c2
nano inventory.ini
```

Contenu (adapter l’IP à celle de la **cible**) :

<img width="575" height="173" alt="fichier_inventory_ini" src="https://github.com/user-attachments/assets/10a6d87d-859e-4762-965b-c0951c902ce2" />

```ini
[targets]
server1 ansible_host=192.168.1.17 ansible_user=jiji

[all:vars]
ansible_python_interpreter=/usr/bin/python3
```

Vérifier :

<img width="734" height="287" alt="verif_inventaire" src="https://github.com/user-attachments/assets/5e6e5898-0d75-400c-bc27-58819c0b4c2b" />


```bash
ansible-inventory -i inventory.ini --list -y
```

---

## 4. Phase 0 – Première connexion SSH et acceptation de la clé de la cible

Sur le **controller** :

```bash
ssh jiji@192.168.1.17
# répondre "yes" pour accepter la clé (host key)
# entrer le mot de passe de jiji
exit
```

Cette première phase permet de vérifier la connectivité SSH avant d’utiliser Ansible.

---

## 5. Tester la connexion Ansible → cible (mot de passe)

<img width="805" height="112" alt="test_ping_OK_depuis_C2_cible" src="https://github.com/user-attachments/assets/0cffbbc0-3619-472e-baf6-5bb1e7c9afaa" />

```bash
ansible targets -i inventory.ini -m ping --ask-pass
```

- `SSH password:` → mot de passe de `jiji`.

Si tout va bien, on obtient un `SUCCESS` avec `"ping": "pong"`.

---

## 6. Playbook 1 : mise à jour OS (`1-update-os.yml`)

Créer le playbook :

<img width="503" height="247" alt="playbook_OS" src="https://github.com/user-attachments/assets/85b90db3-5ef2-43fb-90db-028c1466c6df" />


```bash
cd ~/ansible-c2
nano 1-update-os.yml
```

Contenu :

```yaml
***
- name: Update OS on targets (Debian)
  hosts: targets
  become: true

  tasks:
    - name: Update APT cache and upgrade packages
      apt:
        update_cache: yes
        upgrade: dist
```

Lancer (au début du TP, avec mot de passe SSH) :

<img width="1081" height="309" alt="Playbook_ansible_Maj_OS" src="https://github.com/user-attachments/assets/4bc21ce5-f1b4-4090-b85a-5eae781f3f0c" />

<img width="713" height="164" alt="explication_Resultat_playbook_OS" src="https://github.com/user-attachments/assets/1f951e06-0a2b-4562-822f-1d0284d1b8c8" />

```bash
ansible-playbook -i inventory.ini 1-update-os.yml --ask-pass --ask-become-pass
```

- `SSH password` → mot de passe de `jiji`.
- `BECOME password` → mot de passe sudo de `jiji` (souvent le même).

**But :** automatiser la gestion des mises à jour système sur la VM cible.

---

## 7. Générer une clé SSH pour Ansible (controller)

Sur le **controller** :

<img width="864" height="410" alt="creation_clé_ssh_C2" src="https://github.com/user-attachments/assets/1d61aeab-1375-4366-b93e-be27427e3200" />

```bash
ssh-keygen -t ed25519 -f ~/.ssh/jiji_key -C "ansible jiji"
```

- Clé privée : `~/.ssh/jiji_key`  
- Clé publique : `~/.ssh/jiji_key.pub`

Copier la clé sur la cible :

<img width="932" height="233" alt="copie_clé_ssh_sur_cible" src="https://github.com/user-attachments/assets/42d3ec91-0eaf-4dd8-afd8-e27022403403" />

```bash
ssh-copy-id -i ~/.ssh/jiji_key.pub jiji@192.168.1.17
```

Tester la connexion avec la clé :

<img width="934" height="231" alt="connexion_sans_mot_passe_cible_ac_passphrase" src="https://github.com/user-attachments/assets/1a355c83-1494-42e8-8922-4962f7aab143" />

```bash
ssh -i ~/.ssh/jiji_key jiji@192.168.1.17
```

---

## 8. Mettre à jour `inventory.ini` pour utiliser la clé

Modifier `inventory.ini` :

<img width="932" height="186" alt="modif_inventory_ini_avec_clé_ssh" src="https://github.com/user-attachments/assets/63afa475-0e38-4dae-b607-0f00fcbeb1a5" />

```ini
[targets]
server1 ansible_host=192.168.1.17 ansible_user=jiji ansible_ssh_private_key_file=~/.ssh/jiji_key

[all:vars]
ansible_python_interpreter=/usr/bin/python3
```

---

## 9. Utiliser ssh-agent pour la clé avec passphrase (sur le C2)

<img width="943" height="520" alt="connexion_sans_mot_passe" src="https://github.com/user-attachments/assets/63fcc21f-afcc-4cf5-a10c-0fa1e90ba606" />

Si la clé `~/.ssh/jiji_key` a une passphrase, charger la clé dans `ssh-agent` à chaque nouvelle session :

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/jiji_key
```

- `ssh-add` demandera la passphrase de la clé.
- Tant que la session reste ouverte, SSH et Ansible peuvent utiliser la clé sans redemander la passphrase.

Tester Ansible sans mot de passe SSH :

```bash
ansible targets -i inventory.ini -m ping
ansible-playbook -i inventory.ini 1-update-os.yml --ask-become-pass
```

À ce stade :
- Authentification SSH par **clé** (plus besoin de `--ask-pass`).
- `--ask-become-pass` reste nécessaire pour le mot de passe sudo (BECOME password).

---

## 10. Playbook 2 : création user + clé SSH + durcissement SSH (`2-create-user.yml`)

Ce playbook permet de gérer complètement l’utilisateur de service et la configuration SSH via Ansible.

Créer le fichier :

<img width="955" height="825" alt="playbook2_creation_user_sudo avec_ansible" src="https://github.com/user-attachments/assets/06925a80-e9c8-4fb5-a9fd-2aec0ad220c6" />


```bash
cd ~/ansible-c2
nano 2-create-user.yml
```

Contenu :

```yaml
***
- name: Create service user jiji with SSH key (Debian)
  hosts: targets
  become: true

  tasks:
    - name: Create user jiji with sudo
      user:
        name: jiji
        groups: sudo
        append: true
        create_home: true
        shell: /bin/bash

    - name: Add SSH public key for jiji
      ansible.posix.authorized_key:
        user: jiji
        state: present
        key: "{{ lookup('file', '~/.ssh/jiji_key.pub') }}"

    - name: Disable root SSH password login
      lineinfile:
        path: /etc/ssh/sshd_config
        regexp: '^#?PermitRootLogin'
        line: 'PermitRootLogin prohibit-password'
        state: present

    - name: Disable password authentication (only keys)
      lineinfile:
        path: /etc/ssh/sshd_config
        regexp: '^#?PasswordAuthentication'
        line: 'PasswordAuthentication no'
        state: present

    - name: Restart SSH service
      service:
        name: ssh
        state: restarted
```

Lancer :

<img width="936" height="841" alt="playbook2_resultats" src="https://github.com/user-attachments/assets/50aece24-cf3d-4ee7-9692-97695eafda21" />

```bash
ansible-playbook -i inventory.ini 2-create-user.yml --ask-become-pass
```

**But :** ne plus gérer la machine en root ou par mot de passe, mais via un utilisateur dédié (`jiji`) et une clé SSH, avec un serveur SSH durci.

---

## 11. Playbook 3 : Nginx + page web (`3-deploy-website.yml`)

### 11.1. Préparer la page sur le controller

Créer le dossier `site` puis le fichier `index.html` :

```bash
mkdir -p ~/ansible-c2/site
nano ~/ansible-c2/site/index.html
```

Exemple de contenu HTML simple :

<img width="749" height="293" alt="page_index_html" src="https://github.com/user-attachments/assets/724f2059-e595-447a-b367-f243d5c13291" />

```html
<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="UTF-8">
  <title>TP Ansible C2</title>
</head>
<body>
  <h1>TP Ansible - Déploiement Nginx</h1>
  <p>Ceci est ma première page déployée automatiquement avec Ansible.</p>
</body>
</html>
```

### 11.2. Créer le playbook

<img width="811" height="630" alt="creation_playbook3_nginx" src="https://github.com/user-attachments/assets/cc4649bb-de0a-490d-8d19-29590ff75780" />

```bash
cd ~/ansible-c2
nano 3-deploy-website.yml
```

Contenu :

```yaml
***
- name: Deploy Nginx and static website (Debian)
  hosts: targets
  become: true

  vars:
    web_root: /var/www/html

  tasks:
    - name: Install Nginx
      apt:
        name: nginx
        state: present
        update_cache: yes

    - name: Copy index.html to web root
      copy:
        src: site/index.html
        dest: "{{ web_root }}/index.html"
        owner: www-data
        group: www-data
        mode: '0644'

    - name: Ensure Nginx is enabled and running
      service:
        name: nginx
        state: started
        enabled: yes
```

### 11.3. Lancer le playbook et vérifier

Lancer le playbook :

```bash
ansible-playbook -i inventory.ini 3-deploy-website.yml --ask-become-pass
```

Installer `curl` sur le C2 si besoin :

```bash
sudo apt update
sudo apt install -y curl
```

Vérifier l’affichage de la page web :

<img width="708" height="255" alt="affiche_site_web_depuis_c2" src="https://github.com/user-attachments/assets/798ba63d-0be0-4863-ab3d-bf808a2e82cc" />

```bash
curl http://192.168.1.17
```

---

## 12. Résumé et notions importantes

### 12.1. SSH / Authentification

- Sur la cible :
  - `openssh-server` installé.
  - `sudo` installé.
  - `jiji` dans le groupe `sudo`.
- Clé SSH ED25519 `jiji_key` avec passphrase, copiée sur la cible.
- Sur le C2 :
  - `ssh-agent` + `ssh-add ~/.ssh/jiji_key` pour utiliser la clé sans retaper la passphrase.

### 12.2. Inventaire final

```ini
[targets]
server1 ansible_host=192.168.1.17 ansible_user=jiji ansible_ssh_private_key_file=~/.ssh/jiji_key

[all:vars]
ansible_python_interpreter=/usr/bin/python3
```

### 12.3. Commandes clés

```bash
# Test ping
ansible targets -i inventory.ini -m ping

# Update OS
ansible-playbook -i inventory.ini 1-update-os.yml --ask-become-pass

# Création user + durcissement SSH
ansible-playbook -i inventory.ini 2-create-user.yml --ask-become-pass

# Déploiement site
ansible-playbook -i inventory.ini 3-deploy-website.yml --ask-become-pass
```

### 12.4. Idempotence

- Si je relance `1-update-os.yml`, la machine est déjà à jour, donc peu ou pas de changements.
- Si je relance `2-create-user.yml`, `jiji` existe déjà, la clé SSH est en place, SSH est déjà durci, les tâches passent en `ok` sans casser la configuration.
- Si je relance `3-deploy-website.yml`, Nginx est déjà installé, `index.html` est recopié si je l’ai modifié côté controller.

Ansible décrit un **état cible** et remet la machine dans cet état à chaque exécution, de façon répétable.

---

## 13. Ce que ce TP m’a appris

- Installer et configurer un **controller Ansible** et une cible Debian.
- Créer et utiliser un **inventaire** Ansible.
- Distinguer :
  - l’authentification SSH (mot de passe → clé),
  - et l’élévation de privilèges (`become` / `sudo`).
- Générer et utiliser une **clé SSH protégée par passphrase** avec `ssh-agent`.
- Automatiser :
  - les mises à jour système,
  - la gestion des utilisateurs et des droits `sudo`,
  - la configuration SSH (durcissement),
  - l’installation et la configuration d’un service (Nginx),
  - le déploiement d’un site statique.

Ce TP est mon **premier pas sérieux avec Ansible** et l’automatisation d’infrastructures dans le cadre de la formation AIS (Simplon).

Parfait, on ajoute une **section bonus Vault** à la fin de ton README détaillé, sans casser ce que tu as déjà.

Tu peux coller ce bloc **après la section 13** (“Ce que ce TP m’a appris”) de ton README actuel.


---

## 14. Bonus – Ansible Vault (optionnel)

Pour ce TP, je n’ai pas besoin de stocker de mot de passe en clair dans les fichiers :  
j’utilise `--ask-pass` et `--ask-become-pass`, donc Ansible me demande les mots de passe au moment de l’exécution.  

Pour aller plus loin, j’ai testé **Ansible Vault** afin de voir comment chiffrer un mot de passe sudo dans un fichier de variables.

### 14.1. Création d’un fichier de secrets chiffré

Dans le dossier de travail Ansible (`~/ansible-c2`) :

```bash
mkdir -p vars
ansible-vault create vars/secrets.yml
```

Ansible demande un **mot de passe de coffre** (vault password), puis ouvre un éditeur.  
Je renseigne par exemple :

```yaml
ansible_become_password: "MonMotDePasseSudo"
```

En enregistrant et fermant, le fichier `vars/secrets.yml` est **chiffré** : on ne voit plus le mot de passe en clair.

### 14.2. Utilisation du vault dans un playbook

Exemple avec le playbook `1-update-os.yml` :

```yaml
***
- name: Update OS on targets (Debian)
  hosts: targets
  become: true

  vars_files:
    - vars/secrets.yml

  tasks:
    - name: Update APT cache and upgrade packages
      apt:
        update_cache: yes
        upgrade: dist
```

Ici :
- `vars_files` charge le fichier `vars/secrets.yml` (chiffré).
- La variable `ansible_become_password` est automatiquement utilisée par Ansible comme mot de passe sudo.

### 14.3. Exécution avec Vault

Pour lancer ce playbook avec Vault :

```bash
ansible-playbook -i inventory.ini 1-update-os.yml --ask-vault-pass
```

- `--ask-vault-pass` : Ansible demande le **mot de passe du coffre** pour déchiffrer `vars/secrets.yml`.

Dans ce mode :
- Le mot de passe sudo n’apparaît **dans aucun fichier en clair**.
- Il est stocké dans un fichier chiffré (`vars/secrets.yml`), et seulement déchiffré en mémoire le temps de l’exécution du playbook.

---

### 14.4. Intérêt de Vault pour la suite

Sur ce TP simple, Vault n’est pas indispensable, mais il devient très utile quand :

- On commence à gérer plusieurs serveurs de production avec des mots de passe différents.
- On veut **committer le code Ansible sur Git** sans exposer de secrets.
- On doit stocker d’autres informations sensibles : mots de passe de bases de données, tokens API, etc.

Cette section me permet surtout de montrer que je sais :
- Créer un fichier de secrets chiffré avec `ansible-vault`.
- Charger ces secrets dans un playbook avec `vars_files`.
- Lancer un playbook avec `--ask-vault-pass` pour protéger les mots de passe.

