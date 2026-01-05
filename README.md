CCNA COMMAND CHEAT SHEET (ENGLISH TRANSLATION)
References: visualsubnetcalc.com | exampointers.com | itexamanswers.net

================================================================================
PART 1: BASIC DEVICE CONFIGURATION
================================================================================

1. Basic Setup (Hostname & Banner)
----------------------------------
Router# configure terminal
Router(config)# hostname R1
! Banner Message of the Day (Delimiters must match, e.g., #)
R1(config)# banner motd #Access Denied! Authorized Personnel Only#

2. Line Passwords (Console & VTY)
---------------------------------
! -- Console (Physical) --
R1(config)# line console 0
R1(config-line)# password cisco
R1(config-line)# login
R1(config-line)# exit

! -- VTY (Remote Telnet/SSH) --
R1(config)# line vty 0 4
R1(config-line)# password cisco
R1(config-line)# login
R1(config-line)# exec-timeout 10 0    ! 10 mins 0 sec idle timeout
R1(config-line)# exit

! -- Encrypt Plaintext Passwords --
R1(config)# service password-encryption

3. Admin Privileges (Enable Password)
-------------------------------------
! Recommended (MD5 Hash)
R1(config)# enable secret mysecurepass
! Not Recommended (Plaintext)
! R1(config)# enable password myweakpass

4. IP Address & Gateway (Switch SVI)
------------------------------------
S1(config)# interface vlan 1
S1(config-if)# ip address 192.168.1.10 255.255.255.0
S1(config-if)# no shutdown
S1(config-if)# exit
! Default Gateway (for Switch management)
S1(config)# ip default-gateway 192.168.1.1

5. SSH Configuration (Secure Remote Access)
-------------------------------------------
! Prerequisite: Hostname must be set
R1(config)# ip domain-name example.com
R1(config)# username admin secret strongpassword123
R1(config)# crypto key generate rsa general-keys modulus 1024
R1(config)# ip ssh version 2
R1(config)# ip ssh authentication-retries 2
R1(config)# ip ssh time-out 60
! Apply to VTY
R1(config)# line vty 0 4
R1(config-line)# transport input ssh
R1(config-line)# login local
R1(config-line)# exit

6. Misc / Best Practices
------------------------
! Disable DNS Lookup (Prevent typos causing delays)
R1(config)# no ip domain-lookup
! Enforce Minimum Password Length
R1(config)# security passwords min-length 10
! Shortcuts:
! Ctrl+Shift+6 = Interrupt Process
! Ctrl+Z = Return to Privileged Exec (#)

================================================================================
PART 2: SWITCHING (VLANs, STP, EtherChannel)
================================================================================

1. VLANs & Trunking
-------------------
S1(config)# vlan 10
S1(config-vlan)# name Sales

! -- Access Port --
S1(config)# interface range f0/1-10
S1(config-if-range)# switchport mode access
S1(config-if-range)# switchport access vlan 10

! -- Voice VLAN --
S1(config-if)# mls qos trust cos
S1(config-if)# switchport voice vlan 20

! -- Trunk Port --
S1(config)# interface g0/1
S1(config-if)# switchport mode trunk
S1(config-if)# switchport trunk native vlan 99
S1(config-if)# switchport trunk allowed vlan 10,20,99
S1(config-if)# switchport nonegotiate

2. Spanning Tree Protocol (STP)
-------------------------------
S1(config)# spanning-tree mode pvst
! Root Bridge Selection
S1(config)# spanning-tree vlan 1 priority 24576
! OR Automatic
S1(config)# spanning-tree vlan 10 root primary
S1(config)# spanning-tree vlan 10 root secondary

! Edge Ports (PortFast & BPDUGuard)
S1(config-if)# spanning-tree portfast
S1(config-if)# spanning-tree bpduguard enable

3. EtherChannel (Link Aggregation)
----------------------------------
! LACP (Open Standard) - Active/Passive
S1(config)# interface range f0/21-22
S1(config-if-range)# shutdown
S1(config-if-range)# channel-group 1 mode active
S1(config-if-range)# no shutdown

! PAgP (Cisco) - Desirable/Auto
S1(config-if-range)# channel-group 2 mode desirable

! Configure Logical Interface
S1(config)# interface port-channel 1
S1(config-if)# switchport mode trunk
S1(config-if)# switchport trunk native vlan 99

4. Inter-VLAN Routing
---------------------
! Method A: Router-on-a-Stick
R1(config)# interface g0/0
R1(config-if)# no shutdown
R1(config)# interface g0/0.10
R1(config-subif)# encapsulation dot1q 10
R1(config-subif)# ip address 192.168.10.1 255.255.255.0

! Method B: L3 Switch (SVI)
S1(config)# ip routing
S1(config)# interface vlan 10
S1(config-if)# ip address 192.168.10.1 255.255.255.0
S1(config-if)# no shutdown

5. HSRP (Gateway Redundancy)
----------------------------
R1(config)# interface g0/1
R1(config-if)# standby version 2
R1(config-if)# standby 1 ip 192.168.1.254
R1(config-if)# standby 1 priority 150
R1(config-if)# standby 1 preempt

6. DHCPv4
---------
R1(config)# ip dhcp excluded-address 192.168.1.1 192.168.1.10
R1(config)# ip dhcp pool LAN-POOL
R1(dhcp-config)# network 192.168.1.0 255.255.255.0
R1(dhcp-config)# default-router 192.168.1.1
R1(dhcp-config)# dns-server 8.8.8.8

! DHCP Relay (Helper)
R2(config-if)# ip helper-address 10.1.1.2

7. IPv6 (SLAAC & DHCPv6)
------------------------
R1(config)# ipv6 unicast-routing

! Method 1: SLAAC (Auto)
R1(config-if)# ipv6 address 2001:db8:acad:1::1/64
R1(config-if)# no ipv6 nd managed-config-flag
R1(config-if)# no ipv6 nd other-config-flag

! Method 2: Stateless DHCPv6
R1(config-if)# ipv6 nd other-config-flag
R1(config)# ipv6 dhcp pool DNS-ONLY
R1(config-dhcp)# dns-server 2001:4860::8888

! Method 3: Stateful DHCPv6
R1(config-if)# ipv6 nd managed-config-flag
R1(config-if)# ipv6 nd prefix default no-autoconfig

8. Switch Security
------------------
! Port Security
S1(config-if)# switchport port-security
S1(config-if)# switchport port-security maximum 2
S1(config-if)# switchport port-security mac-address sticky
S1(config-if)# switchport port-security violation restrict

! DHCP Snooping
S1(config)# ip dhcp snooping
S1(config)# ip dhcp snooping vlan 10
S1(config)# interface g0/1
S1(config-if)# ip dhcp snooping trust

! DAI (Dynamic ARP Inspection)
S1(config)# ip arp inspection vlan 10
S1(config-if)# ip arp inspection trust
S1(config)# ip arp inspection validate src-mac ip

================================================================================
PART 3: ROUTING & WAN (OSPF, ACL, NAT, VPN)
================================================================================

1. OSPFv2
---------
! Enable Process
R1(config)# router ospf 10
R1(config-router)# network 10.0.0.0 0.0.0.3 area 0
! OR Interface method
R1(config-if)# ip ospf 10 area 0

! Optimization
R1(config-router)# passive-interface g0/1
R1(config-if)# ip ospf priority 255   ! Force DR
R1(config-if)# ip ospf network point-to-point
R2(config-router)# default-information originate

2. ACLs (Access Control Lists)
------------------------------
! Standard (Source only, 1-99)
R1(config)# access-list 1 deny 192.168.11.0 0.0.0.255
R1(config)# access-list 1 permit any
R1(config-if)# ip access-group 1 out

! Extended (Src/Dst/Port, 100-199)
R1(config)# access-list 100 permit tcp host 1.1.1.1 host 2.2.2.2 eq 80
R1(config)# ip access-list extended HTTP_ONLY
R1(config-ext-nacl)# permit tcp any any eq 80

3. NAT (Network Address Translation)
------------------------------------
! Static NAT
R2(config)# ip nat inside source static 172.16.1.1 64.100.50.1

! PAT (Overload)
R2(config)# access-list 1 permit 192.168.0.0 0.0.0.255
R2(config)# ip nat inside source list 1 interface s0/1/0 overload
! Define Interfaces
R2(config-if)# ip nat inside
R2(config-if)# ip nat outside

4. WAN (PPP & GRE)
------------------
! PPP & CHAP
R1(config)# username R2 password SecretPass
R1(config-if)# encapsulation ppp
R1(config-if)# ppp authentication chap

! GRE Tunnel
R1(config)# interface tunnel 0
R1(config-if)# ip address 10.0.0.1 255.255.255.0
R1(config-if)# tunnel source 1.2.3.4
R1(config-if)# tunnel destination 6.7.8.9
R1(config)# ip route 192.168.100.0 255.255.255.0 10.0.0.2

5. IPsec over GRE
-----------------
! Phase 1 (ISAKMP)
R1(config)# crypto isakmp policy 1
R1(config-isakmp)# hash sha
R1(config-isakmp)# authentication pre-share
R1(config-isakmp)# group 5
R1(config-isakmp)# encryption aes
R1(config)# crypto isakmp key KEY123 address 6.7.8.9

! Phase 2 (IPsec)
R1(config)# crypto ipsec transform-set MY_SET esp-aes esp-sha-hmac

! Map & ACL
R1(config)# ip access-list extended VPN_ACL
R1(config-ext-nacl)# permit gre host 1.2.3.4 host 6.7.8.9
R1(config)# crypto map MY_MAP 1 ipsec-isakmp
R1(config-crypto-map)# set peer 6.7.8.9
R1(config-crypto-map)# set transform-set MY_SET
R1(config-crypto-map)# match address VPN_ACL

! Apply to Outside Interface
R1(config)# interface s0/1/0
R1(config-if)# crypto map MY_MAP
