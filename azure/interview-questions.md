1. Difference between Azure Firewall and NSG?
   - NSG is a basic stateful network traffic filtering mechanism applied at the subnet or NIC level, mainly using source/destination IP, port and protocol.
   - Azure Firewall is a centralized managed stateful firewall that provides more advanced capabilities such as application and network rules, FQDN filtering, DNAT
   - NSG = "Who can talk to this subnet/NIC on which port?"
   - Azure Firewall = "What traffic is allowed to pass through this network security boundary?"

<br>

2. What is UDR(User Defined routes) and why would you use it?
   - A UDR allows us to define custom routing for subnet traffic. Basically ovewrite auto generated system routes.
   - Its attached to subnet
   - Commonly to force traffic through Azure Firewall or another network appliance for security, inspection, or connectivity requirements.
   - By default, Azure automatically creates system routes for traffic between VNets, subnets, and Azure services. With UDRs, you can override or supplement those routes and define your own next hop.
   - Example: Route all outbound traffic from AKS subnet via the Azure Firewall
`
AKS subnet
 |
 | 0.0.0.0/0
 v
Azure Firewall
 |
Internet
`

3. What is VNet Peering? How is it different from VPN Gateway?
   - VNet Peering connects Azure VNets privately through the Azure backbone. Traffic doesn't need to traverse the public internet.
   - VPN Gateway is a Encrypted VPN connectivity, commonly Azure-to-on-premises or other networks over the public internet.


4. Why do we need a Private DNS Zone with a Private Endpoint?
   - A Private Endpoint provides a private IP and private network connectivity to an Azure service, but applications normally connect using the service's FQDN.
   - A Private DNS Zone allows that FQDN to resolve to the Private Endpoint's private IP instead of the public IP.
   - Therefore, Private Endpoint provides private connectivity, while Private DNS provides private name resolution.
   - We link the Private DNS Zone to the required VNets so workloads such as AKS pods or VMs can resolve the service privately.
