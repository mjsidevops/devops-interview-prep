1. What is Persistant Volume (PV) and Persistant Volume Claim(PVC)?
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

2. What is Storage Class?
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

<br><br>

5. What is Stateful sets?
    - Stateful sets are for Stateful application. And Deployments are for stateless appn.
    - For application like Database it needs a certain order how the pods are come up or setup.
    - If there are 3 pods, first pod will be master and rest 2 are slaves. Pods will be created in sequential order first master, then slave1 and last slave2. Slave1 copy data from master, slave2 copies from slave1 and both slave1 and slave2 are synched with master for continuous replication of data.
    - The write operation will only be on master pod, slave1 and slave2 are for read operations for high availability.
    - With Deployment we can't achieve this setup because here all the pods are comes up at the same time.
    - Stateful set assigns a unique index for each pod, a number starting from 0,1,3...
    - So each pod gets unique name which combines stateful set name and index. Ex: mysql-0, mysql-1, mysql-2. But deployment pods has random names.
    - If master fails and new pod created, it will still come up with same name.
    - When we delete stateful set, the pods will be deleted in the riverse order, slave2, slave1 and finally master.
    - We can also set the stateful set not create the pods in order by setting a field podManagementPolicy: parallel, so pods will be created in parallel but will have unique name.

<br> <br>

6. what is headless service in k8s?
   - A headless Service is a Kubernetes Service configured with clusterIP: None.
   - Unlike a normal Service, it doesn't provide a virtual ClusterIP or perform normal Service-level load balancing.
   - It creates DNS entry for each pod using the pod name and sub-domain. <pod name>. <service name>.<namespace>.<svc>.<cluster>.<local>
   - So when web application wants to do a write operation it uses the DNS name of MASTER pod (mysql-0.mysql-h.default.svc.cluster.local).
   - Headless services are mostly used in Stateful set. In stateful set definition YAML we should specify the name of the headless service to use.
   - It is commonly used with StatefulSets and distributed applications such as Kafka, Cassandra, or databases where clients need to discover and communicate with individual Pods.
   - ```yaml
     Normal Service:

      DNS → ClusterIP → Pod

     Headless Service:

      DNS → Pod IP(s)
     ```

7. How persistent volume used in the Stateful set?
    - Add "volumeClaimTemplates" in the stateful set definition, this is same as creating separate PVC manually.
    - So what it does is that, it will create a separate PVC volume for each pods. Even after pods are re created it will be attached to the same PVC as before.
    - <img width="3024" height="1892" alt="image" src="https://github.com/user-attachments/assets/802c315c-f72e-49ff-8874-320581e981ae" />



                     
