# Mise en place d'un serveur DNS récursif

## 1.  Vérification préalable

Mettez à jour votre serveur

```console
admineleve@template-debianSISR:~$ sudo apt update && sudo apt upgrade
```

Sur votre serveur Debian 12, installez le service de journalisation rsyslog à la place de journalctl. Cela vous permettra de disposer de fichiers de log clairs au format texte situés dans /var/log.

```bash
admineleve@template-debianSISR:~$ sudo apt install rsyslog
```

## 2. Définir les paramètres réseaux du serveur

```bash
admineleve@template-debianSISR:~$ sudoedit /etc/network/interfaces
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
address 192.168.1.10
netmask 255.255.255.0
gateway 192.168.1.254
```

## 3. Définir le serveur DNS récursif à utiliser

```bash
admineleve@template-debianSISR:~$ sudoedit /etc/resolv.conf
```

```bash
nameserver 8.8.8.8
```

!!! Warning  "Attention"
    Lorsque le service Unbound sera opérationnel, remplacer 8.8.8.8 par 127.0.0.1 et ajouter ensuite le second serveur récursif 

## 4. Prendre en compte les modifications des paramètres réseaux

```bash
admineleve@template-debianSISR:~$ sudo systemctl restart networking
```

## 5. Configurer correctement les fichiers /etc/hostname et /etc/hosts

Le fichier **hostname** sert à donner un nom à votre serveur.

```bash
admineleve@template-debianSISR:~$ sudoedit /etc/hostname
```

```bash
dns0
```
!!! Info  "Information"
    Le fichier hosts, ancêtre des stubresolver DNS, permet de faire la correspondance entre un nom et une IP. Il est généralement prioritaire sur la résolution DNS (pour modifier l’ordre de préférence, éditez le fichier /etc/nsswitch.conf). Dans ce fichier, il est important de renseigner la correspondance entre votre adresse de boucle locale et un nom. Ainsi, si votre machine sollicite le nom indiqué lors d'un processus particulier, cela la renverra vers l'adresse de loopback.

```bash
admineleve@template-debianSISR:~$ sudoedit /etc/hosts
```
```bash
127.0.0.1	localhost
127.0.1.1	dns0.tours.tierslieux86.fr	dns0

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```
Il est nécessaire de redémarrer le serveur pour prendre en compte le changement de nom.

```bash
admineleve@template-debianSISR:~$ sudo shutdown -r now
```

## 6. Installer Unbound et les outils d'administration appropriés

```bash
admineleve@dns0:~$ sudo apt install unbound dnsutils tcpdump tmux curl
```

## 7. Configurer Unbound

```bash
admineleve@dns0:~$ sudoedit /etc/unbound/unbound.conf
```

```bash
# Unbound configuration file for Debian.
#
# See the unbound.conf(5) man page.
#
# See /usr/share/doc/unbound/examples/unbound.conf for a commented
# reference config file.
#
# The following line includes additional configuration files from the
# /etc/unbound/unbound.conf.d directory.
include: "/etc/unbound/unbound.conf.d/*.conf"

server:

# Interface d’écoute IPv4 sur le réseau
interface: 192.168.1.10
interface: 127.0.0.1

# Quels réseaux ont le droit de se servir du serveur DNS recursif
# Attention !! Ne pas laisser votre serveur récursif ouvert à tous !
# Allow_snoop autorise le tracage des requetes DNS avec la 
# commande dig +trace
access-control: 192.168.1.0/24 allow_snoop
access-control: 127.0.0.0/8 allow_snoop

# Fichier indiquant les serveurs DNS racines
root-hints: "/var/lib/unbound/root.hints"

# 0n cache la version de Unbound
# et on augmente la securite
hide-version: yes
hide-identity: yes
qname-minimisation: yes


# On autorise l’IPv4
do-ip4: yes

# On journalise et garde une trace des evenements
# Tres important !!!!
logfile: /var/log/unbound.log
verbosity: 1
log-queries: yes
```

Il est important de vérifier ensuite que la syntaxe des lignes contenues dans le fichier de configuration est correcte pour cela il existe la commande **unbound-checkconf**.

```bash
admineleve@dns0:~$ sudo unbound-checkconf
```

Notre serveur récursif va nativement s’adresser aux serveurs faisant autorité sur Internet en sollicitant en premier l’un des serveurs racines. Dans le cas où le serveur récursif serait amené à devoir traiter des domaines locaux qui se trouvent en dehors de l’arborescence DNS réelle (ex : btssio.lan ou epoka.local), il est important de l’indiquer dans le fichier de configuration (/etc/unbound/unbound.conf) de la manière suivante :

```bash
# Comme le domaine btssio.lan est local et en dehors de l’arborescence officielle, il est indispensable
# d'indiquer au récursif qu'il doit contacter les serveurs internes faisant autorité
# sur cette zone plutot que l'un des serveurs racines.

# On precise ici que le domaine local ne gère pas DNSSEC et qu’il faut donc desactiver la verification 
# realisee par Unbound
domain-insecure: "btssio.lan."
private-domain: btssio.lan.

stub-zone:
name: "btssio.lan."
stub-addr: 172.16.20.10
stub-addr: 172.16.20.11
```

On récupère les adresses des serveurs racines et nous les stockons dans le fichier /var/lib/unbound/root.hints. Ce fichier est indispensable et permet au service Unbound de savoir comment contacter le serveur racine le plus proche ou rapide.

```bash
admineleve@dns0:~$ sudo curl --output /var/lib/unbound/root.hints https://www.internic.net/domain/named.cache
admineleve@dns0:~$ sudo chown -R unbound:unbound /var/lib/unbound/
```

On crée le fichier de log spécifique à Unbound.

```bash
admineleve@dns0:~$ sudo touch /var/log/unbound.log
admineleve@dns0:~$ chown unbound:unbound /var/log/unbound.log
admineleve@dns0:~$ sudo systemctl restart unbound
admineleve@dns0:~$ sudo systemctl status unbound
```

!!! Warning  "Attention"
    Sur les systèmes Debian récents, un logiciel de sécurité de type MAC (Mandatory Access Control) nommé AppArmor est activé par défaut. Il surveille entre autres les droits d’accès des différents processus lancés sur le système. Par défaut, AppArmor empêche le service unbound de lire et d’écrire dans le répertoire /var/log/. Il est donc indispensable de changer ces permissions.

```bash
admineleve@dns0:~$ sudoedit /etc/apparmor.d/usr.sbin.unbound
```

```bash
# On autorise le daemon unbound à lire et ecrire dans le fichier /var/log/unbound.log

/var/log/unbound.log rw,
```

On vérifie que le nouveau fichier de configuration de AppArmor ne contient pas d’erreurs puis on redémarre le service.

```bash
admineleve@dns0:~$ sudo apparmor_parser -r /etc/apparmor.d/usr.sbin.unbound
admineleve@dns0:~$ sudo systemctl restart apparmor
```

Si l’on souhaite observer les évènements journalisés :

```bash
admineleve@dns0:~$ sudo cat /var/log/unbound.log
```

Si l’on souhaite observer les derniers évènements enregistrés dans le fichier de log en temps réel :

```bash
admineleve@dns0:~$ sudo  tail -f /var/log/unbound.log
```