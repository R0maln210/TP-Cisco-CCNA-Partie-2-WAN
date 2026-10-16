---
title: Labs WAN 05 - VXLAN

---

# Labs WAN 05 - VXLAN

## Compte rendu : blocage sur l'étape 0

Pour réaliser ce TP, les VTEP doivent être des machines Linux (avec iproute2), car les routeurs c7200 ne gèrent pas VXLAN. Avec GNS3, ces nœuds sont des conteneurs Docker (appliance « Network Automation » ou « Ubuntu Docker Guest ») qui doivent tourner sur la GNS3 VM. Je n'ai pas réussi à les mettre en place.

### Ce que j'ai fait

- J'ai analysé la topologie de l'énoncé et je l'ai transposée à ma maquette (interfaces et adressage différents) : le LAN de R1 en 192.168.20.0/24 et celui de R3 en 192.168.27.0/24. J'ai aussi repéré qu'il fallait annoncer ces LAN dans EIGRP 100 (underlay sans IPsec) et non dans EIGRP 200 (tunnel IPsec du module 02), pour ne pas fausser les tests de MTU.
- J'ai tenté d'installer l'appliance Linux depuis le serveur GNS3 : erreur « Cannot use a node on the GNS3 VM server with the GNS3 VM not configured » (capture 01).
- J'ai importé la GNS3 VM (version 2.2.61) dans VMware Workstation, puis je l'ai activée dans GNS3 (Edit → Preferences → GNS3 VM). GNS3 n'arrive pas à la piloter : « vmrun has returned an error: Unknown error » (capture 02).
- VMware refuse de démarrer la VM avec : « VMware Workstation does not support nested virtualization on this host. Module 'HV' power on failed » (capture 03), après le message « Virtualized AMD-V/RVI is not supported on this platform ».
- J'ai décoché « Virtualize Intel VT-x/EPT or AMD-V/RVI » dans les paramètres de la VM (capture 06). La VM a alors pu démarrer et afficher son menu (capture 05).
- Malgré cela, la création d'un nœud depuis GNS3 échoue avec : « Cannot connect to compute 'GNS3 VM' with request POST /projects » (capture 04).

### Cause probable

Windows occupe l'hyperviseur (Hyper-V, isolation du noyau ou WSL2), ce qui empêche VMware d'exposer la virtualisation imbriquée à la GNS3 VM. La liaison entre GNS3 et la VM n'est donc pas stable et les nœuds Linux ne peuvent pas être créés.

### Suite

Faute de temps, je n'ai pas exploré d'autres solutions et je priorise les TP suivants.

## Preuves du non-fonctionnement sur GNS3 et VMware Workstation

### 01 - Impossible de créer le nœud Linux

![01 - can not use node](https://hackmd.io/_uploads/rycAkSRoMe.png)

### 02 - Erreur vmrun

![02 - vmrun has returned an error](https://hackmd.io/_uploads/rJj01BAiMe.png)

### 03 - VMware ne supporte pas la virtualisation imbriquée

![03 - vmware workstation does not support nested virtualization](https://hackmd.io/_uploads/B1oRyrCsMl.png)

### 04 - GNS3 ne joint pas la GNS3 VM

![04 - cannot connect to compute with request POST](https://hackmd.io/_uploads/Hko0kHRjGx.png)

### 05 - VM démarrée avec les bons réglages

![05 - vm bien demarre avec les bons settings](https://hackmd.io/_uploads/rJi0yB0iMx.png)

### 06 - Case de virtualisation décochée dans VMware

![06 - case virtualize bien decochee dans vmware](https://hackmd.io/_uploads/Bks01rCszx.png)