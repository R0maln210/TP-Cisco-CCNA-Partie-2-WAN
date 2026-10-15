---
title: Labs WAN 01 - BGP

---

# Labs WAN 01 - BGP

## Lab complet : 

Le lab est adapté à ma configuration initiale avant les TP et les adresses IP aussi

![lab complet](https://hackmd.io/_uploads/Skz9bX9jMg.png)

## Étape 1 : 

### R1 :

```
en
conf t

interface Loopback100
description Bloc public Siege 100.64.2.0/26
ip add 100.64.2.1 255.255.255.192
exit 

do wr
```

### R2 : 

```
en
conf t

router eigrp 100
passive-interface g0/0
exit

ip route 0.0.0.0 0.0.0.0 dhcp

do wr
```

### R4 : 

```
en
conf t

interface Loopback100
description Bloc public Agence 100.64.2.64/26
ip add 100.64.2.65 255.255.255.192
exit

do wr
```

## Commandes pour la vérification de l'étape 1 : 

### show ip eigrp Neighbors
Sur R2

![01 - show ip eigrp neighbors](https://hackmd.io/_uploads/S1zoZX5ofl.png)

### show ip route 100.64.2.64
Sur R1

![02 - show ip route 100.64.2.64](https://hackmd.io/_uploads/BJ6i-75sMe.png)

### show ip route 0.0.0.0
Sur R2

![03 - show ip route 0.0.0.0](https://hackmd.io/_uploads/rk7h-7cjMx.png)


## Étape 2 :

### R1 :

```
conf t
router bgp 65002
bgp router-id 10.2.255.1
bgp log-neighbor-changes
neighbor 10.2.255.5 remote-as 65002
neighbor 10.2.255.5 update-source Loopback0
network 100.64.2.0 mask 255.255.255.192
exit
do wr
```

### R2 : 

```
conf t
router bgp 65002
bgp router-id 10.2.255.2
bgp log-neighbor-changes
neighbor 10.2.255.5 remote-as 65002
neighbor 10.2.255.5 update-source Loopback0
exit
do wr
```

### R3 : 

```
conf t
router bgp 65002
bgp router-id 10.2.255.3
bgp log-neighbor-changes
neighbor 10.2.255.5 remote-as 65002
neighbor 10.2.255.5 update-source Loopback0
exit
do wr
```

### R4 : 

```
conf t
router bgp 65002
bgp router-id 10.2.255.4
bgp log-neighbor-changes
neighbor 10.2.255.5 remote-as 65002
neighbor 10.2.255.5 update-source Loopback0
network 100.64.2.64 mask 255.255.255.192
exit
do wr
```

### R5 : 

```
conf t
router bgp 65002
bgp router-id 10.2.255.5
bgp log-neighbor-changes
neighbor IBGP-RR peer-group
neighbor IBGP-RR remote-as 65002
neighbor IBGP-RR update-source Loopback0
neighbor IBGP-RR route-reflector-client
neighbor IBGP-RR next-hop-self
neighbor 10.2.255.1 peer-group IBGP-RR
neighbor 10.2.255.2 peer-group IBGP-RR
neighbor 10.2.255.3 peer-group IBGP-RR
neighbor 10.2.255.4 peer-group IBGP-RR
exit
do wr
```

## Commandes pour la vérification de l'étape 2 : 

### show ip bgp summary
Sur R2

![01 - show ip bgp summary](https://hackmd.io/_uploads/HJyTZXciGg.png)

### show ip route bgp
Sur R2

![02 - show ip route bgp](https://hackmd.io/_uploads/Skmp-7coMl.png)

### show ip bgp 100.64.2.64
Sur R1

![03 - show ip bgp 100.64.2.64](https://hackmd.io/_uploads/r1v6bQ5iGl.png)

### showip route bgp
Sur R1

![04 - show ip route bgp](https://hackmd.io/_uploads/HkaaZQ9sGe.png)

### ping 100.64.2.65 source Loopback100
Sur R1

![05 - ping 100.64.2.65 source Loopback100](https://hackmd.io/_uploads/SkXR-79sfe.png)

### traceroute 100.64.2.65 source Loopback100
Sur R1

![06 - traceroute 100.64.2.65 source Loopback100](https://hackmd.io/_uploads/SkOAZ7qsMl.png)


## Étape 3 : 

### R2 :

```
conf t

interface GigabitEthernet1/0
description Lien vers Partenaire R22
ip address 10.40.10.2 255.255.255.0
no shutdown
exit

ip prefix-list PL-OUT-EBGP seq 5 permit 100.64.2.0/24

ip prefix-list PL-IN-EBGP seq 5 deny 100.64.2.0/24 le 32
ip prefix-list PL-IN-EBGP seq 10 permit 100.64.0.0/10 ge 24 le 24

router bgp 65002
aggregate-address 100.64.2.0 255.255.255.0
neighbor 10.40.10.22 remote-as 65022
neighbor 10.40.10.22 description eBGP vers Entreprise 22
neighbor 10.40.10.22 password BGP-2-22
neighbor 10.40.10.22 ttl-security hops 1
neighbor 10.40.10.22 prefix-list PL-IN-EBGP in
neighbor 10.40.10.22 prefix-list PL-OUT-EBGP out
neighbor 10.40.10.22 maximum-prefix 20 80
neighbor 10.40.10.22 soft-reconfiguration inbound
exit

do wr
```

### Simulation d'un routeur entreprise externe R22 :

```
en
conf t
hostname R22

interface GigabitEthernet0/0
description Lien vers Enterprise 2 (R2)
ip address 10.40.10.22 255.255.255.0
no shutdown
exit

interface Loopback100
description Siege Entreprise 22
ip address 100.64.22.1 255.255.255.192
exit

ip prefix-list PL-OUT-EBGP seq 5 permit 100.64.22.0/24
ip prefix-list PL-IN-EBGP seq 5 deny 100.64.22.0/24 le 32
ip prefix-list PL-IN-EBGP seq 10 permit 100.64.0.0/10 ge 24 le 24

router bgp 65022
bgp router-id 10.22.255.1
aggregate-address 100.64.22.0 255.255.255.0
neighbor 10.40.10.2 remote-as 65002
neighbor 10.40.10.2 description eBGP vers Entreprise 2 (R2)
neighbor 10.40.10.2 password BGP-2-22
neighbor 10.40.10.2 ttl-security hops 1
neighbor 10.40.10.2 prefix-list PL-IN-EBGP in
neighbor 10.40.10.2 prefix-list PL-OUT-EBGP out
neighbor 10.40.10.2 maximum-prefix 20 80
neighbor 10.40.10.2 soft-reconfiguration inbound

network 100.64.22.0 mask 255.255.255.0
exit

ip route 100.64.22.0 255.255.255.0 Null0

do wr
```

## Commandes pour la vérification de l'étape 3 : 

### show ip bgp summary
Sur R2

![01 - show ip bgp summary](https://hackmd.io/_uploads/SJJlG7qifx.png)

### show ip bgp Neighbors 10.40.10.22 advertised-routes
Sur R2

![02 - show ip bgp neighbors 10.40.10.22 advertised-routes](https://hackmd.io/_uploads/r1mlG7cszx.png)

### show ip bgp Neighbors 10.40.10.22 received-routes
Sur R2

![03 - show ip bgp neighbors 10.40.10.22 received-routes](https://hackmd.io/_uploads/H1wlMX9sze.png)

### show ip bgp Neighbors 10.40.10.22 routes
Sur R2

![04 - show ip bgp neighbors 10.40.10.22 routes](https://hackmd.io/_uploads/BkjezXqjGe.png)

### show ip route 100.64.22.0
Sur R1

![05 - show ip route 100.64.22.0](https://hackmd.io/_uploads/BkJbGmcjfx.png)

### clear ip bgp 10.40.10.22
Sur R2 après avoir enlevé le mot de passe sur R22 --> 

```
conf t
router bgp 65022
no neighbor 10.40.10.2 password BGP-2-22
do wr
```
![06 - clear ip bgp 10.40.10.22](https://hackmd.io/_uploads/SJeMfm5sGx.png)

### show ip bgp neighbors 10.40.10.22 | include External|Hops|TTL
Sur R2

![07 - show ip bgp neighbors 10.40.10.22 include External Hops TTL](https://hackmd.io/_uploads/HJUGGQqiGe.png)

## Étapes 4 : 

### R2 : 

```
conf t

interface g1/0
bfd interval 300 min_rx 300 multiplier 3
no bfd echo
exit

router bgp 65002
neighbor 10.40.10.22 fall-over bfd
exit

do wr
```

### R22 : 

```
conf t

interface g0/0
bfd interval 300 min_rx 300 multiplier 3
no bfd echo
exit

router bgp 65022
neighbor 10.40.10.2 fall-over bfd
exit

do wr
```

Ces commandes ne passent pas sur cette version du routeur mais le BFD est quand même actif et opérationnel pour BGP

## Commandes pour la vérification de l'étape 4 : 

### show ip bgp neighbors 10.40.10.22 | include BFD|fall-over
Sur R2

![01 - show ip bgp neighbors 10.40.10.22 include BFD fall-over](https://hackmd.io/_uploads/SyTmM75jzl.png)

## Étape 5 : 

### R2 : 

```
conf t

interface g2/0
ip add 10.50.10.2 255.255.255.0
no shutdown
exit

ip as-path access-list 1 permit ^$

router bgp 65002
neighbor 10.50.10.33 remote-as 65033
neighbor 10.50.10.33 description eBGP vers Partenaire Q (R33)
neighbor 10.50.10.33 password CISCO
neighbor 10.50.10.33 ttl-security hops 1
neighbor 10.50.10.33 prefix-list PL-OUT-EBGP out
neighbor 10.50.10.33 filter-list 1 out
exit

do wr
```

### R33 : 

```
en
conf t
hostname R33

interface g0/0
ip add 10.50.10.33 255.255.255.0
no shutdown
exit

router bgp 65033
bgp router-id 10.33.255.33
neighbor 10.50.10.2 remote-as 65002
neighbor 10.50.10.2 password CISCO
neighbor 10.50.10.2 ttl-security hops 1
exit

do wr
```

## Commandes pour la vérification de l'étape 5 :

### show ip bgp 100.64.2.0
Sur R22

![01 - show ip bgp 100.64.2.0](https://hackmd.io/_uploads/SyzBMX9ofx.png)

### show ip bgp 100.64.2.0
Sur R22

![02 - show ip bgp 100.64.2.0](https://hackmd.io/_uploads/SkUrz79iMx.png)

### show ip bgp Neighbors 10.50.10.33 advertised-routes
Sur R2

![03 - show ip bgp neighbors 10.50.10.33 advertised-routes](https://hackmd.io/_uploads/r1FSzmqofe.png)


## Étape 6 : 

R2 : 

```
conf t
ip prefix-list PL-OUT-TRANSIT seq 5 permit 100.64.0.0/10 ge 24 le 24

router bgp 65002
no neighbor 10.40.10.22 filter-list 1 out
no neighbor 10.40.10.33 filter-list 1 out
neighbor 10.40.10.22 prefix-list PL-OUT-TRANSIT out
neighbor 10.50.10.33 prefix-list PL-OUT-TRANSIT out
exit

route-map RM-IN-P permit 10
set local-preference 200

router bgp 65002
neighbor 10.40.10.22 route-map RM-IN-P in
exit

route-map RM-OUT-Q permit 10
set as-path prepend 65002 65002 65002

router bgp 65002
neighbor 10.50.10.33 route-map RM-OUT-Q out
exit

route-map RM-OUT-P permit 10
set community no-export
exit

router bgp 65002
neighbor 10.40.10.22 send-community
neighbor 10.40.10.22 route-map RM-OUT-P out
exit

end
clear ip bgp 10.50.10.33 soft out
clear ip bgp 10.40.10.22 soft in
clear ip bgp 10.40.10.22 soft out
wr
```

## Commandes pour la vérification de l'étape 6 :

### show ip bgp
Sur R2

![01 - show ip bgp](https://hackmd.io/_uploads/SkE8f7qsGg.png)

### show ip bgp 100.64.22.0
Sur R1

![02 - show ip bgp 100.64.22.0](https://hackmd.io/_uploads/S1_Izmcszx.png)

### show ip bgp 100.64.2.0
Sur R22 et R33

![03 - show ip bgp 100.64.2.0](https://hackmd.io/_uploads/Bk0UGm9oze.png)


## Défis

### Défis redondance du Route Reflector

#### R1 :

```
conf t
router bgp 65002
bgp cluster-id 10.2.255.250
end
wr
```

#### R5 : 

```
conf t
router bgp 65002
bgp cluster-id 10.2.255.250

interface Loopback0
shutdown
end
wr
```

### Vérification défis Route Reflector :

#### show ip bgp
Sur R4

![Défis 1 - show ip bgp](https://hackmd.io/_uploads/Bk3vfQcoGe.png)

## Questions d'analyse : 

Voici les réponses détaillées aux questions d'analyse sur BGP et les architectures réseaux :

### 1. Pourquoi BGP utilise-t-il TCP, alors qu'EIGRP et OSPF n'en ont pas besoin ?

- BGP (Border Gateway Protocol) est conçu pour router à grande échelle sur Internet entre des Systèmes Autonomes (AS) différents, souvent sur des liaisons non fiables ou non adjacentes directement. Il utilise **TCP (port 179)** pour bénéficier d'un transport fiable, du contrôle de flux, de la gestion des accusés de réception et de la retransmission des paquets perdus, lui évitant d'implémenter ses propres mécanismes de fiabilité.

- OSPF et EIGRP sont des protocoles de passerelle interne (IGP) optimisés pour la convergence rapide au sein d'un même domaine. OSPF s'encapsule directement au-dessus d'IP (protocole 89) et gère sa propre fiabilité, tandis qu'EIGRP utilise son propre protocole de transport (RTP) par-dessus IP.

---

### 2. Explique la règle du split-horizon iBGP et les deux solutions possibles.

- La règle : Une route apprise d'un voisin iBGP (Internal BGP) ne peut pas être réannoncée à un autre voisin iBGP. Comme BGP n'utilise pas le champ AS_PATH pour détecter les boucles au sein du même AS, cette règle empêche les boucles de routage iBGP.

- Les deux solutions :
    1. Route Reflector (RR) : Désigner un ou plusieurs routeurs comme « réflecteurs » autorisés à redistribuer les routes iBGP apprises d'un client vers d'autres clients ou routeurs non-clients.
    2. iBGP Full Mesh (Maillage complet) : Connecter directement chaque routeur iBGP avec tous les autres routeurs de l'AS ($N(N-1)/2$ liaisons), ce qui devient difficile à gérer à grande échelle.

---

### 3. Dans `show ip bgp`, que signifient `*`, `>`, `i`, `r` et le code d'origine `i` / `?` ?

- `*` : La route est **valide** (le saut suivant est joignable).

- `>` : La route est la **meilleure** (*best path*), installée dans la table de routage IP.

- `i` : La route a été apprise via l'**iBGP**.

- `r` : Route rejetée (*RIB-failure* ou filtrée par politique).

- Code d'origine `i` : Origine **IGP** (le préfixe a été injecté dans BGP depuis un protocole interne ou statique).

- Code d'origine `?` : Origine **Incomplete** (souvent injecté par redistribution directe sans certitude sur l'origine).

---

### 4. Pourquoi `next-hop-self` est-il nécessaire sur R5 pour les routes eBGP ?

Lorsqu'un routeur apprend une route d'un voisin eBGP externe, il conserve par défaut le Next-Hop d'origine (l'IP du routeur externe). Lorsqu'il la transmet à ses voisins iBGP, si ces derniers ne connaissent pas cette adresse IP externe dans leur table IGP, le saut suivant devient injoignable et la route est invalidée. La commande `next-hop-self` force le routeur (R5) à remplacer le Next-Hop par sa propre adresse IP (ou Loopback) avant de l'annoncer en iBGP.

---

### 5. LOCAL_PREF et AS_PATH prepend : lequel agit sur le trafic sortant, lequel sur le trafic entrant ? Pourquoi le second est-il seulement une « suggestion » ?

- LOCAL_PREF agit sur le trafic sortant (et iBGP) : plus sa valeur est élevée, plus la route est privilégiée pour quitter l'AS.

- AS_PATH prepend agit sur le trafic entrant : en allongeant artificiellement la liste des AS traversés, on décourage les AS distants d'emprunter ce chemin.

- Pourquoi une suggestion ? Parce qu'un AS externe reste entièrement souverain et libre d'appliquer ses propres politiques de routage (il peut décider d'ignorer la longueur de l'AS_PATH s'il préfère privilégier un autre critère comme le MED ou un accord commercial spécifique).

---

### 6. Quelle différence entre password (MD5) et ttl-security ? Contre quelle attaque protège chacun ?

- Password (MD5) : Authentifie chaque segment TCP de la session BGP à l'aide d'une empreinte cryptographique. Il protège contre l'usurpation d'identité (spoofing TCP) et l'injection de faux paquets pour perturber la session.

- TTL-Security (GTSM) : Vérifie que les paquets IP arrivent avec un TTL extrêmement élevé (proche de 255), sachant qu'un voisin direct doit avoir un TTL de 255 ou 254. Il protège contre les attaques de Déni de Service (DoS) et le spoofing de paquets IP distants tentant de deviner les numéros de séquence TCP.

---

### 7. Pourquoi l'expérience « BGP-free core » montre-t-elle l'intérêt de MPLS ?

Dans une architecture traditionnelle sans MPLS, chaque routeur du cœur (*core*) d'un opérateur doit stocker l'intégralité de la table de routage Internet mondiale (plus de 900 000 routes), ce qui sature la mémoire et le CPU. Grâce à **MPLS**, le cœur n'a besoin de faire tourner qu'un protocole IGP simple (comme OSPF/IS-IS) et de commuter des paquets à l'aide de libellés (*labels*), libérant ainsi les routeurs internes de la charge BGP (*BGP-free core*).

---

### 8. Quel risque prend une entreprise qui oublie le filtre sortant vers deux opérateurs ? Cite un incident réel lié à une fuite de routes BGP.

- Risque : L'entreprise risque de devenir un point de transit non intentionnel (*transit leak*). En annonçant à l'opérateur A les routes qu'elle apprend de l'opérateur B, elle risque de voir transiter par son réseau des flux de données extérieurs qui satureront ses liens et poseront des risques majeurs de sécurité.

- Incident réel : L'incident de Pakistan Telecom (AS 17557) en 2008, qui a accidentellement annoncé une route trop spécifique pour YouTube (204.42.167.0/24) au reste du monde à la suite d'une mauvaise configuration de filtrage, entraînant une panne mondiale de l'accès à YouTube pendant plusieurs heures.
