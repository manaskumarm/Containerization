# Kubernetes
Kubernetes, also known as K8s, is an open source system for automating deployment, scaling, and management of containerized applications.

![image](https://github.com/manaskumarm/Containerization/assets/14363425/5dd202c1-810e-44cc-8ae6-9092e4aad6be)

# Components 
1️⃣ Control Plane Components: These components manage the cluster and make global decisions.

**1.1 API Server (kube-apiserver)**

Acts as the gateway for all Kubernetes operations.

Exposes the Kubernetes API (REST interface).

All requests from kubectl, helm, or external clients go through the API Server.

Ensures authentication, authorization, and admission control.

🔹 Check API Server logs: kubectl logs -n kube-system kube-apiserver-<node-name>

**1.2 Controller Manager (kube-controller-manager)**

Runs various background controllers to maintain the desired state.

Examples of controllers:

Node Controller → Detects and replaces failed nodes.

Replication Controller → Ensures the correct number of pod replicas are running.

Service Account Controller → Manages service accounts and tokens.

🔹 Check Controller Manager logs: kubectl logs -n kube-system kube-controller-manager-<node-name>

**1.3 Scheduler (kube-scheduler)**

Assigns new pods to appropriate nodes based on:

Resource availability (CPU, memory)

Node taints & tolerations

Node affinity & anti-affinity

Uses various scheduling algorithms.

🔹 Check Scheduler logs:

kubectl logs -n kube-system kube-scheduler-<node-name>

**1.4 etcd (Key-Value Store)**

Stores cluster configuration & state (e.g., deployments, services, secrets).

Highly available and supports leader election.

Every change to Kubernetes objects is stored here.

🔹 Check etcd health:

ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 endpoint status --write-out=table

2️⃣ Worker Node Components

Worker nodes are where application containers actually run.

**2.1 Kubelet**

Agent running on each node.

Manages the lifecycle of pods (ensures containers are running).

Talks to the API Server and pulls container specs.

🔹 Check Kubelet logs: journalctl -u kubelet -f

2.2 Kube Proxy

Manages networking for services.

Maintains iptables or IPVS rules to route traffic inside the cluster.

Ensures communication between pods & services.

🔹 Check Kube Proxy logs: kubectl logs -n kube-system kube-proxy-<node-name>

**2.3 Container Runtime**

Runs containers inside pods.

Supported runtimes:

containerd (default in Kubernetes)
Docker
CRI-O
🔹 Check Container Runtime logs (containerd example):

sudo journalctl -u containerd -f

**3️⃣ Add-Ons (Optional but Important Components)**

**3.1 CoreDNS**

Provides internal DNS resolution.

Allows pods to communicate using service names (my-service.default.svc.cluster.local).
🔹 Check CoreDNS status:

kubectl get pods -n kube-system | grep coredns

3.2 Ingress Controller

Manages external HTTP/HTTPS traffic to services.

Supports path-based and host-based routing.

🔹 Example:

https://api.myapp.com/orders → Routed to orders-service

https://api.myapp.com/users → Routed to users-service

**3.3 Network Plugins (CNI)**

Controls pod networking.

Common CNIs:

Calico (network policies, BGP routing)

Flannel (simple overlay network)

Weave (automatic encryption)

🔹 Check CNI plugin logs:

kubectl get pods -n kube-system | grep calico

**3.4 Metrics Server**

Enables kubectl top commands.

Provides CPU and memory usage metrics.

🔹 Install Metrics Server:

kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml


# Commands
![image](https://github.com/user-attachments/assets/8e580d9e-7840-4c2a-96ba-90c1c5ed5a42)



 # Debugging
**A. Pod & Container Details**

Pod name, namespace

Pod status (Running, Pending, CrashLoopBackOff, OOMKilled, etc.)

Container logs and events

Resource limits (CPU, Memory)

**B. Service & Network Details**

Service type (ClusterIP, NodePort, LoadBalancer, ExternalName)

DNS resolution (service-name.namespace.svc.cluster.local)

Network policies

Kubernetes API server & connectivity

**C. Node Details**
Node status (Ready, NotReady, Unschedulable)

Disk, CPU, memory usage

Network connectivity

Kernel logs for failures

****Commands****
**A. Check Pod & Container Status**

kubectl get pods -A -o wide   # Get all pods with node info

kubectl describe pod <pod-name> -n <namespace>   # Detailed events and pod status

kubectl logs <pod-name> -c <container-name> -n <namespace>   # Check container logs

kubectl logs <pod-name> --previous   # Get logs of the crashed container

kubectl exec -it <pod-name> -n <namespace> -- sh   # Get shell inside the container

👉 If a pod is stuck in CrashLoopBackOff, check logs and events.

**B. Check Node Status**

kubectl get nodes -o wide    # Get all nodes

kubectl describe node <node-name>   # Check node conditions, taints

kubectl top node   # Check CPU/memory usage

dmesg | tail -50   # Check kernel logs for node-level issues

👉 If a node is NotReady, check network or resource issues.

**C. Check Service & Network Issues**

kubectl get svc -A   # List all services

kubectl describe svc <service-name> -n <namespace>   # Check service endpoints

kubectl get endpoints <service-name> -n <namespace>   # Ensure the service has endpoints

kubectl exec -it <pod-name> -- curl http://<service-name>:<port>   # Test connectivity from a pod

👉 If a service has no endpoints, check if the pod labels match the service selector.

**D. Check Network Policies & DNS**

kubectl get networkpolicy -A   # List all network policies

kubectl describe networkpolicy <policy-name> -n <namespace>   # Check rules

kubectl exec -it <pod-name> -- nslookup <service-name>   # Check DNS resolution

kubectl get pods -n kube-system | grep coredns   # Check CoreDNS status

👉 If DNS is failing, restart CoreDNS and verify network policies.

**E. Debugging Kubernetes API & Scheduling Issues**

kubectl get events --sort-by=.metadata.creationTimestamp   # List recent events

kubectl get pods --field-selector=status.phase=Pending   # Find pods stuck in Pending

kubectl describe pod <pod-name>   # Check pod scheduling issues

kubectl get deployments -A   # Ensure deployments are healthy

👉 If a pod is Pending, check events, taints, and resource constraints.

**F. Debugging CNI & Network Plugin Issues**

kubectl get pods -n kube-system | grep calico  # If using Calico

kubectl get pods -n kube-system | grep weave   # If using Weave

kubectl get pods -n kube-system | grep flannel # If using Flannel

iptables -L -v -n    # Check if iptables is blocking traffic

👉 If network communication is blocked, check CNI logs and iptables.

****Metrics****
**A. Container Logs**

kubectl logs <pod-name> (Get logs of running containers)

kubectl logs <pod-name> --previous (Get logs of crashed containers)

kubectl exec -it <pod-name> -- cat /var/log/app.log (Check custom application logs)

**B. Node & System Logs**

Kernel Logs:

journalctl -u kubelet -n 50

dmesg | tail -50

Kubelet Logs:

cat /var/log/kubelet.log

**C. Cluster Events**

kubectl get events --sort-by=.metadata.creationTimestamp

👉 Helps in tracking failures, OOM kills, and scheduling issues.

**If you want quick insights, try:**

kubectl-debug: Debug pods interactively

kubectl debug pod/<pod-name> -it --image=busybox

stern: Real-time log monitoring of multiple pods(need to install manually)

stern <pod-name>

k9s: Interactive Kubernetes CLI tool

k9s
