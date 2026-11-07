# Installation de GLPI sur Ubuntu Server (VMware)

Déploiement d'une plateforme de gestion de parc et de helpdesk (ITSM) **GLPI** sur **Ubuntu Server 26.04 LTS**, avec Apache, MariaDB, PHP et **HTTPS** (certificat auto-signé).

## Environnement

| Composant | Version |
|---|---|
| Hyperviseur | VMware Workstation |
| OS | Ubuntu Server 26.04.1 LTS |
| Serveur web | Apache 2 |
| Base de données | MariaDB 11.8 |
| PHP | 8.5 |
| GLPI | (version) |
| Accès distant | SSH depuis Windows PowerShell |

## Architecture

PC Windows (navigateur + PowerShell) → réseau VMware → VM Ubuntu Server (Apache + PHP + MariaDB + GLPI)

## Étapes

### 1. Préparation de la VM
Création de la VM (2 CPU, 4 Go RAM, 25 Go disque) et accès SSH depuis PowerShell.

![VM](screenshots/01_vmware_vm.png)
![Hostname et IP](screenshots/02_hostname_ip.png)
![SSH](screenshots/03_ssh_powershell.png)

### 2. Installation d'Apache, MariaDB et PHP
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y apache2 mariadb-server php libapache2-mod-php \
php-mysql php-curl php-gd php-intl php-xml php-mbstring php-zip \
php-bcmath php-bz2 php-ldap php-apcu php-cli
```
![PHP et MariaDB](screenshots/04_php_mariadb.png)
![Services](screenshots/05_services.png)

### 3. Base de données
```sql
CREATE DATABASE glpidb CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'glpiuser'@'localhost' IDENTIFIED BY '<MOT_DE_PASSE>';
GRANT ALL PRIVILEGES ON glpidb.* TO 'glpiuser'@'localhost';
GRANT SELECT ON mysql.time_zone_name TO 'glpiuser'@'localhost';
FLUSH PRIVILEGES;
```
![Base de données](screenshots/06_database.png)

### 4. Téléchargement de GLPI
Récupération de la dernière version depuis GitHub, extraction dans `/var/www/glpi`, propriétaire `www-data`.

![Fichiers GLPI](screenshots/07_glpi_files.png)

### 5. Configuration d'Apache
Virtual host pointant vers `/var/www/glpi/public`.

![Config Apache](screenshots/08_apache_conf.png)
![Statut Apache](screenshots/09_apache_status.png)

### 6. HTTPS avec certificat auto-signé
Certificat généré avec OpenSSL (RSA 2048, SAN IP + DNS, validité 825 jours), redirection HTTP → HTTPS, certificat importé dans le magasin racine de Windows.

![Certificat](screenshots/10_certificat_ssl.png)
![Détails](screenshots/11_certificat_details.png)
![GLPI en HTTPS](screenshots/12_glpi_https.png)

### 7. Installation via l'assistant web et sécurisation
Changement des mots de passe par défaut, désactivation des données de démonstration, tâche cron GLPI, snapshot de la VM.

![Dashboard](screenshots/13_dashboard.png)
![Snapshot](screenshots/14_snapshot.png)

## Problèmes rencontrés et solutions

| Problème | Cause | Solution |
|---|---|---|
| `mysql_secure_installation` introuvable | MariaDB 11.8 est sécurisée par défaut sur Ubuntu | Utilisation de `mariadb-secure-installation`, devenue facultative |
| Accès root MariaDB refusé | Mot de passe root hérité d'une ancienne installation | Réinitialisation via `--skip-grant-tables`, puis retour à `unix_socket` |
| Apache inactif | Service arrêté après l'ancienne installation | `systemctl enable --now apache2` |
| Copier-coller impossible dans la console VMware | Limitation de la console | Connexion SSH depuis PowerShell |
| `tar` : fichier introuvable | Mauvais répertoire et session SSH coupée | Retour dans `/tmp` et extraction depuis le bon dossier |
| Commande `openssl` multiligne échouée | Collage incomplet | Exécution sur une seule ligne |
| Données de démonstration dans le Dashboard | Option par défaut de GLPI | Désactivation depuis l'interface |

## Compétences mises en œuvre

Linux (Ubuntu Server), Apache, MariaDB, PHP, SSH, OpenSSL / HTTPS, VMware, GLPI, ITSM, documentation technique.

## Suite du projet

Entités, groupes et profils, gestion des tickets, inventaire avec GLPI Agent.
