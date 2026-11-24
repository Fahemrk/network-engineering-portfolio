Small Office Network – VLAN, DHCP, Inter-VLAN Routing \& NAT

1\. Project Overview



This lab demonstrates how to configure a small office network using Cisco Packet Tracer.



The network includes:



VLAN segmentation

Router-on-a-Stick (Inter-VLAN Routing)

DHCP configuration

Switch trunking (802.1Q)

WAN connectivity

ISP router configuration

NAT Overload (PAT)

Public server connectivity



Main objective: Allow different departments to communicate through a router and access a public server through an ISP connection.



2\. Network Topology

text

&#x20;                PUBLIC SERVER

&#x20;                198.51.100.10

&#x20;                      |

&#x20;                      |

&#x20;                 ISP ROUTER

&#x20;             Gi0/1: 198.51.100.1

&#x20;             Gi0/0: 203.0.113.1

&#x20;                      |

&#x20;                      |

&#x20;                 R1 ROUTER

&#x20;             Gi0/1: 203.0.113.2

&#x20;             Gi0/0: VLAN TRUNK

&#x20;                      |

&#x20;                      |

&#x20;                   SW1

&#x20;                      |

&#x20;         +------------+------------+

&#x20;         |            |            |

&#x20;       VLAN 10      VLAN 20      VLAN 30

&#x20;       SALES          IT          GUEST



3\. IP Addressing Scheme

VLAN Configuration

VLAN	Name	Network	Gateway

10	SALES	192.168.10.0/24	192.168.10.1

20	IT	192.168.20.0/24	192.168.20.1

30	GUEST	192.168.30.0/24	192.168.30.1

99	MANAGEMENT	192.168.99.0/24	192.168.99.1

WAN Addressing

Device	Interface	IP Address

R1	Gi0/1	203.0.113.2/30

ISP	Gi0/0	203.0.113.1/30

Public Server Network

Device	IP Address

ISP	198.51.100.1

Public Server	198.51.100.10

Subnet Mask	255.255.255.0

Default Gateway	198.51.100.1

4\. Switch Configuration – SW1

VLAN Creation

cisco

enable

configure terminal



hostname SW1

no ip domain-lookup



vlan 10

&#x20;name SALES



vlan 20

&#x20;name IT



vlan 30

&#x20;name GUEST



vlan 99

&#x20;name MANAGEMENT



Access Port Configuration



Fa0/4 is reserved for the router trunk.



cisco

interface range fa0/1-3

&#x20;switchport mode access

&#x20;switchport access vlan 10

&#x20;spanning-tree portfast

&#x20;no shutdown



interface range fa0/5-10

&#x20;switchport mode access

&#x20;switchport access vlan 10

&#x20;spanning-tree portfast

&#x20;no shutdown



interface range fa0/11-20

&#x20;switchport mode access

&#x20;switchport access vlan 20

&#x20;spanning-tree portfast

&#x20;no shutdown



interface range fa0/21-24

&#x20;switchport mode access

&#x20;switchport access vlan 30

&#x20;spanning-tree portfast

&#x20;no shutdown



Trunk Configuration



The physical connection is:



SW1 Fa0/4 → R1 Gi0/0



cisco

interface fa0/4

&#x20;description TRUNK\_TO\_R1

&#x20;switchport mode trunk

&#x20;switchport trunk native vlan 99

&#x20;switchport trunk allowed vlan 10,20,30,99

&#x20;no shutdown



Management SVI

cisco

interface vlan 99

&#x20;description MANAGEMENT\_SVI

&#x20;ip address 192.168.99.2 255.255.255.0

&#x20;no shutdown



exit



ip default-gateway 192.168.99.1



end

write memory



5\. Router Configuration – R1

Basic Configuration

cisco

enable

configure terminal



hostname R1

no ip domain-lookup



DHCP Configuration



Excluded addresses:



cisco

ip dhcp excluded-address 192.168.10.1 192.168.10.10

ip dhcp excluded-address 192.168.20.1 192.168.20.10

ip dhcp excluded-address 192.168.30.1 192.168.30.10





SALES DHCP Pool:



cisco

ip dhcp pool SALES\_POOL

&#x20;network 192.168.10.0 255.255.255.0

&#x20;default-router 192.168.10.1

&#x20;dns-server 198.51.100.10





IT DHCP Pool:



cisco

ip dhcp pool IT\_POOL

&#x20;network 192.168.20.0 255.255.255.0

&#x20;default-router 192.168.20.1

&#x20;dns-server 198.51.100.10





GUEST DHCP Pool:



cisco

ip dhcp pool GUEST\_POOL

&#x20;network 192.168.30.0 255.255.255.0

&#x20;default-router 192.168.30.1

&#x20;dns-server 198.51.100.10



Router-on-a-Stick Configuration



Physical trunk interface:



cisco

interface gigabitEthernet 0/0

&#x20;description TRUNK\_TO\_SW1

&#x20;no ip address

&#x20;no shutdown





SALES:



cisco

interface gigabitEthernet 0/0.10

&#x20;encapsulation dot1Q 10

&#x20;ip address 192.168.10.1 255.255.255.0

&#x20;ip nat inside





IT:



cisco

interface gigabitEthernet 0/0.20

&#x20;encapsulation dot1Q 20

&#x20;ip address 192.168.20.1 255.255.255.0

&#x20;ip nat inside





GUEST:



cisco

interface gigabitEthernet 0/0.30

&#x20;encapsulation dot1Q 30

&#x20;ip address 192.168.30.1 255.255.255.0

&#x20;ip nat inside





MANAGEMENT:



cisco

interface gigabitEthernet 0/0.99

&#x20;encapsulation dot1Q 99 native

&#x20;ip address 192.168.99.1 255.255.255.0

&#x20;ip nat inside



WAN Configuration

cisco

interface gigabitEthernet 0/1

&#x20;description WAN\_TO\_ISP

&#x20;ip address 203.0.113.2 255.255.255.252

&#x20;ip nat outside

&#x20;no shutdown



Default Route

cisco

ip route 0.0.0.0 0.0.0.0 203.0.113.1



NAT Overload (PAT)

cisco

access-list 1 permit 192.168.10.0 0.0.0.255

access-list 1 permit 192.168.20.0 0.0.0.255

access-list 1 permit 192.168.30.0 0.0.0.255

access-list 1 permit 192.168.99.0 0.0.0.255



ip nat inside source list 1 interface gigabitEthernet 0/1 overload



end

write memory



6\. ISP Router Configuration

Basic Configuration

cisco

enable

configure terminal



hostname ISP

no ip domain-lookup



Connection to Customer Router

cisco

interface gigabitEthernet 0/0

&#x20;description LINK\_TO\_CUSTOMER\_R1

&#x20;ip address 203.0.113.1 255.255.255.252

&#x20;no shutdown



Public Server Network

cisco

interface gigabitEthernet 0/1

&#x20;description PUBLIC\_SERVER\_SEGMENT

&#x20;ip address 198.51.100.1 255.255.255.0

&#x20;no shutdown



end

write memory



7\. Public Server Configuration



Configure the server in Packet Tracer:



Setting	Value

IP Address	198.51.100.10

Subnet Mask	255.255.255.0

Default Gateway	198.51.100.1



The DNS service can be enabled separately if the server is intended to provide DNS.



8\. Verification Commands

Switch Verification

cisco

show vlan brief

show interfaces trunk

show spanning-tree

show ip interface brief





Check:



All four VLANs exist.

PC ports belong to the correct VLANs.

Fa0/4 is operating as a trunk.

Native VLAN is 99.

R1 Verification

cisco

show ip interface brief

show ip route

show ip dhcp pool

show ip dhcp binding

show ip nat translations

show ip nat statistics





Check:



All router interfaces are up/up.

All VLAN routes are connected.

DHCP pools are available.

PCs receive correct IP addresses.

NAT translations appear after generating traffic.

ISP Verification

cisco

show ip interface brief

show ip route





Check:



Gi0/0 is up/up.

Gi0/1 is up/up.

Both networks appear as directly connected.

9\. Connectivity Testing

Test 1 – DHCP



On each PC:



text

ipconfig /all





Expected:



Department	IP Range	Gateway

SALES	192.168.10.11–254	192.168.10.1

IT	192.168.20.11–254	192.168.20.1

GUEST	192.168.30.11–254	192.168.30.1

Test 2 – Default Gateway



From SALES:



text

ping 192.168.10.1





From IT:



text

ping 192.168.20.1





From GUEST:



text

ping 192.168.30.1



Test 3 – Inter-VLAN Routing



From SALES:



text

ping 192.168.20.11

ping 192.168.30.11





From IT:



text

ping 192.168.10.11



Test 4 – WAN Connectivity



From R1:



cisco

ping 203.0.113.1



Test 5 – Public Server Connectivity



From ISP:



cisco

ping 198.51.100.10





From R1:



cisco

ping 198.51.100.10





From SALES PC:



text

ping 198.51.100.10



Test 6 – NAT



On R1:



cisco

show ip nat translations

show ip nat statistics





Generate traffic from an internal PC to the public server before checking translations.



10\. Troubleshooting Lessons



During this lab, the following issues were encountered:



Issue 1: DHCP Request Failed



Cause: The switch trunk was configured on the wrong physical port.



Solution: Used CDP to identify the actual router connection.



cisco

show cdp neighbors





Identified:



SW1 Fa0/4

R1 Gi0/1 (initial connection)



The physical cabling was then corrected to match the intended configuration.



Issue 2: Wrong VLAN Assignment



Cause: A PC port was assigned to VLAN 10 instead of VLAN 20.



Solution:



cisco

interface fa0/3

&#x20;switchport mode access

&#x20;switchport access vlan 20



Issue 3: Inter-VLAN Ping Failure



Troubleshooting:



Checked DHCP assignments.

Checked VLAN membership.

Checked trunk status.

Checked router subinterfaces.

Checked the routing table.

Issue 4: Public Server Unreachable



Cause: Incorrect physical port connection.



Solution: Corrected the connection to the ISP public server interface (Gi0/1).



11\. What I Learned



By completing this lab, I practised:



Creating and naming VLANs.

Assigning switch ports to VLANs.

Configuring 802.1Q trunking.

Understanding native VLANs.

Configuring Router-on-a-Stick.

Creating DHCP pools.

Understanding IP addressing and subnet masks.

Configuring static default routes.

Configuring NAT Overload.

Connecting a customer router to an ISP.

Troubleshooting physical connections using CDP.

Verifying network connectivity using ping and show commands.

12\. Final Goal



The intended final network behaviour:



SALES, IT and GUEST PCs receive DHCP addresses.

PCs can reach their respective gateways.

Different VLANs communicate through R1.

R1 communicates with the ISP.

Internal PCs can access the public server.

NAT translates private addresses into the public WAN address.



Project: Small Office Network

Platform: Cisco Packet Tracer

Technologies: VLAN, DHCP, 802.1Q, Inter-VLAN Routing, Static Routing, NAT/PAT

