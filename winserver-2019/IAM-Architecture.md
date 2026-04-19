\# Identity and Access Management (IAM) As-Built Documentation



\## Overview

To secure the network infrastructure, local router authentication was alternated in favor of centralized AAA (Authentication, Authorization, and Accounting). A Windows Server environment was deployed to serve as the Active Directory Domain Controller and Network Policy Server (NPS).



This document outlines the GUI configurations applied to the Windows Server to support RADIUS authentication for the Cisco routing and switching core.



\---



\##  1. Active Directory Domain Services (AD DS)

Instead of creating local admin accounts on every individual router, access is controlled centrally via Active Directory.



\* \*\*Domain Name:\*\* `lab.local`

\* \*\*Security Group Created:\*\* `NET\_ADMINS`

\* \*\*Access Strategy:\*\* Role-Based Access Control (RBAC). Only personnel placed inside the `NET\_ADMINS` security group are granted authorization to log into the network infrastructure.



!\[Active Directory Users Group](./images/ad-users.jpg)



\---



\##  2. Network Policy Server (NPS) / RADIUS

The Windows Server was configured to listen for and process authentication requests coming from the Cisco hardware.



\### RADIUS Clients Configured

The following network devices were manually added to the NPS console as authorized RADIUS clients:

1\. \*\*DST\_R001\*\* (Core Router 1) - IP: `10.10.10.2`

2\. \*\*DST\_R002\*\* (Core Router 2) - IP: `10.10.10.3`

3\. \*\*ACC\_SW\_01\*\* (Access Switch) - IP: `192.168.99.5`



\*Note: All devices were configured using the standard Cisco vendor template and secured with a unified Shared Secret key.\*



!\[NPS RADIUS Clients](./images/nps-clients.png)



\---



\##  3. Network Policy Rules

To bridge the Cisco routers to Active Directory, a custom Network Policy was created within NPS.



\* \*\*Policy Name:\*\* `NET\_ADMINS`

\* \*\*Condition:\*\* User's Active Directory account MUST be a member of the `lab.local\\NET\_ADMINS` Windows User Group.

\* \*\*Authentication Method:\*\* Unencrypted authentication (PAP/SPAP) enabled to support legacy Cisco AAA requests via standard RADIUS ports (1812/1813).

\* \*\*Action:\*\* Grant Access.



!\[NPS Network Policy Conditions](./images/nps-policy.png)



\### Traffic Flow Summary:

When an engineer attempts an SSH connection to a router, the router forwards the credentials to `192.168.55.10` (Windows Server). The NPS service checks the `NET\_ADMINS` policy, validates the password against Active Directory, and returns an "Access-Accept" packet, dropping the engineer directly into Privilege Level 15.

