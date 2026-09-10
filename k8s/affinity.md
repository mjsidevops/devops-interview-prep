Affinity:
    
  Node affinity:
      - Node affinity has advanced expression to select the nodes unlike the Node selector.
      - options:
         - requiredDuringSchedulingIgnoredDuringExecution -> Pods will not be scheduled if no matching nodes available.
         - preferredDuringSchedulingIgnoredDuringExecution -> Pods can be scheduled if it did not find the matching node
         - requiredDuringSchedulingRequiredDuringExecution -> Pod can't scheduled or executi
         - We could use operator like "in, NotIn, Exists"
         - Example
         - ```yaml
           kubectl label node node-1 environment=production
           <br>
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: environment
                operator: In
                values:
                  - production

  containers:
    - name: my-app
      image: nginx
```
