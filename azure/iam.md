1. What is the difference between Service principal and Managed Identity?

```yaml
|                                     | Service Principal                           | Managed Identity                                     |
| ----------------------------------- | ------------------------------------------- | ---------------------------------------------------- |
| **What is it?**                     | Identity for an application/service         | Identity for an **Azure resource/workload**          |
| **Credentials**                     | Secret or certificate typically required    | **No secret to manage**                              |
| **Who manages credentials?**        | **You**                                     | **Azure**                                            |
| **Credential rotation**             | You need to rotate them                     | Azure handles it                                     |
| **Azure VM can use it?**            | ✅ Yes                                       | ✅ Yes                                                |
| **On-prem VM can directly use it?** | ✅ Yes                                       | ❌ Not as a native managed identity                   |
| **Typical use**                     | External/on-prem apps, CI/CD                | Azure VM, AKS, App Service accessing Azure resources |
| **Security**                        | Good, but credential management is required | Usually preferred for Azure workloads                |
```
