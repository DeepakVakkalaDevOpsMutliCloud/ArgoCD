## Argo CD Tutorial: Deploying Helm Charts
### Prerequisites:
* A Kubernetes cluster
* Git client installed
* Argo CD installed and running in your cluster

### Step 1: Set Up the Git Repository with Helm Charts
1. Create a New Git Repository:
   * Create a new Git repository on your preferred Git hosting  service (e.g., GitHub, GitLab).
2. Create the Helm Chart Directory Structure:


```yaml
           
my-repo/
  helm/
    guestbook/
      Chart.yaml
      values.yaml
      templates/
        deployment.yaml
        service.yaml
  ```

3. Create the Helm Chart Files:
     *    Chart.yaml:
 ```yaml
apiVersion: v2
name: guestbook
description: A sample Helm chart for a Kubernetes application
version: 1.0.0
appVersion: 1.0.0
```

* values.yaml:
```yaml
replicas: 3
image:
  repository: redis
  tag: alpine
service:
  type: ClusterIP
  port: 6379
```

* templates/deployment.yaml: 
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Chart.Name }}
  labels:
    app: {{ .Chart.Name }}
spec:
  replicas: {{ .Values.replicas }}
  selector:
    matchLabels:
      app: {{ .Chart.Name }}
  template:
    metadata:
      labels:
        app: {{ .Chart.Name }}
    spec:
      containers:
      - name: {{ .Chart.Name }}
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        ports:
        - containerPort: {{ .Values.service.port }}
```
* templates/service.yaml:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ .Chart.Name }}
spec:
  type: {{ .Values.service.type }}
  ports:
  - port: 80
    targetPort: {{ .Values.service.port }}
  selector:
    app: {{ .Chart.Name }}
```
4. Commit and Push the Changes:
```bash
git add .
git commit -m "Add Helm chart for guestbook application"
git push origin main
```

### Step 2: Deploy the Helm Chart Using Argo CD
1. Add the Git Repository to Argo CD:
```bash
argocd repo add <GIT_REPO_URL>
```
             
2. Create an Argo CD Application:
```bash
argocd app create guestbook-helm \
  --repo <GIT_REPO_URL> \
  --path helm/guestbook \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default \
  --helm-set replicas=3 \
  --helm-set image.repository=redis \
  --helm-set image.tag=alpine
```
3. Sync the Application:
```bash
argocd app sync guestbook-helm
```
### Step 3: Use Helm Repository (Optional)
1. Add the Helm Repository:
```bash
argocd repo add https://charts.bitnami.com/bitnami --type helm
```
2. Create an Application Using a Helm Chart:
```bash
argocd app create guestbook-helm \
  --repo https://charts.bitnami.com/bitnami \
  --helm-chart redis \
  --revision 17.4.0 \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default \
  --helm-set replicaCount=3
```

### Step 4: Monitor and Manage the Application

* Check Application Status:
```bash
argocd app get guestbook-helm 
```

* View Application Details and Logs:
* Access the Argo CD web UI (http://localhost:8080 by default).

####  Update Helm Values:
```bash
argocd app set guestbook-helm --helm-set replicas=5
argocd app sync guestbook-helm
```

* Rollback to a Previous Version:

```bash
argocd app rollback guestbook-helm <REVISION_NUMBER>
```

### Additional Tips:
* Enable Auto-Sync:
```bash
argocd app set guestbook-helm --sync-policy automated
```

* Clean Up:
```bash
argocd app delete guestbook-helm
```
#### Conclusion 
  
   By following these steps, you can effectively deploy and manage your Helm-based applications using Argo CD.