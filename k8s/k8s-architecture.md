Architecture:
 - K8s is broadly divided into Control plane(Master) and Data plane(worker nodes)
 - Components in Control plane:
    - API Server:
       - Its a front end gate for K8s server
       - All communications to the k8s server will happen via API Server.
         
    - etcd:
       - key-value store
       - Contains all data used to maintain the cluster
    
    - Scheduler:
       - Responsible for distributing work or containers/pods across the nodes.
       - Assigns the pods to the nodes

    - Controller:
       - Responsible for keep up the desired number of containers or pods,
       - if a pod goes down, controller will make decision to bring up the new pod


 - Components in Data plane:
    - Kubelet:
       - Runs on each node on the cluster
       - Responsible for making sure that containers are running as expected.
       - Each Kubelet on the nodes will talk to API server
     
    - Container runtime:
       - Underlying software that is used to run container.
       
