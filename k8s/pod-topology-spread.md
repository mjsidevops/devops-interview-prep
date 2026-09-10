Pod Topology Spread Constraints:
 - To equally distribute pods across Kubernetes nodes.
 - Suppose you have 3 nodes and a Deployment with 6 replicas. You want:
```yaml
Node 1 → 2 pods
Node 2 → 2 pods
Node 3 → 2 pods
```

- You can use:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 6
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      topologySpreadConstraints:
        - maxSkew: 1
          topologyKey: kubernetes.io/hostname
          whenUnsatisfiable: DoNotSchedule
          labelSelector:
            matchLabels:
              app: my-app
      containers:
        - name: my-app
          image: nginx
```

- maxSkew: 1
  - This means the difference between the node having the most matching pods and the node having the fewest should not exceed 1
- topologyKey: kubernetes.io/hostname
  - Spread the pods across different nodes. We can say "availability zone" as well. Spread equally across the availabilty zones.
- whenUnsatisfiable: DoNotSchedule
  - If Kubernetes cannot maintain the required distribution, it will not schedule the new pod
