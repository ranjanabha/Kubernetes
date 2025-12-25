---

The demo is implemented using Azure Kubernetes Service (AKS).

---

Tech Stack
Azure AKS
Helm
Unix / CLI

Architecture Overview

We will use the popular vote-app from Docker samples and extend it to demonstrate a Blue-Green deployment strategy using separate AKS node pools.
Node Pool Strategy

The AKS cluster contains two node pools:
Blue Node Pool

Hosts system pods
Hosts Blue application pods

2. Green Node Pool
Hosts Green application pods only

We leverage:
Taints to repel unwanted pods
Tolerations to explicitly allow scheduling
Node Affinity to guarantee pod placement

Each component of the vote-app has two deployments:
One for Blue
One for Green

Initially, the vote service points to the Blue pods.
 Later, we switch the service selector to Green pods, achieving zero downtime deployment.
📌 End Result
Initially: voting between Cats vs Dogs
After switch: voting between Democrat vs Republican

Step 1: Create AKS Cluster
Search for Kubernetes Service in Azure Portal
Click Create

Step 2: Configure Basic Settings
Choose a resource group
Provide a cluster name
Keep remaining settings default

Step 3: Configure Node Pools
By default, AKS provides one agent pool.
 Since this demo uses Azure Free Tier (4 vCPU limit), we customize node pools.
Blue Node Pool
Rename default pool to blue
Set min/max node count to 1
Enable Public IP per node

Add label:
environment=blue
Add taints
environment=blue:NoSchedule
This ensures:
Only pods with matching tolerations are scheduled
System pods can still run in this pool

Green Node Pool
Add new node pool named green
Node count: 1
Add label:

environment=green
. Add taint:
environment=green:NoSchedule

Click Review + Create and wait for the cluster to be provisioned.
Step 4: Blue-Green Deployment Configuration
Helm Chart Structure
Created a chart/ directory
Copied existing vote-app manifests
Created separate deployments for:

 1. vote
2. result
3. worker
4. redis
5. postgres
 
for both Blue and Green
Blue deployment example
apiVersion: apps/v1
kind: Deployment
metadata:
  name: vote-blue
  labels:
    app: vote-blue
    environment: blue
spec:
  replicas: 1
  selector:
    matchLabels:
      app: vote-blue
  template:
    metadata:
      labels:
        app: vote-blue
    spec:
      containers:
      - name: vote-blue
        image: azureciregistry.azurecr.io/votingapp/vote:57
        ports:
        - containerPort: 80
        startupProbe:
          httpGet:
            path: /
            port: 80
          failureThreshold: 6
          periodSeconds: 5
        readinessProbe:
          httpGet:
            path: /
            port: 80
          periodSeconds: 10
        livenessProbe:
          httpGet:
            path: /
            port: 80
          periodSeconds: 30
      tolerations:
      - key: "environment"
        operator: "Equal"
        value: "blue"
        effect: "NoSchedule"
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: environment
                operator: In
                values:
                - blue
      resources:
        requests:
          cpu: "100m"
          memory: "128Mi"
        limits:
          cpu: "200m"
          memory: "256Mi"
Green deployment example
apiVersion: apps/v1
kind: Deployment
metadata:
  name: vote-green
  labels:
    app: vote-green
    environment: green
spec:
  replicas: 1
  selector:
    matchLabels:
      app: vote-green
      environment: green
  template:
    metadata:
      labels:
        app: vote-green
        environment: green
    spec:
      containers:
      - name: vote-green
        image: azureciregistry.azurecr.io/votingapp/vote:57
        ports:
        - containerPort: 80
        envFrom:
        - configMapRef:
            name: vote-config
        startupProbe:
          httpGet:
            path: /
            port: 80
          failureThreshold: 6
          periodSeconds: 5
        readinessProbe:
          httpGet:
            path: /
            port: 80
          periodSeconds: 10
        livenessProbe:
          httpGet:
            path: /
            port: 80
          periodSeconds: 30
      tolerations:
      - key: "environment"
        operator: "Equal"
        value: "green"
        effect: "NoSchedule"
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: environment
                operator: In
                values:
                - green
      resources:
        requests:
          cpu: "100m"
          memory: "128Mi"
        limits:
          cpu: "200m"
          memory: "256Mi"

---

ConfigMap for UI Customization
apiVersion: v1
kind: ConfigMap
metadata:
  name: vote-config
data:
  OPTION_A: Democrat
  OPTION_B: Republican

---

Helm Chart Metadata (Chart.yaml)
apiVersion: v2
name: vote-app
description: A Helm chart for vote-app
type: application
version: 1.0.0
appVersion: "1.0.0"
maintainers:
- name: Ranjanabha Bhattacharya
  email: ranjanabha@gmail.com
kubeVersion: ">=1.25.0-0"

---

Step 5: Install Helm
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh

---

Step 6: Configure kubeconfig
From Azure Portal:
Open AKS cluster
Click Connect
Run the provided commands locally

Step 7: Deploy the Application
git clone <GITHUB_URL>
cd vote-app
helm install blue ./chart
Verify:
kubectl get pods -o wide
Step 8: Access the Application
Allow NodePort access in:
VM Scale Set → Node Pool → Networking → Inbound Port Rules
Get node IP:
kubectl get nodes -o wide

Access:
http://<NODE_IP>:<NODE_PORT>
Step 9: Switch Traffic to Green (Zero Downtime)
kubectl edit svc vote
Change selector from:
app: vote-blue
To
app: vote-green
Refresh the page.
✨ Voila! Zero-downtime Blue → Green switch completed.
Conclusion
In this article, we:
Created an AKS cluster with isolated node pools
Deployed Blue and Green versions using Helm
Enforced strict isolation using taints, tolerations, and affinity
Switched traffic instantly by updating the service selector

The application transitioned from Cats vs Dogs to Democrat vs Republican without any downtime.
