# 🚀 Blue-Green Deployment in AKS Using Taints, Tolerations & Node Affinity

This repository demonstrates a **Blue-Green deployment strategy in Azure Kubernetes Service (AKS)** using **node pool isolation** instead of the traditional namespace-based approach.

The goal is to achieve **hard isolation** between Blue and Green environments by leveraging:

- Kubernetes **taints**
- **tolerations**
- **node affinity**
- **Helm** for deployment management

---

## 📌 What This Demo Shows

- Separate **AKS node pools** for Blue and Green environments
- Explicit scheduling of workloads using **taints & tolerations**
- Guaranteed pod placement using **node affinity**
- **Zero-downtime traffic switch** by updating Kubernetes Service selectors
- Real-world, production-aligned Blue-Green deployment pattern

---

## 🧱 Architecture Overview

### Node Pools

| Node Pool | Purpose |
|----------|--------|
| **Blue Pool** | System pods + Blue application pods |
| **Green Pool** | Green application pods only |

Each node pool is configured with:
- Labels (`environment=blue|green`)
- Taints (`environment=blue|green:NoSchedule`)

Pods explicitly opt in using matching **tolerations** and **node affinity rules**.

<img width="1536" height="1024" alt="ChatGPT Image Dec 25, 2025, 09_58_47 AM" src="https://github.com/user-attachments/assets/8370dd4e-9734-450b-9062-0e7e87a88c8f" />


---

## 🛠 Tech Stack

- Azure Kubernetes Service (AKS)
- Helm
- Docker Vote App
- kubectl
- Unix/Linux shell

---

## 🗂 Repository Structure

```bash
.
├── chart/
│   ├── templates/
│   │   ├── vote-blue.yaml
│   │   ├── vote-green.yaml
│   │   ├── worker-blue.yaml
│   │   ├── worker-green.yaml
│   │   ├── result-blue.yaml
│   │   ├── result-green.yaml
│   │   ├── redis.yaml
│   │   ├── postgres.yaml
│   │   └── service.yaml
│   ├── Chart.yaml
│   └── values.yaml
├── README.md
⚙️ AKS Cluster Prerequisites
AKS cluster with two node pools

Each node pool configured with:

Labels

Taints

kubectl configured locally

Helm v3 installed

🏗 Node Pool Configuration
Blue Node Pool
text
Copy code
Label: environment=blue
Taint: environment=blue:NoSchedule
Green Node Pool
text
Copy code
Label: environment=green
Taint: environment=green:NoSchedule
This ensures:

Only explicitly allowed pods are scheduled

No accidental cross-environment placement

📦 Deployment Strategy
Each application component has two deployments:

Blue deployment

Green deployment

The Kubernetes Service initially points to the Blue deployment.

Traffic is switched to Green by simply updating the service selector.

🚀 Deploying the Application
1️⃣ Install Helm
bash
Copy code
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
2️⃣ Configure kubeconfig
From Azure Portal:

Open AKS cluster

Click Connect

Run the provided commands locally

3️⃣ Deploy Using Helm
bash
Copy code
helm install blue ./chart
Verify deployment:

bash
Copy code
kubectl get pods -o wide
🌐 Accessing the Application
This demo uses NodePort for simplicity.

⚠️ For production, use Azure Load Balancer or an Ingress Controller.

Get node IP:

bash
Copy code
kubectl get nodes -o wide
Access the app:

text
Copy code
http://<NODE_IP>:<NODE_PORT>
🔁 Zero-Downtime Blue → Green Switch
Edit the vote service:

bash
Copy code
kubectl edit svc vote
Change selector from:

yaml
Copy code
app: vote-blue
To:

yaml
Copy code
app: vote-green
🎉 Traffic is instantly switched with zero downtime.

🧪 Demo Behavior
Phase	Vote Options
Blue	Cats vs Dogs
Green	Democrat vs Republican

This confirms that traffic has moved to the Green deployment.

🔄 Rollback Strategy
Rollback is immediate and safe:

bash
Copy code
kubectl edit svc vote
Change selector back to:

yaml
Copy code
app: vote-blue
No redeployment required.

📖 Blog Post
This repository accompanies the Medium article:

Blue-Green Deployment in AKS Using Taints, Tolerations & Node Affinity
(Link coming soon)

🚧 Next Steps
In the next iteration, we will:

Implement Canary deployments

Gradually shift traffic using Argo Rollouts

Compare Blue-Green vs Canary strategies

🤝 Contributing
Contributions, suggestions, and improvements are welcome.

📜 License
This project is provided for educational and demonstration purposes.
