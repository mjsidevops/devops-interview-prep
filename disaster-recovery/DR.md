DR Strategy:

 - We have an active-passive multi-region AKS DR architecture.
 - The primary application runs in UK South and the secondary runs in Central US.
 - Each region has its own Azure Application Gateway and Azure Firewall.
 - Our public DNS hostname resolves to the primary region's Azure Firewall public IP. The firewall performs DNAT to the regional Application Gateway, which routes traffic to AKS.
 - During a regional disaster, we change the DNS record to the secondary region's Azure Firewall public IP. The secondary firewall then performs DNAT to the secondary Application Gateway, routing traffic to the secondary AKS cluster.
 - Once the primary region is recovered and validated, we perform the reverse DNS change for failback.
 - Take Uksouth(ups) as Primary and CentralUs(cus) as Secondary
 - Steps:
     1. Scale-down UKS
         - Set application replicas to 0
         - Disable HPA
         - Nodepool count set to 0
     2. Reverese Storage Replication
         - CUS storage account will be primary or source and UKS storage account will become secondary or destination

     3. SQL Failover:
         - Failover from UKS to CUS SQL server
         - CUS will become primary server and UKS will be secondary
     4. Scale-up CUS
         - Bring up the nodepools
         - Enable HPA
         - set application replicas to desired number.
  - Architecture:
```yam
                         Users
                           |
                           v
                    app.company.com
                           |
                         DNS
                           |
                +----------+----------+
                |                     |
             PRIMARY              SECONDARY
             UK South             Central US
                |                     |
                v                     v
        Azure Firewall        Azure Firewall
        Public IP: UK         Public IP: US
                |                     |
              DNAT                  DNAT
                |                     |
                v                     v
        Application GW        Application GW
                |                     |
                v                     v
             AKS UK               AKS US
                |                     |
               App                   App
```
