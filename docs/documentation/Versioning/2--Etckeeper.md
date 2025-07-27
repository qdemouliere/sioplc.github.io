# Gestion des versions de /etc avec Etckeeper

!!! Info  "Information"
    Cette documentation est issue du [site](https://blog.stephane-robert.info/docs/outils/systeme/etckeeper/) de Stéphane Robert, spécialiste de la culture DevOps et DevSecOps. Elle a ainsi été retranscrite ici après obtention préalable de son accord. 

## 1. Pourquoi mettre en place une gestion de versions du répertoire /etc ?

Le répertoire **/etc** contient l’ensemble des fichiers de configuration système sur une machine Linux. Cela inclut des éléments aussi critiques que la configuration réseau, les comptes utilisateurs, les services démarrés au boot, les règles de sécurité et bien plus encore. Si un fichier est accidentellement modifié ou supprimé, les conséquences peuvent être lourdes : panne réseau, impossibilité de démarrer des services essentiels, voire blocage complet du serveur.

C’est ici que **gérer les versions du répertoire /etc avec Git via etckeeper prend tout son sens**. Voici pourquoi :

* **Historique complet des changements** : chaque modification est enregistrée, datée et associée à un auteur. Cela me permet d’identifier facilement quand et pourquoi un changement a été fait.
* **Retour en arrière simplifié** : en cas de problème, je peux restaurer rapidement une ancienne version d’un fichier ou de l’ensemble du répertoire.
* **Audit facilité** : lors d’une analyse de sécurité, pouvoir montrer un historique clair de la configuration est un vrai plus.
* **Travail en équipe sécurisé** : sur des systèmes gérés par plusieurs administrateurs, cela permet d’éviter des conflits ou des erreurs silencieuses.
* **Sauvegarde efficace** : même si ce n’est pas une sauvegarde complète du système, garder une copie versionnée de /etc est un élément clé d’une bonne stratégie de sauvegarde des fichiers _/etc_.

## 2. Installation d'etckeeper sur Debian

```bash
sudo apt install etckeeper
```

Vérification de la présence d'etckeeper sur le système

```bash
etckeeper --version
```

## 3. Choix du système de contrôle de version (VCS)

Par défaut, Git est utilisé, ce qui est parfait pour la majorité des besoins. Si je souhaite utiliser un autre VCS (comme Mercurial ou Bazaar), je peux modifier la ligne suivante :

```bash
sudoedit /etc/etckeeper/etckeeper.conf
```

```bash
VCS="git"
```

Il est nécessaire (si cela n'a pas été fait automatiquement) d'initialiser manuellement le dépôt Git pour le répertoire _/etc_.

```bash
sudo etckeeper init
sudo etckeeper commit "Commit inital pour /etc"
```
Cela crée une première version (snapshot) de l’état actuel de tous les fichiers de configuration.

## 4. Intégration de etckeeper avec le gestionnaire de paquets Debian APT

L’un des grands avantages d’etckeeper est son intégration automatique avec les gestionnaires de paquets. À chaque installation, mise à jour ou suppression de paquet, un commit Git est généré pour enregistrer les modifications dans_/etc_.

### 4.1 Comment çela fonctionne ?

etckeeper utilise des scripts de hooks qui sont déclenchés automatiquement par le gestionnaire de paquets. À chaque opération sur les paquets, un commit automatique est réalisé si des changements sont détectés.

### 4.2 Exemple pratique

Quand j’installe un nouveau paquet avec _apt_ :

```bash
sudo apt install nginx
```

**etckeeper** va, en coulisses :

* Effectuer un commit avant l’installation pour enregistrer l’état actuel.
* Installer le paquet.
* Effectuer un commit après l’installation pour capturer toute modification apportée à /etc.

Je peux voir les changements avec une simple commande Git :

```bash
cd /etc
sudo git log
```

```bash
commit acd94001bbdb8ed8c0e3196e8c760566d8b546e9 (HEAD -> master)
Author: Stéphane ROBERT <toto@toto.com>
Date:   Sat Apr 26 12:33:37 2025 +0000

    committing changes in /etc made by "apt install nginx"

    Packages with configuration changes:
    +nginx-common 1.24.0-2ubuntu7.3 all

    Package changes:
    +nginx 1.24.0-2ubuntu7.3 amd64
    +nginx-common 1.24.0-2ubuntu7.3 all
```

### 4.3 Activation ou désactivation des commits automatiques

Dans le fichier _/etc/etckeeper/etckeeper.conf_, je peux ajuster cette option :

```bash
sudoedit /etc/etckeeper/etckeeper.conf
```

```bash
COMMIT_AFTER_INSTALL="yes"
```

Si je préfère gérer manuellement les commits, je peux changer ce paramètre en "no".

Grâce à cette automatisation des commits de configuration, je garde un historique précis sans devoir y penser à chaque manipulation de mon système.


## 5. Gestion des métadonnées et des permissions

Quand je met en place une gestion de version de _/etc_ avec etckeeper, je dois aussi penser à préserver les métadonnées des fichiers, comme les permissions, les propriétaires et les groupes. Ces informations sont essentielles pour garantir le bon fonctionnement du système après une restauration.

### 5.1 Métadonnées prises en charge

etckeeper s’occupe automatiquement de préserver les métadonnées essentielles, telles que :

* Les permissions (chmod)
* Les propriétaires (chown)
* Les groupes (chgrp)
* Les dates de modification (dans certaines configurations)

Pour cela, il génère des fichiers spéciaux contenant les métadonnées à chaque commit. Ces fichiers sont stockés dans le dépôt Git sous forme de fichiers supplémentaires, comme :

```bash
.etckeeper
.etckeeper-meta
```

### 5.2 Activation de la gestion avancée

Dans le fichier de configuration _/etc/etckeeper/etckeeper.conf_, je peux vérifier cette option :

```bash
PRESERVE_METADATA="yes"
```

Elle est activée par défaut sur la plupart des distributions. Si elle est désactivée, je risque de perdre des réglages critiques après une restauration.

### 5.3 Bonnes pratiques pour les métadonnées

* Toujours réaliser un commit après une modification importante dans /etc pour éviter d’oublier les changements de permissions.
* Vérifier les fichiers de métadonnées en cas de migration ou de restauration d’un système sur une nouvelle machine.
* Restaurer avec précaution : si je fais un git checkout sur un fichier sensible, je dois m’assurer que ses permissions sont correctement rétablies.

Grâce à cette gestion des permissions et des propriétaires, l’usage d’etckeeper devient fiable même pour des environnements sensibles où la moindre erreur de droit d’accès pourrait compromettre la sécurité du système.

## 6. Sauvegarde et restauration des configurations

L’un des principaux atouts d’etckeeper est de faciliter la sauvegarde et la restauration de la configuration système sans efforts complexes.

### 6.1 Sauvegarder le répertoire /etc

Pour une sauvegarde complète, je m’assure de copier non seulement les fichiers de /etc, mais aussi l’historique Git associé. Je peux utiliser une commande simple comme :

```bash
sudo tar czf etc-backup.tar.gz /etc
```

Cela inclut tout le contenu de /etc, y compris le dossier caché .git qui contient l’historique des changements. Je peux ensuite stocker cette archive sur un serveur de sauvegarde externe ou dans une solution de stockage cloud.

### 6.2 Restaurer la configuration avec etckeeper

En cas de problème, je peux restaurer l’archive sauvegardée :

```bash
sudo tar xzf etc-backup.tar.gz -C /
```

Puis je peux revenir à un état antérieur avec Git :

```bash
cd /etc
sudo git log --oneline
sudo git checkout <id_commit>
```

Cela me permet de restaurer rapidement un fichier spécifique ou l’ensemble du répertoire /etc à une date donnée.

**Exemple : restaurer un fichier spécifique**

Si, par exemple, j’ai cassé ma configuration nginx, je peux restaurer uniquement ce fichier.

```bash
cd /etc
sudo git checkout HEAD^ -- nginx/nginx.conf
```

Je récupère ainsi la dernière version fonctionnelle sans affecter le reste du système.

Grâce à la sauvegarde des fichiers /etc avec etckeeper, je bénéficie d’une solution fiable pour sécuriser mes configurations critiques et réduire le temps de récupération en cas d’incident.

### 6.3 Restaurer des fichiers ou la totalité de /etc avec etckeeper

Avec etckeeper, restaurer une ancienne version de /etc est simple grâce à Git.

**Voir l’historique des changements**

Listez les commits passés :

```bash
cd /etc
sudo git log --oneline
```

Exemple :
```bash
e2d9c24 Ajout de la config SSL nginx
a7d0f91 Commit après update des paquets
1f3c6b2 Commit initial
```

### 6.4 Restaurer tout /etc à un état précédent

Si vous voulez restaurer tout le répertoire /etc à une version antérieure :

```bash
sudo git checkout <ID_du_commit>
```

Par exemple :

```bash
sudo git checkout a7d0f91
```

!!! Warning  "Attention"
    Cela remplace l’ensemble du contenu de /etc avec l’état du commit. Pensez à faire une sauvegarde avant en cas de doute !

### 6.5 Restaurer un fichier spécifique

Si seule une partie doit être restaurée (ex: nginx.conf cassé) :

```bash
sudo git checkout <ID_du_commit> -- chemin/vers/le/fichier
```

Exemple :
```bash
sudo git checkout HEAD^ -- nginx/nginx.conf
```

Cela récupère uniquement l’ancien fichier sans toucher au reste.

### 6.6 Annuler des changements non commités

Si vous avez fait des modifications locales non désirées :

```bash
sudo git checkout -- etc/[<dossier>/]<fichier>
```

Cela annule les modifications non encore commités.
 




