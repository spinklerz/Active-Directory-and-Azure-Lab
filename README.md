# Active-Directory-and-Azure-Lab
Designed and deployed a hybrid environment lab integrating on-premises Active Directory with a Microsoft Azure virtual networks using a site-to-site IPsec VPN. The environment simulates hybrid infrastructure with segmented networking, firewall enforcement, IPsec config, and secure cross-network authentication.

# Architecture Overview

## Network Topology

![Connection Vpn Details](/images/Network_Diagram.png)

## Network & Subnet Breakdown

### On-Prem Environment (VMware)

| Network | CIDR | Purpose |
|----------|------|----------|
| Home LAN | 192.168.1.0/24 | Upstream router network |
| DMZ / AD Network | 192.168.67.0/24 | Domain Controller + Workstations |


### Key On-Prem Hosts

| Device | IP Address | Role |
|----------|------------|------|
| pfSense WAN | 192.168.1.228 | Firewall uplink |
| pfSense LAN | 192.168.67.1 | Internal gateway |
| DC01 | 192.168.67.67 | AD DS, DNS |
| Windows 11 Client | 192.168.67.2 | Domain-joined workstation |


## Azure Environment

### Azure Network Configuration

| Resource | CIDR / IP | Purpose |
|------------|------------|----------|
| VNet | 10.0.0.0/24 | Azure address space |
| Workload Subnet | 10.0.0.0/25 | Azure VM subnet |
| GatewaySubnet | 10.0.0.128/25 | Required for VPN Gateway |
| Azure VM | 10.0.0.4 | Cloud-hosted Windows Server |


## Site-to-Site IPsec VPN Configuration
### Established a route-based IPsec tunnel between pfSense and Azure VPN Gateway.

| Type | Network |
|------|----------|
| Local Network | 192.168.67.0/24 |
| Remote Network | 10.0.0.0/24 |


### Phase 1 – IKEv2 Configuration

| Setting | Value |
|----------|--------|
| Authentication | Pre-Shared Key |
| Encryption | AES-256 |
| Integrity | SHA-256 |
| Diffie-Hellman Group | 2 |


### Phase 2 – IPsec (ESP) Configuration

| Setting | Value |
|----------|--------|
| Encryption | AES-256 |
| Integrity | SHA-256 |
| PFS | Enabled |
| Mode | Tunnel |





## On-Prem Deployment

- Created a VMware vnet
- Installed Windows Server 2022 on VMware
- Promoted server to Domain Controller (AD DS)
- Configured DNS for internal name resolution
- Created Organizational Units (OUs), security groups, and users
- Joined Windows 11 Pro workstation to the domain 
- Configured internal DHCP and gateway routing

## Firewall & Routing Configuration (pfSense)

- Defined WAN and LAN interfaces
- Configured Outbound NAT
- Enabled NAT-T (UDP 4500) for IPsec traversal
- Created firewall rules allowing IPsec traffic
- Configured static routes for Azure subnet
- Configured Phase 1 and 2 in the IPsec tunnel
- Used system logs to debug IPsec or connection issues
- Deployed pfSense with 2 network interfaces, WAN and LAN

## Azure Configuration

- Created a resource group called S2S-VPN
- Created Virtual Network (VNet) and subnets
- Deployed Azure VPN Gateway
- Configured Local Network Gateway 
- Defined Pre shared key and matching IPsec policies
- Configured Virtual Machine [smalldisk] Windows Server 2025 Datacenter: Azure Edition Core - x64 Gen2
- Verified tunnel status and successful data movement

### Resource Group
![Azure Resource Group](/images/Resource_Group.png)

### VPN Gateway
![Azure VPN Gateway](/images/VPN_Gateway.png)

### Local Network Gateway
![Azure Network Gateway](/images/Local_Network_Gateway.png)
![Azure Connection VPN Local](/images/Connection_VPN_Local.png)

### Virtual Machine
![Azure Virtual Machine](/images/Virtual_Machine.png)
![Azure WorkStation](/images/WorkStationAzure.png)
![Azure WorkStation](/images/AzureConfig.png)
![Azure WorkStation](/images/AzureTerminal.png)

## Tunnel Status pfSense
![Azure Virtual Machine](/images/IPsec.png)
## Deployment Outcome

- On prem AD env and with remote machines added to the domain
- Secured site to site IPsec VPN connectivity and tunneling
- Hybrid routing between on-prem (192.168.67.0/24) and Azure (10.0.0.0/24)
- Configured firewall policies to allow certain types of traffic into the network

## Validation & Testing

- Successfully tunnel connection via pfSense
- Network connection test validation via RDP/IMCP and domain joining
- DNS resolution across subnets
- `route print` and `arp -a` verification of remote subnet routing



## Troubleshooting & Challenges

### IPsec Phase Mismatch
Resolved encryption and Diffie-Hellman Group inconsistencies between Azure and pfSense. Originally I used DH Group 2 on the Azure VPN and configured IPsec on pfsense with DH Group 14, which caused phase 1 to fail. I fixed this by checking the system logs and tracking the connection that detailed a DH group mismatch. When configuring IPsec on pfSense, I didn't know you had to configure phases 1 and 2 separately, and this caused a phase 2 failure.

### Network Infra Issue
Corrected internal DNS forwarder configuration to support hybrid name resolution. The big lesson here is I need to configure the router or the gateway first, or else the other machines will have trouble and inconsistencies. 

### Understanding of Virtual Networks
I just found it conceptually challenging at first because in my home network we are given an address space by the ISP, to my knowledge. In a virtual network we basically just create an address space and fill it with machines. 

### Azure NSG Blocking Traffic
Modified the pfSense router to accept ICMP traffic as well as DNS traffic. This was causing DNS resolution issues. 

### RDP Azure 
When a machine is added to a domain, I did not know that the RDP credentials change to a specific user. For example, I logged in via RDP on the credentials I configured on Azure, then when I added the machine to the domain, those credentials from Azure were no longer valid, and I needed to log in as an admin or user. 

### Very Slow DNS resolution
DNS resolution was very slow and timing out very often, so I decided to go to the DNS settings on the Domain Controller and noticed that the forwarders were not configured correct and reaching the proper forwarding location. 

## Lessons Learned

- The role a domain controller plays in a network environment can be a single point of failure; thus, the importance of two domain controllers is essential. 
- IPsec tunnels fail silently when Phase 1/Phase 2 policies mismatch.
- Azure requires a properly sized GatewaySubnet.
- Firewall rule order directly impacts tunnel stability.
- How Vnets, Local Network Gateways, Gateway VPNs function
- DMZ networking
- Manually configuring DHCP, DNS, and IPv4 on Windows machines
- Using debugging tools like ping and nslookup to verify DNS and machine connection.
- How to configure an AD server
- How to add users, DNS configs, and machines to a domain
- Learned a ton about pfSense using system logs and visualization graphs to check throughput, firewall filtering, and management

## Future Improvements

- Integrate SIEM logging (Azure Sentinel or Wazuh)
- Deploy redundant domain controller
- Transition to certificate-based VPN authentication
- Conduct internal penetration testing against the segmented network (current)

