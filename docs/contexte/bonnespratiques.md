# Référentiel défini par la DSI

## Qu'est-ce qu'un référentiel ?

Un référentiel est une collection de bonnes pratiques sur un sujet donné. Lorsqu'il fait l'objet d'une large diffusion et qu'il est reconnu par le marché, on parle alors de standard. Les référentiels sont donc des boîtes à outils permettant d'extraire la bonne pratique dont on a besoin.

**ex: ITIL 4, Guides de recommandation de l'ANSSI, COBIT.**

La norme se différencie des référentiels dans la mesure où il s'agit d'un document édité par une instance de normalisation indépendante faisant état de l'art d'un sujet donné. La norme peut aussi avoir un niveau de contrainte supplémentaire par rapport au référentiel. Elle peut enfin amener une entreprise à être certifiée via un audit réalisé par un organisme indépendant. C'est l'AFNOR qui assure les certifications en France.[^1]

**ex: ISO-9001, ISO-27001.**

!!! Warning  "Attention !"
    La DSI impose donc à l'ensemble de son personnel de respecter le référentiel présenté ci-dessous

## Extrait des bonnes pratiques réseaux

1. Réalisez **OBLIGATOIREMENT** un schéma physique et un schéma logique du réseau de votre agence (à imprimer).
2. Respectez le processus suivant : maquettage, prototypage, mise en production.
3. Une maquette sous le simulateur Packet Tracer doit obligatoirement être validée individuellement avant la phase de prototypage.
4. Pour réduire les risques Cyber, il est nécessaire de procéder à un découpage du SI administré en zones différenciées en appliquant le principe du moindre privilège.
5. Les fichiers de configuration de tous vos équipements réseaux devront être sauvegardés.
6. Les règles de filtrage définies en production doivent être maintenues. Elles ne peuvent être désactivées que lors d'opérations de test ou de diagnostic.

## Extrait des bonnes pratiques systèmes

1. Sur vos serveurs Debian GNU/Linux, il est obligatoire de mettre en place une gestion de version de vos fichiers de configuration à l'aide du logiciel **etckeeper**.

## Hygiène numérique

1. Les mots de passe de chaque équipe doivent être stockés dans un gestionnaire de mots de passe accessible par l'ensemble de l'équipe (Bitwarden, LastPass, KeePass, etc).
2. Les connexions administrateur aux serveurs de chaque agence devront se faire via une authentification multifacteur à l'aide d'un mot de passe et d'un code à usage unique TOTP (utilisation des smartphones avec une application dédiée type FreeOTP+).

[^1]: [Les référentiels de la DSI, état de l'art, usages et bonnes pratiques](https://www.cigref.fr/cigref_publications/RapportsContainer/Parus2009/Referentiels_de_la_DSI_CIGREF_2009.pdf) publié par le CIGREF
