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
    - helm upgrade myapp ./chart -f values-prod.yaml
    
  - Also you can use --set
    - helm upgrade myapp ./chart --set image.tag=1.6.0

  - Multiple values files, 
    - helm upgrade myapp ./chart -f values.yaml -f values-prod.yaml
    - You can specify the '--values'/'-f' flag multiple times. The priority will be given to the last (right-most) file specified. For example, if both myvalues.yaml and override.yaml contained a key called 'Test', the value set in override.yaml would take precedence.

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

- What is the difference between helm install and helm upgrade?
  - Helm install creates new release where helm upgrade updates an existing release
  - If you use both install and upgrade together
     - If the release doesn't exist → install it.
     - If it exists → upgrade it.
    
- Helm hooks:
   - Helm hooks let you tell Helm: "Run this Kubernetes resource before or after a particular Helm operation."
   - For example, you might want to:
       Run database migrations before deploying the application.
       Run cleanup tasks after uninstalling a release.
   - Some important hooks are: pre-install, post-install, pre-upgrade, post-upgrade, pre-rollback, post-rollback, post-delete

- Subcharts:
   - In Helm, a subchart is a Helm chart included inside another Helm chart.
   - Parent chart → contains one or more child/subcharts
   - Subcharts are defined as a dependency in the parent's Chart.yaml
   - Example:
```yaml
my-app/
├── Chart.yaml
├── values.yaml
├── templates/
│   └── deployment.yaml
└── charts/
    ├── redis/
    └── postgresql/
```

- helms commands:
   - helm create myapp
      - Creates a new Helm chart with a standard directory structure.<br>
   - helm show values ./myapp
      - Displays the chart's default values.yaml<br>
   - helm lint ./myapp
      - Checks the chart for the possible issues or errors<br>
   - helm template myapp ./myapp
      - Renders Helm templates into Kubernetes YAML without deploying them
   - helm install myapp ./myapp --dry-run --debug
      - Simulates an installation without actually installing the release.
   - helm list -A
      - Lists all releases in all namespaces
   - helm status myapp
      - checks the status of the release
   - helm history myapp
      - Shows the revision history of the release
      - For every new release there will be Revision like 1, 2, 3 ... with status(failed, deployed)
   - helm rollback myapp 3
      - Rolls the release back to revision 3
   - helm get values myapp
      - Shows values associated with the deployed release.
   - helm uninstall myapp
      - Removes a Helm release and its associated Kubernetes resources.
   - helm repo add bitnami https://charts.bitnami.com/bitnami
      - Adds a Helm chart repository.
   - helm upgrade myapp ./chart --wait
      - Waits for resources to become ready before considering the operation successful.
   - helm upgrade myapp ./chart --wait --timeout 10m
      - Controls how long Helm waits for operations.
   - helm upgrade myapp ./chart --atomic
      - If the upgrade fails, Helm rolls the release back.
  
  
   
