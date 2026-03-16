#  Projet NAS Debian - Déploiement Automatisé (Infrastructure as Code)

##  Contexte et Objectifs
L'objectif principal est de mettre en place un serveur NAS robuste et flexible en utilisant le système d'exploitation Debian, privilégiant une approche sans interface graphique pour optimiser les ressources. 

Ce serveur offrira une palette complète de fonctionnalités, notamment le transfert de fichiers via SFTP, l'accès sécurisé via WebDAV, et la création d'un espace partagé avec un Dossier Public. Un aspect central de ce projet est la mise en place d'une gestion multisessions, offrant ainsi la possibilité à plusieurs utilisateurs d'accéder simultanément au serveur NAS.

###  Le Pivot Architectural : De l'Impératif au Déclaratif
Initialement, la gestion des utilisateurs et des dossiers a été prototypée via un script Bash (disponible dans `/v1_bash_imperatif`). Cependant, pour répondre aux standards de l'industrie et garantir l'idempotence, la traçabilité et la scalabilité du serveur, l'intégralité de l'infrastructure a été refactorisée sous **Ansible**. Ce dépôt présente l'infrastructure finale entièrement automatisée (IaC).

---

##  Architecture et Choix Techniques

### 1. Stockage et Tolérance aux pannes
L'infrastructure de stockage repose sur une grappe RAID 5 logicielle (`mdadm`) composée de 3 disques, permettant de survivre à la défaillance physique d'un disque sans perte de données. Pour éviter qu'un utilisateur n'accapare l'espace disque, des quotas stricts ont été configurés au niveau du système de fichiers `ext4` (Soft limit: 4 Go, Hard limit: 5 Go).

### 2. Sécurité et Cloisonnement (SFTP)
Chaque session sera attribuée de manière distincte, garantissant ainsi une confidentialité et une sécurité des données pour chaque utilisateur.
* **Chroot Jail :** Le serveur OpenSSH a été configuré pour emprisonner les utilisateurs du groupe `nas_users` (`ChrootDirectory`). Lorsqu'un utilisateur se connecte, il est verrouillé à la racine de son espace personnel, incapable de remonter dans l'arborescence du système Debian.
* **Restriction des commandes :** L'accès shell standard a été révoqué (`ForceCommand internal-sftp`). Les utilisateurs ne peuvent effectuer que du transfert de fichiers, neutralisant ainsi les risques d'exécution de scripts malveillants.
* **Conflit de permissions :** Pour satisfaire les exigences de sécurité strictes d'OpenSSH (qui refuse un Chroot si le dossier n'appartient pas à root), la racine de chaque prison appartient à `root:root`, tandis qu'un sous-dossier d'écriture dédié a été provisionné pour l'utilisateur.

### 3. Travail Collaboratif (Le Dossier Public)
La mise en place d'un espace partagé collaboratif nécessitait de contourner le comportement par défaut de Linux où le créateur d'un fichier en est le seul propriétaire absolu. 
* **Le mécanisme SetGID :** Le dossier public a été configuré avec le mode octal `2775`. Grâce au SetGID (le `2`), tout fichier déposé par un utilisateur hérite automatiquement du groupe propriétaire parent (`nas_users`).
* **Résultat :** Alice et Bob peuvent créer, lire et modifier les fichiers de l'un et de l'autre de manière transparente, sans conflit de permissions "Access Denied".

### 4. Gouvernance et Supervision
Un point clé de la démarche est la création d'une session administrateur, bénéficiant de privilèges étendus pour superviser l'ensemble du système. 
* **Élévation de privilèges :** Le compte `superviseur_nas` a été intégré au groupe `sudo` (avec mot de passe haché cryptographiquement en SHA-512 par Ansible) pour assurer la maintenance.
* **Monitoring :** Pour faciliter l'administration globale du serveur NAS, une interface de gestion intuitive a été installée. **Cockpit** permet d'avoir une vue d'ensemble des sessions utilisateurs sur le port `9090`.

---

##  Déploiement (Comment utiliser ce dépôt)

### Prérequis
* Vagrant & VirtualBox installés sur la machine hôte.
* Ansible installé.

### Instructions de lancement
1. Cloner le dépôt :
   ```bash
   git clone [https://github.com/sami-farroudj/nas_debian.git](https://github.com/sami-farroudj/nas_debian.git)
   cd nas_debian
   ```
2.Démarrer l'infrastructure virtuelle :
  ```bash
  vagrant up
  vagrant ssh
  ```

3.Lancer l'orchestration complète :
```bash
sudo ansible-playbook /vagrant/nas_deploy.yml
