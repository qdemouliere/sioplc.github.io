# Mise en place d'un serveur DNS esclave faisant autorité

!!! Warning  "Attention"
    Puisque vous allez être amené à modifier des fichiers de configuration présents dans le dossier /etc, vous devez utiliser **etckeeper** pour faire du versioning.

## 1.  Vérification préalable

Mettez à jour votre serveur

```bash
sudo apt update && sudo apt upgrade
```

Sur votre serveur Debian 12, installez le service de journalisation rsyslog à la place de journalctl. Cela vous permettra de disposer de fichiers de log clairs au format texte situés dans /var/log.

```bash
sudo apt install rsyslog
```

Installez le service Bind 9 et les outils diagnostics DNS

```bash
sudo apt install bind9 dnsutils
```

## 2. Définir les paramètres réseaux du serveur

```bash
sudoedit /etc/network/interfaces
```

```bash
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
allow-hotplug ens192
auto ens192
iface ens192 inet static
address 172.16.3.11
netmask 255.255.255.0
gateway 172.16.3.254
```

## 3. Définir le serveur DNS récursif à utiliser

```bash
sudoedit /etc/resolv.conf
```

```bash
nameserver 86.54.11.100
nameserver 9.9.9.9
```

??? info "Pourquoi choisir ces serveurs DNS récursifs ?"
    Pour rappel, les serveurs DNS faisant autorité gèrent, dans la majorité des cas, des zones publiques appartenant à l'arborescence réelle. Ainsi, ils seront la plupart du temps hébergés dans une DMZ contrairement aux serveurs DNS récursifs qui seront installés dans le LAN. Pour des raisons évidentes de sécurité, il est plus pertinent que les serveurs définis dans le fichier /etc/resolv.conf soient des résolveurs publics plutôt que les résolveurs internes paramétrés précédemment. 

## 4. Prendre en compte les modifications des paramètres réseaux

```bash
sudo systemctl restart networking
```

## 5. Configurer correctement les fichiers /etc/hostname et /etc/hosts

Le fichier **hostname** sert à donner un nom à votre serveur.

```bash
sudoedit /etc/hostname
```

```bash
ns1
```
!!! Info  "Information"
    Le fichier hosts, ancêtre des stubresolver DNS, permet de faire la correspondance entre un nom et une IP. Il est généralement prioritaire sur la résolution DNS (pour modifier l’ordre de préférence, éditez le fichier /etc/nsswitch.conf). Dans ce fichier, il est important de renseigner la correspondance entre votre adresse de boucle locale et un nom. Ainsi, si votre machine sollicite le nom indiqué lors d'un processus particulier, cela la renverra vers l'adresse de loopback.

```bash
sudoedit /etc/hosts
```
```bash
127.0.0.1	localhost
127.0.1.1	ns1.tours.tierslieux86.fr	ns1

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```
Il est nécessaire de redémarrer le serveur pour prendre en compte le changement de nom.

```bash
sudo shutdown -r now
```
## 6. Exemple de configuration d'un serveur DNS esclave faisant autorité

Déclaration de la zone que l’on souhaite transférer depuis le serveur maître sur le serveur esclave :

```bash
sudoedit /etc/bind/named.conf.local

zone "tours.tierslieux86.fr" {
	type slave;
	masters { 172.16.3.10; };
	file "/var/cache/bind/db.tours.tierslieux86.fr";
};
```

Grâce à cette configuration, le serveur esclave réalisera un transfert de zone auprès du serveur maître. Pour que le contenu du transfert soit stocké dans un fichier de zone, il est nécessaire de le créer au préalable avec les permissions adéquates :

```bash
sudo touch /var/cache/bind/db.tours.tierslieux86.fr
sudo chown bind:bind /var/cache/bind/db.tours.tierslieux86.fr
```

## 7. Mise en place de la journalisation

```bash
sudoedit /etc/bind.named.conf.log
```bash

```bash
logging {

	channel bind_log {
	                file "/var/log/bind.log" versions 3 size 100m;
 	               severity info;
	                print-category  yes;
	                print-severity  yes;
	                print-time      yes;
	};

	category default { bind_log; };
};
```

```bash
sudo touch /var/log/bind.log
sudo chown bind:bind /var/log/bind.log
```

```bash
sudoedit /etc/bind/named.conf
```

```bash
// This is the primary configuration file for the BIND DNS server named.
// If you are just adding zones, please do that in /etc/bind/named.conf.local

include "/etc/bind/named.conf.options";
include "/etc/bind/named.conf.local";
include "/etc/bind/named.conf.default-zones";

// Ajout du fichier de parametrage de la journalisation du service BIND
include "/etc/bind/named.conf.log";
```

!!! Warning  "Attention"
    Sur les systèmes Debian récents, un logiciel de sécurité de type MAC (Mandatory Access Control) nommé AppArmor est activé par défaut. Il surveille entre autres les droits d’accès des différents processus lancés sur le système. Par défaut, AppArmor empêche le service Bind 9 de lire et écrire dans le répertoire /var/log/. Il est donc indispensable de changer ces permissions.

```bash
sudoedit /etc/apparmor.d/usr.sbin.named
```

```bash
# On autorise le daemon Bind 9 à lire et ecrire dans le fichier /var/log/bind.log

/var/log/bind.log rw,
```

On vérifie que le nouveau fichier de configuration de AppArmor ne contient pas d’erreurs puis on redémarre le service.

```bash
sudo apparmor_parser -r /etc/apparmor.d/local/usr.sbin.named
sudo systemctl restart apparmor
```

??? info "named-checkconf"
    La commande named-checkconf permet de vérifier si des erreurs de syntaxe sont présentes et de fournir les éléments ou lignes qui posent problème dans un fichier en particulier.

```bash
sudo named-checkconf -z
sudo systemctl restart bind9
sudo systemctl status bind9
```