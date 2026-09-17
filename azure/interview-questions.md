1. Difference between Azure Firewall and NSG?
   - NSG is a basic stateful network traffic filtering mechanism applied at the subnet or NIC level, mainly using source/destination IP, port and protocol.
   - Azure Firewall is a centralized managed stateful firewall that provides more advanced capabilities such as application and network rules, FQDN filtering, DNAT
   - NSG = "Who can talk to this subnet/NIC on which port?"
   - Azure Firewall = "What traffic is allowed to pass through this network security boundary?"
