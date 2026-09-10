Taint:
 - Taints are used to control which Pods are allowed to run on a node. Mean you do not want every pods to be scheduled on this node, only certain pods.
 - tainting a node
 - ```kubectl taint nodes node1 app=db:NoSchedule ```
 - here "NoSchedule" do not schedule any pods on this node until they tolerate the label app=db.
 - Other effects are "PreferNoschedule, NoExecute"
 - PreferNoSchedule - scheduler will try not to schedule any pods on it but if no other available nodes it might schedule it.
 - NoExecute - Scheduler will not schedule any pods, if any running pods will be evicted.

<br>

Tolerations:
  - Tolerations will be added to pods/deploy definition file.
  - Ex:
```yaml
tolerations:
- key: workload
  operator: Equal
  value: app
  effect: NoSchedule
```

