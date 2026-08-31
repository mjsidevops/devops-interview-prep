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

     
