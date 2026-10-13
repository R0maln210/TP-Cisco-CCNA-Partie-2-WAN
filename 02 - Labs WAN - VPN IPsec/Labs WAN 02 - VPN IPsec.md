---
title: Labs WAN 02 - VPN IPsec

---

# Labs WAN 02 - VPN IPsec

## Lab complet : 

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
