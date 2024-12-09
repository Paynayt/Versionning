>[!info] Qu'est ce que le routage
> Le routage est un processus de sélection de chemin par les routeurs pour diriger les paquets vers la destination requise
> Il existe différents types de routage : 
> - Le routage statique
> - Le routage dynamique
> 
> >[!todo] Routage direct
> On appelle routage direct lorsqu'un routeur est connecté à deux réseaux distincts **directement**

# Routage statique :
^c96666

>[!info] En quoi ça consiste ?
>Le routage statique consiste à entrer toutes les routes dans chaque routeur manuellement.

Prenons comme exemple l'infrastructure suivante : 

![[Exemple_routage_statique.png|Infra]]

Ici nous avons 5 réseaux délimiter par des routeurs
Avec la configuration actuelle il n'y a pas de liaison entre les réseaux.
Nous devons donc commencer par **attribuer** les IP aux **interfaces** :
```bash
Router 1 :
enable 
config terminal 
	interface gigabitEthernet 0/0/0
		ip address 192.168.1.1 255.255.255.0
		no shut
	interface serial 0/1/0
		ip address 192.168.2.1 255.255.255.0
		no shut
Router 2 :
enable 
config terminal 
	interface serial 0/1/0
		ip address 192.168.2.2 255.255.255.0
		no shut
	interface serial 0/1/1 
		ip address 192.168.4.1 255.255.255.0
		no shut
	interface gigabitEthernet 0/0/0
		ip address 192.168.3.1 255.255.255.0
		no shut
Router 3 :
enable 
config terminal 
	interface serial 0/1/0
		ip address 192.168.4.2 255.255.255.0
		no shut
	interface gigabitEthernet 0/0/0
		ip address 192.168.5.1 255.255.255.0
		no shut
```

Ceci fait vous êtes désormais capable de ping votre passerelle (après avoir attribué les adresses IP sur les pc).
Par exemple vous pouvez ping les deux passerelles du **Routeur3** depuis le **PC2** :
```bash 
ping 192.168.4.2
>>> Sucess
ping 192.168.5.1
>>> Sucess
```

Afin de pouvoir joindre les pc entre eux il faut mettre en place les routes manuellement :
```bash
Router 1 :
enable 
config terminal
	ip route 0.0.0.0 0.0.0.0 192.168.2.2
Router 2 :
enable 
config terminal 
	ip route 
Router 2 : 
enable 
config terminal 
	ip route 192.168.1.0 255.255.255.0 192.168.2.1 
	ip route 192.168.5.0 255.255.255.0 192.168.4.2
Router 3 :
enable 
config terminal 
	ip route 0.0.0.0 0.0.0.0 192.168.4.1
```

Ceci fait les pc peuvent se joindre entre eux.

>[!info] A savoir
>Le **premier** terme après `ip route` est le **réseau à joindre**
>- On peut noter `0.0.0.0` pour initier une route par défaut 	
>
>Le **deuxième** est le **masque de sous réseau**
>- On peut noter `0.0.0.0` pour le masque par défaut
>
>Le **troisième** est l'interface d'entrée sur le routeur _en face_ 
>- Pour le routeur 1 on note `192.168.2.2`
>
>`show ip route` : affiche toutes les routes

>[!check] Les avantages de routage statique sont : 
>- Stabilité
>- Facilité de mise en place pour un petit réseau 

>[!failure] Inconvénients :
>- Demande énormément de temps de mise en place pour un vaste réseau
---
## Table de routage :
###### Exemple de table de routage : 
Visible en utilisant la commande `show ip route` 

|Liaison|Réseau| 
|-|-|
|**S**|*0.0.0.0/0 [1/0] via 192.168.4.1*| 
|**C**|*192.168.4.0/24 is directly connected Serial 0/1/0*
|**L**|*192.168.4.2/32 is directly connected,Serial 0/1/0 192.168.5.0/24 is variably subnetted, 2 subnets, 2 masks*
|**C**|*192.168.5.0/24 is directly connected, GigabitEthernet 0/0/0*
|**L**|*192.168.5.1/32 is directly connected, GigabitEthernet 0/0/0*

Signification des codes : 
- **S** : Static
- **C** : Connected : pour les réseaux directement connectés
- **L** : Local : adresse des interfaces des réseaux directement connectés 
----
# Routage dynamique :

>[!info] Qu'est ce que le routage  dynamique ?
>Le routage dynamique consiste à ordonner aux routeurs de transmettre leur table de routage aux routeurs adjacents
>Il existe deux familles pour le routage dynamique : 
>- **RIP**
>- **OSPF**
>
>Ces deux méthodes de routage fonctionnent de manière similaire mais comportent des différences.
>Les deux méthodes utilisent le *interior gateway protocol* (**IGP**)

> A noter qu'il existe une méthode nommée RoolRoolBin (*Tourniquet*) qui consiste à connecter 4 routeurs entre eux pour créer un équilibrage de charge
## RIP : 

>[!info] Qu'est ce que c'est ?
>Le *Routing Information Protocol* (**RIP**) utilise la métrique (vecteur de distance) pour déterminer le chemin le plus court afin de l'utiliser pour envoyer un paquet
>Il existe deux protocole utilisant le RIP :
>- Ripv1 
>- Ripv2

### Ripv1 :

RIPv1 est un protocole normalisé lancé en 1988, il comprend différentes spécificités :

- Ipv4 uniquement dû à l'ancienneté du protocole
- Prend en charge uniquement les classes full de masque de sous réseau c'est à dire :
	- 255.255.255.0
	- 255.255.0.0
	- 255.0.0.0
- Les routeurs échangent en broadcast toutes les 30 secondes en attendant une réponse des routeurs adjacents
	 - Si 3 échanges et pas de réponse le routeur concerné est déclaré comme **dead** et est supprimé de la table de routage
- Pour déterminer le chemin le plus court, le protocole utilise l'algorithme de Ford Belmann
- La **distance administrative** de RIPv1 est de **120**
- La métrique est le nombre de saut effectué pour ce protocole

>[!failure] Attention !
>**RIPv1** permet seulement de faire un maximum de **15 sauts**

Les désavantages de Ripv1 sont :
- Pas de VLSM
- Le protocole **consomme beaucoup de bande passante**
- La **convergence** des routeurs est **lente**. *La convergence c'est la mise à jour de la table de routage lors d'un changement d'état*

### RIPv2 :

>**Ripv2 est la deuxième version du protocole RIP** lancée en 1994. Globalement il garde les même spécificités que son prédécesseur tout en comptant certaines améliorations.

Les changements : 
- Prise en charge du VLSM
	- Cela signifie que le protocole prend en charge les longueurs variables de masque de sous-réseaux
- Les messages de mise à jour sont désormais transmis en **Multicast** :
	- à l'adresse `224.0.0.9`
	- UDP : 520
- 25 routes maximum par message
- Possibilité de crypter les messages de mise à jour en **MD5**
- Transmission de routes découvertes par un moyen externe à RIP et redistribution de ces dernières indiquées comme routes externes 
	%%route tag ? Cohabitation avec OSPF ?%% 
- Permet d'indiquer au protocole de prendre en charge tout les sous-réseaux
- Summerisation des réseaux
	- Activé par défaut
	- Cela signifie qu'il est possible d'indiquer au protocole de prendre en compte les sous-réseaux sans avoir à les indiquer manuellement. Par exemple on indique `192.168.0.0 255.255.0.0`, RIPv2 prendra donc en compte `192.168.1.0 255.255.255.252` mais aussi `192.168.1.8 255.255.255.252`.
	- `no auto-summery` pour la désactiver
%%Mettre les commandes de summerisation%%

Globalement RIPv2 comporte les **avantages de RIPv1** tout en **palliant ses défauts** hormis la mise à jour "lente". A noter qu'il existe aussi RIPv3 concernant l'IPv6 uniquement, il comprend les mêmes fonctionnalités que RIPv2. %%à verifier%%
#### Commandes 

`router rip` *active rip*
	`version 2`
	`network 'réseau1'`  *déclaration réseau participant à Ripv2*
	`network 'réseau2'`
	`network 'réseau3'`
	`no auto-summery`
	`passive-interface 'interface'` *interface ne participant pas*
`show ip protocols` *Affiche tout ce qui concerne les protocoles*
`show ip route dynamic | begin gateway`
`show ip rip database` *database*
`debug ip rip` *log de rip en direct (réception paquets etc.)*


## EIGRP 

>[!info] Qu'est ce que EIGRP ?
>EIGRP est l'enfant de IGRP. IGRP est un protocole mais dont Cisco est propriétaire. Il est plus complexe que RIP ou OSPF car il utilise des technologies des deux protocoles. 
>- Vecteur de distance mais en prenant aussi en compte l'état de lien. 


## OSPF 

>[!info] Qu'est ce que c'est ?
> Il s'agit d'un protocole de routage dynamique comme RIP mais avec une métrique différente.
> *Open Shortest Path First* (**OSPF**) utilise l'**état de lien** pour déterminer le chemin le plus **rapide**. Plus il y a de bande passante, donc de débit, plus le **coût** sera bas. **OSPF** calcule le chemin le plus économe niveau **coût** et donc le chemin le plus rapide.

OSPF a plusieurs spécificités :
- Utilisation du multicast ` 224.0.0.5`
- Concept de Zones 
- Vérification du réseau 
- Différents types de paquets
- Structure de données
- Utilisation Etat de Lien

#### Concept de Zones OSFP

Le protocole OSPF utilise plusieurs zones pour fonctionner. La première zone, unique, est la Zone 0 dite BackBone *(colonne vertébrale)*. Toutes les autres zones, dites zones multiples, doivent se rattacher à la Zone 0.

Un **ABR** *(Area Border Router)* est un routeur qui s'occupe de la jonction inter-zones.
Il reçoit des updates en cas de changements sur les zones adjacentes.

L'intérêt d'avoir plusieurs zones est de segmenter. En cas de problème tout le réseau ne doit pas recalculer la LSDB.

A noter qu'on dit qu'il s'agit d'un système **scalable**. Cela signifie que lorsqu'une zone est en difficulté cela n'influe pas autres zones.



#### Vérification du réseau 
Pour effectuer la vérification de l'état du réseau le protocole OSPF indique aux routeurs d'envoyer 5 types de paquets différents : 

- Hello *indique aux routeurs adjacents que l'émetteur est présent*
- DBD  *description base de données des routeurs*
- LSR  *demande de l'état de lien*
- LSU  *link state upgrade mise à jour d'état de lien*
- LSA  *accusé de réception état de lien*

Ces échanges se font en clair basiquement mais possibilité de **chiffrer en MD5**

#### Structure de données
- Base de données sur le **voisinage** *Table voisin*
	- Unique pour chaque routeur
	- Liste tous les routeurs voisins
	- `show ip ospf neighbor` : observer la table
- LSDB *Link Stage Data Base *Table état de lien*
	- Identique sur tout les routeurs d'une même zone
	- Permet de renseigner les routeurs d'une même zone sur cette même zone
- Base de données de transmissions *Table de routage* 
	- `show ip route`
	- table routage classique
	- O devant et plus R ou L


#### Etapes état de lien 
1. Etablissement des contiguités de voisinage
	- Les routeurs OSPF voisins doivent se reconnaître entre eux
	- Dès qu'un routeur découvre son voisin en OSPF, la relation de voisinage se met en place
2. Echange d'annonces à état de liens (LSA : *Link State Advertisements*)
	- Bande passante élevé = Coût Faible 
	- Chaque routeur transmet son message LSA aux voisins
	- La base de donnée peut donc être créée
3. Création de la LSDB *Link Stage Data Base*
	- Recensement de tout les messages LSA au sein de la LSDB 
4. Execution de l'algo SPF
	-  Le premier routeur crée l'arborescence SPF
	- Une colonne pour le réseau à atteindre, une pour le chemin le plus coût avec les noms des routeurs puis une colonne avec le coût du chemin
5. Choisir le meilleur itinéraire
	- Routes par défaut (statique) sont prioritaires

>Algorithme SPF de Dijkstra : 
>- Permet de calculer le coût d'une route
>- L'algorithme prend une table de routage pour créer une arboressence des coûts

#### Désignation routeur

L'ID d'un routeur est composé de 32 bits (type ipv4) et peut être créée manuellement ou prise automatiquement (interface avec l'IP la plus "haute" en tant que ID)

Routeur avec le routeur ID le plus élevé est nommé DR

Sélection DR ou BDR :
- DR : Designed router : les routeurs principaux
- BDR : Backup Designed router : routeurs de backup aux principaux

>[!info] Deux versions de OSPF
>Il existe 2 versions du protocole OSPF : 
>- OSPFv2 pour l'IPv4
>- OSPFv3 pour l'IPv6


#### Commandes 

`router ospf 10` : définit l'ID du processus à 10 | *pas d'impact, à réaliser en enable*
	`?` : plus de commandes
	`router-id 'id'` : mettre manuellement le router ID
	`end` : retourner en mode enable

`router ospf 'id'`
	`'network-address' 'wildcard-mask' 'area-id'`
		`network-address` *classique*
		`wildcard-mask` *masque inversé* 
			*exemple :
				subnetmask : `255.255.255.0`
				wildcard-mask : `0.0.0.255`
				Pas besoin de déclarer les interfaces*
		`area-id` : de base area 0
	Possibilité de faire une `passive-interface`
	`auto-cost reference-bandwidth 'Mbps'` :
		100 de base 
		1000 pour du 1G
		10 000 pour du 10G

Possibilités sur une interface :
- `ip ospf 'process-id' 'area-id'` 
- `show ip ospf interface 'interface'` : voir les spécificités d'une interface ainsi que DR ou BDR
- `clear ip ospf` : recrée l'arborescence 
- `default-information originate` : route par déaut
-  `show ip ospf database`


## Notion de ClockRate

Le ClockRate (*horloge*) spécifie sur une interface le montant de bits qui peut être transmis en une certaine période (*bande passante théorique*).
 On dit qu'un numModem est lorsqu'un DCE et DTE sont ralliés ensembles pour former un câble Serial.

- `clockrate 'debit'`  : définit le clockrate *(sur l'interface côté DCE)*
- `no clockrate` : supprime le clockrate définit


%% IS-IS à developper / demander prof ? déprécié ?%%

%% 
- EGP : exterior gateway protocol
	- Routage D'AS : BGP
		- protocole BGP
- équilibrage de charge : 
	- RoolRoolBin : tourniquet : 4 routeurs qui équilibrent le trafic entre eux
%%

Une modification ici .


>Sources :
>https://cisco.goffinet.org/ccna/ospf/election-dr-bdr-ospf/
>https://cisco.goffinet.org/ccna/rip/routage-dynamique-ripv2/