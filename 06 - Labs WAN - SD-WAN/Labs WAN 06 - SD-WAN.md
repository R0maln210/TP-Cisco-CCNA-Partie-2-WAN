---
title: Labs WAN 06 - Sd-WAN

---

# Labs WAN 06 - Sd-WAN

## Lab complet : 

Le lab est adapté à ma configuration initiale avant les TP et les adresses IP aussi

![lab complet](https://hackmd.io/_uploads/Hkku-OyhMx.png)

## Étape 0 : 

#### show ip route 192.168.27.0
![01 - show ip route 192.168.27.0](https://hackmd.io/_uploads/HkqzMOyhzg.png)

#### ping 192.168.22.2 source fa4/1
![02 - ping 192.168.22.2 source fa4 1](https://hackmd.io/_uploads/ryozGdJ3Mg.png)

#### ping 192.168.26.2 source g3/0
![03 - ping 192.168.26.2 source g3 0](https://hackmd.io/_uploads/SyozG_12zl.png)

## Étape 1 : 

### Configuration des routeurs

### R1

```
conf t

ip route 192.168.26.2 255.255.255.255 192.168.24.2

crypto isakmp policy 10
encryption aes 256
hash sha256
authentication pre-share
group 14
crypto isakmp key Cisco123! address 192.168.22.2
crypto isakmp key Cisco123! address 192.168.26.2
crypto ipsec transform-set TS-SDWAN esp-aes 256 esp-sha256-hmac
mode transport
crypto ipsec profile IPSEC-SDWAN
set transform-set TS-SDWAN

interface Tunnel101
description TLOC mpls
bandwidth 10000
delay 100
ip address 10.2.101.0 255.255.255.254
ip mtu 1400
ip tcp adjust-mss 1360
tunnel source FastEthernet4/1
tunnel destination 192.168.22.2
tunnel protection ipsec profile IPSEC-SDWAN
interface Tunnel102
description TLOC biz-internet
bandwidth 10000
delay 200
ip address 10.2.102.0 255.255.255.254
ip mtu 1400
ip tcp adjust-mss 1360
tunnel source GigabitEthernet3/0
tunnel destination 192.168.26.2
tunnel protection ipsec profile IPSEC-SDWAN

router eigrp 200
network 10.2.101.0 0.0.0.1
network 10.2.102.0 0.0.0.1
network 192.168.20.0 0.0.0.255
passive-interface FastEthernet4/0
no auto-summary

end
wr
```

### R3 

```
conf t

ip route 192.168.24.1 255.255.255.255 192.168.26.1

crypto isakmp policy 10
encryption aes 256
hash sha256
authentication pre-share
group 14
crypto isakmp key Cisco123! address 192.168.21.1
crypto isakmp key Cisco123! address 192.168.24.1
crypto ipsec transform-set TS-SDWAN esp-aes 256 esp-sha256-hmac
mode transport
crypto ipsec profile IPSEC-SDWAN
set transform-set TS-SDWAN

interface Tunnel101
description TLOC mpls
bandwidth 10000
delay 100
ip address 10.2.101.1 255.255.255.254
ip mtu 1400
ip tcp adjust-mss 1360
tunnel source FastEthernet4/0
tunnel destination 192.168.21.1
tunnel protection ipsec profile IPSEC-SDWAN
interface Tunnel102
description TLOC biz-internet
bandwidth 10000
delay 200
ip address 10.2.102.1 255.255.255.254
ip mtu 1400
ip tcp adjust-mss 1360
tunnel source GigabitEthernet2/0
tunnel destination 192.168.24.1
tunnel protection ipsec profile IPSEC-SDWAN

router eigrp 200
network 10.2.101.0 0.0.0.1
network 10.2.102.0 0.0.0.1
network 192.168.27.0 0.0.0.255
passive-interface FastEthernet4/1
no auto-summary

end
wr
```

### Validation de l'étape 1

#### show interfaces Tunnel101 include line source
![01 - show interfaces Tunnel101 include line source destination](https://hackmd.io/_uploads/SJwOfdknfe.png)

#### show crypto session
![02 - show crypto session](https://hackmd.io/_uploads/rkDOM_12Ml.png)

#### show ip eigrp 200 neighbors
![03 - show ip eigrp 200 neighbors](https://hackmd.io/_uploads/SygPOGu1hGl.png)

#### show ip eigrp 200 topology 192.168.27.0 24
![04 - show ip eigrp 200 topology 192.168.27.0 24](https://hackmd.io/_uploads/BJD_G_13Ml.png)

#### trace 192.168.27.1
![05 - trace 192.168.27.1](https://hackmd.io/_uploads/HyPOfO12ze.png)

#### Wireshark lien R1 - R2
![06 - lien R1 R2](https://hackmd.io/_uploads/HyDdMuJ3zx.png)

#### Wireshark lien R4 - R5
![07 - lien R4 R5](https://hackmd.io/_uploads/rkvOMdk3Gg.png)

--- 

## Étape 2 : 

### Configuration des routeurs

### R1

```
conf t

ip sla responder

ip sla 1
icmp-echo 10.2.101.1 source-interface Tunnel101
frequency 5
ip sla schedule 1 life forever start-time now
ip sla 2
icmp-echo 10.2.102.1 source-interface Tunnel102
frequency 5
ip sla schedule 2 life forever start-time now

ip sla 11
udp-jitter 10.2.101.1 17011 source-ip 10.2.101.0 num-packets 20 interval 20
frequency 10
timeout 1000
threshold 150
ip sla schedule 11 life forever start-time now
ip sla 12
udp-jitter 10.2.102.1 17012 source-ip 10.2.102.0 num-packets 20 interval 20
frequency 10
timeout 1000
threshold 150
ip sla schedule 12 life forever start-time now

track 1 ip sla 1 reachability
delay down 5 up 30
track 2 ip sla 2 reachability
delay down 5 up 30
track 11 ip sla 11 state
delay down 5 up 30
track 12 ip sla 12 state
delay down 5 up 30

end
wr
```

### R3

```
conf t

ip sla responder

end
wr
```

### Validation étape 2

#### show ip sla summary
![01 - show ip sla summary](https://hackmd.io/_uploads/HJfOQdkhfl.png)

#### show ip sla statistics 11
![02 - show ip sla statistics 11](https://hackmd.io/_uploads/rkG_7_yhMe.png)

#### show track brief
![03 - show track brief](https://hackmd.io/_uploads/ryfOXO12Mg.png)

#### Comparaison show ip sla statistics 11
![04 - comparaison show ip sla statistics 11](https://hackmd.io/_uploads/S1fOXd1hGx.png)


#### Comparaison show ip sla statistics 12
![05 - comparaison show ip sla statistics 12](https://hackmd.io/_uploads/B1Q_mOk2Mx.png)

Le transport C est plus lent d’environ 27 ms parce qu’il traverse deux routeurs (R4 et R5) et trois liens, contre un routeur (R2) et deux liens pour le transport A. Chaque saut ajoute du délai de forwarding, de chiffrement ESP et d’émulation. Note cette explication pour ton rapport.

Marge sur le seuil : le RTT maximal mesuré est de 101 ms (SLA 12), contre un seuil de 150 ms. Les tracks ne risquent donc pas de tomber Down sans raison. Au besoin, tu peux relever le seuil si tu veux plus de marge.

Le Number of failures: 1 n’est pas inquiétant. Il reste à 1 alors que les succès augmentent (13, puis 37, puis 57). C’est la toute première mesure, lancée avant que le répondeur et le tunnel soient stables.

## Étape 3 : 

### Configuration des routeurs

### R1 

```
conf t
ip access-list extended APP-VOIX
permit udp any any eq 5060
permit udp any any range 16384 32767

ip access-list extended APP-WEB
permit tcp any any eq 443
permit tcp any any eq 22

route-map SDWAN-POLICY permit 10
description VOIX : mpls puis biz-internet
match ip address APP-VOIX
set ip next-hop verify-availability 10.2.101.1 10 track 11
set ip next-hop verify-availability 10.2.102.1 20 track 12

route-map SDWAN-POLICY permit 20
description WEB/SSH : biz-internet puis mpls
match ip address APP-WEB
set ip next-hop verify-availability 10.2.102.1 10 track 2
set ip next-hop verify-availability 10.2.101.1 20 track 1

route-map SDWAN-POLICY permit 30
description Reste : routage normal (EIGRP 200)

interface FastEthernet4/0
ip policy route-map SDWAN-POLICY

end
wr
```

### R3 

```
conf t

ip sla 1
icmp-echo 10.2.101.0 source-interface Tunnel101
frequency 5
ip sla schedule 1 life forever start-time now

ip sla 2
icmp-echo 10.2.102.0 source-interface Tunnel102
frequency 5
ip sla schedule 2 life forever start-time now

ip sla 11
udp-jitter 10.2.101.0 17011 source-ip 10.2.101.1 num-packets 20 interval 20
frequency 10
timeout 1000
threshold 150
ip sla schedule 11 life forever start-time now

ip sla 12
udp-jitter 10.2.102.0 17012 source-ip 10.2.102.1 num-packets 20 interval 20
frequency 10
timeout 1000
threshold 150
ip sla schedule 12 life forever start-time now

track 1 ip sla 1 reachability
delay down 5 up 30

track 2 ip sla 2 reachability
delay down 5 up 30

track 11 ip sla 11 state
delay down 5 up 30

track 12 ip sla 12 state
delay down 5 up 30

ip access-list extended APP-VOIX
permit udp any any eq 5060
permit udp any any range 16384 32767

ip access-list extended APP-WEB
permit tcp any any eq 443
permit tcp any any eq 22

route-map SDWAN-POLICY permit 10
description VOIX : mpls puis biz-internet
match ip address APP-VOIX
set ip next-hop verify-availability 10.2.101.0 10 track 11
set ip next-hop verify-availability 10.2.102.0 20 track 12

route-map SDWAN-POLICY permit 20
description WEB/SSH : biz-internet puis mpls
match ip address APP-WEB
set ip next-hop verify-availability 10.2.102.0 10 track 2
set ip next-hop verify-availability 10.2.101.0 20 track 1

route-map SDWAN-POLICY permit 30

interface FastEthernet4/1
ip policy route-map SDWAN-POLICY

end
wr
```

### Validation de l'étape 3 

#### clear route-map counters SDWAN-POLICY <br> puis ping <br> puis show route-map SDWAN-POLICY
![01 - clear route-map counters SDWAN-POLICY puis ping puis show route-map SDWAN-POLICY](https://hackmd.io/_uploads/HkyuNO1hGe.png)

#### show interfaces Tunnel101 | include packets output AVANT
![02.1 - show interfaces Tunnel101 include packets output avant](https://hackmd.io/_uploads/rykuV_1hGx.png)

#### show interfaces Tunnel101 |include packets output APRÈS 
![02.2 - show interfaces Tunnel101 include packets output après](https://hackmd.io/_uploads/S1yu4u12fg.png)

#### ping 192.168.27.1 -P 6 -p 443
![03 - ping 192.168.27.1 -P 6 -p 443](https://hackmd.io/_uploads/BJlk_NO13Ge.png)

#### ping 192.168.27.1
![04 - ping 192.168.27.1](https://hackmd.io/_uploads/HJgk_VdJ3Mx.png)

#### debug ip policy <br> puis ping <br> puis debug all
![05 - debug ip policy puis ping puis undebug all](https://hackmd.io/_uploads/r1gOE_JnGl.png)

## Étape 4 : 

### Configuration des routeurs

### R1

```
conf t

service timestamps log datetime msec
logging buffered 16384

event manager applet SDWAN-VOIX-DOWN
 event track 11 state down
 action 1.0 syslog priority warnings msg "SD-WAN: SLA VOIX hors seuil sur mpls (track 11 Down), voix vers biz-internet"

event manager applet SDWAN-VOIX-UP
 event track 11 state up
 action 1.0 syslog priority warnings msg "SD-WAN: SLA VOIX retabli sur mpls (track 11 Up), voix revient sur mpls"

event manager applet SDWAN-INET-DOWN
 event track 12 state down
 action 1.0 syslog priority warnings msg "SD-WAN: SLA biz-internet hors seuil (track 12 Down)"

event manager applet SDWAN-INET-UP
 event track 12 state up
 action 1.0 syslog priority warnings msg "SD-WAN: SLA biz-internet retabli (track 12 Up)"

event manager applet SDWAN-MPLS-DOWN
 event track 1 state down
 action 1.0 syslog priority warnings msg "SD-WAN: transport mpls INJOIGNABLE (track 1 Down)"

event manager applet SDWAN-MPLS-UP
 event track 1 state up
 action 1.0 syslog priority warnings msg "SD-WAN: transport mpls disponible (track 1 Up)"

end
wr
```

### Validateur de l'étape 4

#### show sla statistics 11
![01 - show sla statistics 11](https://hackmd.io/_uploads/ryL5XYknMe.png)

#### show track 11
![02 - show track 11](https://hackmd.io/_uploads/HJUqmFJ3fl.png)

#### ping 192.168.27.1 -P 17 -p 5060
![03 - ping 192.168.27.1 -P 17 -p 5060](https://hackmd.io/_uploads/Hk8cQKJnGg.png)

#### ping 192.168.27.1
![04 - ping 192.168.27.1](https://hackmd.io/_uploads/SkU57K12Gl.png)

#### Dégradation retirée
![05 - dégradation retirée](https://hackmd.io/_uploads/SyvcQK1nGe.png)

#### Lien R2 - R3 coupé
![06 - L2 coupé](https://hackmd.io/_uploads/S1DcmYJ2fl.png)

#### Tout est bien remis
![07 - tout est bien remis](https://hackmd.io/_uploads/rJI5XFk2Mg.png)

## Étape 5 : 

| Indicateur | Commande | Résultat |
| ---------- | -------- | -------- |
| RTT mpls (SLA 11) min/moy/max | `show ip sla statistics 11` | 51 / 60 / 65 ms |
| Gigue mpls (SD / DS) min/moy/max | `show ip sla statistics 11` | 0/4/19 ms et 0/3/10 ms |
| Perte mpls | `show ip sla statistics 11` | 0 % |
| RTT biz-internet (SLA 12) min/moy/max | `show ip sla statistics 12` | 83 / 89 / 98 ms |
| Gigue biz-internet (SD / DS) min/moy/max | `show ip sla statistics 12` | 1/6/14 ms et 0/4/7 ms |
| Perte biz-internet | `show ip sla statistics 12` | 0 % |
| Historique mpls (depuis 11:00:42, environ 26 min) | `show ip sla statistics aggregated 11` | RTT 39 / 61 / 76 ms sur 3100 mesures, gigue moy. 5 / 3 ms, perte 0 |
| État des transports | `show track brief` | Tracks 1, 2, 11, 12 : Up depuis 26 min |
| Répartition par classe | `show route-map SDWAN-POLICY` | Séquences 10, 20, 30 : 0 paquet (aucun trafic de test en cours) |
| Volume Tunnel101 (mpls) | `show interfaces Tunnel101` | 2000 bit/s, 3 paquets/s |
| Volume Tunnel102 (biz-internet) | `show interfaces Tunnel102` | 3000 bit/s, 4 paquets/s |
| Événements de bascule | `show logging \| include SD-WAN` | 11:01:32 transport mpls disponible ; 11:01:37 SLA VOIX rétabli ; 11:01:37 SLA biz-internet rétabli |
 
### Comparaison des états relevés
 
| Indicateur | Nominal (09:20) | Dégradé (10:13) | Nominal (11:26) |
| ---------- | --------------- | --------------- | --------------- |
| RTT mpls moyen | 64 ms | 5268 ms | 60 ms |
| RTT biz-internet moyen | 91 ms | non relevé | 89 ms |
| Track 11 | Up | Down (Over threshold) | Up |
| Voix | Tunnel101 | Tunnel102 | Tunnel101 |
 
### Remarques
 
- Le débit de 2 à 3 kbit/s correspond aux sondes SLA et aux Hello EIGRP, aucun trafic de test ne tournait.
- Les compteurs de route-map à 0 sont normaux : ils avaient été remis à zéro.
- Le transport biz-internet est plus lent (environ 89 ms contre 60 ms) car il traverse deux routeurs intermédiaires (R4 et R5) contre un seul (R2) pour le mpls.
- Limite : relevés espacés de plusieurs dizaines de minutes plutôt que toutes les 5 minutes, suite à un incident GNS3 sur les liens R1–R3 et R3–R5. L'historique agrégé couvre environ 26 minutes.
 
## Questions d'analyse

### 1) Quelle différence fondamentale entre le routage EIGRP et l'App-Aware Routing ?

EIGRP choisit un chemin selon la topologie et une métrique statique (bande passante et délai configurés, delay 100 contre delay 200 dans le lab). Tous les flux vers une même destination suivent le même chemin (Tunnel101), quelle que soit l'application et quelle que soit la qualité réellement mesurée : un lien peut être très lent sans que EIGRP ne bouge, tant que l'adjacence reste active.

L'App-Aware Routing choisit un chemin par application, à partir de mesures réelles (latence, gigue, perte). Dans le lab, la voix part sur Tunnel101 tant que le SLA est respecté, et bascule sur Tunnel102 quand le RTT dépasse 150 ms (track 11 Down), alors que le web va sur Tunnel102 et que le reste suit EIGRP. La dégradation à 5268 ms l'a montré : la voix a basculé, le trafic ICMP est resté sur Tunnel101.

### 2)Pourquoi faut-il épingler chaque tunnel sur son transport ?

Sans route /32 vers l'extrémité de chaque tunnel, c'est EIGRP 100 (l'underlay) qui choisit le chemin des paquets chiffrés, et il peut faire passer les deux tunnels par le même chemin (le plus rapide). Il n'y aurait alors plus deux transports distincts : une panne ou une dégradation toucherait les deux tunnels en même temps, et les mesures SLA de chaque tunnel ne décriraient plus un transport propre.

Les routes /32 forcent Tunnel101 à passer par R2 (transport A) et Tunnel102 par R4 et R5 (transport C). Cela garantit la diversité de transport, condition nécessaire pour que le SD-WAN ait un vrai choix.

### 3) Pourquoi l'hystérésis (delay up 30) est-elle indispensable ? Que se passerait-il sans elle sur un lien instable ?

L'hystérésis (delay down 5 up 30) rend la bascule rapide (5 s) mais le retour lent (30 s), pour ne revenir sur un transport que lorsqu'il est stable.

Sans elle, un lien instable qui passe sans cesse de « bon » à « mauvais » ferait basculer le trafic en permanence (flapping) : voix saccadée, sessions TCP perturbées, logs et recalculs en boucle. Les logs du lab montrent ce phénomène : à 10:19, le track 11 est remonté puis retombé en quinze secondes pendant la dégradation.

### 4) Explique la différence entre track … reachability et track … state.

Les deux suivent une sonde IP SLA, mais pas sur le même critère :

reachability : le track est Up tant que la sonde obtient une réponse, même lente. Il tombe seulement si la cible ne répond plus.
state : le track est Up seulement si la sonde est OK. Il tombe aussi quand la sonde est hors seuil (Over threshold, RTT supérieur à 150 ms).

Dans le lab, les tracks 1 et 2 (reachability, ping ICMP) mesurent la disponibilité d'un transport, et les tracks 11 et 12 (state, udp-jitter) mesurent sa qualité. Pendant la dégradation, le track 11 est tombé (RTT de 5268 ms, Over threshold) alors que le track 1 est resté Up : le transport répondait, mais trop lentement pour la voix.

### 5) Pourquoi la politique est-elle appliquée à l'entrée du LAN et pas sur les tunnels ?

Le PBR s'applique aux paquets qui arrivent sur une interface, avant la décision de routage. À l'entrée du LAN (FastEthernet4/0 sur R1), les paquets sont en clair : le routeur voit les ports UDP et TCP d'origine et peut les classer (voix, web, reste) avant de choisir le tunnel de sortie.

Sur un tunnel, le paquet est déjà routé et encapsulé en GRE puis chiffré en ESP : les ports applicatifs ne sont plus visibles, et le choix du tunnel est déjà fait. Le PBR sur le tunnel ne permettrait donc ni de classer le trafic ni de choisir le chemin.

### 6) Complète un tableau « Brique SD-WAN Cisco ↔ implémentation du lab ↔ limite de notre implémentation ».

| Brique SD-WAN Cisco (Architecture Enterprise) | Implémentation de notre Lab (CLI / GNS3 / TP) | Limite de notre implémentation |
| :--- | :--- | :--- |
| **vManage** (Orchestrateur & GUI centralisée) | Configuration manuelle ligne par ligne en CLI sur chaque routeur. | Pas de supervision centralisée unique ; risque d'erreur humaine et lourdeur de configuration à l'échelle. |
| **vSmart** (Contrôleur du plan de contrôle OMP) | Protocoles de routage classiques / iBGP VPNv4 et MP-BGP. | Pas de séparation stricte entre le plan de contrôle et les routeurs de bordure (les PE font tout). |
| **vBond** (Orchestrateur d'authentification initiale) | Établissement manuel des sessions de voisinage et des tunnels. | Absence de Zero-Touch Provisioning (ZTP) automatisé pour l'enrôlement des équipements. |
| **vEdges / cEdges** (Routeurs de périphérie) | Routeurs Cisco IOS / IOSv faisant office de PE. | Ressources logicielles limitées, pas de chiffrement IPsec matériel natif géré par un orchestrateur. |

### 7) Quels sont les avantages opérationnels d'un contrôleur centralisé pour 300 agences ?

- Déploiement zero-touch : un routeur branché dans une agence récupère seul sa configuration auprès du contrôleur, sans technicien sur place.

- Modèles et politiques centralisés : une règle (SLA voix, routage applicatif) est modifiée une fois et diffusée sur les 300 sites, au lieu de configurer 300 routeurs un par un comme dans ce lab avec R1 et R3.
Cohérence : tous les sites appliquent la même politique, ce qui supprime les écarts de configuration et les erreurs de saisie.

- Supervision globale : état de chaque transport, mesures de latence, gigue et perte, et alertes sur tous les sites depuis une seule interface, là où le lab demande des show routeur par routeur.

- Maillage automatique : les tunnels entre sites sont créés par le contrôleur, alors qu'ici chaque tunnel et chaque clé IPsec est saisi à la main (300 agences représenteraient des milliers de configurations).
Mises à jour et dépannage plus rapides : changement de transport, correctifs et diagnostics sont faits depuis un point unique.

### 8) Pourquoi le SD-WAN chiffre-t-il aussi le trafic qui passe par le MPLS de l'opérateur ?

Le MPLS est un réseau privé, mais l'entreprise n'a pas de contrôle sur le réseau de l'opérateur : le trafic y est en clair pour lui, et peut être lu en cas d'erreur de configuration, de compromission ou d'accès interne. Le chiffrement IPsec garantit la confidentialité et l'intégrité des données sur tout le trajet, quel que soit le transport.

Cela permet aussi un traitement uniforme : la même politique de sécurité s'applique que le flux passe par le MPLS ou par Internet, et une bascule de l'un vers l'autre ne change rien au niveau de protection. Dans le lab, les captures Wireshark montrent de l'ESP sur les deux transports (R1–R2 et R4–R5).

## Rapport d'intervention

### Objectif
Reproduire les principes du SD-WAN entre R1 (siège) et R3 (agence) : deux transports, mesure de la qualité, routage par application et supervision.

### Intervention réalisée
- **Étape 0** : état initial, contrôle des deux transports (A via R2, C via R4/R5).
- **Étape 1** : deux tunnels IPsec (Tunnel101 = mpls, Tunnel102 = biz-internet) épinglés par des routes /32, EIGRP 200 dans les tunnels.
- **Étape 2** : sondes IP SLA (icmp-echo pour la disponibilité, udp-jitter pour la qualité, seuil 150 ms), tracks 1, 2, 11 et 12 avec hystérésis (`delay down 5 up 30`).
- **Étape 3** : PBR `SDWAN-POLICY` à l'entrée du LAN : voix (UDP 5060 et 16384-32767) sur mpls puis biz-internet, web/SSH sur biz-internet puis mpls, le reste en routage normal.
- **Étape 4** : applets EEM qui journalisent les bascules de tracks.
- **Étape 5** : relevés nominal / dégradé / nominal.

### Vérifications
- `show ip sla statistics`, `show track brief`
- Compteurs de route-map et de Tunnel101
- `debug ip policy`
- Pings applicatifs (`-P 6 -p 443`, `-P 17 -p 5060`)
- Dégradation injectée puis retirée
- Coupure du lien R2-R3 puis remise en état

### Résultats

| Indicateur | Nominal (09:20) | Dégradé (10:13) | Nominal (11:26) |
| ---------- | --------------- | --------------- | --------------- |
| RTT mpls moyen | 64 ms | 5268 ms | 60 ms |
| RTT biz-internet moyen | 91 ms | non relevé | 89 ms |
| Track 11 | Up | Down (Over threshold) | Up |
| Voix | Tunnel101 | Tunnel102 | Tunnel101 |

### Problèmes rencontrés
- Incident GNS3 sur les liens R1-R3 et R3-R5 : relevés espacés de plusieurs dizaines de minutes plutôt que toutes les 5 minutes. L'historique agrégé couvre environ 26 minutes.
- Le transport biz-internet est plus lent (environ 89 ms contre 60 ms) car il traverse deux routeurs intermédiaires (R4 et R5) contre un seul (R2) pour le mpls.

### Conclusion
La voix bascule sur biz-internet quand le SLA du mpls est dépassé et revient sur mpls une fois rétabli, sans flapping grâce à l'hystérésis. Le reste du trafic n'est pas impacté. La limite principale est l'absence de contrôleur centralisé : tout est configuré à la main, routeur par routeur.
