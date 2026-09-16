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
    
 5. Create the Federated Identity Credential which associates AKS OIDC issuer + k8s service account + MI
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
   3. Use RBAC, use k8s RBAC role and role binding or azure RBAC wherever applicable. Follow least privilege
   4. Pod Security:
         - Use Pod Security Admission (PSA) and enforce appropriate security standards.
         - For example, prevent containers from:
            - Running as root
            - Using privileged mode
            - Accessing host filesystem
            - Using host networking unnecessarily
            - Adding dangerous Linux capabilities
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
   7. Network Policy:
       - Use Network Policies to control pod-to-pod communication.
       - By default all pods in the k8s cluster can talks to each other.
       - We can create Network policy with Ingress or Egress rules to control the inbound and outbound traffic to the pod.
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
   9. Protect secrets, integrate AKS with Azure Key vault with CSI driver enabled.
   10. Runtime security for containers, monitor and alert any vulnarabilities using tools like Falco, Aqua
   11. Monitoring and auditing: Azure Activity Logs, Azure Monitor, Log Analytics, Microsoft Defender for Cloud, Datadog
   12. Azure Policy: Use Azure Policy for AKS to enforce security rules.
         Ex: "Privileged containers are not allowed."

<br><br>

5. What is container networking? And what are the networking options in AKS?
    - Container networking is how pods get ip addresses, how it communicates with other pods, nodes and external services and how traffic enters and leaves the cluster.
    - Networking is implemented via Azure CNI plugins
    - Other CNI plugins are Calico, planner, cilium
    - AKS supports only Azure CNI
    - Types
       1. Azure CNI - Node Subnet:
           - Pods gets IP address directly from the node's subnet
           - Requires more IPs, leads subnet exhaustion
           - Provides full VNet connectivity for pods, allowing them to be directly reached via their private IP address from connected networks.
       2. Azure CNI Overlay:
           - Here each node gets IP from the subnet
           - Pods gets IP from the Overlay CIDR
           - Communication with external endpoints uses network address translation (NAT) through the node IP. This model conserves IP address space and supports large-scale clusters.
         
       3. Azure CNI Pod Subnet:
           - Azure CNI Pod Subnet is a networking model in Azure Kubernetes Service (AKS) that assigns IP addresses to pods from a separate subnet than the one used for cluster nodes.

<br><br>

6. What is Network policies?
    - Controls how pods are allowed to communicates with each other by ingress and egress rules.
    - In Azure it uses Azure Network policy engine to enforce n/w policies.
  
<br><br>

7. How do you achieve High Availability(HA) in AKS?
    - Control Plane HA will be managed by Azure itself
    - For node pool HA, provision it in multiple Availability zones (1, 2, 3)
    - For Pods use HPA or VPA
    
<br><br>

8. How do you upgrade K8s cluster with Zero-downtime?
   - Pre requisite:
      1. Cordon nodes -> Make nodes unschedulable, so no new deployments will happen on the node.
      2. Inform the team about the upgrade and scheduled upgrade time
      3. Release notes:
          - Understand the impact of the upgrade to a existing workload.
          - Read the change logs
          - K8s upgrades are irreversible so you can't rollback to the previous version, you need to create new cluster.
      4. Upgrade the lower environment first, test it properly in Dev and Preprod before moving to Prod
      5. Upgrade process:
          1. Upgrade the Control plane
          2. Upgrade the Node pools
          3. Upgrade the Add-ons, may be upgrade the ingress to sync with latest k8s version
      6. K8s will do rolling update, that means it will upgrade 1 node at a time inside nodepools
      7. A new buffer node will be created with the specified k8s version, then it will cordon and drain the older node.
      8. Once the older node is fully drained, it will be upgraded to the newer version, same will repeat for all the nodes.
      9. Force upgrade:
           - Tells k8s upgrade even if there are blocking conditions.
           - If normal upgrade fails repeatedly use force upgrade
           - It will ignore the Pod disruption budget.

<br><br>

9. How do you troubleshoot a PENDING pod?
   - describe the pod to see the events, which will tell us why the pod can't be scheduled.
      - ```kubectl describe pod my-app```
      - It could be due to insufficient CPU or MEM resources
      - Node unavailability, node pool has reached max count 
      - Node had taint for which pod did not tolerate
      - Affinity rules, node, pod or anti pod affinity rules might cause the issue
      - Pod spread constraints topology might be blocking it
   - How to fix it?
      - If the node is not available, increase the node pool max count
      - Resolve the taint and toleration
      - Check the affinity rules and apply it accordingly 

<br><br>

10. Pod is RUNNING but application is not accessible?
    - Check the application is listening on the right port
    - ```kubectl get pod my-app -o yaml```
    - Check the Service is correctly discovering the pod by comparing the Selector labels in Service and labels in pods and also targetPort is matching.
    - ```kubectl get endpoints myapp```
    - Test if the pod is reachable from another pod, may be a busybox pod
    - ```kubectl exec -it busybox -- curl http://<pod-service-ip>:port```
    - Check the network policy blocking the traffic, may be the pod allows traffic only from certain application
    - Now check the Kong ingress is routing the traffic correctly:
      - Check host, path, backend service, port
      - Check kong pods and logs
      -  Check the kong service kong-kong-proxy is has LoadBalancer IP, load balancer is Azure Load Balancer created in the backend.
    - check Kong Ingress load balancer is reachable
    - check the application gateway backend is properly set to kong load balancer private DNS zone
    - troubleshoot the connection from application gateway to kong load balancer, check backend health
    - Check the SSL certificate issue with Application gateway or Load balancer.
    - Check WAF rules
    - Check Firewall DNAT rule is correctly pointing to App gateway

<br><br>

11. Troubleshoot why POD is keep RESTARTING?
    - This will be due to the application failure
    - Pod will keep restarting and eventually go to CrashLoopBackOff
    - Describe pod to see if any events
    - Then check the pod logs, might be previous logs where it failed 
    - ```kubectl logs myapp-xxx --previous```
    - I would investigate:
       - Environment variables missing, may be connectivity to App config where it has config vars
       - Application startup failure
       - ConfigMap or Secrets missing
       - Database connectivity
       - Liveness probe failure
       - OOMKilled error 
<br><br>

12. What is Startup, Liveness and Readiness Probe?
    - Startup probe:
      - Startup probes verify whether the application within a container is started.
      - If a startup probe is configured, Kubernetes does not execute liveness or readiness probes until the startup probe succeeds.
      - This type of probe is only executed at startup, unlike liveness and readiness probes, which are run periodically.
      - a failed startup probe eventually causes the container to be restarted.
      - Startup probes are useful for Pods that have containers that take a long time to come into service. 
    - Liveness probe:
      - Checks the pod if it is alive and running
      - If the probe fails the Kubelet will restart the pod.
      - For example, liveness probes could catch a deadlock, where an application is running, but unable to make progress.
      - Restarting a container in such a state can help to make the application more available despite bugs.
   - Readiness Probe:
      - Readiness probes determine when a container is ready to accept traffic.
      - This is useful when waiting for an application to perform time-consuming initial tasks, such as establishing network connections, loading files, and warming caches
      - If readiness fails, Kubernetes removes the pod from the Service endpoints
    
   - Example:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: probe-example
spec:
  containers:
  - name: app
    image: registry.k8s.io/e2e-test-images/agnhost:2.40
    ports:
    - containerPort: 8080
    startupProbe:
      httpGet:
        path: /healthz
        port: 8080
      failureThreshold: 30
      periodSeconds: 10
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
      initialDelaySeconds: 10
      periodSeconds: 5
      timeoutSeconds: 3
      failureThreshold: 3
    readinessProbe:
      httpGet:
        path: /ready
        port: 8080
      periodSeconds:
```

<br><br>

13. How do you troubleshoot if node is Not Ready in AKS?
    - AKS continuously monitors the health state of worker nodes, and automatically repairs the nodes if they become unhealthy.
    1. Check node status
       ```yaml kubectl get nodes```
    2. Describe the node to see the events
       ```yaml kubectl describe node node-name```
    3. The Conditions and Events section is usually the first major clue.
```yaml
Conditions:
  Ready              False
  MemoryPressure     False
  DiskPressure       False
  PIDPressure        False

Events:
  ...
```
    
  4. MemoryPressure=True: The node is running low on memory. Check ```kubectl top node <node-name>```
  5. DiskPressure=True: The node is running low on disk space. Check `df -h`
  6. PIDPressure=True: The node has too many processes. Check `ps -e | wc -l`
  7. Check Kubelet status
  8. Check the Node resource utilization
  9. NetworkUnavailable, if Kubelets not able to connect to the API server

<br><br>

14. How do you troubleshoot Pod can't pull image from ACR?
    - I will describe pod to see the events
    - I will verify:
      - Correct image name, tag
      - access to ACR, AKS MI has acrPull permission
      - network connectivity to ACR from AKS, `az aks check-acr`
