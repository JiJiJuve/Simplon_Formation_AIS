# TP3 – Authentification par clé SSH


## Objectifs

- Générer une paire de clés SSH (publique/privée).
- Transférer la clé publique sur un serveur distant Linux.
- Forcer l’authentification par clé sur le serveurns Linux.
- Comprendre le rôle de `ssh-agent` sous Windows.

---

## I. Génération de la clé SSH

### 1. Commande utilisée

Sur le poste client (Windows/Linux), j’ai généré une paire de clés RSA 4096 avec :

```bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa
```

- `~/.ssh/id_rsa` : clé privée (à ne jamais partager).
- `~/.ssh/id_rsa.pub` : clé publique, à copier sur le serveur.

---

## II. Dépôt de la clé publique sur le serveur distant

### 1. Méthode avec `ssh-copy-id`

J’ai copié la clé publique sur le serveur avec la commande suivante :

```bash
ssh-copy-id -i ~/.ssh/id_rsa.pub utilisateur@serveur
```

Cette commande crée automatiquement le dossier `~/.ssh` sur le serveur, ajoute la clé dans `authorized_keys` et ajuste les permissions.

### 2. Méthode manuelle (alternative)

Si on le fait manuellement sur le serveur :

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
cat id_rsa.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

---

## III. Configuration du serveur pour forcer l’authentification par clé

Sur le serveur Linux, j’ai édité le fichier `/etc/ssh/sshd_config` :

```bash
sudo nano /etc/ssh/sshd_config
```

J’ai modifié (ou vérifié) les lignes suivantes :

```text
PubkeyAuthentication yes
PasswordAuthentication no
```

Ensuite, j’ai redémarré le service SSH :

```bash
sudo systemctl restart ssh
```

Le serveur ne permet plus la connexion par mot de passe et exige une authentification par clé.

---

## IV. Validation du fonctionnement

### 1. Test de connexion

Depuis le client, j’ai testé la connexion :

```bash
ssh utilisateur@serveur
```

La connexion s’est établie **sans mot de passe**, uniquement à l’aide de la clé privée, ce qui confirme que la configuration est correcte.

### 2. Réponses aux questions

- **Que se passe-t-il si l’on supprime la clé privée sur le client ?**  
  La connexion par clé ne fonctionne plus, car on ne possède plus la clé privée pour prouver l’identité. Si `PasswordAuthentication` est resté désactivé, la connexion est impossible jusqu’à remise en place de la clé privée ou réactivation d’une autre méthode.

- **Comment réactiver l’authentification par mot de passe en cas de besoin ?**  
  Sur le serveur, il faut modifier `/etc/ssh/sshd_config` :

  ```text
  PasswordAuthentication yes
  ```

  Puis redémarrer le service SSH :

  ```bash
  sudo systemctl restart ssh
  ```

---

## V. Bonus : Activation et utilisation de ssh-agent

### 1. Rôle de ssh-agent

`ssh-agent` est un service qui garde la clé privée en mémoire (après déverrouillage avec la phrase de passe) afin de ne pas avoir à la saisir à chaque nouvelle connexion SSH. Il permet un usage plus pratique et sécurisé des clés SSH, surtout dans des environnements avec de nombreuses connexions ou scripts.

### 2. Sur Windows (client SSH natif)

J’ai activé et utilisé `ssh-agent` comme suit :

```powershell
# Activer le service
Get-Service ssh-agent | Set-Service -StartupType Automatic

# Démarrer le service
Start-Service ssh-agent

# Charger ma clé privée
ssh-add $env:USERPROFILE\.ssh\id_rsa
```

Après cela, les connexions SSH ne demandent plus la phrase de passe tant que `ssh-agent` est en fonction.

### 3. Sur PuTTY

L’équivalent de `ssh-agent` sous Windows pour PuTTY est **Pageant** :

- Lancer Pageant.
- Charger la clé privée (format PuTTY `.ppk`).
- Configurer PuTTY pour utiliser cette session ; la clé sera utilisée automatiquement sans redemander la phrase de passe.

### 4. Sur MobaXterm

MobaXterm intègre un agent SSH. Il suffit de :

- Charger la clé privée dans l’agent SSH de MobaXterm.
- Établir une session SSH normale ; MobaXterm utilise alors la clé stockée dans l’agent, ce qui évite de retaper la phrase de passe à chaque connexion.

---

## Conclusion

Ce TP permet de comprendre et mettre en œuvre l’authentification par clé SSH sur un serveur Linux, de sécuriser le serveur en désactivant le mot de passe et d’optimiser l’usage avec `ssh-agent`, notamment sous Windows. Les mêmes principes s’appliquent pour de nombreux services (Git, déploiement, tunnel, etc.).
```

