1. What is `terraform init`?
   - Initializes the provider plugin, eg: AzureARM provider for Azure
   - Initializes backend
   - Downloads the modules if any
   - Creates/updates terraform.lock.hcl, it lock the terraform provider version.
  
<br><br>

2. What is terraform state file?
   - It's a terraform's record of infrastructure it manages. Current state of the infrastructure.
   - It is stored in terraform.tfstate

<br><br>

3. Diff between count and for_each?
   - count
     - count is useful when the instances are essentially identical or indexed.
     - best for Identical/similar resources
     - access with count.index
     - input can be number or list
`  
resource "azurerm_resource_group" "rg" {
  count    = 3
  name     = "app-rg-${count.index}"
  location = "UK South"
} `
   
   - for_each
    - Best for Resources with meaningful names/configurations
    - access with each.key, each.value
    - input is mostly map

   - count creates multiple resource instances using numeric indexes, while for_each creates instances using keys from a map or set.
   - I use count when the resources are essentially identical and quantity-based.
   - I prefer for_each when resources have meaningful names or different configurations because the resource addresses are key-based and are generally more stable when items are added or removed.

<br><br>

4. How do you setup terraform for multiple environments say DEV, QA and PROD?
   - Use terraform modules
   - Example setup
`
terraform/
│
├── environments/
│   ├── dev/
│   ├── staging/
│   └── prod/
│
└── modules/
    ├── network/
    ├── aks/
    ├── keyvault/
    ├── storage/
    └── monitoring/
`
  - Inside environment it will call module

```
module "network" {
  source = "../../modules/network"

  vnet_name     = "dev-vnet"
  address_space = ["10.10.0.0/16"]
}
```

  - A module normally contains: main.tf, variables.tf and output.tf
  - Output from one module can be consumed in another module
  - For example network module output has
```yaml
output "vnet_id" {
  value = azurerm_virtual_network.this.id
}
```
  - Now AKS module can consume this vnet id
```yaml
module "aks" {
  source = "../../modules/aks"

  vnet_id = module.network.vnet_id
}
```

