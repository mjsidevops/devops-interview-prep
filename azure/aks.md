1. How to integrate Azure Key vault to AKS pods or how to securely provide secrets to running pods/containers?
   

Steps:
 1. While creating AKS enable the Key Vault CSI provider, which actually installs the CSI driver on the k8s cluster.
 2. Create a User Assigned Managed Identity and give access(Key Vault Secrets User) to respective key vault
 3. Create the Kubernetes ServiceAccount with client ID of the MI associated apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-app-sa
  namespace: my-app
  annotations:
    azure.workload.identity/client-id: "<managed-identity-client-id>"
 4. Create the Federated Identity Credential which associates AKS OIDC issue + k8s service account + MI
 5. Create SecretProvideClass apiVersion: secrets-store.csi.x-k8s.io/v1
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

  6. Mount it into the Pod

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


2. And how do you achieve the secret rotation with zero-downtime or without restarting the pod?

  - While creating the AKS, enable Secrets Store CSI Driver with auto-rotation enabled.
  - When I rotate the secret in Key Vault, the CSI driver periodically polls Key Vault and updates the mounted secret. The default rotation interval is two minutes, and it can be customized. 
  - The Pod itself doesn't need to restart when the secret is consumed through the mounted CSI volume.

  key_vault_secrets_provider {
     secret_rotation_enabled  = true
     secret_rotation_interval = "2m"
  }

