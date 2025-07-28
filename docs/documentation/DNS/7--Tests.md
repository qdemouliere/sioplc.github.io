# Tests et diagnostics DNS

!!! Info  "Information"
    Cette partie de la documentation DNS est issue de la production [Mise en oeuvre du système DNS](https://www.reseaucerta.org/content/mise-en-oeuvre-du-dns) publiée par Frédéric Varni et Apollonie Raffalli sur le site du [réseau CERTA](https://www.reseaucerta.org/).

## 1. Les outils de diagnostic dont on doit disposer

Le paquet **dnsutils** (parfois installé par défaut sur le système debian) fournit quelques commandes utiles :

* Pour vérifier les fichiers de configuration globale : named-checkconf et named-checkconf -z

Si la commande ne renvoie rien, c'est qu'il n'y a pas d'erreurs de syntaxe dans les fichiers. 

* Pour vérifier la syntaxe des fichiers de zone : named-checkzone zonename filename

Il ne faut pas toujours se fier au "OK" final mais lire les phrases qui précèdent... 

* Les commandes nslookup, host et dig permettent de vérifier les données relatives à chaque zone.

Mais pour ne pas perdre de temps, il est important de procéder méthodiquement.

## 2. Première batterie de tests : le serveur bind 9 a-t'il correctement démarré ou redémarré ?

!!! Warning  "Attention"
    SAUF s'il y a une erreur de syntaxe dans les fichiers de configuration globaux, le serveur va apparemment démarrer ou redémarrer sans erreur. Pour voir si tout c'est réellement bien passer, il faut lire les fichiers de trace (logs).

Lorsque vous lancez ou relancez le serveur, il est judicieux d'ouvrir une autre console, d'y lancer la commande "**tail -f /var/log/bind.log**" pour lire en direct les fichiers de trace concernant le démon _named_.

Voici par exemple 2 cas où il y a un problème :

**Premier cas - le fichier de zone n'est pas trouvé :**
```bash
Jan 26 16:01:33 ns1 named[21056]: zone tours.tierslieux86.fr/IN: loading from master file /var/named/db.tours.tierslieux86.fr failed: file not found
```

Il n'y a pas ici une erreur de syntaxe, la commande named-checkconf n'aurait rien renvoyée.

**Deuxième cas : le fichier de zone est trouvé mais présente une erreur :**
```bash
Jan 26 16:13:57 ns1 named[21652]: zone tours.tierslieux86.fr/IN: loading from master file /var/named/db.tours.tierslieux86.fr failed: unexpected end of input
```

Une erreur se situe ici dans le fichier de configuration de la zone.

On a le détail des erreurs avec la commande :
```bash
named-checkzone zonename filename /var/named/db.tours.tierslieux86.fr
```

Erreurs courantes :
* les nombreux ";"  des fichiers de configuration globale ;
* les noms DNS pleinement qualifiés dans les fichiers de zone non suffixés par un point ;
* les parenthèses mal placées de l'enregistrement SOA ;
* la non relecture de bind9 lorsqu'un fichier de configuration est modifié et le non vidage du cache.

## 3. Deuxième batterie de tests : vérification des données relatives à chaque zone.

L'utilisation des commandes _nslookup, host ou dig_ vous permettra de vérifier les données relatives à chaque zone.
Nous ne détaillerons ici que la commande dig (Domain Information Groper) qui donne beaucoup de détails... À l'inverse de nslookup, cette commande n'est pas interactive.

```bash
dig www.reseaucerta.org

; <<>> DiG 9.5.0-P2 <<>> www.reseaucerta.org
;; global options:  printcmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 26702
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 3, ADDITIONAL: 3




;; QUESTION SECTION: (1)
;www.reseaucerta.org.           IN      A

;; ANSWER SECTION: (2)
www.reseaucerta.org.    1785    IN      CNAME   strasbourg.reseaucerta.org.
strasbourg.reseaucerta.org. 3585 IN     A       130.79.130.89

;; AUTHORITY SECTION: (3)
reseaucerta.org.        10785   IN      NS      c.dns.gandi.net.
reseaucerta.org.        10785   IN      NS      b.dns.gandi.net.
reseaucerta.org.        10785   IN      NS      a.dns.gandi.net.

;; ADDITIONAL SECTION: (4)
a.dns.gandi.net.        117683  IN      A       217.70.179.40
b.dns.gandi.net.        117683  IN      A       217.70.184.40
c.dns.gandi.net.        117683  IN      A       217.70.182.20

;; Query time: 2 msec (5)
;; SERVER: 192.168.1.1#53(192.168.1.1)
;; WHEN: Wed Jan 28 12:20:18 2009
;; MSG SIZE  rcvd: 187
```

(1) La section QUESTION reprend  la requête émise.\
(2) La section ANSWER donne la réponse à la requête.\
(3) La section AUTHORITY donne les serveurs de noms ayant autorité sur la zone.\
(4) La section ADDITIONAL donne les adresses IP des serveurs de noms autoritaires.\
(5) La section Query time donne le temps de réponse de la requête. Cette valeur indique donc si la réponse est en cache ou pas.

Pour filtrer les résultats, on peut préciser, dans la requête, les différents types de RR :
```bash
dig www.reseaucerta.org NS
dig www.reseaucerta.org A
dig www.reseaucerta.org MX
dig -x 192.168.1.124
...
``` 

L'option _+trace_ de la commande _dig_ permet de faire la recherche en parcourant l'arborescence depuis la racine jusqu'à la réponse.

```bash
dig +trace www.domaine-grp1.com

; <<>> DiG 9.5.0-P2 <<>> +trace www.domaine-grp1.com
;; global options:  printcmd
.                       3600000 IN      NS      A.ROOT-SERVERS.NET.
;; Received 48 bytes from 192.168.1.2#53(192.168.1.2) in 0 ms

domaine-grp1.com.      3600    IN      NS      ns.domaine-grp1.com.
;; Received 72 bytes from 192.168.1.124#53(A.ROOT-SERVERS.NET) in 14 ms

www.domaine-grp1.com. 86400 IN      CNAME      ns.domaine-grp1.com.
ns1.domaine-grp1.com. 86400 IN      A       	192.168.10.10
domaine-grp1.com.     86400 IN      NS         ns.domaine-grp1.com.
;; Received 72 bytes from 192.168.10.10#53(ns.domaine1-grp1.com) in 3 ms
```

Observons maintenant une requête qui fournit une réponse négative.

```bash
dig www.domaine-grp1.com

; <<>> DiG 9.5.0-P2 <<>> www.domaine1-grp1.com
;; global options:  printcmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 9054
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 0

;; QUESTION SECTION:
;www.domaine-grp1.com.          IN      A

;; AUTHORITY SECTION:
domaine-grp1.com.       3600    IN      SOA     ns.domaine-grp1.com. hostmaster.domaine-grp1.com. 20090126 86400 21600 3600000 3600

;; Query time: 3 msec
;; SERVER: 192.168.1.2#53(192.168.1.2)
;; WHEN: Wed Jan 28 15:31:44 2009
;; MSG SIZE  rcvd: 97
```

Il existe un autre type de réponse négative : NODATA  qui indique qu'aucune donnée pour le triplet (nom, type, classe) demandé n'existe ; mais il existe d'autres enregistrements possédant ce nom, mais de  type différent.

Si l'on trouve, après status, le mot clé _SERVFAIL_, il n'y a pas de réponse et il faut revenir à la première batterie de test car le serveur a mal démarré.

!!! Warning  "Attention"
    Cela ne vient pas forcément de votre serveur mais d'un des serveurs interrogés de manière récursive.

!!! Info  "Remarque"
    Attention au cache qui pourrait fournir des réponses qui ne sont plus valables ; la commande rndc flush  permet de supprimer toutes données en cache.

## 4.Troisième batterie de tests : vérification de la non récursivité d'un serveur DNS

Vous devez configurer le client DNS d'un poste n'appartenant pas à votre réseau sur votre serveur DNS qui n'accepte pas la récursivité externe.

Une même requête doit être :

* refusée si demande récursive à partir d'un poste n'appartenant pas au réseau.

```bash
dig www.domaine-grp1.com
; <<>> DiG 9.5.0-P2 <<>> www.domaine-grp1.com
;; global options:  printcmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: REFUSED, id: 20371
;; flags: qr rd; QUERY: 1, ANSWER: 0, AUTHORITY: 0, ADDITIONAL: 0
;; WARNING: recursion requested but not available

;; QUESTION SECTION:
;www.domaine-grp1.com.                        IN      A

;; Query time: 0 msec
;; SERVER: 192.168.1.2#53(192.168.1.2)
;; WHEN: Wed Jan 28 19:06:41 2009
;; MSG SIZE  rcvd: 32
```

* acceptée si provenant du réseau interne.

```bash
dig www.domaine-grp1.com

; <<>> DiG 9.5.0-P2 <<>> www.domaine-grp1.com
;; global options:  printcmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 12560
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 1, ADDITIONAL: 0

;; QUESTION SECTION:
;www.domaine-grp1.com.                        IN      A

;; ANSWER SECTION:
www.domaine-grp1.com. 	86233   	IN   CNAME   ns.domaine-grp1.com.
ns1.domaine-grp1.com. 	86233 	IN      A       192.168.10.10

;; AUTHORITY SECTION:
domaine-grp1.com.     86233   IN      NS      ns.domaine-grp1.com.

;; Query time: 17 msec
;; SERVER: 192.168.1.2#53(192.168.1.2)
;; WHEN: Wed Jan 28 19:14:01 2009
;; MSG SIZE  rcvd: 90
```

## 5.Quatrième batterie de tests : contrôle des transferts de zone

Il suffit en fait de lire les fichiers de trace (logs) lorsque vous lancez pour la première fois votre serveur secondaire ou que vous modifiez le contenu d'un fichier de zone (sans oublier de modifier le numéro de série).

Dans les fichiers de trace du serveur secondaire, vous devriez trouver quelque chose de ce style :

```bash
Jan 29 16:15:26 nsd named[20736]: zone domaine-grp1.com/IN: Transfer started.
Jan 29 16:15:26 nsd named[20736]: transfer of 'domaine-grp1.com/IN' from 192.168.1.2#53: connected using 192.168.10.100#56170
Jan 29 16:15:26 nsd named[20736]: zone domaine-grp1.com/IN: transferred serial 20090126
Jan 29 16:15:26 nsd named[20736]: transfer of 'domaine-grp1.com/IN' from 192.168.10.10#53: Transfer completed: 1 messages, 7 records, 225 bytes, 0.015 secs (15000 bytes/sec)
Jan 29 16:15:26 nsd named[20736]: zone domaine-grp1.com/IN: sending notifies (serial 20090126)
```

Sur le serveur maître :

```bash
Jan 29 16:21:29 ns1 named[8190]: client 192.168.10.100#56170: transfer of 'domaine-grp1.com/IN': AXFR started
Jan 29 16:21:29 ns1 named[8190]: client 192.168.10.100#56170: transfer of 'domaine-grp1.com/IN': AXFR ended
```

Vous pouvez maintenant vérifier que les fichiers de zones sur le serveur secondaire ont bien été créés.

L'erreur fréquente consiste à ne pas accorder le droit d'écrire au groupe système bind sur le répertoire _/var/cache/bind/_. C'est certainement le cas si vous avez les traces suivantes sur le serveur secondaire :

```bash
Jan 29 16:20:23 ns1 named[20970]: dumping master file: /var/cache/bind/slave/tmp-FqJO3iWCxz: open: permission denied
Jan 29 16:20:23 ns1 named[20970]: transfer of 'domaine-grp1.com/IN' from 192.168.10.10#53: failed while receiving responses: permission denied
```

Pour tester le serveur secondaire, vous devez configurer les DNS du client de manière à ce qu'il pointe vers le serveur secondaire.