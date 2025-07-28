# Serveur DNS maître faisant autorité

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
address 172.16.3.10
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
ns0
```
!!! Info  "Information"
    Le fichier hosts, ancêtre des stubresolver DNS, permet de faire la correspondance entre un nom et une IP. Il est généralement prioritaire sur la résolution DNS (pour modifier l’ordre de préférence, éditez le fichier /etc/nsswitch.conf). Dans ce fichier, il est important de renseigner la correspondance entre votre adresse de boucle locale et un nom. Ainsi, si votre machine sollicite le nom indiqué lors d'un processus particulier, cela la renverra vers l'adresse de loopback.

```bash
sudoedit /etc/hosts
```
```bash
127.0.0.1	localhost
127.0.1.1	ns0.tours.tierslieux86.fr	ns0

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```
Il est nécessaire de redémarrer le serveur pour prendre en compte le changement de nom.

```bash
sudo shutdown -r now
```

## 6. Principes généraux concernant Bind9 et le service DNS.

1. Créer dans le répertoire /var/named/ les zones adéquates à l’aide d’un fichier db.xxxxxx (ou xxxxxx correspond à votre nom de domaine) par exemple, db.tours.btsssio.fr.
2. Initialiser la ou les zones dans /etc/bind/named.conf.local
3. Ajouter ou gérer les options globales du daemon Bind 9 dans /etc/bind/named.conf.options
4. Faire particulièrement attention à la syntaxe. 
5. Penser aux commandes de test et redémarrer ou recharger le service.

Les commandes à connaître absolument sont les suivantes et vous seront utiles tout au long de votre activité :

* dig
* host
* nslookup (sous Windows)
* named-checkconf
* named-checkzone
* service bind9 stop|start|restart|reload|status (fonctionne avec SysVinit, systemd)
* avec systemd, systemctl stop|start|restart|reload|status bind9

## 7. Exemple de configuration d'un serveur DNS maître faisant autorité

!!! Warning  "Attention"
    Il est recommandé de supprimer le contenu des fichiers de configuration par défaut et d'utiliser le contenu fourni ci-dessous. N'oubliez pas que **etckeeper** doit être mobilisé.

```bash
sudo cat /etc/bind/named.conf
``` 

```bash
// This is the primary configuration file for the BIND DNS server named.
// If you are just adding zones, please do that in /etc/bind/named.conf.local

include "/etc/bind/named.conf.options";
include "/etc/bind/named.conf.local";
include "/etc/bind/named.conf.default-zones";
```

Le fichier /etc/bind/named.conf est le fichier de configuration global du service DNS. Il est possible d'y renseigner tous les paramètres de configuration du service.

Sous Debian, il a été décidé dans un souci de lisibilité, de scinder la configuration en plusieurs fichiers. Ces fichiers sont appelés par le fichier de configuration général à l'aide de la directive include.

* Le fichier named.conf.options est utilisé pour déclarer les options de configuration liées à Bind.
* Le fichier named.conf.local sert à déclarer les zones nouvellement créées.
* Le fichier named.conf.default-zones contient les déclarations de fichier de zone concernant la racine DNS et la boucle locale.
* Le fichier named.conf.log n'existe pas mais vous permettra de gérer la journalisation de votre service dans un fichier texte dédié.

```bash
sudo cat /etc/bind/named.conf.options
```

```bash
options {
	// Directory permet de définir où se situe le cache du serveur DNS
	directory "/var/cache/bind";
	
	// Définit le port et la ou les adresses IPv4 d’écoute du service Bind
	listen-on port 53 { 127.0.0.1; 172.16.3.10; };
	
	
	// Forwarders (redirecteurs) permet de définir les serveurs DNS récursifs qui seront sollicités par 
	// Bind lorsqu'il ne sera pas répondre à une requête.
	// forwarders {
	// 	0.0.0.0;
	// };

	// Recursion permet  d'autoriser ou d'interdire la récursivité sur un serveur DNS. Par défaut un
	// serveur DNS faisant autorité ne doit pas être récursif.
	recursion no;

	// Empêche la diffusion de la version du service Bind utilisée. Mécanisme de sécurité qui ralentit
	// un potentiel attaquant à la recherche d’une faille liée à une version précise du service.
	version none;
};
```
```bash
sudoedit /etc/bind/named.conf.local
```

```bash
zone "tours.tierslieux86.fr" {
	type master;
	allow-transfer { 172.16.3.11; };
	file "/var/cache/bind/db.tours.tierslieux86.fr";
};
```

Ce fichier sert à déclarer les zones que vous aurez à gérer. Votre serveur peut-être maître sur une zone ou esclave. La directive file sert à déclarer le fichier de zone contenant les enregistrements liés (SOA, NS, A…). La directive allow-transfer permet de déclarer les serveurs esclaves habilités.

!!! Warning  "Attention"
	Le numéro de série ne doit jamais être choisi au hasard et **systématiquement incrémenté à chaque modification du fichier de zone**. Les guides de bonnes pratiques recommande de choisir la date du jour sous **2025090101** (année/mois/jour/id). Dans le cas d'une architecture maître-esclave si le numéro de série présent sur le maître se retrouve **décrémenté** et inférieur à celui du serveur esclave, **le transfert de zone ne se fera plus !**

```bash
sudoedit /var/cache/bind/db. tours.tierslieux86.fr
sudo chown bind:bind /var/cache/bind/db.tours.tierslieux86.fr
```

```bash
; La directive TTL définie la durée de conservation en cache d'informations concernant cette zone par les 
; serveurs DNS résolveurs. Lorsque ce TTL arrive à expiration les résolveurs vident leur cache et 
; consultent à nouveau le serveur faisant autorité.
$TTL 43200; 	12 heures

; Pour le domaine défini dans le fichier de zone, on déclare le serveur SOA ainsi que le mail de la 
; personne à contacter pour cette zone (sans le @).

; Le numéro de série est important. Souvent, il correspond à la date du jour suivi d'un chiffre ou d'une 
; valeur que l'on incrémente lors des modifications. Le numéro de série d'une zone stockée par un 
; serveur esclave ne doit jamais être supérieur à celle présente sur un serveur maître. Lorsque l'on 
; incrémente le numéro de série, cela signifie aux serveurs esclaves qu'une modification sur la zone a été 
; opérée.

; Les valeurs suivantes permettent aux serveurs esclaves d'obtenir les délais maximums avant de 
; rafraîchir leur zone, de réessayer à contacter le serveur maître en cas d'interruption de service. En cas 
; d'interruption du service sur le maître pendant plus d'une semaine, l'esclave considérera ce dernier 
; comme expiré.

tours.tierslieux86.fr. IN SOA ns0.tours.tierslieux86.fr. postmaster.tours.tierslieux86.fr. (
	2021070601	; Serial
		1D	; Refresh
		1H	; Retry
		1W	; Expire
		3H )	; Negative Cache TTL

; Déclaration des serveurs DNS faisant autorité sur la zone tours.tierslieux86.fr à l’aide de 
; l’enregistrement NS.

tours.tierslieux86.fr.	IN 	NS 	ns0.tours.tierslieux86.fr.
tours.tierslieux86.fr.	IN 	NS 	ns1.tours.tierslieux86.fr.

; Déclaration des correspondances entre un nom de domaine et une adresse IP à l’aide de l
; ’enregistrement A. Il est indispensable de déclarer les correspondances pour les serveurs DNS (nsx).

ns0		IN	A	172.16.3.10
ns1		IN	A	172.16.3.11
www.	IN	A	172.16.3.2

; Exemple d’utilisation d’un alias. Ici extranet.tours.tierslieux86.fr renvoie vers 
; www.tours.tierslieux86.fr. 

extranet IN CNAME www.tours.tierslieux86.fr.
```

!!! Warning  "Attention"
    Ceci est un exemple de fichier de zone à réadapter sans les commentaires (;). Tous les enregistrements présents dans cet exemple ne servent pas forcément dans le TP. Par contre, il est pertinent de rechercher leurs significations sur Internet pour comprendre à quoi ils peuvent servir dans un contexte professionnel.

Mise en place d’une journalisation des évènements du service DNS

```bash
sudoedit /etc/bind/named.conf.log
```

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

Ce fichier permet d'activer la journalisation des évènements pour le service DNS. Vous pouvez préciser le niveau de verbosité avec la directive severity mais aussi la taille maximale du fichier de log. Comme pour le fichier de zone esclave, il est nécessaire de créer un fichier vide de log avec les bonnes permissions au préalable :

```bash
sudo touch /var/log/bind.log
sudo chown bind:bind /var/log/bind.log
```

Enfin, on n’oublie pas de déclarer ce nouveau fichier de configuration dans /etc/bind/named.conf.

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
sudo apparmor_parser -r /etc/apparmor.d/usr.sbin.named
sudo systemctl restart apparmor
```

??? info "named-checkconf"
    La commande named-checkconf permet de vérifier si des erreurs de syntaxe sont présentes et de fournir les éléments ou lignes qui posent problème dans un fichier en particulier.

```bash
sudo named-checkconf -z
sudo systemctl restart bind9
sudo systemctl status bind9
```