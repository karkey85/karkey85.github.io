---
layout: post
title: A route to BGP
date: 2022-07-25 09:45 +0530
---

## A route to BGP
BGP doesn't need any introduction. Network engineers come across this term every single day at their work. When we are surfing the internet, BGP works behind the hoods. Lets see how the BGP routes propagates in a tnetwork.

BGP runs as iBGP (internal BGP) in routers inside an Autonomous System (AS) and as eBGP (external BGP) in AS border routers across two different AS.

![img-description](/img/bgp1.png)
_BGP Topology_

**Internal BGP**
             
If two routers residing in the same Autonomous System are configured using same ASN, internal BGP kicks in. In above example,  R1 and R2 are present in same ASN 100. In order for BGP peering to happen, we need to ensure basic connectivity between these two routers. Even directly connected routes are sufficient for BGP peering to happen. We can have direct connectivity, static routes or any dynamic routing protocol like OSPF can be run on these R1 and R2.

GNS3 is used to simulate this topology with Cisco 7200 soft router.

Configure router R1:
 
    R1(config)#interface fastEthernet 0/0
    R1(config-if)#ip address 10.1.1.1 255.255.255.0
    R1(config-if)#no shutdown
    R1(config-if)#ex
    R1(config)#router bgp 100
    R1(config-router)#neighbor 10.1.1.2 remote-as 100

Configure router R2:

    R2(config)#interface fastEthernet 0/0
    R2(config-if)#ip address 10.1.1.2 255.255.255.0
    R2(config-if)#no shutdown
    R2(config-if)#ex
    R2(config)#router bgp 100
    R2(config-router)#neighbor 10.1.1.1 remote-as 100

Now, lets check if BGP is up and running on these routers

    R1#show ip bgp summary
    BGP router identifier 10.1.1.1, local AS number 100
    BGP table version is 1, main routing table version 1
    
    Neighbor        V          AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
    10.1.1.2        4        100       2       2        0    0    0 00:00:14        0

In bgp summary output, we can see bgp peering happened,  two messages are received and sent and Up/Down status started ticking. This shows BGP peering is successful. 
The "PfxRcd" field shows 0 which means there are no prefix exchanged from both peers over bgp protocol.

Lets add one more router R3 in the same AS 100.

    R3(config)#interface gigabitEthernet 1/0
    R3(config-if)#ip address 20.1.1.2 255.255.255.0
    R3(config-if)#no shutdown

Ping R1 from R3. Not reachable. 

    R3#ping 10.1.1.1
    
    Type escape sequence to abort.
    Sending 5, 100-byte ICMP Echos to 10.1.1.1, timeout is 2 seconds:
    .....
    Success rate is 0 percent (0/5)

It is time to run OSPF on R1, R2 and R3  to ensure reachability. Before that, let us experiment BGP peering using loopback address as neighbours.

Configure loopback interface and bgp in R2.

    R2(config)#interface Loopback 0
    R2(config-if)#ip address 2.2.2.2 255.255.255.255
    R2(config-if)#no shutdown
    R2(config-if)#ex
    R2(config)#router bgp 100
    R2(config-router)#neighbor 3.3.3.3 remote-as 100

Similarly we do this for R3.

    R3(config)#interface Loopback 0
    R3(config-if)#ip address 3.3.3.3 255.255.255.255
    R3(config-if)#no shutdown
    R3(config-if)#ex
    R3(config)#router bgp 100
    R3(config-router)#neighbor 2.2.2.2 remote-as 100

Check BGP peering in R3.

    R3#show ip bgp summary
    BGP router identifier 3.3.3.3, local AS number 100
    BGP table version is 1, main routing table version 1
    
    Neighbor        V          AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
    2.2.2.2         4        100       0       0        0    0    0 never    Idle

Since there is no routing information for loopback interfaces in R2 and R3, BGP peering didn't happen.

Lets enable OSPF in R2 and R3 an put them in area 0

    R2(config)#router ospf 100
    R2(config-router)#network 20.1.1.0 0.0.0.255 area 0
    R2(config-router)# network 2.2.2.2 0.0.0.0 area 0

**Please note that adding loopback alone to OSPF will not establish neighbourship. we need to add physical interface's network to area 0**

Configuring R3:

    R3(config)#router ospf 100
    R3(config-router)#network 20.1.1.0 0.0.0.255 area 0
    R3(config-router)#network 3.3.3.3 0.0.0.0 area 0

Finally show ip bgp summary outputs something but BGP peering is not done. 

    R3#show ip bgp summary
    BGP router identifier 3.3.3.3, local AS number 100
    BGP table version is 1, main routing table version 1
    
    Neighbor        V          AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
    2.2.2.2         4        100       0       0        0    0    0 never    Active

Hmm. Another important concept to note.
When R3 peers with R2 using R2's loopback address, BGP update packet is sent froom R3 to R2. The source address of the BGP packet would be physical address of R3 and destination address would be loopback 0 address of R2. 
But when R2 receives the packet, it checks the configured neighbor statements, and verifies that the neighbor address that is configured in R2, is the loopback 0 address of R3 and not the physical address of R3. So R2 discards the packet and BGP connection fails. In such a scenario, if the source address of the BGP update packet is modified as loopback 0 address of R3, BGP connection will succeed.

Configure R3 saying source is loop back interface. 

    R3(config-router)#neighbor 2.2.2.2  update-source Loopback 0
    R3(config-router)#ex
    R3#show ip bgp summary
    BGP router identifier 3.3.3.3, local AS number 100
    BGP table version is 1, main routing table version 1
    
    Neighbor        V          AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
    2.2.2.2         4        100       3       2        1    0    0 00:00:12        0


Now BGP peering is established between R2 and R3. Lets do the ping test from R3 to R1.

    R3#ping 10.1.1.1
    
    Type escape sequence to abort.
    Sending 5, 100-byte ICMP Echos to 10.1.1.1, timeout is 2 seconds:
    .....
    Success rate is 0 percent (0/5)

Ok. Time to enable OSPF between R1 and R2. 

    R2(config)#router ospf 100
    R2(config-router)#network 10.1.1.0 0.0.0.255 area 0
    
    R2#sho ip ospf neighbor
    
    Neighbor ID     Pri   State           Dead Time   Address         Interface
    10.1.1.1          1   FULL/DR         00:00:38    10.1.1.1        FastEthernet0/0
    3.3.3.3           1   FULL/DR         00:00:38    20.1.1.2        GigabitEthernet1/0

Now, R1 routes  are visible to R3 via ospf. Ping is successful between R3 and R1.

    R3#show ip route
    Codes: C - connected, S - static, R - RIP, M - mobile, B - BGP
           D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
           N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
           E1 - OSPF external type 1, E2 - OSPF external type 2
           i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
           ia - IS-IS inter area, * - candidate default, U - per-user static route
           o - ODR, P - periodic downloaded static route
    
    Gateway of last resort is not set
    
         2.0.0.0/32 is subnetted, 1 subnets
    O       2.2.2.2 [110/2] via 20.1.1.1, 00:17:24, GigabitEthernet1/0
         3.0.0.0/32 is subnetted, 1 subnets
    C       3.3.3.3 is directly connected, Loopback0
         20.0.0.0/24 is subnetted, 1 subnets
    C       20.1.1.0 is directly connected, GigabitEthernet1/0
         10.0.0.0/24 is subnetted, 1 subnets
    O       10.1.1.0 [110/2] via 20.1.1.1, 00:00:10, GigabitEthernet1/0
    R3#ping 10.1.1.1
    
    Type escape sequence to abort.
    Sending 5, 100-byte ICMP Echos to 10.1.1.1, timeout is 2 seconds:
    !!!!!
    Success rate is 100 percent (5/5), round-trip min/avg/max = 16/33/60 ms


