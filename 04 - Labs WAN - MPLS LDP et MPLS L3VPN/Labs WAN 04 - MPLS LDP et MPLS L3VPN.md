---
title: Labs WAN 04 - MPLS LDP et MPLS L3VPN

---

# Labs WAN 04 - MPLS LDP et MPLS L3VPN

## Lab complet : 

Le lab est adapté à ma configuration initiale avant les TP et les adresses IP aussi

<img width="2229" height="1191" alt="lab complet" src="https://github.com/user-attachments/assets/7401e31c-e016-4c4c-ad5e-a022b8cd72ea" />

## Étape 1 : 

### Configuration des routeurs

### R1 : 

```
conf t

mpls label protocol ldp
mpls ldp router-id Loopback0 force

interface fa4/1
mpls ip

interface S6/0
mpls ip

interface g3/0
mpls ip

end
wr
```

### R2 :

```
conf t

mpls label protocol ldp
mpls ldp router-id Loopback0 force

interface fa4/0
mpls ip

interface fa4/1
mpls ip

end
wr
```

### R3 : 

```
conf t

mpls label protocol ldp
mpls ldp router-id Loopback0 force

interface fa4/0
mpls ip

interface s6/0
mpls ip

interface g2/0
mpls ip

end
wr
```

### R4 : 

```
conf t

mpls label protocol ldp
mpls ldp router-id Loopback0 force

interface g3/0
mpls ip

interface g1/0
mpls ip

end
wr
```

### R5 : 

```
conf t

mpls label protocol ldp
mpls ldp router-id Loopback0 force

interface g1/0
mpls ip

interface g2/0
mpls ip

end
wr
```

### Validation étape 1

#### show mpls interfaces
<img width="1230" height="267" alt="01 - show mpls interfaces" src="https://github.com/user-attachments/assets/c4f92379-176d-49eb-98a6-a97eaf48179f" />
Sur R1

#### show mpls ldp neighbor
<img width="1233" height="917" alt="02 - show mpls ldp neighbor" src="https://github.com/user-attachments/assets/7f4d7d80-eb89-49eb-b353-afea42d3bb44" />
Sur R1

#### show mpls ldp bindings 10.2.255.3 32
<img width="1233" height="278" alt="03 - show mpls ldp bindings 10 2 255 3 32" src="https://github.com/user-attachments/assets/bfe31886-cf42-4a20-bbee-0f1506f5b133" />
Sur R1

#### show mpls forwarding-table 10.2.255.3 32
<img width="1232" height="213" alt="04 - R1 show mpls forwarding-table 10 2 255 3 32" src="https://github.com/user-attachments/assets/be973e05-8144-4229-ab27-8cf48884de21" />
Sur R1

#### show mpls forwarding-table 10.2.255.3 32
<img width="1238" height="233" alt="05 - R3 show mpls forwarding-table 10 2 255 3 32" src="https://github.com/user-attachments/assets/bd4c2b89-9609-416f-a599-68a9b0d3e5a3" />
Sur R3

#### traceroute 10.2.255.3 source Loopback0
<img width="1232" height="318" alt="06 - traceroute 10 2 255 3 source Loopback0" src="https://github.com/user-attachments/assets/eebe86df-4172-4d3c-baa9-e1e7fd3bf4f8" />
Sur R1

#### Preuve du traffic Wireshark pendant le ping 10.2.255.3
<img width="3200" height="1910" alt="07 - preuve wireshark pendant ping 10 2 255 3" src="https://github.com/user-attachments/assets/7e7abb34-3934-4d79-affe-e2edd4f29983" />
Sur le lien R1 - R4

---

### Étape 2 : 

### Configuration des routeurs

### R1 : 

```
conf t

ip vrf CLIENT-A
rd 65100:1
route-target export 65100:1
route-target import 65100:1

ip vrf CLIENT-B
rd 65100:2
route-target export 65100:2
route-target import 65100:2

interface fa4/0
description LAN PC1 - Client A
no ip add
ip vrf forwarding CLIENT-A
ip add 192.168.20.254 255.255.255.0
no sh

interface Loopback202
ip vrf forwarding CLIENT-B
ip add 192.168.0.1 255.255.255.0

end
wr
```

### R3 : 

```
conf t

ip vrf CLIENT-A
rd 65NNN:1
route-target export 65NNN:1
route-target import 65NNN:1

ip vrf CLIENT-B
rd 65NNN:2
route-target export 65NNN:2
route-target import 65NNN:2

interface fa4/1
description LAN PC2 - Client A
no ip add
ip vrf forwarding CLIENT-A
ip add 192.168.27.254 255.255.255.0
no sh

end
wr
```

### Validation étape 2 

#### show vrf
<img width="1233" height="221" alt="01 - show vrf" src="https://github.com/user-attachments/assets/37abb6a1-9bcf-463f-b9da-290c73aba67b" />
Sur R1

#### show ip route vrf CLIENT-A et CLIENT-B
<img width="1235" height="1178" alt="02 - show ip route vrf CLIENT-A et CLIENT-B" src="https://github.com/user-attachments/assets/0eccc57e-e730-4980-aa8e-8ff5037d1054" />
Sur R1

#### show ip route 192.168.0.0
<img width="1233" height="154" alt="03- show ip route 192 168 0 0" src="https://github.com/user-attachments/assets/94e5cf69-d69f-47e5-b459-2bf21f042a86" />

Sur R1

#### ping192.168.20.254
<img width="1234" height="330" alt="04 - ping 192 168 20 254" src="https://github.com/user-attachments/assets/27d3961e-b8af-4d06-a71b-00ded8923649" />
Sur PC1

---

## Étape 3 : 

### Configuration des routeurs

### R1 : 

```
conf t

router bgp 65100
neighbor 10.2.255.3 remote-as 65100
neighbor 10.2.255.3 update-source Loopback0
address-family vpnv4
neighbor 10.2.255.3 activate
neighbor 10.2.255.3 send-community extended
exit-address-family

address-family ipv4 vrf CLIENT-A
redistribute connected
exit-address-family
 
address-family ipv4 vrf CLIENT-B
redistribute connected
exit-address-family

end
wr
```

### R3 : 

```
conf t

router bgp 65100
neighbor 10.2.255.1 remote-as 65100
neighbor 10.2.255.1 update-source Loopback0
address-family vpnv4
neighbor 10.2.255.1 activate
neighbor 10.2.255.1 send-community extended
exit-address-family
 
address-family ipv4 vrf CLIENT-A
redistribute connected
exit-address-family
 
address-family ipv4 vrf CLIENT-B
redistribute connected
exit-address-family

end
wr
```

### Validation étape 3

#### show bgp vpnv4 unicast all summary
<img width="1235" height="609" alt="01 - show bgp vpnv4 unicast all summary" src="https://github.com/user-attachments/assets/cc42210e-719e-448b-a48d-64d5746fd196" />
Sur R1

#### show bgp vpnv4 unicast all
<img width="1232" height="590" alt="02 - show bgp vpnv4 unicast all" src="https://github.com/user-attachments/assets/4e6ba6b5-d3b1-4be2-989e-dcfa08e57d87" />
Sur R1

#### show bgp vpnv4 unicast all 192.168.7.0
<img width="1236" height="792" alt="03 - show bgp vpnv4 unicast all 192 168 7 0" src="https://github.com/user-attachments/assets/1cbbd217-0515-4800-8fc4-400d7341289c" />
Sur R1

#### show bgp vpnv4 unicast all labels
<img width="1233" height="398" alt="04 - show bgp vpnv4 unicast all labels" src="https://github.com/user-attachments/assets/b4d9c311-f5fc-42df-8d30-1904ee98c528" />
Sur R1

#### show ip route vrf CLIENT-A
<img width="1230" height="685" alt="05 - show ip route vrf CLIENT-A" src="https://github.com/user-attachments/assets/cef513e4-2b3b-4426-9553-7445e0f632ef" />
Sur R1

#### show ip bgp
<img width="1234" height="205" alt="06 - show ip bgp" src="https://github.com/user-attachments/assets/cab2f61a-b70e-4b2c-8726-f061be358493" />
Sur R2

---

## Étape 4 : 

### Configuration des routeurs

Il n'y a pas de configuration pour cette étape

### Vérification pour l'étape 4

#### ping 192.168.7.10
<img width="1236" height="278" alt="01 - ping 192 168 7 10" src="https://github.com/user-attachments/assets/38fd97b6-6645-4255-b815-be2e63883fab" />
Sur PC1

#### trace 192.168.7.10
<img width="1236" height="393" alt="02 - trace 192 168 7 10" src="https://github.com/user-attachments/assets/96147aa2-801b-4b98-b5da-4632423daa2e" />
Sur PC1

#### ping vrf CLIENT-B 192.168.7.1 source Loopback202
<img width="1234" height="303" alt="03 - ping vrf CLIENT-B 192 168 7 1 source Loopback202" src="https://github.com/user-attachments/assets/6d00e3fc-4e9f-4136-859d-c812de900e36" />
Sur R1

#### ping 192.168.7.10
<img width="1228" height="269" alt="04 - ping 192 168 7 10" src="https://github.com/user-attachments/assets/98b7e3eb-caaa-46fa-a20f-a629074e3108" />
Sur R1

#### ping vrf CLIENT-B 192.168.7.10
<img width="1235" height="269" alt="05 - ping vrf CLIENT-B 192 168 7 10" src="https://github.com/user-attachments/assets/0b772d77-3c0c-48c6-9086-f83924486d09" />
Sur R1

#### show ip cef vrf CLIENT-A 192.168.7.0 detail
<img width="1233" height="236" alt="06 - show ip cef vrf CLIENT-A 192 168 7 0 detail" src="https://github.com/user-attachments/assets/b2126d93-5c2f-4c74-84dd-1f038f412cf5" />
Sur R1

#### Preuve Wireshark pendant ping 192.168.7.10
<img width="3200" height="1904" alt="07 - preuve wireshark R1-R4 pendant ping 192 168 7 10" src="https://github.com/user-attachments/assets/0d6179a5-699a-455f-963d-bcb7b3304eed" />
Sur le lien R1 - R4

#### Preuve Wireshark pendant ping 192.168.7.10
<img width="3200" height="1902" alt="08 - preuve wireshark R3-R5 pendant ping 192 168 7 10" src="https://github.com/user-attachments/assets/39a8b056-82c0-4291-a6f2-07604511315a" />
Sur le lien R3 - R5

#### show forwarding table 10.2.255.3 32 <br> puis shutdown <br> puis show forwarding table 10.2.255.3 32
<img width="1235" height="1706" alt="09 - show forwarding table 10 2 255 3 32 puis sh puis re la commande" src="https://github.com/user-attachments/assets/878d4046-9bdc-4381-9f00-d049ee0ca91c" />

--- 

## Étape 5 : 

### Configuration des routeurs

### R1 : 

```
conf t

ip vrf SERVICES
rd 65100:99
route-target export 65100:99
route-target import 65100:1
route-target import 65100:2

ip vrf CLIENT-A
route-target import 65100:99
ip vrf CLIENT-B
route-target import 65100:99

interface Loopback203
ip vrf forwarding SERVICES
ip address 10.2.203.53 255.255.255.255

router bgp 65100
address-family ipv4 vrf SERVICES
redistribute connected
exit-address-family

ip vrf SERVICES
no route-target import 65100:2

end
clear ip bgp vpnv4 unicast * soft
wr
```

### R3 : 

```
conf t

ip vrf CLIENT-A
route-target import 65100:99
ip vrf CLIENT-B
route-target import 65100:99

end
wr
```

### Vérification de l'étape 5 

#### ping 10.2.203.53
<img width="1234" height="328" alt="01 - ping 10 2 203 53" src="https://github.com/user-attachments/assets/5b10afa9-96e2-4a30-b170-232cae8aaf6b" />
Sur PC1

#### ping vrf CLIENT-A source fa4/1 et CLIENT-B source Loopback202
<img width="1116" height="422" alt="02 - ping vrf CLIENT-A et B source fa4 1 puis Loopback202" src="https://github.com/user-attachments/assets/ca2285b8-1080-4586-99fd-dd70e749b3fb" />
Sur R3

#### show ip route vrf SERVICES
<img width="1233" height="706" alt="03 - show ip route vrf SERVICES" src="https://github.com/user-attachments/assets/c62dc1c3-15d5-41d3-94b7-b96c49d13733" />
Sur R1

La VRF SERVICES importe les RT 65100:1 et 65100:2. Comme les clients A et B utilisent les mêmes adresses, elle reçoit deux routes pour chaque préfixe (192.168.0.0/24 et 192.168.7.0/24), distinguées dans BGP par leur RD (65100:1 et 65100:2). Mais une table de routage ne peut contenir qu’une seule route par préfixe : BGP n’en retient donc qu’une.

Avant correction, la route retenue (best) pour 192.168.7.0/24 était celle du client A (RD 65100:1), et pour 192.168.0.0/24 celle du client B (RD 65100:2, via Loopback202). Les deux chemins ont des attributs identiques (local-pref 100, métrique 0, origine incomplete) : c’est un critère de départage final, sans lien avec le client, qui décide. Conséquence : le trafic de retour du DNS ne joint qu’un seul client par préfixe. PC1 (192.168.0.0/24) échouait car la réponse partait vers le client B, alors que PC2 (192.168.7.0/24) réussissait car elle partait vers le client A.

Pour corriger, j’ai supprimé route-target import 65100:2 dans la VRF SERVICES. Elle ne garde plus qu’une route par préfixe, celle du client A : PC1 et PC2 atteignent désormais le DNS, mais le client B n’a plus de route de retour (le ping depuis Loopback202 échoue). L’isolation entre A et B reste intacte.

On en conclut qu’un extranet est incompatible avec des plans d’adressage qui se chevauchent. Pour servir les deux clients, il faut des adresses uniques par client, du NAT, ou des routes statiques dédiées dans SERVICES.

---

## Questions d'analyse

### 1) Pourquoi les routeurs P n'ont-ils pas besoin de BGP ? Relie ta réponse à l'expérience « BGP-free core » du module 01.

Les routeurs du cœur (routeurs P) n'ont pas besoin de faire tourner BGP car ils n'ont aucune visibilité sur les préfixes IP privés des clients. Dans un cœur MPLS, le transport des paquets s'appuie uniquement sur un protocole de routage interne (IGP comme OSPF ou IS-IS) pour joindre les Loopbacks des routeurs PE, et sur la commutation d'étiquettes (MPLS). C'est le principe du « BGP-free core » : le cœur se contente de commuter des étiquettes d'un saut à l'autre sans se soucier du trafic client encapsulé à l'intérieur.

### 2) Quelle est la différence entre RD et RT ? Peut-on avoir le même RD sur deux VRF d'un même PE ?

- Le Route Distinguisher (RD) est un préfixe de 8 octets ajouté à une adresse IPv4 pour la rendre unique dans le monde VPNv4 (ce qui permet de différencier deux clients qui utilisent la même plage IP privée, par exemple 192.168.1.0/24). Il n'a aucune portée politique de routage.

- Le Route Target (RT) est une communauté étendue BGP qui sert de balise politique pour contrôler l'exportation et l'importation des routes entre les VRF des différents PE.

- Oui, on peut tout à fait configurer le même RD sur deux VRF d'un même PE (bien que ce soit déconseillé pour la clarté du troubleshooting), du moment que les Route Targets (RT) associés permettent de bien cloisonner les flux.

### 3) Explique le PHP et l'étiquette implicit-null. Quel bénéfice pour le PE de sortie ?

- Le PHP (Penultimate Hop Popping) consiste à faire dépiler l'étiquette extérieure de transport par l'avant-dernier routeur (le routeur P juste avant le PE de sortie) au lieu du PE lui-même.

- L'étiquette implicit-null (valeur 3) est affectée par le PE de sortie à son voisin (l'avant-dernier routeur) pour lui signifier : « Retire l'étiquette extérieure avant de m'envoyer le paquet ».

- Le bénéfice pour le PE de sortie est qu'il reçoit directement un paquet avec une seule étiquette (l'étiquette VPN) ou un paquet IP brut (selon le cas), ce qui lui évite d'avoir à effectuer deux lookups (recherches successives) dans sa table de commutation pour dépiler lui-même le transport puis le VPN.

### 4) Pourquoi faut-il deux étiquettes dans un L3VPN ? Que se passerait-il avec une seule ?
Pourquoi les Loopback0 doivent-elles être en /32 pour MPLS ?

- Il faut deux étiquettes : une étiquette extérieure (transport) pour traverser le cœur MPLS de PE en PE, et une étiquette intérieure (VPN) pour indiquer au PE de destination à quelle VRF (et donc à quel client) appartient le paquet reçu.

- Avec une seule étiquette, le cœur MPLS ne saura pas router le paquet si l'étiquette est celle du VPN, ou à l'inverse, le PE de sortie ne saura pas vers quelle VRF diriger le paquet s'il ne reçoit qu'une étiquette de transport globale.

- Pourquoi les Loopback0 doivent-elles être en /32 pour MPLS ?
Les interfaces Loopback0 servent de identifiants stables (Router-ID et destination des tunnels LDP/BGP) pour les routeurs PE. Un masque en /32 garantit une route hôte exacte et unique dans l'IGP, évitant toute ambiguïté ou agrégation de routes dans le cœur MPLS.

### 5) Décris le chemin complet d'un paquet de PC1 vers PC2 : étiquettes imposées, échangées, retirées à chaque saut.

- Sur PE1 (R1) : PC1 envoie un paquet IP. R1 consulte la VRF, trouve la route vers le site distant, lui associe l'étiquette VPN (apprise par BGP) et l'étiquette de transport (attribuée par LDP pour joindre la Loopback du PE distant). Le paquet part avec une pile de 2 étiquettes.

- Sur le routeur P (R2) : Il lit l'étiquette de transport extérieure, effectue un échange d'étiquettes (swap) ou applique le PHP (si avant-dernier saut), et transmet le paquet sans toucher à l'étiquette VPN intérieure.

- Sur PE2 (R3) : Il reçoit le paquet (avec l'étiquette VPN grâce au PHP, ou dépile la dernière). Il lit l'étiquette VPN, identifie la VRF CLIENT-A, retire l'étiquette, et transmet le paquet en IP pur sur l'interface physique vers PC2.

### 6) Comment l'opérateur facture-t-il plusieurs clients sur la même infrastructure ? (indice : VRF, QoS, TC)

L'opérateur mutualise sa structure physique tout en garantissant l'isolation et la qualité de service grâce aux VRF (pour l'étanchéité logique des données), au marquage des classes de service (CoS / DiffServ via le champ TC - Traffic Class, anciennement EXP, dans l'en-tête MPLS), et à des politiques de QoS (Quality of Service) garantissant les débits et la priorisation par client (SLA).

### 7) Que montre la question 5.2 sur les extranets avec des adresses qui se chevauchent ?

Que montre la question 5.2 sur les extranets avec des adresses qui se chevauchent ?
Elle démontre la puissance et la flexibilité du modèle L3VPN/VRF : même si deux clients distincts utilisent exactement les mêmes plan d'adressage privé (par exemple 192.168.1.0/24), les VPN et les politiques d'import/export de Route Targets permettent de les faire cohabiter sur le même réseau opérateur sans aucun conflit de routage, et d'autoriser ou restreindre sélectivement les communications inter-clients (extranet).
