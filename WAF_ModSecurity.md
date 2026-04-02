# 🛡️ Déploiement d'un WAF ModSecurity de niveau production avec Apache sur Ubuntu 24.04

## Le guide de laboratoire définitif pour le déploiement de ModSecurity et de l'OWASP CRS**

Ce guide fournit une méthodologie complète et éprouvée pour la mise en place d'un pare-feu applicatif web (**WAF**) robuste basé sur **ModSecurity** et l'**OWASP Core Rule Set (CRS)** sur un serveur web Apache fonctionnant sous Ubuntu 24.04. Chaque étape est conçue pour construire une couche de défense sécurisée, maintenable et prête pour la production.


## Pré-requis / contexte

- Distribution : Debian / Ubuntu. 
- Service : Apache 2 installé (`apache2`). 

**But :** partir d’un serveur qui fonctionne déjà, puis le sécuriser.

## Étape 1 – Mettre Apache à jour

**But :** ne pas travailler sur une version vulnérable, appliquer les patchs de sécurité.
1. Mettre l’index des paquets à jour et à appliquer toutes les mises à jour :
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```
