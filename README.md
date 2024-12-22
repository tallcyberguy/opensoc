# OpenSoC
OpenSource Security Operations Center

# Virtual Security Operations Center (VSOC) on Hetzner Cloud  

This repository contains the infrastructure-as-code (IaC) and documentation for our VSOC deployed on Hetzner Cloud.  This setup provides 
- Real-time threat monitoring
- Incident response capabilities
-Log aggregation for our internal network.  

## Architecture  

Key components include:  

✅ Wazuh as a SIEM to collect logs and create custom alerts.

✅TheHive as Security Incident Response Platform to create tickets and report.

✅Opnsense as a Firewall & OpenVPN to securely access the environment.

✅Tpot as a honeypot to generate alerts from real attack done to my public IP.

✅Windows machine to gather telemetry about attacks done manually.

✅Velociraptor to acquire forensics data from Windows machines.

## Deployment  

This VSOC is deployed using Hetzner Robot using EX44 server.
Server Specs 
AMD Radeon 7700 64 GB Ram 2x1 TB SSD Nvme 

**Prerequisites:**  

* Hetzner account  
* Motive to build things
* Technical reading
* Commandline skills

**Steps:** 

**Installing Proxmox
**
1. Create an account on Hetzner.  
2. Use Rescue system to install Custom image proxmox-debian-wormhole edition. Follow this >>  https://community.hetzner.com/tutorials/install-and-configure-proxmox_ve
3. Modify the partitioning settings before selecting start so right amount of storage is used by proxmox for localdisk. (I initially set it to 15 GB for no reason but was able extend  :) (https://www.tecmint.com/extend-and-reduce-lvms-in-linux/ )
4. Access Proxmox from Publicip:8006
5. Disable rpcbind on proxmox( automatically starts rpc so unused services should be disabled 
systemctl stop rpcbind
systemctl disable rpcbind)

**Networking**
Creating vmbr0 network
This is needed so that opnsense can be attached to host network. MAC Address should be the second IP's mac address given to you in Hetzner. DHCP on hetzner automatically assigns public IP based on the MAC.

<img width="1284" alt="Screenshot 2024-12-22 at 20 14 31" src="https://github.com/user-attachments/assets/eb5cff96-39df-41a1-9e12-69b9ad2b5709" />

Create vmb1 and vmbr2 for LAN and DMZ purposes and assing IP blocks. Do not provide a gateway.

<img width="1387" alt="Screenshot 2024-12-22 at 20 19 47" src="https://github.com/user-attachments/assets/1b5abe9a-98e6-4110-a3d7-bd755a91781d" />

VM Solution Installations ( Assuming you know your way around vms)

**Installing Opnsense** >> https://www.youtube.com/watch?v=vJBoCgptF-0 ( login with installer opnsense ) 

**Configuring DMZ** https://pfsensesetup.com/opnsenseinstall.com/configuring-the-dmz-in-opnsense-in-3-easy-steps/

**Installing TheHive** >> The easiest deployment is via docker. Follow the steps mentioned here and you are good to go. https://docs.strangebee.com/thehive/installation/docker/#step-2-run-the-application-stack

**Installing Wazuh** >> Create an ubuntu machine or cent os 7/8 and follow the steps. https://documentation.wazuh.com/current/quickstart.html

**Installing TPOT** >> Follow the steps in here. Basically download repo and run install.sh https://github.com/telekom-security/tpotce?tab=readme-ov-file#installation

**Installing Velociraptor** >> Can't go wrong with Markus Schober the man the myth if you want to install on Windows https://youtu.be/-bj0c158Wlo?si=l7sX6WcZiwwD2sfr on ubuntu https://www.youtube.com/watch?v=Jj0t4iXvdcU


## Monitoring & Maintenance
Create a documentation %100 you will need it. Otherwise machine passwords can be lost in a dust which is bad like really bad. 

In OPNSense set static dhcp leasings by going to Services>DHCP4>Leases>Add

In OPNSense enable IDS to generate alerts and inspect traffic which will be forwarded to wazuh via syslog.

Before updating opnsense or installing a plugin download the config System>Configuration>Backups>Download. You never know when you will need it.

In order to wazuh to generate it needs agents that delivers 7/24 so install agent on the machines.

Forward firewall logs/ suricata logs to wazuh. OPNSense > System>Settings>Logging>Remote create one.

Wazuh doesn't log all logs only alerts by default which is a wise policy but since we don't have storage issues we can enable logall option which is going to log all logs so that we can do queries in large logset.

Change these settings to true/yes to log all logs.

![Screenshot 2024-12-22 at 20 44 19](https://github.com/user-attachments/assets/400c5951-7a61-4fa0-a722-1ccfdf579763)

For maintenance and ease of deleting old records we can create a retention policy for logs older than 30 days to be deleted from the disk. 

https://documentation.wazuh.com/current/user-manual/manager/event-logging.html

Securing Linux computers is recommended but not needed since this is not a production environment.
useradd newuser -m -G sudo 
passwd newuser 

Installing ssh-server on host 
Sudo apt-get install openssh-server 
Ssh-copy-id –i ~/.ssh/id_ras.pub root@yourserver

SSH Config file modifications /etc/ssh/sshd_config
Port 12222 
PermitRootLogin no 
PasswordAuthentication yes 
PermitEmptyPasswords no 
PubkeyAuthentication yes 
AuthorizedKeysFile      .ssh/authorized_keys 


## Contact  
Linkedin > https://www.linkedin.com/in/hacksi/

Youtube > https://www.youtube.com/@ihacksi
