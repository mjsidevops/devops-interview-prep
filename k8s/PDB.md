Pod Disruption Budget:
 - A Pod Disruption Budget (PDB) is a Kubernetes resource that helps keep an application available when Kubernetes performs voluntary disruptions, such as draining a node for maintenance.
 - It makes sure that minimum number of pods that must be available during the voluntary disruption.
 - PDBs do not protect against involuntary failures such as a node suddenly crashing, hardware failure, or certain pod/container failures.
```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: my-app-pdb
spec:
  minAvailable: 4
  selector:
    matchLabels:
      app: my-app
```
