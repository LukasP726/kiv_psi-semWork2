# Síťová konfigurace – projekt

## Topologie
cisco R3 Gi1/0 --- frrouter R2 eth0  
frrouter R2 eth1 --- cisco R1 Gi0/0  
cisco R1 Gi1/0 --- switch1 eth0  
cisco R1 Gi2/0 --- switch2 eth0  
switch1 eth1 --- psi-base-node1 eth0  
switch1 eth2 --- psi-base-node2 eth0  
switch2 eth1 --- psi-base-node3 eth0  
switch2 eth2 --- psi-base-node4 eth0  

## R1 – Přidání slotu
Configure → Slots → Slot2: PA-GE

## R1 – Konfigurace rozhraní a DHCP
conf t  
interface GigabitEthernet 1/0  
 ip address 10.0.1.254 255.255.255.0  
 no shutdown  
 exit  
interface GigabitEthernet 2/0  
 ip address 10.0.2.254 255.255.255.0  
 no shutdown  
CTRL+Z  
write  

ip dhcp excluded-address 10.0.1.254  
ip dhcp pool 1  
 network 10.0.1.0 255.255.255.0  
 default-router 10.0.1.254  
 dns-server 8.8.8.8 8.8.4.4  
 exit  

ip dhcp excluded-address 10.0.2.254  
ip dhcp pool 2  
 network 10.0.2.0 255.255.255.0  
 default-router 10.0.2.254  
 dns-server 8.8.8.8 8.8.4.4  
CTRL+Z  
write  

Na všech klientech povolit DHCP:  
nano /etc/network/interfaces  
auto eth0  
iface eth0 inet dhcp  

## R2 – frrouter
conf t  
interface eth1  
 ip address 192.168.1.2/30  
 exit  
interface eth0  
 ip address 192.168.2.1/30  
 exit  
write  

Ověření:  
show interface brief  
show running-config  
show ip route  

## R1 – Připojení k R2
conf t  
interface GigabitEthernet 0/0  
 ip address 192.168.1.1 255.255.255.252  
 no shutdown  
CTRL+Z  
write  

## R3 – Konfigurace rozhraní a NAT
conf t  
interface GigabitEthernet 1/0  
 ip address 192.168.2.2 255.255.255.252  
 no shutdown  
 exit  
interface GigabitEthernet 1/0  
 ip nat inside  
 no shutdown  
 exit  
interface GigabitEthernet 0/0  
 ip address dhcp  
 ip nat outside  
 no shutdown  
 exit  
access-list 100 permit ip 192.168.1.0 0.0.0.3 any  
access-list 100 permit ip 192.168.2.0 0.0.0.3 any  
access-list 100 permit ip 10.0.0.0 0.0.255.255 any  
ip nat inside source list 100 interface GigabitEthernet 0/0 overload  
ip route 0.0.0.0 0.0.0.0 GigabitEthernet 0/0  
CTRL+Z  
write  

## OSPF konfigurace
Nejprve povolit OSPF na frrouteru: pravé tlačítko → Configure → General settings → Environment variables → ENABLE_OSPF=yes  

### R1 – OSPF
conf t  
router ospf 1  
 network 10.0.0.0 0.0.255.255 area 0  
 network 192.168.1.0 0.0.0.3 area 0  
CTRL+Z  
write  

### R2 – OSPF
conf t  
router ospf  
 network 192.168.1.0/30 area 0  
 network 192.168.2.0/30 area 0  
CTRL+Z  
write  

### R3 – OSPF
conf t  
router ospf 1  
 network 192.168.1.0 0.0.0.3 area 0  
 network 192.168.2.0 0.0.0.3 area 0  
 default-information originate  
CTRL+Z  
write  

## Kontrola a ověření
Na R1/R2/R3:  
show ip int brief  
show ip dhcp binding  
show ip route  
show ip nat translations  
show ip nat statistics  

Na klientovi:  
Ujistit se, že DHCP aktivní a přidělená IP odpovídá síti.  
Test pingem do interní sítě i na internet.  

