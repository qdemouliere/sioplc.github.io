# La résolution inverse

## 1. Qu'est-ce que la résolution inverse ?

Une recherche DNS inversée est une requête DNS pour obtenir le nom de domaine associé à une adresse IP donnée. Cette méthode est le contraire de la recherche DNS directe, plus couramment utilisée, qui consiste à interroger le système DNS pour obtenir une adresse IP.

Il existe des normes de l'Internet Engineering Task Force (IETF) suggérant que chaque domaine devrait être capable de recherche DNS inversée, mais comme les recherches inversées ne sont pas essentielles au fonctionnement normal d'Internet, elles ne constituent pas une exigence stricte. Par conséquent, les recherches DNS inversées ne sont pas universellement adoptées.

Les recherches DNS inversées interrogent les serveurs DNS pour obtenir un enregistrement PTR (pointeur). Si le serveur n'a pas d'enregistrement PTR, il ne peut pas résoudre une recherche inversée. Les enregistrements PTR stockent les adresses IP avec leurs segments inversés, et leur ajoutent « .in-addr.arpa ». Par exemple, si un domaine a pour adresse IP 192.0.2.1, l'enregistrement PTR stockera cette information sous la forme 1.2.0.192.in-addr.arpa.

Dans IPv6, la dernière version du protocole Internet, les enregistrements PTR sont stockés dans le domaine « .ip6.arpa » au lieu de « .in-addr.arpa. » [^1]

## 2. Mise en place d'une résolution inverse

### 2.1 Création d’un fichier de zone pour le réseau 172.16.3.0/24

```bash
sudoedit /var/cache/bind/db.172.16.3
```

```bash
$TTL    43200

3.16.172.in-addr.arpa.       IN      SOA     ns0.tours.tierslieux86.fr. postmaster.tours.tierslieux86.fr. (
                         2021070601; Serial
                         604800         ; Refresh
                        86400         ; Retry
                        2419200         ; Expire
                        604800 )       ; Negative Cache TTL
;
3.16.172.in-addr-arpa.       IN      NS      ns0.tours.tierslieux86.fr.
10.3.16.172.in-addr.arpa.      IN      PTR     ns0.tours.tierslieux86.fr.

2.3.16.172.in-addr.arpa.      IN      PTR     www.tours.tierslieux86.fr.
```

Ce fichier de zone inverse ressemble à un fichier de zone classique. Il sert à mettre en œuvre la résolution DNS inversée. Il est nécessaire que le service Bind dispose des droits afin d’accéder au fichier de zone inverse nouvellement créé.

```bash
sudo chown bind:bind /var/cache/bind/db.172.16.3
```

### 2.2 Déclaration de la zone inverse dans le fichier /etc/bind/named.conf.local

```bash
sudoedit /etc/bind/named.conf.local
```

```bash
zone "3.16.172.in-addr.arpa" {
	type master;
	file "/var/cache/bind/db.172.16.3";
};
```

### 2.3 Prise en compte des modifications

```bash
sudo systemctl reload bind9
```

[^1] Source : [Cloudflare](https://www.cloudflare.com/fr-fr/learning/dns/glossary/reverse-dns/#:~:text=Une%20recherche%20DNS%20invers%C3%A9e%20est,pour%20obtenir%20une%20adresse%20IP.)