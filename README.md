# README
## **Project:** S.C.A.R.L.E.T.
Secure, Cloud-based Automation, Redundancy, Logging, Exploitations, and Tactics 

 **Main Objective:** Demonstrate an automated, load balanced ELK stack solution to monitor and log exploits on the Damn Vulnerable Web Application (DVWA)
   - Built in Azure
   - Docker containers automated with Ansible
   - DVWA servers monitored by Elastic's Filebeat and Metricbeat
   - *Project will be expanded to include more vulnerable applications and more Elastic beats for monitoring*

**SCARLET Network:**

![Link an image](https://raw.githubusercontent.com/DigitalHammer/Scarlet/main/Resources/Images/network-diagram.png "Scarlet Network Diagram")

---

**This document contains the following details:**
- Azure Network Breakdown
- Accessing Virtual Machines
- Building DVWA Servers
- Building ELK Server
- Elastic Beats Setup: Filebeat and Metricbeat

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
*Note: These directions assume previous knowledge of Azure fundamentals and basic Linux command-line skills.*
- Build one or more virtual machines in Azure
  - Utilize Azure size stated in `Virtual Machines (5)` section above
    - The sizes listed are minimum requirements to run the server(s)
  - Building multiple DVWA servers creates redundancy but only one virtual machine is required to get DVWA up and running
  - Group all servers intended to host DVWA into the same `resource group, virtual network, and availability set`
  - Do not give them a public IP. The load balancer will provide the public IP to access DVWA over HTTP
- Verify network security group inbound rules have been created
  - See rules `Local-to-WebServers` and `JumpBox-to-VNet` under **Network Security Groups** section above
- Install Docker on JumpBox
  ```
  > sudo apt install docker.io
  > sudo systemctl status docker
  > sudo systemctl start docker
  ```
- Pull ansible container
  ```
  > sudo docker pull cyberxsecurity/ansible
  ```
- Start and attach to container
  ```
  > sudo docker start [container_name]
  > sudo docker attach [container_name]
  ```
- Creat new SSH key while inside Docker container
  ```
  > ssh-keygen
  > cat ~/.ssh/id_rsa.pub
  ```
- Copy new SSH key and update DVWA virtual machines in Azure
  - You can input the SSH key by going into each individual machine and going to the section `Support + troubleshooting` and click on `Reset password`
- Update Ansible hosts file with web server VMs' internal IP addresses
  - The Ansible hosts file is located in: `/etc/ansible/`
    ```
    > nano /etc/ansible/hosts
    ```
  - Input the group name in brackets (in this example `[webservers]` is the group name) then list the internal IP addresses below 
  - Include `ansible_python_interpreter=/usr/bin/python3` after each IP
  - Example:

    ![Link an image](https://raw.githubusercontent.com/DigitalHammer/Scarlet/main/Resources/Images/ansible-hosts-webservers.png "Scarlet Network Diagram")

- Update Ansible config file with the username of DVWA virtual machines
  - The Ansible config file is located in: `/etc/ansible/`
     ```
    > nano /etc/ansible/ansible.cfg
    ```
  - The syntax is `remote_user = [username]` 
  - In this example all the virual machines had the same username:

  ![Link an image](https://raw.githubusercontent.com/DigitalHammer/Scarlet/main/Resources/Images/ansible-config-remoteuser.png "Scarlet Network Diagram")

- Create and run the DVWA playbook
  - Create a YAML file in the `/etc/ansible/` directory
     ```
    > nano /etc/ansible/ansible-playbook.dvwa.yml
    ```
    - Copy and paste this [DVWA Playbook](Ansible-Playbooks/ansible-playbook-dvwa.yml) into the new YAML file, save, and exit  
    - (*Note: change `remote_user` and any directories if your setup is different than the ones in this file*)
  - Run DVWA playbook
    ```
     > ansible-playbook ansible-playbook.dvwa.yml
     ```
  
- Place DVWA servers behind a load balancer
  - Create a load balancer in the Azure portal
  - The load balancer needs to be in the same `resource group and location` as the DVWA servers
  - Give the load balancer a `frontend IP address`
    - This creates a public IP address so DVWA can be accessed through a browser over HTTP
  - Create a `backend pool` and add all DVWA servers to it
  - Create a `load balancing rule` and set the `backend pool` and `backendpool` to the newly created ones and set the ports to `80`

- Access DVWA
  - In a web browser use the following URL with the load balancer frontend IP address:
    > http://20.69.153.158/setup.php

---

## **Building the ELK Server**

- Build one virtual machine in Azure
  - Utilize Azure size stated in `Virtual Machines (5)` section above
  - Place VM in the same resource group as all other resources
  - Use SSH key inside the Ansible container created earlier
  - The server will be in its own `virtual network`, `network security group` and have a `public IP address`
- Verify network security group inbound rules have been created
  - See rules `JumpBox-to-ELK` and `Local-to-ELK` under **Network Security Groups** section above
- Create a VNET Peering if one is not in place between both virtual networks
- Update ansible hosts with new group name and ELK server IP address
  - SSH into the JumpBox, then start and attach to the Ansible container
    ```
    > ssh admin@20.69.167.144
      > sudo docker start [container_name]
      > sudo docker attach [container_name]
    ```
  - Open Ansible hosts file in `/etc/ansible/` directory
    ```
    > nano /etc/ansible/hosts
    ```
  - Input another group name in brackets (in this example `[elk]` is the group name) then list the internal IP address below 
  - Include `ansible_python_interpreter=/usr/bin/python3` the IP
  - Example:

    ![Link an image](https://raw.githubusercontent.com/DigitalHammer/Scarlet/main/Resources/Images/ansible-hosts-elk.png "Scarlet Network Diagram")

- Create and run the ELK playbook
  - Create a YAML file in the `/etc/ansible/` directory -- (*Note: directories can be setup based on preference*)
     ```
    > nano /etc/ansible/ansible-playbook-elk.yml
    ```
    - Copy and paste this [ELK Playbook](Ansible-Playbooks/ansible-playbook-elk.yml) into the new YAML file, save, and exit
    - (*Note: change `remote_user` and any directories if your setup is different than the ones in this file*)
  - Run ELK playbook
    ```
     > ansible-playbook ansible-playbook.elk.yml
     ```
- Access Kibana Dashboard
  - In a web browser use the following URL with the ELK server public IP address:
    > http://13.48.210.202:5601/app/kibana

---

## **Elastic Beats Setup: Filebeat and Metricbeat**
- Download and edit the [Filebeat](Elastic-beats-configs/filebeat-configuartion.yml) configuration file to the Ansible container:
    - SSH into the JumpBox, then start and attach to the Ansible container (if you aren't already there)
      ```
      > ssh admin@20.69.167.144
       > sudo docker start [container_name]
       > sudo docker attach [container_name]
      ```
  - Download the Filebeat configuration file:
    ```
    > curl https://raw.githubusercontent.com/DigitalHammer/Scarlet/main/Elastic-beats-configs/filebeat-configuartion.yml > /etc/ansible/filebeat-config.yml
    ```
  - Edit the Filebeat configuration file:
    ```
    > nano /etc/ansible/filebeat-config.yml
    ```
    - Under the `Elasticsearch Output` section - input the ELK server's internal IP, HTTP Port 9200, and Elastic'c default username and password (*if you have an Elastic account then you can use the username and password you have setup*)

        ![Link an image](https://raw.githubusercontent.com/DigitalHammer/Scarlet/main/Resources/Images/filebeat-config-add-host.png "Scarlet Network Diagram")

    - Under the `Kibana` section - input the ELK server internal IP and HTTP Port 5601

      ![Link an image](https://raw.githubusercontent.com/DigitalHammer/Scarlet/main/Resources/Images/filebeat-config-kibana.png "Scarlet Network Diagram")

    - Save and exit the Filebeat config file

- Download and edit the [Metricbeat](Elastic-beats-configs/metricbeat-configuartion.yml) configuration file to the Ansible container: 
  - SSH into the JumpBox, then start and attach to the Ansible container (if you aren't already there)
      ```
      > ssh admin@20.69.167.144
       > sudo docker start [container_name]
       > sudo docker attach [container_name]
      ```
  - Download the Metricbeat configuration file:
    ```
    > curl https://raw.githubusercontent.com/DigitalHammer/Scarlet/main/Elastic-beats-configs/metricbeat-configuartion.yml > /etc/ansible/metricbeat-config.yml
    ```
  - Edit the Metricbeat configuration file:
    ```
    > nano /etc/ansible/filebeat-config.yml
    ```
    - Under the `Elasticsearch Output` section - input the ELK server's internal IP, HTTP Port 9200, and Elastic'c default username and password (*if you have an Elastic account then you can use the username and password you have setup*)

        ![Link an image](https://raw.githubusercontent.com/DigitalHammer/Scarlet/main/Resources/Images/filebeat-config-add-host.png "Scarlet Network Diagram")

    - Under the `Kibana` section - input the ELK server internal IP and HTTP Port 5601

      ![Link an image](https://raw.githubusercontent.com/DigitalHammer/Scarlet/main/Resources/Images/filebeat-config-kibana.png "Scarlet Network Diagram")

    - Save and exit the Metricbeat config file

- Download and run the Elastic beats playbook 
  - *Notes:* 
    - *This example was done with a single playbook for both Filebeat and Metricbeat. These can be done with individual playbooks by keeping the same header and removing the commands of the beat you do not want*
    - *If you have organized your directories differently, you'll need to modify the source and destination directories in the playbook*
  - Download the Ansible playbook for the beats:
    ```
    > curl https://raw.githubusercontent.com/DigitalHammer/Scarlet/main/Ansible-Playbooks/ansible-playbook-elastic-beats.yml > /etc/ansible/ansible-playbook-elastic-beats.yml
    ```
  - Run Elastic beats playbook
    ```
     > ansible-playbook ansible-playbook-elastic-beat.yml
     ```
- Verify Kibana is receiving Filebeat logs:
  - Go to your Kibana Dashboard with your ELK Server public IP address:
    > http://13.48.210.202:5601/app/kibana
  - In the Kibana dashboard under `Logs` click on `Add log data`
    - Click on `System logs`
    - Scroll down to `Module status` and click on `Check data`
    - *It will prompt you of a successful or unsuccessful data retreival*
- Verify Kibana is receiving Metricbeat logs:
  - Go to your Kibana Dashboard with your ELK Server public IP address:
    > http://13.48.210.202:5601/app/kibana
  - In the Kibana dashboard under `Metrics` click on `Add metric data`
    - Click on `System metrics`
    - Scroll down to `Module status` and click on `Check data`
    - *It will prompt you of a successful or unsuccessful data retreival*


---

## **Future of the project**
- SCARLET will be utilized dominantely for training and educational purposes. I intend to use this for security write-ups as I pen test more machines and capture logs and packets in real-time. 
- **Next additions include:**
  - More virtual machines with varied vulnerable servers
    - I will rotate these machines to keep Azure costs down
  - More monitoring
    - Packetbeat will be the next monitoring tool to be added