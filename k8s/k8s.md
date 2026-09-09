1. what is headless service in k8s?
   - A headless Service is a Kubernetes Service configured with clusterIP: None.
   - Unlike a normal Service, it doesn't provide a virtual ClusterIP or perform normal Service-level load balancing.
   - Instead, Kubernetes DNS returns the IP addresses of the individual Pods behind the Service. It is commonly used with StatefulSets and distributed applications such as Kafka, Cassandra, or databases where clients need to discover and communicate with individual Pods.
   - ```yaml
     Normal Service:

      DNS → ClusterIP → Pod

     Headless Service:

      DNS → Pod IP(s)
     ```

2. What is Persistant Volume (PV) and Persistant Volume Claim(PVC)?
    - Persistant Volume(PV):
       - A Persistent Volume (PV) is a piece of storage that exists independently of a Pod.
       - The main reason we use a PV is that Pod storage is normally temporary. If a Pod is deleted or recreated, data stored inside the Pod's container filesystem can be lost.
       - The PV is mostly used for Stateful applications.
       - <img width="1060" height="576" alt="image" src="https://github.com/user-attachments/assets/1b64b058-bad2-4850-aa6d-984789ab3dbe" />
       - A PV represents the actual storage resource available to Kubernetes.
       - <img width="1624" height="554" alt="image" src="https://github.com/user-attachments/assets/fd847121-31b3-40ad-8e5d-1f3a9f01dc88" />
       - Azure Disk and Azure Files are from AKS. And EBS from AWS.
       - Its static provision, you manually create the storage and PV
    - Persistant Volume Claim(PVC):
       - A PVC is a request for storage from PV
       - <img width="1654" height="602" alt="image" src="https://github.com/user-attachments/assets/a1815132-14a2-4f84-b323-41a5164ee950" />
    - Pod will mount PVC as volume.
    - <img width="1682" height="778" alt="image" src="https://github.com/user-attachments/assets/36905f5f-da6a-479d-a746-25c4fc7b11a2" />

<br><br>

3. What is Storage Class?
    - StorageClass tells Kubernetes what type of storage to create and how to create it automatically.
    - Without Storage Class we need to create storage (Azure Disk) manually in the cloud then PV and PVC.
    - But with Storage Class we don't need to create a storage manually, it will auto create it based on the request by PVC.
    - Its uses CSI driver
    - Its dynamic provisioning, you create Storage Class and Kubernetes/CSI driver automatically provisions Storage(Azure Disk) and PV in the backend.
```yam
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: azure-disk-sc

provisioner: disk.csi.azure.com

parameters:
  skuName: Premium_LRS

reclaimPolicy: Delete

volumeBindingMode: WaitForFirstConsumer
```




                     
