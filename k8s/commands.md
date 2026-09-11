```kubectl version```
- shows k8s version


```kubectl cluster-info```
- Shows information about the Kubernetes control plane and services

```kubectl get nodes```
- lists all worker nodes with status 


```kubectl describe node node-name```
- Shows detailed information about a node

```kubectl get namespaces```
- Lists namespaces


```kubectl get pods -n dev```
- get pods from dev namespace

```kubetcl get pods -A```
- get pods from all namespaces


```kubectl get pods -o wide```
- additional info such as Pod IP, Node where pod running

```kubectl describe pod pod-name```
- when you need detailed info and events on the pod
- Can also describe service, ingress, nodes etc


```kubectl logs pod-name```
- to see the application logs on the pod
- can specify `-c container-name` to get logs from particular container


`kubectl exec -it pod-name -- /bin/bash`
- execute commands inside the pod

`kubectl apply -f deployment.yaml`
- to create or update the resources

`kubectl delete pod pod-name`
- to delete the pod
- can also delete deployments, service, ingress etc

`kubectl rollout status deployment my-app`
- check the status of the deployment

`kubectl rollout history deployment my-app`
- shows deployment history

`kubectl rollout undo deployment my-app`
- Rollback to the last deployment

`kubectl rollout undo deployment my-app --to-revision=2`
- Rollback to the specific version

`kubectl rollout restart deployment my-app`
- Restart the deployment
- Used when pods are stuck or need to restart all pods

`kubectl scale deployment my-app --replicas=3`
- to scale the Deployment replicas

`kubectl top node node- name`
- CPU and MEM usage of nodes

`kubectl top pod pod-name`
- CPU and MEM usage of Pods
