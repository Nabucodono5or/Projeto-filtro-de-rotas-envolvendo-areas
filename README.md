# Projeto-filtro-de-rotas-envolvendo-areas

## **Descrição**
Projeto que visa testar o filtro em ambiente configurado com OSPF com areas diferentes.

## **Topologia**

A topologia é a mesma para todos os cenários.

![topologia](Pasted%20image%2020250304113030.png)

As **LAN01** e **LAN02** cloud são **Linux Vrin** configurados para gerar rotas as areas OSPF.

### LAN02
![topologia](Pasted%20image%2020250304113230.png)

### LAN01
![topologia](Pasted%20image%2020250304113254.png)


Validação dos protocolos e interfaces.

### NR5

```julia
NR5#show ip protocols 
Routing Protocol is "ospf 5"
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  Router ID 5.5.5.5
  Number of areas in this router is 1. 1 normal 0 stub 0 nssa
  Maximum path: 4
  Routing for Networks:
  Routing on Interfaces Configured Explicitly (Area 0.0.0.0):
    FastEthernet0/1
    FastEthernet0/0
 Reference bandwidth unit is 100 mbps
  Routing Information Sources:
    Gateway         Distance      Last Update
    2.2.2.2              110      00:01:47
    1.1.1.1              110      00:02:23
    6.6.6.6              110      00:01:52
  Distance: (default is 110)

NR5#show running-config | section ospf
 ip ospf 5 area 0.0.0.0
 ip ospf 5 area 0.0.0.0
router ospf 5
 router-id 5.5.5.5
 log-adjacency-changes

NR5#show running-config interface fastEthernet 0/0
Building configuration...

Current configuration : 143 bytes
!
interface FastEthernet0/0
 description NR5 to NR6
 ip address 10.1.1.1 255.255.255.252
 ip ospf 5 area 0.0.0.0
 duplex auto
 speed auto
end

NR5#show running-config interface fastEthernet 0/1
Building configuration...

Current configuration : 138 bytes
!
interface FastEthernet0/1
 description LAN01
 ip address 10.1.1.5 255.255.255.252
 ip ospf 5 area 0.0.0.0
 duplex auto
 speed auto
end

```


### NR6

```julia
Routing Protocol is "ospf 6"
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  Router ID 6.6.6.6
  It is an area border router
  Number of areas in this router is 2. 2 normal 0 stub 0 nssa
  Maximum path: 4
  Routing for Networks:
  Routing on Interfaces Configured Explicitly (Area 0):
    FastEthernet0/0
  Routing on Interfaces Configured Explicitly (Area 1.1.1.1):
    FastEthernet0/1
 Reference bandwidth unit is 100 mbps
  Routing Information Sources:
    Gateway         Distance      Last Update
    2.2.2.2              110      00:11:51
    1.1.1.1              110      00:12:12
    5.5.5.5              110      00:12:32
    7.7.7.7              110      00:12:49
  Distance: (default is 110)


NR6#show running-config | section ospf
 ip ospf 6 area 0
 ip ospf 6 area 1.1.1.1
router ospf 6
 router-id 6.6.6.6
 log-adjacency-changes


NR6#show running-config interface fastEthernet 0/0
Building configuration...

Current configuration : 137 bytes
!
interface FastEthernet0/0
 description NR6 to NR5
 ip address 10.1.1.2 255.255.255.252
 ip ospf 6 area 0
 duplex auto
 speed auto
end

NR6#show running-config interface fastEthernet 0/1
Building configuration...

Current configuration : 143 bytes
!
interface FastEthernet0/1
 description NR6 to NR7
 ip address 10.1.2.2 255.255.255.252
 ip ospf 6 area 1.1.1.1
 duplex auto
 speed auto
end

```


### NR7

```julia
NR7#show ip protocols 
Routing Protocol is "ospf 7"
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  Router ID 7.7.7.7
  Number of areas in this router is 1. 1 normal 0 stub 0 nssa
  Maximum path: 4
  Routing for Networks:
  Routing on Interfaces Configured Explicitly (Area 1.1.1.1):
    FastEthernet0/1
    FastEthernet0/0
 Reference bandwidth unit is 100 mbps
  Routing Information Sources:
    Gateway         Distance      Last Update
    2.2.2.2              110      00:13:38
    1.1.1.1              110      00:13:53
    6.6.6.6              110      00:13:58
  Distance: (default is 110)


NR7#show running-config | section ospf
 ip ospf 7 area 1.1.1.1
 ip ospf 7 area 1.1.1.1
router ospf 7
 router-id 7.7.7.7
 log-adjacency-changes

NR7#show running-config interface fastEthernet 0/0
Building configuration...

Current configuration : 138 bytes
!
interface FastEthernet0/0
 description LAN02
 ip address 10.1.2.5 255.255.255.252
 ip ospf 7 area 1.1.1.1
 duplex auto
 speed auto
end


NR7#show running-config interface fastEthernet 0/1
Building configuration...

Current configuration : 143 bytes
!
interface FastEthernet0/1
 description NR7 to NR6
 ip address 10.1.2.1 255.255.255.252
 ip ospf 7 area 1.1.1.1
 duplex auto
 speed auto
end

```


## **Endereçamento**

Endereçamento ipv4  utilizados nos cenários

| DISPOSITIVO | INTERFACE | ENDEREÇO IPV4 |
| ----------- | --------- | ------------- |
| NR5         | F0/0      | 10.1.1.1/30   |
| NR5         | F0/1      | 10.1.1.5/30   |
| NR6         | F0/0      | 10.1.1.2/30   |
| NR6         | F0/1      | 10.1.2.2/30   |
| NR7         | F0/0      | 10.1.2.5/30   |
| NR7         | F0/1      | 10.1.2.1/30   |

## Cenário 1

Foi feita uma tentativa de filtra rotas da **area 0** para a **area 1**, porém como as rotas são do tipo externa, mais específicamente, `E2`.  Não foi possível a realização através de `distribute-list` e nem `area x filter-list`, neste caso ficou com a configuração sendo realizada no próprio router da area 0 que precisava filtrar as rotas.

No entanto, existe uma possível solução a ser avaliada. Seria a criação de uma **NSSA**, onde rotas externas seriam filtradas. O que não foi testado no momento.

Para o teste de filtragem entre areas foi criado um interface de **loopback**, na area 1 para que eu pudesse testar o filtro para a area 0.

### Conifguração

#### NR5

```julia
conf t
hostname NR5
router ospf 5
router-id 5.5.5.5
exit
interface FastEthernet 0/0
description NR5 to NR6
ip address 10.1.1.1 255.255.255.252
ip ospf 5 area 0
no shut
exit
interface FastEthernet 0/1
description LAN01
ip address 10.1.1.5 255.255.255.252
ip ospf 5 area 0
no shut
end
```


#### NR6

```julia
conf t
hostname NR6
router ospf 6
router-id 6.6.6.6
interface FastEthernet 0/0
description NR6 to NR5
ip address 10.1.1.2 255.255.255.252
ip ospf 6 area 0
no shut
exit
interface FastEthernet 0/1
description NR6 to NR7
ip address 10.1.2.2 255.255.255.252
ip ospf 6 area 1
no shut
exit

ip prefix-list seq 10 deny 192.168.2.1/32
ip prefix-list FILTER-AREA-1 seq 15 deny 8.8.8.8/32
ip prefix-list seq 16 deny 192.168.2.2/32
ip prefix-list seq 20 deny 192.168.2.0/24 ge 25
ip prefix-list seq 30 permit 0.0.0.0/0 le 32

router ospf 6
area 0 filter-list prefix FILTER-AREA-1 in
end

```

#### NR7

```julia
conf t
hostname NR7
router ospf 7
router-id 7.7.7.7
interface FastEthernet 0/1
description NR7 to NR6
ip address 10.1.2.1 255.255.255.252
ip ospf 7 area 1
no shut
exit
interface FastEthernet 0/0
description LAN02
ip address 10.1.2.5 255.255.255.252
ip ospf 7 area 1
no shut
interface Loopback 1
ip address 8.8.8.8 255.255.255.255
ip ospf 7 area 1
no shut
end

```

### Testes e validação

#### NR6

```julia
NR6#show ip prefix-list FILTER-AREA-1
ip prefix-list FILTER-AREA-1: 5 entries
   seq 10 deny 192.168.2.1/32
   seq 15 deny 8.8.8.8/32
   seq 16 deny 192.168.2.2/32
   seq 20 deny 192.168.2.0/24 ge 25
   seq 30 permit 0.0.0.0/0 le 32


NR6#sh ip route   
Codes: C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route

Gateway of last resort is not set

     1.0.0.0/32 is subnetted, 1 subnets
O       1.1.1.1 [110/30] via 10.1.1.1, 02:40:21, FastEthernet0/0
     2.0.0.0/32 is subnetted, 1 subnets
O       2.2.2.2 [110/30] via 10.1.2.1, 02:40:21, FastEthernet0/1
     8.0.0.0/32 is subnetted, 1 subnets
O       8.8.8.8 [110/11] via 10.1.2.1, 02:40:21, FastEthernet0/1
     10.0.0.0/30 is subnetted, 4 subnets
C       10.1.2.0 is directly connected, FastEthernet0/1
C       10.1.1.0 is directly connected, FastEthernet0/0
O       10.1.2.4 [110/20] via 10.1.2.1, 02:40:21, FastEthernet0/1
O       10.1.1.4 [110/20] via 10.1.1.1, 02:40:23, FastEthernet0/0
     192.168.1.0/32 is subnetted, 15 subnets
O E2    192.168.1.9 [110/30] via 10.1.1.1, 02:40:23, FastEthernet0/0
O E2    192.168.1.8 [110/30] via 10.1.1.1, 02:40:23, FastEthernet0/0
O E2    192.168.1.11 [110/30] via 10.1.1.1, 02:40:23, FastEthernet0/0
O E2    192.168.1.10 [110/30] via 10.1.1.1, 02:40:23, FastEthernet0/0
O E2    192.168.1.13 [110/30] via 10.1.1.1, 02:40:23, FastEthernet0/0
O E2    192.168.1.12 [110/30] via 10.1.1.1, 02:40:23, FastEthernet0/0
O E2    192.168.1.15 [110/30] via 10.1.1.1, 02:40:23, FastEthernet0/0
O E2    192.168.1.14 [110/30] via 10.1.1.1, 02:40:23, FastEthernet0/0
O E2    192.168.1.1 [110/30] via 10.1.1.1, 02:40:23, FastEthernet0/0
O E2    192.168.1.3 [110/30] via 10.1.1.1, 02:40:23, FastEthernet0/0
O E2    192.168.1.2 [110/30] via 10.1.1.1, 02:40:23, FastEthernet0/0
O E2    192.168.1.5 [110/30] via 10.1.1.1, 02:40:23, FastEthernet0/0
O E2    192.168.1.4 [110/30] via 10.1.1.1, 02:40:23, FastEthernet0/0
O E2    192.168.1.7 [110/30] via 10.1.1.1, 02:40:23, FastEthernet0/0
O E2    192.168.1.6 [110/30] via 10.1.1.1, 02:40:23, FastEthernet0/0
     192.168.2.0/32 is subnetted, 4 subnets
O E2    192.168.2.3 [110/30] via 10.1.2.1, 02:40:25, FastEthernet0/1
O E2    192.168.2.1 [110/30] via 10.1.2.1, 02:40:25, FastEthernet0/1
O E2    192.168.2.4 [110/30] via 10.1.2.1, 02:40:25, FastEthernet0/1
O E2    192.168.2.5 [110/30] via 10.1.2.1, 02:40:25, FastEthernet0/1

```


#### NR5
```julia
NR5#sh ip route                  
Codes: C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route

Gateway of last resort is not set

     1.0.0.0/32 is subnetted, 1 subnets
O       1.1.1.1 [110/20] via 10.1.1.6, 01:57:34, FastEthernet0/1
     2.0.0.0/32 is subnetted, 1 subnets
O IA    2.2.2.2 [110/40] via 10.1.1.2, 01:57:34, FastEthernet0/0
     10.0.0.0/30 is subnetted, 4 subnets
O IA    10.1.2.0 [110/20] via 10.1.1.2, 01:57:34, FastEthernet0/0
C       10.1.1.0 is directly connected, FastEthernet0/0
O IA    10.1.2.4 [110/30] via 10.1.1.2, 01:57:34, FastEthernet0/0
C       10.1.1.4 is directly connected, FastEthernet0/1
     192.168.1.0/32 is subnetted, 15 subnets
O E2    192.168.1.9 [110/30] via 10.1.1.6, 01:57:35, FastEthernet0/1
O E2    192.168.1.8 [110/30] via 10.1.1.6, 01:57:35, FastEthernet0/1
O E2    192.168.1.11 [110/30] via 10.1.1.6, 01:57:35, FastEthernet0/1
O E2    192.168.1.10 [110/30] via 10.1.1.6, 01:57:36, FastEthernet0/1
O E2    192.168.1.13 [110/30] via 10.1.1.6, 01:57:36, FastEthernet0/1
O E2    192.168.1.12 [110/30] via 10.1.1.6, 01:57:36, FastEthernet0/1
O E2    192.168.1.15 [110/30] via 10.1.1.6, 01:57:36, FastEthernet0/1
O E2    192.168.1.14 [110/30] via 10.1.1.6, 01:57:36, FastEthernet0/1
O E2    192.168.1.1 [110/30] via 10.1.1.6, 01:57:36, FastEthernet0/1
O E2    192.168.1.3 [110/30] via 10.1.1.6, 01:57:36, FastEthernet0/1
O E2    192.168.1.2 [110/30] via 10.1.1.6, 01:57:36, FastEthernet0/1
O E2    192.168.1.5 [110/30] via 10.1.1.6, 01:57:36, FastEthernet0/1
O E2    192.168.1.4 [110/30] via 10.1.1.6, 01:57:36, FastEthernet0/1
O E2    192.168.1.7 [110/30] via 10.1.1.6, 01:57:36, FastEthernet0/1
O E2    192.168.1.6 [110/30] via 10.1.1.6, 01:57:36, FastEthernet0/1

```
