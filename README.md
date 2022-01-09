# README
## **Project:** S.C.A.R.L.E.T.
Secure, Cloud-based Automation, Redundancy, Logging, Exploitations, and Tactics 

 **Main Objective:** Demonstrate an automated, load balanced ELK stack solution to monitor and log exploits on the Damn Vulnerable Web Application (DVWA)
   - Built in Azure and automated with Ansible

**SCARLET Network:**

![Link an image](https://raw.githubusercontent.com/DigitalHammer/Scarlet/main/network-diagram.png "Scarlet Network Diagram")

---

**This document contains the following details:**
- Azure Network Breakdown
- Accessing Virtual Machines
- Building DVWA Servers
- Building ELK Server
- Elastic Beats Setup
- How to Use the Ansible Build

---

## **Azure Network Breakdown**

**Azure Resource Group (1):**
- All Azure resources were created under the same resource group: **ScarletRG**

**Virtual Networks (2):**
- (1) Virtual network is hosting the "Scarlet network."
  - *Includes:* the JumpBox, all DVWA servers, and a load balancer
- (1) Virtual network is hosting the ELK Server
  - *Includes:* the ELK server
- Both virtual networks are connected via VNET Peering

  | Name     | Network Range | Subnet |
  |----------|----------|------------|
  | Scarlet-Vnet | 10.0.0.0/16  | 10.0.0.0/24
  | ELK-VNet   | 10.10.0.0/16| 10.0.0.0/24 

**Virtual Machines (5):**
- (1) Jumpbox - Acts as the gateway to access all servers
- (3) Web servers - Used to host DVWA and create redundancy by being placed behind a load balancer
  - All web servers are placed inside the same availability set and within the same Azure region
  - Only one DVWA server is necessary for generating logs, others are for redundancy
- (1) ELK server - Provides monitoring of web servers
    - Hosted in a separate virtual network and Azure region than the JumpBox and DVWA servers

  | Name     | Function | IP | OS | Azure Size (minimum) |
  |----------|----------|------------|------------------|------------------|
  | Scarlet-JumpBox | Gateway  | 10.0.0.4   | Ubuntu 18.04 | Standard B1s |
  | Scarlet-Web1    | Host DVWA | 10.0.0.9  | Ubuntu 18.04 | Standard B1ms |
  | Scarlet-Web2    | Host DVWA | 10.0.0.10 | Ubuntu 18.04 | Standard B1ms |
  | Scarlet-Web3    | Host DVWA | 10.0.0.11 | Ubuntu 18.04 | Standard B1ms |
  | ELK-Server     | Host ELK Stack | 10.1.0.4 | Ubuntu 18.04 | Standard B2s |

**Network Security Groups (2):**
- (1) NSG created for the Scarlet-Vnet

  | Inbound Rules Name | Port     | Function |
  |----------|----------|----------|
  | Local-to-JumpBox | 22 | Allows public SSH access from local machine to JumpBox VM |
  | JumpBox-to-VNet | 22 |Allows internal SSH access from JumpBox to the DVWA servers |
  | Local-to-WebServers | 80 | Allows public HTTP access from local machine to DVWA servers |

- (1) NSG created for the ELK-Vnet

  | Inbound Rules Name  | Port   | Function |
  |----------|----------|----------|
  | JumpBox-to-ELK | 22 | Allows internal SSH access from JumpBox to ELK Server |
  | Local-to-ELK | 5601 |Allows public HTTP access from local machine to ELK Server to access Kibana Dashboard |

**Load Balancer (1)**
- (1) Load Balancer - Permits public HTTP access to DVWA and creates redundancy for all DVWA servers
  - All DVWA servers need to be placed in the same backend pool
  
  | Name     | Frontend IP | Backend Pool | 
  |----------|----------|----------|
  | Scarlet-LB | 20.69.153.158  | Scarlet-Backend-Pool|

 
---

## **Accessing Virtual machines**
*Note: In this setup, all public IP addresses are dynamic. Unless you have static IPs, you will need to update your network security groups (NSGs) with the public IP address assigned to the respective device at the time of access*
- **JumpBox:** SSH from local machine
  
  | Access From     | Access Type | Allowed IP Address | Protocol (Port) | Purpose | 
  |----------|---------------------|----------------------|----------------------|----------------------|
  | Scarlet-JumpBox | Public | Local Machine's Public IP | SSH (22) | Administration |
  
  Steps to access via SSH:
  - Generate SSH key from local machine and updated the VM in Azure
  - Verify inbound rule is created within NSG (see *Local-to-JumpBox* rule under Network Security Groups)
  - Example SSH command in GitBash:
    ```
    > ssh admin@20.69.167.144
    ```
- **DVWA Servers:** SSH from the Docker container inside the JumpBox to access servers administratively or access the Damn Vulnerable Web Application via HTTP in a web browser on local machine

  | Access From | Access Type | Allowed IP Address | Protocol (Port) | Purpose |
  |----------|---------------------|----------------------|----------------------|----------------------|
  | Scarlet-JumpBox | Internal | 10.0.0.4 | SSH (22) | Administration |
  | Local Machine | Public | Local Machine's Public IP | HTTP (80) | Access DVWA |

  Steps to access via SSH:
  - Generate SSH key while attached to Docker container in JumpBox and updated the VM in Azure
  - Verify inbound rule is created within NSG (see *JumpBox-to-VNet* rule under Network Security Groups)
  - Example string of commands to connect to server while in JumpBox:
    ```
    > sudo docker start [container_name]
    > sudo docker attach [container_name]
      > ssh admin@[web.server.internal.ip]
    ```
  Steps to access via HTTP:
    - Verify inbound rule is created within NSG (see *Local-to-WebServers* rule under Network Security Groups)
    - Input the following URL with the load balancer public IP address:
      > http://20.69.153.158/setup.php

- **ELK-Server** SSH from the Docker container inside the JumpBox to access servers administratively or access the Kibana Dashboard via HTTP in a web browser on local machine

  | Access From | Access Type | Allowed IP Address | Protocol (Port) | Purpose |
  |----------|---------------------|----------------------|----------------------|----------------------|
  | Scarlet-JumpBox | Internal | 10.0.0.4 | SSH (22) | Administration |
  | Local Machine | Public | Local Machine's Public IP | HTTP (5601) | Access Kibana Dashboard |

  - Generate SSH key while attached to Docker container in JumpBox and updated the VM in Azure
  - Verify inbound rule is created within NSG (see *JumpBox-to-ELK* rule under Network Security Groups)
  - Example string of commands to connect to server while in JumpBox:
    ```
    > sudo docker start [container_name]
    > sudo docker attach [container_name]
      > ssh admin@[elk.server.internal.ip]
    ```

  Steps to access via HTTP:
  - Verify inbound rule is created within NSG (see *Local-to-ELK* rule under Network Security Groups)
  - Input the following URL with the ELK server public IP address:
    > http://13.48.210.202:5601/app/kibana

---

## **Building the DVWA Servers**
- 
---

## **Building the ELK Server**
-
---

## **Elastic Beats Setup**
- 
---

## **How to Use the Ansible Build**
- 
  



