# README
## **Project:** S.C.A.R.L.E.T.
Secure, Cloud-based Automation, Redundancy, Logging, Exploitations, and Tactics 

 **Main Objective:** Demonstrate an automated, load balanced ELK stack solution to monitor and log exploits on the Damn Vulnerable Web Application (DVWA)
   - Built in Azure and automated with Ansible

SCARLET Network:

![Link an image](https://raw.githubusercontent.com/DigitalHammer/Scarlet/main/network-diagram.png "Scarlet Network Diagram")

**This document contains the following details:**
- Network Breakdown
- Accessing Virtual Machines
- Building DVWA Servers
- Building ELK Server
- Elastic Beats Setup
- How to Use the Ansible Build

### **Network Breakdown**
**Virtual Machines (5):**
  - 1 Jumpbox - Acts as the gateway to access all servers
  - 3 Web servers - Used to host DVWA and create redundancy by being placed behind a load balancer
    - All web servers are placed inside the same availability set and within the same Azure region
  - 1 ELK server - Provides monitoring of web servers
    - hosted in a separate virtual network and Azure region than other VMs

| Name     | Function | IP Address | Operating System | Azure Size |
|----------|----------|------------|------------------|------------------|
| Scarlet-JumpBox | Gateway  | 10.0.0.4   | Ubuntu 18.04 | Standard B1s |
| Scarlet-Web1    | Host DVWA | 10.0.0.9  | Ubuntu 18.04 | Standard B1ms |
| Scarlet-Web2    | Host DVWA | 10.0.0.10 | Ubuntu 18.04 | Standard B1ms |
| Scarlet-Web3    | Host DVWA | 10.0.0.11 | Ubuntu 18.04 | Standard B1ms |
| ELK-Server     | Host ELK Stack | 10.1.0.4 | Ubuntu 18.04 | Standard B2s |

**Virtual Networks (2):**
  - One vnet is hosting the "Scarlet network." This consists 
  - One vnet is hosting the ELK Server
  - Connected via VNET Peering

| Name     | Network Range | Subnet |
|----------|----------|------------|
| Scarlet-Vnet | 10.0.0.0/16  | 10.0.0.0/24
| ELK-VNet   | 10.10.0.0/16| 10.0.0.0/24



**Load Balancer**
- 1 Load Balancer


---
<br>

### Accessing Virtual machines
- 

| Name     | Publicly Accessible | Allowed IP Addresses |
|----------|---------------------|----------------------|
| Scarlet-JumpBox | Yes              | Local Machine's Public IP    |
| Scarlet-Web1 |  |                      |
| Scarlet-Web2 |                     |                      |
| Scarlet-Web3 |                     |                      |
| ELK-Server |                     |                      |

---
<br>

### **Building DVWA Servers**
- 
---
<br>

### **Building ELK Server**
-
---
<br>

### **Elastic Beats Setup**
- 
---
<br>

### **How to Use the Ansible Build**
- 
  



