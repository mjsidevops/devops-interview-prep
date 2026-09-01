1. How to integrate Azure Key vault to AKS pods or how to securely provide secrets to running pods/containers?
   

Steps:
 1. While creating AKS enable the Key Vault CSI provider, which actually installs the CSI driver on the k8s cluster.
 2. Create a User Assigned Managed Identity and give access(Key Vault Secrets User) to respective key vault
 3. Create the Kubernetes ServiceAccount with client ID of the MI associated

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-app-sa
  annotations:
    azure.workload.identity/client-id: <managed-identity-client-id>
```
    
 5. Create the Federated Identity Credential which associates AKS OIDC issue + k8s service account + MI
 6. Create SecretProvideClass
```yaml
apiVersion: secrets-store.csi.x-k8s.io/v1
kind: SecretProviderClass
metadata:
  name: azure-kv-secrets
  namespace: my-app

spec:
  provider: azure

  parameters:
    usePodIdentity: "false"
    clientID: "<managed-identity-client-id>"
    keyvaultName: "my-keyvault"
    tenantId: "<tenant-id>"

    objects: |
      array:
        - |
          objectName: DB_PASSWORD
          objectType: secret

        - |
          objectName: API_KEY
          objectType: secret
```

  6. Mount it into the Pod
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app

spec:
  replicas: 3

  template:
    metadata:
      labels:
        app: my-app
        azure.workload.identity/use: "true"

    spec:
      serviceAccountName: my-app-sa

      containers:
        - name: my-app
          image: myapp:1.0

          volumeMounts:
            - name: secrets-store
              mountPath: /mnt/secrets
              readOnly: true

      volumes:
        - name: secrets-store
          csi:
            driver: secrets-store.csi.k8s.io
            readOnly: true

            volumeAttributes:
              secretProviderClass: azure-kv-secrets
```
Note: SecretProvideClass can also create k8s secrets and in pod it can be referenced as environment variable reference.
<br><br>

2. And how do you achieve the secret rotation with zero-downtime or without restarting the pod?

  - While creating the AKS, enable Secrets Store CSI Driver with auto-rotation enabled.
  - When I rotate the secret in Key Vault, the CSI driver periodically polls Key Vault and updates the mounted secret. The default rotation interval is two minutes, and it can be customized. 
  - The Pod itself doesn't need to restart when the secret is consumed through the mounted CSI volume.

  ```icl
  key_vault_secrets_provider {
     secret_rotation_enabled  = true
     secret_rotation_interval = "2m"
  }
  ```
<br><br>

3. How do you troubleshoot if the cluster node pool unable to scale?
   Answer:
    1. Check Cluster Autoscaler events/status
    2. Node pool minCount / maxCount and autoscaler configuration
    3. Subnet IP availability
  
<br><br>

4. How do you secure AKS cluster?
   1. Use Private AKS cluster, instead exposing k8s API over public internet
   2. Secure Authentication and Authorization using Microsoft Entra ID, Users, Groups and Managed Identities
   3. Use RBAC, use k8s role and role binding or azure RBAC wherever applicable. Follow least privilege
   4. Pod Security:
         - Use Pod Security Admission (PSA) and enforce appropriate security standards.
         - For example, prevent containers from:
            Running as root
            Using privileged mode
            Accessing host filesystem
            Using host networking unnecessarily
            Adding dangerous Linux capabilities
      ```yaml
      securityContext:
        runAsNonRoot: true
        allowPrivilegeEscalation: false
        readOnlyRootFilesystem: true
        capabilities:
          drop:
           - ALL
      ```
   5. Secure container images before deploying to AKS
   6. Private ACR, access it via Private Endpoint and proper MI(acrPull)
   7. Network Policy: Use Network Policies to control pod-to-pod communication.
        Without network policies:
          ```yaml
          Pod A ───────→ Pod B
          Pod A ───────→ Pod C
          Pod A ───────→ Pod D
          ```
        With network policies:
          ```yaml
           Pod A ──→ Pod B       ALLOWED
           Pod A ──X→ Pod C      DENIED
           Pod A ──X→ Pod D      DENIED
           ```
        For example:
        ```yaml
        apiVersion: networking.k8s.io/v1
        kind: NetworkPolicy
        metadata:
          name: allow-api
        spec:
          podSelector:
            matchLabels:
              app: api
        policyTypes:
         - Ingress
        ingress:
         - from:
           - podSelector:
               matchLabels:
                 app: frontend
        ```
        This means only the frontend pods can communicate with the API pods.
   8. Protect secrets, integrate AKS with Azure Key vault with CSI driver enabled.
   9. Runtime security for containers, monitor and alert any vulnarabilities using tools like Falco, Aqua
   10. Monitoring and auditing: Azure Activity Logs, Azure Monitor, Log Analytics, Microsoft Defender for Cloud, Datadog
   11. Azure Policy: Use Azure Policy for AKS to enforce security rules.
         Ex: "Privileged containers are not allowed."
