---
title: Labs WAN 02 - VPN IPsec

---

# Labs WAN 02 - VPN IPsec

## Lab complet : 

Le lab est adapté à ma configuration initiale avant les TP et les adresses IP aussi

![lab complet](https://hackmd.io/_uploads/H14uC_oiMg.png)

## Étape 0 : preuve de l'isolement

![01 - preuve de l'isolement show ip route](https://hackmd.io/_uploads/rkLletoozg.png)

![02 - preuve de l'isolement ping](https://hackmd.io/_uploads/B1nlxKojzg.png) 

## Partie A : 

### Configuration des routeurs

### R1 

```
conf t
crypto isakmp policy 10
encryption aes 256
hash sha256
authentication pre-share
group 14
lifetime 86400

crypto isakmp key VPN-Siege-Agence-N address 192.168.23.2

crypto ipsec transform-set TS-AES256 esp-aes 256 esp-sha256-hmac
mode tunnel

ip access-list extended VPN-SIEGE-AGENCE
permit ip 192.168.20.0 0.0.0.255 192.168.27.0 0.0.0.255

crypto map CMAP-WAN 10 ipsec-isakmp
description VPN vers Agence (R3 s6/0)
set peer 192.168.23.2
set transform-set TS-AES256
set pfs group14
match address VPN-SIEGE-AGENCE

interface s6/0
crypto map CMAP-WAN

ip route 192.168.23.2 255.255.255.255 192.168.23.2
ip route 192.168.27.0 255.255.255.0 192.168.23.2

end
wr
```

### R3

```
conf t

crypto isakmp policy 10
encryption aes 256
hash sha256
authentication pre-share
group 14
lifetime 86400

crypto isakmp key VPN-Siege-Agence-N address 192.168.23.1

crypto ipsec transform-set TS-AES256 esp-aes 256 esp-sha256-hmac
mode tunnel

ip access-list extended VPN-AGENCE-SIEGE
permit ip 192.168.27.0 0.0.0.255 192.168.20.0 0.0.0.255

crypto map CMAP-WAN 10 ipsec-isakmp
description VPN vers Siege (R1 s6/0)
set peer 192.168.23.1
set transform-set TS-AES256
set pfs group14
match address VPN-AGENCE-SIEGE

interface s6/0
crypto map CMAP-WAN

ip route 192.168.23.1 255.255.255.255 192.168.23.1
ip route 192.168.0.0 255.255.255.0 192.168.23.1

end
wr
```

### Validation Partie A : 

#### ping 192.168.27.1
Sur PC1
![01 - ping 192.168.27.1](https://hackmd.io/_uploads/S10Hhqiofx.png)

#### show crypto isakmp sa
Sur R1
![02 - show crypto isakmp sa](https://hackmd.io/_uploads/Hk0r25ioMe.png)

#### show crypto ipsec sa
Sur R1
![03 - show crypto ipsec sa](https://hackmd.io/_uploads/SyRH25isfl.png)

#### show crypto session detail
Sur R1
![04 - show crypto session detail](https://hackmd.io/_uploads/BJphncsofg.png)

#### Lien Wireshark entre R1 et R4
![05 - wireshark entre R1 et R4](https://hackmd.io/_uploads/ByyIhqsofl.png)

#### Lien Wireshark entre R1 et R2
![06 - wireshark entre R1 et R2](https://hackmd.io/_uploads/Hk1In9jofg.png)

#### debug crypto isakmp et clear crypto session
Sur R1
![07 - debug crypto isakmp et clear crypto session](https://hackmd.io/_uploads/SykL3qsjzl.png)

---

## Partie B : 

### Configuration des routeurs

### R1

```
conf t

interface s6/0
no crypto map CMAP-WAN
exit
no ip route 192.168.27.0 255.255.255.0 192.168.23.2

crypto ikev2 proposal PROP-AES256
encryption aes-cbc-256
integrity sha256
group 14
crypto ikev2 policy POL-1
proposal PROP-AES256
crypto ikev2 keyring KR-VTI
peer R3
address 192.168.22.2
pre-shared-key Cisco123!
crypto ikev2 profile PROF-VTI
match identity remote address 192.168.22.2 255.255.255.255
authentication remote pre-share
authentication local pre-share
keyring local KR-VTI
dpd 10 3 periodic
crypto ipsec profile IPSEC-PROF-VTI
set transform-set TS-AES256
set pfs group14
set ikev2-profile PROF-VTI

interface Tunnel10
ip address 10.2.10.0 255.255.255.254
tunnel source FastEthernet4/1
tunnel destination 192.168.22.2
tunnel mode ipsec ipv4
tunnel protection ipsec profile IPSEC-PROF-VTI
ip mtu 1400
ip tcp adjust-mss 1360

ip route 192.168.22.2 255.255.255.255 192.168.21.2

router eigrp 200
network 10.2.10.0 0.0.0.1
network 192.168.20.0 0.0.0.255
network 100.64.2.0 0.0.0.63
passive-interface FastEthernet4/0
no auto-summary

end
wr
```

### R3

```
conf t

interface Serial6/0
no crypto map CMAP-WAN
exit
no ip route 192.168.20.0 255.255.255.0 192.168.23.1

crypto ikev2 proposal PROP-AES256
encryption aes-cbc-256
integrity sha256
group 14
crypto ikev2 policy POL-1
proposal PROP-AES256
crypto ikev2 keyring KR-VTI
peer R1
address 192.168.21.1
pre-shared-key Cisco123!
crypto ikev2 profile PROF-VTI
match identity remote address 192.168.21.1 255.255.255.255
authentication remote pre-share
authentication local pre-share
keyring local KR-VTI
dpd 10 3 periodic
crypto ipsec profile IPSEC-PROF-VTI
set transform-set TS-AES256
set pfs group14
set ikev2-profile PROF-VTI

interface Tunnel10
ip address 10.2.10.1 255.255.255.254
tunnel source FastEthernet4/0
tunnel destination 192.168.21.1
tunnel mode ipsec ipv4
tunnel protection ipsec profile IPSEC-PROF-VTI
ip mtu 1400
ip tcp adjust-mss 1360

ip route 192.168.21.1 255.255.255.255 192.168.22.1

router eigrp 200
network 10.2.10.0 0.0.0.1
network 192.168.27.0 0.0.0.255
passive-interface FastEthernet4/1
no auto-summary

end
wr
```

### Validation Partie B : 

#### show crypto ikev2 sa
Sur R1 et R3
![01 - show crypto ikev2 sa](https://hackmd.io/_uploads/rypn69isMx.png)

#### show crypto ipsec sa interface Tunnel10
Sur R1 et R3
![02 - show crypto ipsec sa interface Tunnel10](https://hackmd.io/_uploads/HJah69jjGg.png)

#### show interfaces Tunnel10
Sur R1 et R3
![03 - show interfaces Tunnel10](https://hackmd.io/_uploads/rk6n6csoGe.png)

#### show ip eigrp 200 neighbors
Sur R1 et R3
![04 - show ip eigrp 200 neighbors](https://hackmd.io/_uploads/rJahp5ojGg.png)

#### show ip route 192.168.27.1

Sur R1 et R3
![05 - show ip route 192.168.27.1](https://hackmd.io/_uploads/HkR2Tcjofx.png)

#### trace 192.168.27.1
Sur PC1
![06 - trace 192.168.27.1](https://hackmd.io/_uploads/Hyah6cijGl.png)

#### Preuve de l'ESP sur Wireshark sur le lien R2 - R3
![07 - preuve ESP](https://hackmd.io/_uploads/BkCn69sjGg.png)

#### show ip route 100.64.2.0
Sur R3
![08 - show ip route 100.64.2.0](https://hackmd.io/_uploads/Sy63Tqijzg.png)

---

## Partie C : 

### Configuration des routeurs

### R5

```
conf t
interface Loopback200
ip add 100.64.2.129 255.255.255.192

crypto ikev2 proposal PROP-B2B
encryption aes-cbc-256
integrity sha256
group 14

crypto ikev2 policy POL-B2B
proposal PROP-B2B
crypto ikev2 keyring KR-B2B
peer R4-PARTENAIRE
address 192.168.25.1
pre-shared-key Cisco123B2B!

crypto ikev2 profile PROF-B2B
match identity remote address 192.168.25.1 255.255.255.255
authentication remote pre-share
authentication local pre-share
keyring local KR-B2B
dpd 10 3 periodic

crypto ipsec transform-set TS-AES256 esp-aes 256 esp-sha256-hmac
mode tunnel
crypto ipsec profile IPSEC-PROF-B2B
set transform-set TS-AES256
set pfs group14
set ikev2-profile PROF-B2B

interface Tunnel20
ip unnumbered Loopback0
tunnel source GigabitEthernet1/0
tunnel destination 192.168.25.1
tunnel mode ipsec ipv4
tunnel protection ipsec profile IPSEC-PROF-B2B
ip mtu 1400
ip tcp adjust-mss 1360

ip route 100.64.3.0 255.255.255.0 Tunnel20
end
wr
```

### R4

```
conf t
interface Loopback300
ip add 100.64.3.1 255.255.255.0

crypto ikev2 proposal PROP-B2B
encryption aes-cbc-256
integrity sha256
group 14

crypto ikev2 policy POL-B2B
proposal PROP-B2B

crypto ikev2 keyring KR-B2B
peer R5-ENTREPRISE-N
address 192.168.25.2
pre-shared-key Cisco123B2B!

crypto ikev2 profile PROF-B2B
match identity remote address 192.168.25.2 255.255.255.255
authentication remote pre-share
authentication local pre-share
keyring local KR-B2B
dpd 10 3 periodic

crypto ipsec transform-set TS-AES256 esp-aes 256 esp-sha256-hmac
mode tunnel

crypto ipsec profile IPSEC-PROF-B2B
set transform-set TS-AES256
set pfs group14
set ikev2-profile PROF-B2B

interface Tunnel20
ip unnumbered Loopback0
tunnel source GigabitEthernet1/0
tunnel destination 192.168.25.2
tunnel mode ipsec ipv4
tunnel protection ipsec profile IPSEC-PROF-B2B
ip mtu 1400
ip tcp adjust-mss 1360

ip route 100.64.2.128 255.255.255.192 Tunnel20
end
wr
```

### Validation Partie C : 

#### show crypto ikev2 sa
Sur R5
![01 - show crypto ikev2 sa](https://hackmd.io/_uploads/Sk15yjiozx.png)

#### ping 100.64.2.1 source Loopback100
Sur R1
![02 - ping 100.64.2.1 source Loopback100](https://hackmd.io/_uploads/B11cyojoMg.png)

#### show ip route 100.64.3.0
Sur R5 : 
![03 - show ip route 100.64.3.0](https://hackmd.io/_uploads/rJJc1joifl.png)

#### show crypto ipsec sa interface Tunnel20 | include pkts
Sur R5
![04 - show crypto ipsec sa interface Tunnel20 include pkts](https://hackmd.io/_uploads/H1ycyoiiMx.png)

#### Preuve de l'ESP dans Wireshark
![05 - preuve wireshark esp](https://hackmd.io/_uploads/Hyl5yoooGe.png)

#### Preuve de la non présence d'ICMP dans Wireshark
![06 - preuve wireshark icmp](https://hackmd.io/_uploads/Skeckosjfg.png)

#### ping et shutdown sur le tunnel
SurR5
![07 - ping et shutdown sur le tunnel](https://hackmd.io/_uploads/BJl9ysisfg.png)

## Questions d'analyse

### 1) Pourquoi une SA IPsec est-elle unidirectionnelle ? Combien de SA existent pour un tunnel IKEv2 actif ?

Une SA IPsec porte un seul jeu de clés, un SPI et un compteur anti-rejeu pour un seul sens de circulation. Chaque sens a donc ses propres clés et son propre numéro de séquence, ce qui évite de partager un compteur entre deux émetteurs. Un tunnel IKEv2 actif compte 1 IKE SA (le canal de contrôle) et 2 IPsec SA (une par sens). Ton capture le montre : deux SPI différents (0x153304a1 et 0x36894bd1).

### 2) Quelle est la différence entre phase 1 et phase 2 ? Que protège chacune ?

- Phase 1 (IKE SA) : les routeurs s’authentifient (PSK) et créent un canal sécurisé par Diffie-Hellman (groupe 14). Elle protège les échanges de négociation entre les routeurs, pas les données.

- Phase 2 (IPsec SA) : dans ce canal, ils négocient les SA ESP (AES-256, SHA-256). Elle protège le trafic utilisateur (les paquets des PC).
- 
En IKEv2, les deux étapes sont fusionnées : IKE_SA_INIT + IKE_AUTH créent à la fois l’IKE SA et la première IPsec SA.

### 3) Pourquoi le premier ping de la partie A perd-il des paquets ?

Le tunnel n’existe pas avant le premier paquet « intéressant » (qui correspond à la crypto ACL). Ce paquet déclenche la négociation IKE (phase 1 puis phase 2), qui prend quelques secondes. Pendant ce temps, les premiers paquets sont jetés ou expirent (timeout de 2 s). Une fois les SA créées, les pings suivants passent.

### 4) Dans la partie A, pourquoi faut-il une route hôte vers le pair ? Que se passerait-il sans elle ?

Elle force le trafic IKE/ESP vers le pair à sortir par l’interface qui porte la crypto map. Sans elle, EIGRP aurait appris le chemin vers le pair par un autre lien (R2/R3 dans le sujet), qui n’a pas de crypto map : les paquets IPsec ne seraient pas générés sur la bonne interface, le tunnel ne monterait pas ou le trafic partirait en clair.

### 5) Donne 3 avantages du VTI sur la crypto map pour une entreprise de 50 agences.

- Routage dynamique (EIGRP/OSPF/BGP) dans le tunnel : une nouvelle agence ou un nouveau réseau s’annonce par le protocole, sans toucher à IPsec. Avec une crypto map, il faut modifier les ACL des deux côtés à chaque ajout, et les garder en miroir.

- Configuration plus simple et évolutive : un profil IPsec réutilisable et une interface Tunnel par agence, au lieu d’ACL qui grossissent et d’une crypto map unique difficile à maintenir.

- Supervision et QoS : une interface par tunnel, avec compteurs, état up/down, et possibilité d’appliquer QoS, ACL ou redondance (double tunnel avec bascule rapide). La crypto map n’offre rien de tel.

### 6) Explique le rôle du PFS et du DPD.

- PFS : à chaque renouvellement d’une IPsec SA, un nouvel échange Diffie-Hellman génère des clés indépendantes des précédentes. Si une clé est compromise, les trafics passés et futurs restent protégés.

- DPD (Dead Peer Detection) : envoi périodique de tests (ici toutes les 10 s, 3 essais) pour détecter qu’un pair est mort. Les SA devenues inutiles sont supprimées et le tunnel peut être renégocié ou basculé sur un chemin de secours, au lieu de rester un « trou noir ».

### 7) Pourquoi la route statique de la partie C l'emporte-t-elle sur la route BGP ? Quel risque cela crée-t-il (question C.6) ?

Le routeur choisit la route ayant la distance administrative la plus basse : statique = 1, eBGP = 20. La statique via Tunnel20 l’emporte donc, et le trafic B2B passe dans le tunnel même si BGP annonce aussi le préfixe.
Le risque : si le tunnel tombe, la statique disparaît (interface down) et la route BGP, en clair, prend le relais. Le trafic continue de passer sans chiffrement, et personne ne le remarque puisque les pings répondent toujours. Parade : route statique vers Null0 avec une distance inférieure à celle de la route de secours, ou filtre BGP entrant sur le préfixe du partenaire. Chez toi, sans route de secours, le comportement était fail-closed (0/20 tunnel coupé).

### 8) Pourquoi faut-il réduire le MSS TCP dans un tunnel IPsec ?

IPsec ajoute un surcoût à chaque paquet : nouvel en-tête IP, en-tête ESP, IV, bourrage et authentification (environ 60 à 73 octets). Un paquet TCP de 1500 octets dépasse alors le MTU du lien une fois encapsulé et doit être fragmenté, ou il est jeté si le bit DF est positionné (et si le PMTUD est bloqué, le transfert se fige alors que le ping passe). Avec ip tcp adjust-mss 1360, le routeur réduit le MSS annoncé dans les SYN TCP, de sorte que les hôtes envoient des segments assez petits pour tenir dans le tunnel sans fragmentation, en accord avec ip mtu 1400.

## Rapport d'intervention - TP 02 : VPN IPsec

### Objectif
Sécuriser la liaison entre le siège (PC1) et l'agence (PC2) par IPsec, d'abord en crypto map, puis en VTI avec IKEv2, puis en B2B avec un partenaire.

### Intervention réalisée
- **Étape 0** : preuve de l'isolement (pas de route entre les deux LAN, ping en échec).
- **Partie A (crypto map, IKEv1)** : ISAKMP AES-256 / SHA-256 / DH14 avec clé pré-partagée, transform-set ESP, crypto ACL en miroir, crypto map sur s6/0, routes hôtes vers le pair.
- **Partie B (VTI, IKEv2)** : suppression de la crypto map, proposal / policy / keyring / profile IKEv2, profil IPsec, Tunnel10 avec `tunnel protection`, MTU 1400 et MSS 1360, EIGRP 200 dans le tunnel, DPD.
- **Partie C (B2B)** : tunnel IKEv2 R5 ↔ R4 (Tunnel20, `ip unnumbered Loopback0`), routes statiques vers les blocs du partenaire.

### Vérifications
- Ping PC1 → PC2
- `show crypto isakmp sa`, `show crypto ipsec sa`, `show crypto session detail`
- `show crypto ikev2 sa`, `show crypto ipsec sa interface Tunnel10`
- Voisinage EIGRP 200 et traceroute
- Captures Wireshark : ESP visible, ICMP absent en clair
- Test de coupure du tunnel (comportement fail-closed)

### Problèmes rencontrés
Perte des premiers paquets du ping en partie A : le premier paquet « intéressant » déclenche la négociation IKE, le temps que le tunnel monte.

### Conclusion
Les trois variantes fonctionnent et le trafic est chiffré en ESP sur l'underlay. Le VTI est plus évolutif que la crypto map (routage dynamique, une interface par tunnel). Le B2B reste fail-closed en cas de coupure du tunnel, faute de route de secours.
