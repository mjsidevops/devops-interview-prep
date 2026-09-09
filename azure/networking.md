1. What is Hub and Spoke network topology
   
   <img width="1600" height="1168" alt="image" src="https://github.com/user-attachments/assets/fbb6079c-7ff2-4cfc-9b09-c0228fc8d55e" />

   - In Azure, a Hub-and-Spoke network is a common architecture where you create one central Hub VNet for shared networking services, and connect multiple Spoke VNets to it.
   - In a hub-and-spoke topology:
      - A hub virtual network acts as the central point of connectivity. It contains shared network services such as a firewall, gateway, and Bastion host.
      - Spoke virtual networks peer to the hub. Each spoke hosts a workload: an application, a team environment, or an isolated service. Spokes contain the actual application workloads.
      - VNet peering is non-transitive. Spokes can reach the hub, but spokes can't reach each other directly through the hub unless you configure routing or direct peering between them.
      - The Hub and Spokes are normally connected using VNet peering.
    


   Summary:
   Hub-and-Spoke is a network architecture where a central Hub VNet provides shared networking and security services, while multiple Spoke VNets host application workloads. The hub and    spokes are connected using VNet peering. Services such as Azure Firewall, VPN/ExpressRoute Gateway, Bastion and DNS can be centralized in the hub. This provides network isolation,  centralized security and routing, and makes the architecture easier to manage as the number of workloads grows.

<br><br>

2. What is Azure Bastion?
    - Bastion is a service that lets you securely connect to the VM using RDP/SSH without exposing the VMs to public internet.
  
<br><br>

3. What is Azure Resource Group?
    - Its a logical organization of resources
    - Contains group of resources work towards a common goal or application or project.
    - Better for resource life cycle management, for example Delete all resource at once, tagging, cost management, apply azure policy or assign a RBAC at RG level.
  
<br><br>

4. Azure Firewall:
    - Network security service that protects vnet resources by controlling and managing the inbound and outbound traffic based on the defined rules.
    - It is attached to the Vnet
    - Its statefull
    - By default it blocks all the traffic
    - Rule types:
       1. DNAT:
           - Destination network Address Translation
           - DNAT is used when you want to allow incoming traffic from the internet to reach a private resource inside your Azure VNet.
           - Translates Firewalls public IP and port to an internal private IP and port.
           - Ex: Reach the private App gateway from the internet via Firewall
       2. Network Rules:
           - Works at network and transport layer (layer 3 and 4)
           - Allows traffic based on IP and port
       3. Application rules:
           - Works at the application layer 7
           - Controls HTTP/HTTPS traffic based on the FQDN
           - Ex: When you want to allow a access to specific website or URL from the AKS subnet
   - Rules processing order: DNAT, n/w rules, and then appn rules
   - In route table we write UDR(user defined route) to send all traffic to Firewall.
      Ex:
     ```yaml
     Destination: 0.0.0.0/0
     Next hop: virtual appliances
     Next hop IP: Firewall private IP
     ```
          
