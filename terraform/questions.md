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
```yaml
terraform/
│. 
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
```
   - Inside environment it will call module

```
module "network" {
  source = "../../modules/network"

  vnet_name     = "dev-vnet"
  address_space = ["10.10.0.0/16"]
}
```
   - Environment normally contains: backend.tf, provider.tf, main.tf, terraform.tfvars
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

<br><br>

5. What is depends_on?
   - Terraform normally creates an implicit dependency when one resource references another.
   - depends_on is used when Terraform cannot automatically determine a dependency.
  
<br><br>

6. What are terraform life cycle options?
   - create_before_destroy
   - prevent_destroy
   - ignore_changes

<br><br>

7. How terraform identifies the drift?
   - Terraform identifies the drift during the terraform plan
   - When someone manually change the configuration via the Azure portal, terraform plan will show that drift.
  
<br><br>

8. How do you securely use secrets in terraform? For example while creating Azure SQL resource, it needs admin password how do you pass it?
   - Store the admin password in the Azure key vault secret
   - Terraform can retrieve it with the azurerm_key_vault_secret data source and pass it to azurerm_mssql_server
```yaml
# Existing Key Vault
data "azurerm_key_vault" "main" {
  name                = "my-prod-kv"
  resource_group_name = "my-prod-rg"
}

# Existing secret in Key Vault
data "azurerm_key_vault_secret" "sql_admin_password" {
  name         = "sql-admin-password"
  key_vault_id = data.azurerm_key_vault.main.id
}

resource "azurerm_mssql_server" "sql" {
  name                = "my-prod-sql-server"
  resource_group_name = "my-prod-rg"
  location            = "UK South"

  version = "12.0"

  administrator_login          = "sqladmin"
  administrator_login_password = data.azurerm_key_vault_secret.sql_admin_password.value

  minimum_tls_version       = "1.2"
  public_network_access_enabled = false
}
```
 - But the password is stored in terraform state file, so we need to securely store and encrypt the state file.
 - Instead of `administrator_login_password` attribute we can use `administrator_login_password_wo` so that the password is not stored in state file.
 - It is recommended to use Microsoft Entra ID authentication instead of SQL authentication.

<br><br>

9. What is terraform import?
   - terraform import is used to bring the manually created resources into terraform state.
   - Ex: terraform import azurerm_storage_account.existing <resource-id>
   - Here resource-id is the actual resource ID of that resource created in Azure.
   - First you need to import the resource so that resource will be comes to terraform state
   - Second you need to create the terraform configuration manually to match the actual resource configuration.

<br><br>

10. What is dynamic block in terraform?
    - A dynamic block is used when we need to generate multiple nested blocks dynamically based on a collection such as a list, set, or map.
    - Instead of manually writing the same nested block multiple times, we can use dynamic.
    - Example: Suppose your app gateway have multiple listeners
```yaml
Application Gateway
 ├── listener: app.example.com
 ├── listener: api.example.com
 └── listener: admin.example.com
```
   - Instead of hard-coding each http_listener block, we can generate them dynamically.
   - Define variable for listeners
```yaml
variable "listeners" {
  type = map(object({
    host_name = string
    port      = number
    protocol  = string
  }))

  default = {
    app = {
      host_name = "app.example.com"
      port      = 443
      protocol  = "Https"
    }

    api = {
      host_name = "api.example.com"
      port      = 443
      protocol  = "Https"
    }

    admin = {
      host_name = "admin.example.com"
      port      = 443
      protocol  = "Https"
    }
  }
}
```
  - Dynamic block
```yaml
resource "azurerm_application_gateway" "appgw" {
  name                = "my-appgw"
  resource_group_name = azurerm_resource_group.example.name
  location            = azurerm_resource_group.example.location

  # Other required Application Gateway configuration...

  dynamic "http_listener" {
    for_each = var.listeners

    content {
      name                           = "${http_listener.key}-listener"
      frontend_ip_configuration_name = "public-frontend"
      frontend_port_name             = "https-port"
      protocol                       = http_listener.value.protocol
      host_name                      = http_listener.value.host_name
    }
  }
}
```
  - A dynamic block allows me to generate repeatable nested blocks dynamically. For example, in Azure Application Gateway, if I have multiple HTTP listeners, instead of manually creating multiple http_listener blocks, I can define the listeners as a map and use a dynamic http_listener block with for_each. This makes the Application Gateway configuration reusable and easier to maintain when the number of listeners changes.
   
