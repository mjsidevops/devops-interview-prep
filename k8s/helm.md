Helm:
   - Helm is a package manager for Kubernetes.
   - It allows us to package Kubernetes manifests into reusable charts and manage them as versioned releases.
   - Instead of maintaining separate static YAML files for every environment, Helm allows us to parameterize Kubernetes manifests using templates and values.
   - Example structure:
   ```yaml
   myapp/
   ├── Chart.yaml
   ├── values.yaml
   ├── charts/
   ├── templates/
   │   ├── deployment.yaml
   │   ├── service.yaml
   │   ├── ingress.yaml
   │   ├── configmap.yaml
   │   └── _helpers.tpl
   ├── .helmignore
   └── README.md
   ```

```bash
   helm upgrade --install myapp ./myapp \
  -f values-prod.yaml \
  -n production
```

 - Chart.yaml contains the metadata of the chart.
```yaml
apiVersion: v2
name: myapp
description: My application Helm chart
type: application
version: 1.2.0
appVersion: "5.4.0"
```

- values.yaml
  - values.yaml contains the default configuration values for a Helm chart. 
```yaml
replicaCount: 3

image:
  repository: myacr.azurecr.io/myapp
  tag: "1.5.0"

service:
  type: ClusterIP
  port: 8080
```

- Templates consume these values using .Values.
```yaml
spec:
  replicas: {{ .Values.replicaCount }}

containers:
  - name: myapp
    image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
```

- How do you override values.yaml?
  - by using -f / --values
    --> helm upgrade myapp ./chart -f values-prod.yaml
    
  - Also you can use --set
    --> helm upgrade myapp ./chart --set image.tag=1.6.0

  - Multiple values files 
    --> helm upgrade myapp ./chart -f values.yaml -f values-prod.yaml
   
