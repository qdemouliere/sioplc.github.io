# Délégation de zone entre un serveur parent et un serveur enfant

Pour réaliser une délégation de zone entre un serveur parent et un serveur enfant, il suffit de se connecter sur le serveur parent.

## 1. Connexion SSH au serveur maître parent

!!! Warning  "Attention"
    Lorsque vous ajoutez les nouveaux enregistrements nécessaires à la mise en place de la délégation de zone, assurez-vous d'être les seuls à éditer le fichier de zone. **Il faut absolument éviter d'éditer le même fichier en simultané !**

```bash
ssh admineleve@ns0.tierslieux86.fr
The authenticity of host 'ns0.sisr.sfda37.fr (172.16.253.10)' can't be established.ECDSA key fingerprint is SHA256:86URTSy4kQ4hHHPu9nTP3oXRAEH9RCTp4/3JHv0qe+g.
Are you sure you want to continue connecting (yes/no/[fingerprint])?y
Warning: Permanently added 'ns0.sisr.sfda37.fr' (ECDSA) to the list of known hosts.
admineleve@ns0.tierslieux86.fr's password:
```

## 2. Ajout des enregistrements nécessaires dans la zone du serveur maître parent

Il est nécessaire d’ajouter les lignes suivantes à la fin du fichier de zone en pensant à incrémenter le numéro de série

```bash
sudoedit /var/cache/bind/db.tierslieux86.fr
```

```bash
$TTL 43200; 	12 heures

sisr.tierslieux86.fr. IN SOA ns0..tierslieux86.fr. postmaster.tierslieux86.fr. (
	2021070601	; Serial
		1D	; Refresh
		1H	; Retry
		1W	; Expire
		3H )	; Negative Cache TTL

; Les points de suspension mettent en évidence que nous sommes en présence d’un extrait de fichier de 
; zone. Ce sont les enregistrements en gras qui doivent être ajoutés à l’existant.
.…

; Déclaration des serveurs faisant autorité pour le sous-domaine. Indispensable à la délégation
tours.tierslieux86.fr.	IN NS	ns0.tours
tours.tierslieux86.fr.	IN NS	ns1.tours

; Enregistrements de type glue ou colle indispensables à la délégation de zone
ns0.tours	IN A	172.16.3.10
ns1.tours	IN A	172.16.3.11
```

??? info "Qu'est-ce les glue records (colle ou enregistrements de raccord) ?"
    Lorsqu'une zone est déléguée à des serveurs dont le nom est dans la zone fille, la résolution DNS se heurte à un problème d'œuf et de poule. Pour trouver l'adresse de ns1.mazone.example, le résolveur doit passer par les serveurs de mazone.example, qui est déléguée à ns1.mazone.example et ainsi de suite... On rompt ce cercle vicieux en ajoutant, dans la zone parente, des données qui ne font pas autorité sur les adresses de ces serveurs (RFC 1034, section 4.2.1). Il faut donc bien veiller à les garder synchrones avec la zone fille[^1]. Si vous voulez en savoir plus, jetez un oeil à [cet article](https://www.afnic.fr/observatoire-ressources/papier-expert/le-dns-ca-colle-ou-ca-ne-colle-pas/) de l'AFNIC.

Enfin, il est nécessaire de tester la validité de la syntaxe des nouveaux enregistrements puis de prendre en compte les modifications du fichier de zone à l’aide des commandes suivantes :

```bash
sudo named-checkonf -z
sudo systemctl reload bind9
```

[^1]: [Définition tirée du site de S. Bortzmeyer et de son article sur la terminologie liée à DNS](https://www.bortzmeyer.org/9499.html) 