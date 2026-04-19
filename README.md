##### Enterprise Network Infrastructure

##### Overview

###### This repository documents a high-availability, multi-vendor enterprise network core built in PNETLab. The architecture features a redundant core/distribution layer, a dual-homed WAN edge with eBGP, and a hardened access layer.


###### 

###### The primary focus of this project was implementing a "Zero Single Point of Failure" design using Active/Active HSRP load sharing and AAA RADIUS authentication integrated with Windows Server.

###### 

###### Architecture and Topology

###### WAN Edge (R1): DELTA\_TELECOM running eBGP (AS 100) peering with internal AS 65001.

###### 

###### Redundant Distribution (R2/R3): DST\_R001 and DST\_R002 running iBGP and HSRP.

###### 

###### Access Security (R5): ACC\_SW\_01 featuring DHCP Snooping, DAI, and Port Security.

###### 

###### Services: Windows Server NPS (RADIUS) for centralized management authentication.

###### 

###### Key Objectives Achieved

###### HSRP Load Sharing: Optimized bandwidth by splitting Active/Standby roles across the distribution pair for different VLAN groups.

###### 

###### External Connectivity: Established eBGP peerings with upstream providers using default-originate to provide internet reachability.

###### 

###### Redundancy Tracking: Implemented Object Tracking on WAN links to trigger sub-second HSRP gateway failover.

###### 

###### Tiered Management Access: Segmented VTY lines into "Daily" (RADIUS) and "Emergency" (Local) methods for resilient administration.

###### 

##### Configuration Highlights

###### 1\. HSRP Active/Active Gateway Setup

###### To ensure no hardware sits idle, I split the virtual gateways between DST\_R001 (R2) and DST\_R002 (R3).

###### 

###### DST\_R001 (Active for VLAN 10 and 99):

###### 

###### Plaintext

###### interface Ethernet0/1.10

###### &#x20;standby 1 ip 192.168.1.1

###### &#x20;standby 1 priority 110

###### &#x20;standby 1 preempt

###### &#x20;standby 1 track 1 decrement 20

###### DST\_R002 (Active for VLAN 20 and 55):

###### 

###### Plaintext

###### interface Ethernet0/1.55

###### &#x20;standby 55 ip 192.168.55.1

###### &#x20;standby 55 priority 110

###### &#x20;standby 55 preempt

###### &#x20;standby 55 track 1 decrement 20



##### 2\. Segmented AAA Management Access

###### I configured a dual-tier authentication strategy to prevent being locked out of the core infrastructure during a server failure.

###### 

###### Plaintext

###### ! AAA Method Definitions

###### aaa authentication login DAILY-METHOD group radius local

###### aaa authentication login EMERGENCY-METHOD local

###### 

###### ! VTY Line Segmentation

###### line vty 0 2

###### &#x20;description Primary Management Lines

###### &#x20;login authentication DAILY-METHOD

###### &#x20;transport input ssh

###### 

###### line vty 3 4

###### &#x20;description Out-of-Band / Emergency Access

###### &#x20;login authentication EMERGENCY-METHOD

###### &#x20;transport input ssh



##### 3\. Edge BGP Architecture

###### The ISP router (R1) provides redundant paths to the internal network (AS 65001).

###### 

###### Plaintext

###### router bgp 100

###### &#x20;neighbor 31.171.111.18 remote-as 65001

###### &#x20;neighbor 31.171.111.18 default-originate

###### &#x20;neighbor 31.171.111.22 remote-as 65001

###### &#x20;neighbor 31.171.111.22 default-originate



##### 4\. Layer 2 Security Hardening (Switch R5)

###### Implemented Dynamic ARP Inspection (DAI) and Port Security to prevent common MITM attacks.

###### 

###### Plaintext

###### ip arp inspection vlan 10,20,99

###### !

###### interface Ethernet0/3

###### &#x20;switchport access vlan 10

###### &#x20;switchport mode access

###### &#x20;switchport port-security maximum 2

###### &#x20;switchport port-security violation restrict



##### Real-World Troubleshooting and Solutions

##### Issue 1: AAA Lockout Prevention

###### Symptom: During the initial RADIUS configuration, the router could not reach the NPS server, and the local admin account would not work on the default VTY lines.

###### Resolution: Implemented the EMERGENCY-METHOD on lines 3-4, ensuring that even if the network path to the Windows Server is broken, local console-level access remains possible.

###### 

###### Issue 2: HSRP Gateway Preemption

###### Symptom: During a WAN failure, the Standby router took over, but the Primary did not reclaim the role after the link was restored.

###### Resolution: Applied the preempt command to all HSRP groups to ensure the "Priority 110" router always regains control when healthy.

###### 

###### Issue 3: DHCP Snooping Trust

###### Symptom: New users on VLAN 10 were not receiving IP addresses.

###### Resolution: Identified that the uplink trunk on ACC\_SW\_01 was missing the "ip dhcp snooping trust" command, causing the switch to drop the DHCP offers from the core.

###### 

###### Configuration Index

###### /network-configs/R1-Edge.cfg: eBGP and WAN connectivity.

###### 

###### /network-configs/DST\_R001.cfg: Core Routing, NAT, and HSRP Group 1.

###### 

###### /network-configs/DST\_R002.cfg: Core Routing, NAT, and HSRP Group 2.

###### 

###### /network-configs/ACC\_SW\_01.cfg: Layer 2 Hardening (Port Security, DAI, Snooping).

###### 

###### Related Projects

###### [SIEM and Security Integration](https://github.com/elmarxelilov10-ux/enterprise-SIEM-lab): See how I layered Splunk Enterprise and Syslog monitoring on top of this network.

