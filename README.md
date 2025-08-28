# small-office-network
Small Office Network - HR / IT / Sales
======================================

IP Plan
-------
HR:    192.168.1.0/26     Gateway: 192.168.1.1     DHCP: 192.168.1.10 - 192.168.1.62
IT:    192.168.1.64/26    Gateway: 192.168.1.65    DHCP: 192.168.1.70 - 192.168.1.126
Sales: 192.168.1.128/26   Gateway: 192.168.1.129   DHCP: 192.168.1.130 - 192.168.1.190

Device List
-----------
- R1 (Router) with 3 GigabitEthernet interfaces:
  - G0/0 -> HR switch (SW1)
  - G0/1 -> IT switch (SW2)
  - G0/2 -> Sales switch (SW3)
- SW1 (HR), SW2 (IT), SW3 (Sales)
- 2 PCs per department (example)

Router R1 Configuration (Cisco IOS style)
-----------------------------------------
enable
configure terminal

! HR interface
interface GigabitEthernet0/0
 description HR LAN
 ip address 192.168.1.1 255.255.255.192
 no shutdown

! IT interface
interface GigabitEthernet0/1
 description IT LAN
 ip address 192.168.1.65 255.255.255.192
 no shutdown

! Sales interface
interface GigabitEthernet0/2
 description Sales LAN
 ip address 192.168.1.129 255.255.255.192
 no shutdown

! Optional: DNS and NTP (adjust as needed)
ip name-server 8.8.8.8
ntp server 0.pool.ntp.org

! DHCP (exclude static ranges first)
ip dhcp excluded-address 192.168.1.1 192.168.1.9
ip dhcp excluded-address 192.168.1.65 192.168.1.69
ip dhcp excluded-address 192.168.1.129 192.168.1.129

! DHCP Pools
ip dhcp pool HR
 network 192.168.1.0 255.255.255.192
 default-router 192.168.1.1
 dns-server 8.8.8.8
 domain-name office.local

ip dhcp pool IT
 network 192.168.1.64 255.255.255.192
 default-router 192.168.1.65
 dns-server 8.8.8.8
 domain-name office.local

ip dhcp pool SALES
 network 192.168.1.128 255.255.255.192
 default-router 192.168.1.129
 dns-server 8.8.8.8
 domain-name office.local

end
write memory

Switch Baselines (SW1/SW2/SW3)
------------------------------
! Assign access ports to PCs
enable
configure terminal
! example for SW1:
interface range FastEthernet0/1 - 24
 switchport mode access
 spanning-tree portfast
end
write memory

Testing
-------
1) Connect PCs to respective department switch.
2) Set PC NICs to DHCP and verify they lease an IP from correct subnet.
3) From PC-HR1, ping 192.168.1.65 and 192.168.1.129 (router gateways for IT and Sales).
4) Ping across departments (e.g., PC-HR1 -> PC-IT1).

Notes
-----
- If using Packet Tracer, choose a router with >= 3 Gigabit/FE ports, or use sub-interfaces + VLANs (router-on-a-stick).
- For router-on-a-stick design, create VLAN 10 (HR), 20 (IT), 30 (Sales), trunk to R1, and use 'encapsulation dot1Q' on sub-interfaces.
