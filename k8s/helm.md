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

- _helpers.tpl
   - It is a template file used to define reusable helper functions/templates that can be used across your Helm chart.
   - It is commonly used to avoid repeating things like names, labels, selectors, and common Kubernetes metadata.
   - We typically use define to create helpers and include to consume them from templates like Deployment and Service, which improves reusability and maintainability of the Helm chart.
 
   Example:
    Suppose you have this in _helpers.tpl:
```yaml
{{- define "myapp.fullname" -}}
{{- printf "%s-%s" .Release.Name .Chart.Name | trunc 63 | trimSuffix "-" }}
{{- end }}
```

   - This defines a reusable template called "my app.fullname"
   - You can use it from deployment.yaml:
```yaml
metadata:
  name: {{ include "myapp.fullname" . }}
```
   - If your Helm release is 'helm install my-release myapp' then name: my-release-myapp
- Why do we need helpers.tpl
  - Imagine you need the same labels in your Deployment, Service and ConfigMap.
  - Instead of repeating:
```yaml
labels:
  app: myapp
  environment: production
  version: "1.0"
```
- you can define a helper:
```yaml
{{- define "myapp.labels" }}
app.kubernetes.io/name: {{ include "myapp.name" . }}
app.kubernetes.io/instance: {{ .Release.Name }}
app.kubernetes.io/managed-by: {{ .Release.Service }}
helm.sh/chart: {{ .Chart.Name }}-{{ .Chart.Version | replace "+" "_" }}
{{- end }}
```
- Then in each k8s manifest
```yaml
metadata:
  labels:
    {{- include "myapp.labels" . | nindent 4 }}
```
   
