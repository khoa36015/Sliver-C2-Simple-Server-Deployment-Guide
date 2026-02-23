# Sliver-C2-Simple-Server-Deployment-Guide
<pre>
██████  ██      ██ ██    ██ ███████ ██████  
 ██       ██      ██ ██    ██ ██      ██   ██ 
  █████   ██      ██ ██    ██ █████   ██████  
      ██  ██      ██  ██  ██  ██      ██   ██ 
 ██████   ███████ ██   ████   ███████ ██   ██ 
</pre>
 >> Stealthy C2 Infrastructure on Azure <<
As a Red Teamer, you must know how to build your own C2 infrastructure. If you're a newcomer, follow the guide below to get started."
<p align="center">
  <img src="whoami.gif" width="600px" alt="C2 Connection Demo">
</p>

**Minimum System Requirements**

**Redirector**: At least 1 vCPU and 1GB RAM.

**C2 Server**: At least 2 vCPUs and 2GB RAM.

In this section, I will guide you through the provisioning and setup process.

**Network Architecture & Security Posture**
<p align="center">
  <img src="diagram.png" width="600px" alt="C2 Diagram">
</p>
The infrastructure is designed with a Tier-2 Stealth Model, ensuring the C2 backend remains completely invisible to the public internet.

**1. Component Roles**
-Redirector (Nginx): The frontline edge node. It handles SSL termination and filters incoming traffic before proxying to the backend.

-C2 Server (Sliver): The isolated "brain" of the operation. It resides in a private VNet with no Public IP, reachable only through the redirector.
**2. Traffic Control & Firewall Rules**
We implement a strict Allow-by-Exception policy using Network Security Groups (NSG):
<img width="834" height="282" alt="image" src="https://github.com/user-attachments/assets/91b06a85-142d-4b46-894e-78a42001ab76" />
<img width="957" height="237" alt="image" src="https://github.com/user-attachments/assets/565322e2-5446-4f76-b5cb-98ee0c28888e" />
**3. Priority-Based Enforcement Logic**
To ensure absolute isolation, we utilize a priority-tiering system for all network rules:

[!IMPORTANT]

Specific Rules (Priority < 1000): All authorized traffic between the Redirector and C2 is assigned a high-priority value.

Global Killswitch (Priority 1000): A catch-all Deny All rule is placed at the end of the stack to drop any unauthorized inbound or outbound traffic, effectively neutralizing internet access for the C2 backend.


