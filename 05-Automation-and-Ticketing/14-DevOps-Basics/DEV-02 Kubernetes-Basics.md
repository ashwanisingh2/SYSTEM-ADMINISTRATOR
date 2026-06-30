---
tags: [desktop-support, devops, orchestration, kubernetes, L2]
aliases: [kubernetes-basics, k8s-basics]
created: 2026-06-25
status: #complete
difficulty: #advanced
cert-relevant: #none
---

# DEV-02: Kubernetes Basics

> [!note] Overview
> This note covers the fundamentals of Kubernetes (K8s) container orchestration. It details cluster architecture (Control Plane and Worker Nodes), core API objects (Pods, Deployments, Services), YAML manifest structures, cluster management, scaling, and self-healing diagnostics.

---
## Concept Overview
- **What it is** — Kubernetes (K8s) is an open-source container orchestration platform designed to automate the deployment, scaling, network routing, load balancing, and lifecycle management of containerized workloads across a distributed cluster of server nodes.
- **Why it matters** — Deploying a few containers on a single host using Docker is simple, but managing container workloads spread across dozens of servers requires orchestration. Kubernetes coordinates these physical or virtual servers as a single resource, automatically correcting crashes (self-healing) and distributing traffic (load balancing).
- **Real job encounter** — Checking application container status, scaling application replicas during high-traffic windows, deploying rolling application updates, and retrieving logs using `kubectl`.
- **L1 vs L2 vs L3 responsibility split:**
  - **L1 Resolution**: Query pod operational statuses (`kubectl get pods`), view stdout container logs (`kubectl logs`), and restart failing pods by deleting them to force replication checks.
  - **Escalation Trigger**: Escalate if pods are stuck in error states (e.g. `CrashLoopBackOff`, `ImagePullBackOff`), or if physical cluster nodes report `NotReady`.
  - **L2 Resolution**: Author YAML manifests defining Deployments and Services, scale deployments manually using CLI commands, configure ingress routing, and update container image tags.
  - **L3 Resolution**: Install and configure production-grade Kubernetes control planes and nodes, manage persistent volume storage classes, define namespace network isolation policies, and configure RBAC security profiles.

---
## Technical Deep Dive

### 1. Kubernetes Cluster Architecture
A Kubernetes cluster is divided into two primary logical layers: the Control Plane (management) and Worker Nodes (compute hosts).

#### The Control Plane (Master Node)
The administrative engine managing cluster state:
- **`kube-apiserver`**: The central entry point. All administrative commands (via `kubectl` or API calls) route through the API server.
- **`etcd`**: A highly available, distributed key-value database storing the entire cluster configuration state and metadata.
- **`kube-scheduler`**: Assigns newly created pods to specific worker nodes based on resource availability (CPU/RAM requirements) and node constraints.
- **`kube-controller-manager`**: Evaluates cluster state and runs loops to maintain compliance (e.g., Node Controller, Replication Controller).

#### Worker Nodes
The hosts executing the actual application workloads:
- **`kubelet`**: The local agent running on each worker node. It communicates with the control plane and ensures the physical containers are running inside pods as defined by configuration manifests.
- **`kube-proxy`**: The network agent on each node. Manages host packet forwarding and implements virtual IP routing for services.
- **Container Runtime** (e.g., `containerd`, Docker): The engine executing the container processes.

```
                  +-----------------------------------+
                  |      Control Plane (Master)       |
                  |  [API Server] <---> [etcd DB]     |
                  |  [Scheduler]   [Controller Mgr]   |
                  +-----------------------------------+
                                    |
                    Monitors and Commands Workers
                                    |
         +--------------------------+--------------------------+
         |                                                     |
         v                                                     v
+-------------------------------+                     +-------------------------------+
|      Worker Node (Node 1)     |                     |      Worker Node (Node 2)     |
|  [Kubelet]      [Kube-Proxy]  |                     |  [Kubelet]      [Kube-Proxy]  |
|  [Containerd]                 |                     |  [Containerd]                 |
|  +-------------------------+  |                     |  +-------------------------+  |
|  |     Pod A (App Container) |  |                     |  |     Pod B (App Container) |  |
|  +-------------------------+  |                     |  +-------------------------+  |
+-------------------------------+                     +-------------------------------+
```

### 2. Core Kubernetes API Objects
Kubernetes resources are defined as declarative objects:
- **Pod**: The smallest deployable unit. A pod hosts one or more containers (e.g. an application container and a helper sidecar) that share network interfaces and storage volumes.
- **Deployment**: Manages the deployment lifecycle of pods. You declare the desired state (e.g., "run 3 replicas of Nginx"), and the deployment controller handles scaling, rolling updates, and self-healing.
- **Service**: Provides a stable network IP address and DNS name to route traffic to a dynamic group of pods (which are ephemeral and change IPs when recreated).
  - *ClusterIP*: Exposes service on internal cluster IP (accessible within cluster only).
  - *NodePort*: Maps a port on all worker nodes to the service, exposing it outside the cluster.
  - *LoadBalancer*: Provisions a public cloud load balancer pointing to the service.

---
## Step-by-Step Lab

> [!warning] Pre-requisites
> - Access to a running single-node Kubernetes cluster (such as Minikube, MicroK8s, or Azure AKS).
> - The Kubernetes command-line tool `kubectl` installed and configured on your workstation.
> - Outbound internet connectivity to pull public image libraries.

### Step 1: Verify Cluster Health
1. Open a terminal and check if cluster nodes are online and ready:
```bash
kubectl get nodes
```
**Expected Output:**
```text
NAME             STATUS   ROLES           AGE   VERSION
minikube-node1   Ready    control-plane   10d   v1.26.1
```

### Step 2: Author and Deploy a Web Server Deployment
We will deploy three replicas of an Nginx web server.

1. Create a file named `nginx-deployment.yaml`:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-web
  labels:
    app: web-server
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-server
  template:
    metadata:
      labels:
        app: web-server
    spec:
      containers:
      - name: nginx
        image: nginx:alpine
        ports:
        - containerPort: 80
```
2. Apply the deployment config manifest:
```bash
kubectl apply -f nginx-deployment.yaml
```
**Expected Output:** `deployment.apps/nginx-web created`

3. List the active deployments and pods:
```bash
kubectl get deployments
kubectl get pods
```
**Expected Output:** Lists three running nginx pods (e.g., `nginx-web-68cfb-xxxxx`).

### Step 3: Expose the Web Deployment using a Service
We will expose the internal Nginx pods to external client traffic.

1. Create a file named `nginx-service.yaml`:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: NodePort
  selector:
    app: web-server
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
```
2. Apply the service config manifest:
```bash
kubectl apply -f nginx-service.yaml
```
3. Test connection: Run `curl http://<Node_IP>:30080`. The default Nginx welcome page loads.

### Step 4: Test Self-Healing Behavior
1. Check running pods:
```bash
kubectl get pods
```
2. Note a pod name and delete it to simulate a container crash:
```bash
kubectl delete pod nginx-web-68cfb-xxxxx
```
**Expected Output:** `pod "nginx-web-68cfb-xxxxx" deleted`
3. Immediately query the pod list again:
```bash
kubectl get pods
```
**Expected Output:** A new pod is instantly spawned and running to maintain the desired state replica count of 3.

---
## Cheat Sheet / Quick Reference

| Command / Option | Scope | Purpose / Example |
|---|---|---|
| `kubectl get <object>` | Query | Lists resources (`pods`, `svc`, `deploy`, `nodes`) |
| `kubectl describe <type> <name>` | Query | Shows detailed configuration and lifecycle event histories |
| `kubectl logs <pod>` | Diagnostics | Displays container standard output logs |
| `kubectl logs <pod> -c <container>` | Diagnostics | Displays logs from a specific container inside a pod |
| `kubectl exec -it <pod> -- sh` | Diagnostics | Opens an interactive terminal shell inside a pod |
| `kubectl apply -f <file.yaml>` | Configuration | Deploys or updates objects defined in a YAML manifest |
| `kubectl delete -f <file.yaml>`| Configuration | Removes resources defined in a YAML manifest |
| `kubectl scale deploy <name> --replicas=N`| Scaling | Manually scales replica counts |
| `kubectl get pods --watch` | Query | Watches pod state changes in real time |

---
## Troubleshooting Table

| Problem | Cause | Fix |
|---|---|---|
| Pod status shows: `ImagePullBackOff` or `ErrImagePull`. | The image tag does not exist, or the private registry requires authentication credentials. | Verify image name spelling. If using a private registry, create an `imagePullSecrets` configuration and reference it in the Deployment spec. |
| Pod status shows: `CrashLoopBackOff`. | The application inside crashed during startup, or lacks required environment variables. | Check logs: `kubectl logs <pod-name>`. Fix configuration errors or missing database connection values. |
| Pod is stuck in `Pending` state. | No worker node has sufficient CPU/RAM resources to host the pod. | Add more nodes to the cluster, or reduce the resource requests in the pod specification. |
| `kubectl` commands fail: "Connection to server refused." | The local kubeconfig file is missing, or the cluster control plane is offline. | Verify the `KUBECONFIG` environment variable is pointing to `~/.kube/config`. Ensure local Kubernetes services (`minikube` / `kubelet`) are running. |
| Service routing fails (Accessing service IP returns timeouts). | Service selector labels do not match pod deployment labels. | Inspect the service description: `kubectl describe svc <name>`. Ensure the `selector` section matches the `labels` in the Deployment template exactly. |

---
## Interview Questions

> [!question] L1 Question
> **Q:** What is a Pod in Kubernetes, and can it host multiple containers?
> **A:** A Pod is the smallest deployable and manageable object in Kubernetes. It represents a single instance of a running process in the cluster. Yes, a pod can host multiple containers (called multi-container pods). Containers sharing a pod also share local storage volumes, network namespaces, IP addresses, and ports, communicating with each other using `localhost`.

> [!question] L2 Question
> **Q:** What is the difference between a ClusterIP and a NodePort service in Kubernetes?
> **A:** A **ClusterIP** service is the default service type. It exposes the application on a private IP address within the cluster, making it accessible only to other internal cluster resources. A **NodePort** service exposes the application on a specific port (ranging from 30000-32767) on all physical worker nodes' external IP addresses, allowing external network clients to access the service directly.

> [!question] L3/Scenario Question
> **Q:** You deploy a database application as a standard Deployment. During a cluster node failure, Kubernetes recreates the database pod on another node, but the application reports that all customer records are lost. Why did this happen, and how do you resolve it?
> **A:** 
> - **Situation:** Database container reallocation results in permanent data loss.
> - **Task:** Implement persistent database storage in Kubernetes.
> - **Action:** 
>   - **Identify Cause**: Standard Deployments treat pods as ephemeral. Any data written inside the container filesystem is deleted when the pod is destroyed or moved.
>   - **Resolution**:
>     1. Re-deploy the database using a **StatefulSet** instead of a Deployment, which guarantees stable identifiers and persistent disk mappings.
>     2. Configure a **PersistentVolumeClaim (PVC)** in the StatefulSet manifest.
>     3. Map the PVC to a **StorageClass** (such as Azure Disk or AWS EBS) that provisions local disk space outside the cluster lifecycle.
>     4. Mount the volume to the database data directory (e.g. `/var/lib/mysql`) inside the container spec.
> - **Result:** When the pod is moved to another node, the external persistent storage volume is detached from the old host and re-attached to the new host, preserving all database records.

---
## Seedha Simple Mein
*Seedha simple mein: Kubernetes (K8s) ek aisa tool hai jo containerised apps ko network me scale, load-balance aur manage karta hai. Jab clusters me hazaron containers chalte hain, toh K8s automatically host servers (nodes) par traffic distribute karta hai. K8s ka core object `Pod` hota hai aur commands run karne ke liye `kubectl` tool use hota hai.*

---
## Related Notes
- [[05-Automation-and-Ticketing/14-DevOps-Basics/DEV-01 Docker-Fundamentals|DEV-01 Docker-Fundamentals]] — Core container packaging engines.
- [[04-Cloud-and-Security/08-Azure/AZ104-08 Azure-Container-Instances|AZ104-08 Azure-Container-Instances]] — Serverless container hosting alternatives.
- [[05-Automation-and-Ticketing/13-Monitoring/MON-03 Prometheus and Grafana|MON-03 Prometheus and Grafana]] — Monitoring Kubernetes clusters.

---
*Tags: #desktop-support #devops #orchestration #kubernetes #L2*
