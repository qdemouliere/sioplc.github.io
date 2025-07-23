# Description de l'infrastructure

## Plan d'adressage IPv4 de l'entreprise

|  			Structure 		         |  			Adresses 			de sous-réseaux 		                                                                                                                  |  			Adresses 			IP des pare-feux 		                                                               |
|---------------------|---------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------|
|  			Agence 			Anvers 		     |  			LAN 1 – 192.168.1.0/24  			 R1-FW1 			– 192.168.11.252/30  			 DMZ 			1 – 192.36.1.0/24  			 WAN 			– 192.36.253.0/24 		                                       |  			In : 			192.168.11.254  			 DMZ : 			192.36.1.254  			 Out : 			192.36.253.10 		                          |
|  			Agence 			Barcelone 		  |  			LAN 			2 – 192.168.2.0/24  			 R2-FW2 			– 192.168.22.252/30  			 DMZ 			2 – 192.36.2.0/24  			 WAN 			– 192.36.253.0/24 		                                       |  			In : 			192.168.22.254  			 DMZ : 			192.36.2.254  			 Out : 			192.36.253.20 		                          |
|  			Agence 			Californie 		 |  			LAN 			3 – 192.168.3.0/24  			 R3-FW3 			– 192.168.33.252/30  			 DMZ 			3 – 192.36.3.0/24  			 WAN 			– 192.36.253.0/24 		                                       |  			In : 			192.168.33.254  			 DMZ : 			192.36.3.254  			 Out : 			192.36.253.30 		                          |
|  			Agence 			Dortmund 		   |  			LAN 			7 – 192.168.4.0/24  			 R4-FW4 			– 192.168.44.252/30  			 DMZ 			7 – 192.36.4.0/24  			 WAN 			– 192.36.253.0/24 		                                       |  			In : 			192.168.44.254  			 DMZ : 			192.36.4.254  			 Out : 			192.36.253.40 		                          |
|  			Agence 			Edimbourg 		  |  			LAN 			5 – 192.168.5.0/24  			 R5-FW5 			– 192.168.55.252/30  			 DMZ 			5 – 192.36.5.0/24  			 WAN 			– 192.36.253.0/24 		                                       |  			In : 			192.168.55.254  			 DMZ : 			192.36.5.254  			 Out : 			192.36.253.50 		                          |
|  			Agence 			Frankfurt 		  |  			LAN 			6 – 192.168.6.0/24  			 R6-FW6 			– 192.168.66.252/30  			 DMZ 			6 – 192.36.6.0/24  			 WAN 			– 192.36.253.0/24 		                                       |  			In : 			192.168.66.254  			 DMZ : 			192.36.6.254  			 Out : 			192.36.253.60 		                          |
|  			Agence 			Galway 		     |  			LAN 			4 – 192.168.7.0/24  			 R7-FW7 			– 192.168.77.252/30  			 DMZ 			4 – 192.36.7.0/24  			 WAN 			– 192.36.253.0/24 		                                       |  			In : 			192.168.77.254  			 DMZ : 			192.36.7.254  			 Out : 			192.36.253.70 		                          |
|  			Agence 			Hong-Kong 		  |  			LAN 			8 – 192.168.8.0/24  			 R8-FW8 			– 192.168.88.252/30  			 DMZ 			8 – 192.36.8.0/24  			 WAN 			– 192.36.253.0/24 		                                       |  			In : 			192.168.88.254  			 DMZ : 			192.36.8.254  			 Out : 			192.36.253.80 		                          |
|  			Siège 			CUB 		         |  			LAN 			Siège – 192.168.20.0/24  			 RS-FWS 			– 192.168.201.252/30  			 DMZ 			Siège – 192.36.250.0/24  			 WAN 			– 192.36.253.0/24  			 Pédago 			– 172.16.28.0/22 		 |  			In : 			192.168.20.254  			 DMZ : 			192.36.250.254  			 WAN : 			192.36.253.254  			 Out : 			172.16.29.199 		 |

## Liste des serveurs présents sur le site du siège

| Serveur           | Configuration réseau                                                          | Description                                                        |
|-------------------|-------------------------------------------------------------------------------|--------------------------------------------------------------------|
| ns0.cub.sioplc.fr | IP : 192.36.250.10/24 Passerelle : 192.36.250.254 DNS : 172.16.20.11, 9.9.9.9 | Serveur DNS maître faisant autorité sur le domaine cub.sioplc.fr.  |
| ns1.cub.sioplc.fr | IP : 192.36.250.11/24 Passerelle : 192.36.250.254 DNS : 172.16.20.11, 9.9.9.9 | Serveur DNS esclave faisant autorité sur le domaine cub.sioplc.fr. |
| www.cub.sioplc.fr | IP : 192.36.250.20/24 Passerelle : 192.36.250.254 DNS : 172.16.20.11, 9.9.9.9 | Serveur web vitrine de l’entreprise CUB                            |

## Description des services présents dans le réseau local de chaque agence

 | Intitulé des services              | Description VLAN | Nombre d’hôtes par service |
|------------------------------------|------------------|----------------------------|
| Production                         | VLAN 5X          | 60 hôtes                   |
| Client 1                           | VLAN 10          | 16 hôtes                   |
| Administration systèmes et réseaux | VLAN 20          | 3 hôtes                    |

**NB :** L'évolution du réseau est un élément primordial à prendre en
compte. Ainsi, en cas de découpage réseau, il est indispensable de
prévoir un plan d'adressage capable d'accueillir au minimum le double
d'équipements par rapport au recensement initial afin d'éviter toute
possibilité de saturation.

## Administration des équipements réseaux

| Matériel                       | Administration                                       | Description                                                                                                                                      |
|--------------------------------|------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------|
| fw.local.agence.cub.sioplc.fr  | 192.168.XX.254/30 depuis 192.168.X.192/28 uniquement | Le pare-feu de l’agence doit être administré uniquement sur son interface IN (réseau d’interco) depuis le VLAN d’Administration.                 |
| sw0.local.agence.cub.sioplc.fr | 192.168.X.205/28 depuis 192.168.X.192/28 uniquement  | Le switch cœur de réseau L3 doit disposer d’une configuration réseau dans le VLAN d’Administration uniquement et être administré depuis ce VLAN. |
| sw1.local.agence.cub.sioplc.fr | 192.168.X.204/28 depuis 192.168.X.192/28 uniquement  | Le switch de distribution doit disposer d’une configuration réseau dans le VLAN d’Administration uniquement et être administré depuis ce VLAN.   |

## Gestion des ports sur les commutateurs

| Ports des commutateurs | Attribution                                                                                                   |
|------------------------|---------------------------------------------------------------------------------------------------------------|
| Les ports 1 à 10       | Réservés aux équipements dit « terminaux » (PC, téléphone, PC portable).                                      |
| Les ports de 11 à 18   | Réservés aux serveurs                                                                                         |
| Les ports de 19 à 24   | Réservés à l’interconnexion avec d’autres équipements réseaux (commutateurs, routeurs, pare-feu, borne-wifi). |

!!! info "Pourquoi cette approche ?"
    L’intérêt d’appliquer une telle méthodologie est d’avoir une rigueur dans l’organisation du réseau permettant de savoir très rapidement repérer la fonction d’un port sur un commutateur.
