Azure blueprints:
   - It will be retired by Jan 2027
   - Think of an Azure Blueprint as a predefined environment template for an organization.
   - Whenever we create a new Azure subscription, it must have certain security policies, RBAC roles, resource groups, and standard resources.
   - Instead of configuring all of these manually every time, you can define them together as a Blueprint and assign that Blueprint to subscriptions.
   - Microsoft describes Blueprints as a way to define a repeatable set of Azure resources and governance components that follows organizational standards.
   - Blueprint contains:

```yaml
| Artifact                    | Purpose                         |
| --------------------------- | ------------------------------- |
| **Azure Policy Assignment** | Enforce organizational rules    |
| **RBAC Role Assignment**    | Give users/services permissions |
| **ARM Template**            | Deploy Azure resources          |
| **Resource Group**          | Create standard resource groups |
```

   - Blueprint Definition = what should be deployed
   - Blueprint Assignment = where it should be deployed
   - Blueprint will be replaced by:
       - Deployment stacks: Microsoft describes a Deployment Stack as a resource that manages a group of Azure resources as a single cohesive unit.
       - Deployment Stacks provide resource lifecycle management and protection
       - Template Spec: is a reusable, versioned ARM template stored inside Azure or in Git.

<br><br>


Azure policy:
  - Azure Policy is a governance service in Azure that allows you to define and enforce rules for Azure resources.
  - Azure Policy checks whether your Azure resources follow your organization's rules and can prevent or flag resources that don't.
  - Example:
     - Suppose your company says "All Azure Storage Accounts must use HTTPS."
     - So you create a Azure policy which enforce Storage account must be created with HTTPS, if not Deny the creation or modify to HTTPS and deploy.
  - Initiative Definition:
      - An initiative Definition contains the actual rule.
      - Example: Allowed locations: East US, West Europe
      - The policy checks whether a resource is deployed in an allowed location.
  - Policy Assignment:
      You could assign the policy at:
        - Management Group
        - Subscription
        - Resource Group
        - Individual resource
  - Policy Effect:
     - The effect determines what Azure Policy does when a resource doesn't comply.
     - Common effect include Deny, Audit, Modify, Disable
  <br><br>
   
  Azure Landing Zone:
    - An Azure Landing Zone is a pre-designed Azure environment that provides the foundational infrastructure, governance, security, networking, and identity controls needed before application teams start deploying workloads.
    - Landing Zone = A ready-to-use, governed Azure environment where applications can be deployed safely.
    - Example:
```yaml
Azure Tenant
│
└── Management Group
      │
      └── Subscription
            │
            └── Resource Group
                  │
                  ├── AKS
                  ├── ACR
                  └── Key Vault
```

  - Tenant

    Your organization's Microsoft Entra ID boundary.

  - Management Group

    A container used to organize subscriptions and apply governance to them.

  - Subscription

    The actual Azure billing/resource boundary where you deploy resources.

  - Resource Group

    A logical container for resources.
  
  - Resources

    AKS, VM, Storage Account, Key Vault, VNet, etc.
  
    
             
