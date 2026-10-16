---
title: Labs WAN 03 - DMVPN

---

# Labs WAN 03 - DMVPN

## Lab complet 

Le lab est adapté à ma configuration initiale avant les TP et les adresses IP aussi

![lab complet](https://hackmd.io/_uploads/rk3YWyaozg.png)

## Étape 0 : Préparation

### Configuration du routeur

### R5 : 

```
conf t

interface Loopback50
ip add 10.2.50.1 255.255.255.0
no shutdown

end
wr
```

![01 - ping 10.2.255.4 source Loopback0](https://hackmd.io/_uploads/HJX0-Jpofx.png)

![02 - ping 10.2.255.5 source Loopback0](https://hackmd.io/_uploads/HJXAWk6jMe.png)

![03 - show ip route 10.2.50.0](https://hackmd.io/_uploads/ryQ0ZJTofl.png)

![04 - ping 192.168.0.10](https://hackmd.io/_uploads/HJm0WyaoGe.png)

## Étape 1 : 

### Configuration des routeurs

### R1 : 

```
conf t

interface Tunnel100
ip add 10.2.20.1 255.255.255.0
no ip redirects
ip mtu 1400
ip tcp adjust-mss 1360
tunnel source Loopback0
tunnel mode gre multipoint
tunnel key 100
ip nhrp network-id 100
ip nhrp authentication dmvpnN
ip nhrp map multicast dynamic
no ip split-horizon eigrp 200

exit

router eigrp 200
network 10.2.20.0 0.0.0.255
network 100.64.2.0 0.0.0.63
network 192.168.20.0 0.0.0.255

end
wr
```

### R3 : 

```
conf t

conf t
interface Tunnel100
ip address 10.2.20.3 255.255.255.0
no ip redirects
ip mtu 1400
ip tcp adjust-mss 1360
tunnel source Loopback0
tunnel destination 10.2.255.1
tunnel mode gre point-to-point
tunnel key 100
ip nhrp network-id 100
ip nhrp authentication dmvpnN
ip nhrp map 10.2.20.1 10.2.255.1
ip nhrp map multicast 10.2.255.1
ip nhrp nhs 10.2.20.1
exit

router eigrp 200
network 10.2.20.0 0.0.0.255
network 192.168.27.0 0.0.0.255
network 100.64.2.0 0.0.0.63

end
wr
```

### R5 :

```
conf t

interface Tunnel100
ip address 10.2.20.5 255.255.255.0
no ip redirects
ip mtu 1400
ip tcp adjust-mss 1360
tunnel source Loopback0
tunnel destination 10.2.255.1
tunnel mode gre point-to-point
tunnel key 100
ip nhrp network-id 100
ip nhrp authentication dmvpnN
ip nhrp map 10.2.20.1 10.2.255.1
ip nhrp map multicast 10.2.255.1
ip nhrp nhs 10.2.20.1
exit

router eigrp 200
network 10.2.20.0 0.0.0.255
network 10.2.50.0 0.0.0.255
network 100.64.2.0 0.0.0.63

end
wr
```

### Validation étape 1 

#### show dmvpn
Sur R1
![01 - show dmvpn](https://hackmd.io/_uploads/SJil7Jaofg.png)

#### show ip nhrp
Sur R1
![02 - show ip nhrp](https://hackmd.io/_uploads/SkolQkToze.png)

#### show ip eigrp 200 neighbors
Sur R1
![03 - show ip eigrp 200 neighbors](https://hackmd.io/_uploads/HJsgXy6sMg.png)

#### show ip route eigrp
Sur R3
![04 - show ip route eigrp](https://hackmd.io/_uploads/ryixXJ6iGl.png)

#### ping 192.168.20.1
Sur PC2
![05 - ping 192.168.20.1](https://hackmd.io/_uploads/H1sgXJpofe.png)

#### traceroute 10.2.50.1 source G2/0
Sur R3
![06 - traceroute 10.2.50.1 source g2 0](https://hackmd.io/_uploads/BJigXyaifg.png)

#### Preuve Wireshark durant le traceroute
Sur le lien R3-R5
![07 - preuve Wireshark](https://hackmd.io/_uploads/SksxXy6sGx.png)

--- 

## Étape 2 : 

### Configuration des routeurs 

### R1 : 

```
conf t

interface Tunnel100
no ip next-hop-self eigrp 200

end
wr
```

### R3 : 

```
conf t

interface Tunnel100
no tunnel destination
tunnel mode gre multipoint

end
wr
```

### R5 : 

```
conf t

interface Tunnel100
no tunnel destination
tunnel mode gre multipoint

end
wr
```

### Validation étape 2 

#### show ip route 10.2.50.0
Sur R3
![01 - show ip route 10.2.50.0](https://hackmd.io/_uploads/B1ZcN1ajMx.png)

#### traceroute 10.2.50.1 source G2/0
Sur R3
![02 - traceroute 10.2.50.1 source g2 0](https://hackmd.io/_uploads/rkz5Ek6sfl.png)

#### show dmvpn
Sur R3
![03 - show dmvpn](https://hackmd.io/_uploads/SJzqNJ6jMl.png)

#### show ip nhrp
Sur R3
![04 - show ip nhrp](https://hackmd.io/_uploads/r1-cV1pjfe.png)

---

## Étape 3 : 

### Configuration des routeurs

### R1 : 

```
conf t

ip cef

interface Tunnel100
no tunnel mode gre point-to-point
tunnel mode gre multipoint
ip nhrp redirect
ip next-hop-self eigrp 200
ip summary-address eigrp 200 192.168.0.0 255.255.248.0
no ip split-horizon eigrp 200

end
wr
```

### R3 : 

```
conf t

interface Tunnel100
ip nhrp shortcut

end
wr
```

### R5 : 

```
conf t

interface Tunnel100
ip nhrp shortcut
exit

rouuter eigrp 200
network 10.2.50.0 0.0.0.255

end
wr
```

**Étape non validée**

La configuration de la phase 3 a été appliquée : `ip nhrp redirect` et `ip summary-address eigrp 200 192.168.16.0 255.255.240.0` sur le hub R1, ip nhrp shortcut sur les spokes R3 et R5. Le résumé a été adapté à mon adressage, car le /21 du TP ne couvrait pas mes LAN.

Le DMVPN est fonctionnel : R3 et R5 sont enregistrés auprès du hub (`show dmvpn`, état UP), les voisinages EIGRP 200 sont établis, et R5 reçoit bien le résumé 192.168.16.0/20 via le hub. Les pings vers le LAN de R3 aboutissent, mais le trafic passe toujours par le hub (`traceroute` à 2 sauts : 10.2.20.1 puis 10.2.20.3).

Le tunnel direct spoke à spoke ne s’est pas formé. Le compteur Traffic Indication de `show ip nhrp traffic` sur R1 est resté à 0 : le hub n’a jamais envoyé de NHRP Redirect, donc les spokes n’ont jamais émis de Resolution Request et aucune route H n’est apparue.

Vérifications effectuées sans trouver d’anomalie : configuration Tunnel100 des trois routeurs, CEF actif sur Tunnel100, next-hops CEF corrects sur R1, no ip redirects, redémarrage du hub, réinitialisation du tunnel, test avec du trafic de transit depuis un PC, debug NHRP. La cause n’a pas été identifiée. L’hypothèse la plus probable est une limite de l’image émulée (c7200 15.2(4)S7 sous GNS3), mais je n’ai pas pu le confirmer. Je suis donc passé à l’étape 4.

### Preuves de la bonne configuration

#### show running config interface Tunnel100
Sur R1, R3 et R4
![01 - show running config interface Tunnel100](https://hackmd.io/_uploads/Hy_DaThifl.png)

#### show dmvpn et show ip eigrp 200 neighbors
Sur R1
![02 - show dmvpn et show ip eigrp 200 neighbors](https://hackmd.io/_uploads/S1uDaahsfg.png)

#### show ip route eigrp
Sur R5
![03 - show ip route eigrp](https://hackmd.io/_uploads/S1_wTa2jfe.png)

#### show ip interface Tunnel100
Sur R1
![04 - show ip interface Tunnel100](https://hackmd.io/_uploads/BydDaT3szl.png)

#### show ip cef
Sur R1
![05 - show ip cef](https://hackmd.io/_uploads/By_waphiGe.png)

#### show ip nhrp traffic
Sur R1
![06 - show ip nhrp traffic](https://hackmd.io/_uploads/Byvv66hofe.png)

--- 

## Étape 4 : 

### Configuration des routeurs


### R1 : 

```
conf t

crypto isakmp policy 10
encryption aes 256
hash sha256
authentication pre-share
group 14
exit

crypto isakmp key Cisco123DMVPN address 0.0.0.0 0.0.0.0
crypto ipsec transform-set TS-DMVPN esp-aes 256 esp-sha256-hmac
mode transport
exit

crypto ipsec profile IPSEC-PROF-DMVPN
set transform-set TS-DMVPN
exit

interface Tunnel100
tunnel protection ipsec profile IPSEC-PROF-DMVPN

end
wr
```

### R3 : 

```
conf t

crypto isakmp policy 10
encryption aes 256
hash sha256
authentication pre-share
group 14
exit

crypto isakmp key Cisco123DMVPN address 0.0.0.0 0.0.0.0
crypto ipsec transform-set TS-DMVPN esp-aes 256 esp-sha256-hmac
mode transport
exit

crypto ipsec profile IPSEC-PROF-DMVPN
set transform-set TS-DMVPN
exit

interface Tunnel100
tunnel protection ipsec profile IPSEC-PROF-DMVPN

end
wr
```

### R5 : 

```
conf t

crypto isakmp policy 10
encryption aes 256
hash sha256
authentication pre-share
group 14
exit

crypto isakmp key Cisco123DMVPN address 0.0.0.0 0.0.0.0
crypto ipsec transform-set TS-DMVPN esp-aes 256 esp-sha256-hmac
mode transport
exit

crypto ipsec profile IPSEC-PROF-DMVPN
set transform-set TS-DMVPN
exit

interface Tunnel100
tunnel protection ipsec profile IPSEC-PROF-DMVPN

wr
end
```

### Validation de l'étape 4

#### show crypto session
Sur R1
![01 - show crypto session](https://hackmd.io/_uploads/ryF4ZAnjzg.png)

#### ping 10.2.50.1 source fa4/1 et show crypto session
Sur R3
![02 - ping 10.2.50.1 et show crypto session](https://hackmd.io/_uploads/BktNbRnozl.png)
le ping vers le LAN de R5 passe par le hub (phase 3 non fonctionnelle, cf. étape 3). R3 n’a donc qu’une session vers R1.

#### show dmvpn detail
Sur R1
![03 - show dmvpn detail](https://hackmd.io/_uploads/HJ5Nb02jMl.png)

#### Preuve de la présence du protocole ESP sur Wireshark
Sur le lien R3-R5
![04 - preuve ESP](https://hackmd.io/_uploads/ry5VWChifl.png)
ESP entre R3 et R1 (SPI 0x2ce45ec1, identique à l’Outbound SPI de R1 vers R3 dans show dmvpn detail). Ce trafic transite par R5 dans l’underlay, donc il est visible sur le lien R3–R5. Les hello EIGRP 192.168.26.x en clair sont ceux de l’underlay (EIGRP 100), qui n’est pas protégé. Le trafic DMVPN, lui, est bien chiffré.

--- 

## Questions d'analyse : 

### 1) Qu'est-ce qu'une adresse NBMA ? Pourquoi utiliser la Loopback0 plutôt qu'une interface physique ?

- Adresse NBMA (Non-Broadcast Multi-Access) : C'est l'adresse IP physique (ou de transport) réelle sur le réseau sous-jacent (Internet/WAN) qu'utilisent les routeurs pour joindre l'interface du tunnel GRE.

- Pourquoi la Loopback0 ? Utiliser une interface logique (Loopback) comme source de tunnel garantit la stabilité : si un lien physique ou une interface physique principale bascule, la Loopback reste active tant qu'un chemin alternatif existe, évitant ainsi le flapping inutile des tunnels DMVPN.

### 2) Décris l'échange NHRP complet (registration, resolution request/reply) lors du premier ping R4 → LAN de R5 en phase 2.

- Registration : Les Spokes (ex: R5) s'enregistrent auprès du Hub (R1) en lui transmettant leur association entre leur adresse IP logique de tunnel et leur adresse NBMA physique.

- Resolution Request : Lorsque R4 veut joindre le LAN de R5, il consulte sa table de routage, voit que le trafic doit passer par le tunnel, et envoie une requête NHRP Resolution au Hub pour demander l'adresse NBMA de R5.

- Resolution Reply : Le Hub (qui connaît l'adresse de R5 grâce à l'enregistrement) renvoie une réponse Resolution Reply à R4 contenant l'adresse NBMA de R5, permettant l'établissement direct du tunnel shortcut.

### 3) Pourquoi faut-il désactiver le split-horizon EIGRP sur le hub ?

Le split-horizon empêche un routeur de réannoncer une route par l'interface sur laquelle il l'a apprise. Sur un Hub DMVPN, toutes les routes des Spokes arrivent par la même interface de tunnel. Le désactiver permet au Hub de réannoncer ces routes aux autres Spokes pour permettre la communication spoke-à-spoke.

### 4) Pourquoi le résumé de routes « casse »-t-il la phase 2 mais pas la phase 3 ?

- En Phase 2 : Les Spokes ont besoin de routes spécifiques (/32 ou /24) pour connaître le véritable next-hop EIGRP du Spoke distant. Si le Hub résume les routes, le Spoke perd cette information de next-hop granulaire et le trafic continue de transiter bêtement par le Hub.

- En Phase 3 : Le Hub utilise le mécanisme NHRP Redirect/Shortcut. Même en présence de résumés de routes, le Hub envoie une redirection au Spoke source pour lui indiquer de router directement vers le Spoke cible.

### 5) Quelle différence entre une entrée NHRP static et dynamic ?

- Static : Configurée manuellement (souvent sur le Hub pour répertorier les Spokes ou fixer des mappings), elle neexpire jamais.

- Dynamic : Apprise automatiquement à la volée lors des requêtes NHRP (enregistrement ou résolution), elle possède un délai d'expiration (holdtime) et disparaît si elle n'est plus utilisée.

### 6) Pourquoi le mode transport suffit-il pour IPsec sur DMVPN ?

Le mode transport chiffre uniquement la charge utile (payload) et l'en-tête du paquet, tandis que le mode tunnel rajoute un nouvel en-tête IP complet. Sur DMVPN, le trafic est déjà encapsulé dans un tunnel GRE (qui ajoute ses propres en-têtes IP/GRE). Le mode transport suffit donc amplement et évite d'alourdir les paquets avec un double en-tête IP redondant.

### 7) Compare, pour 200 agences, le nombre de tunnels configurés en VPN point à point, en DMVPN phase 1 et en DMVPN phase 3.

- VPN point à point (Full Mesh classique) : Nécessite N(N - 1) / 2, soit (200 x 199) / 2 = 19900 tunnels statiques à configurer et maintenir.

- DMVPN (Phase 1, 2 ou 3) : Chaque agence (Spoke) ne configure qu' un seul tunnel permanent vers le Hub. Pour 200 agences, on a 200 tunnels au total sur le Hub (et 1 par Spoke). Les tunnels spoke-à-spoke se créent dynamiquement à la demande.

### 8) En quoi DMVPN prépare-t-il les concepts du SD-WAN (module 06) ?

Le DMVPN introduit la dissociation fondamentale entre le plan de contrôle (le routage dynamique centralisé/distribué) et le plan de données (l'établissement dynamique de tunnels à la demande / shortcuts). C'est exactement le principe fondateur du SD-WAN, qui automatise et orchestre dynamiquement les chemins WAN sans configuration statique lourde.

## Rapport d'intervention - TP 03 : DMVPN

### Objectif
Déployer un DMVPN avec R1 en hub et R3/R5 en spokes, faire évoluer les phases 1, 2 et 3, puis chiffrer avec IPsec.

### Intervention réalisée
- **Étape 0** : Loopback50 sur R5 (10.2.50.0/24), contrôle de l'underlay.
- **Étape 1 (phase 1)** : hub en `gre multipoint`, spokes en `gre point-to-point`, NHRP (network-id, authentication, map multicast), EIGRP 200, split-horizon désactivé sur le hub.
- **Étape 2 (phase 2)** : spokes en `gre multipoint`, `no ip next-hop-self eigrp 200` sur le hub.
- **Étape 3 (phase 3)** : `ip nhrp redirect` et résumé EIGRP sur le hub, `ip nhrp shortcut` sur les spokes.
- **Étape 4** : IPsec en mode transport (profil `IPSEC-PROF-DMVPN`, `tunnel protection`).

### Vérifications
- `show dmvpn`, `show ip nhrp`
- Voisinages EIGRP 200 et tables de routage des spokes
- Traceroute vers le LAN de l'autre spoke
- Captures Wireshark
- `show crypto session`, `show dmvpn detail` (SPI identique à celui de l'ESP capturé)

### Problèmes rencontrés
- **Phase 3 non validée** : le tunnel direct spoke à spoke ne se forme pas. Le compteur Traffic Indication du hub reste à 0 (aucun NHRP Redirect envoyé) et le traceroute passe toujours par le hub.
- Le résumé a été adapté à mon adressage (192.168.16.0/20), car le /21 du TP ne couvrait pas mes LAN.
- Vérifications faites sans anomalie : configuration Tunnel100, CEF, next-hops, `no ip redirects`, redémarrage du hub, reset du tunnel, debug NHRP.
- Cause non identifiée. Hypothèse la plus probable : limite de l'image émulée (c7200 15.2(4)S7 sous GNS3).

### Conclusion
Le DMVPN est fonctionnel en phases 1 et 2 : enregistrements NHRP, EIGRP et trafic chiffré en ESP sont validés. La phase 3 est configurée mais le comportement attendu (shortcut) n'a pas pu être démontré. J'ai donc poursuivi avec l'étape 4 en passant par le hub.
