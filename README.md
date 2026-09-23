# 🚀 Amazon EKS Demo

A hands-on **Amazon Elastic Kubernetes Service (EKS)** project demonstrating how to containerize a web application and deploy it to a Kubernetes cluster running on AWS.

This project is designed to understand the complete flow from application source code to a running workload on Amazon EKS.

---

## 📌 Project Overview

This project demonstrates:

* Docker containerization
* Kubernetes Deployments
* Kubernetes Pods
* Kubernetes Services
* Amazon EKS
* AWS networking
* Container image deployment
* Kubernetes workload management
* Application exposure through a Kubernetes Service

The repository is organized into two main parts:

```text
EKS-Demo/
│
├── k8s/
│   └── Kubernetes configuration files
│
└── website/
    └── Application source code
```

---

# ☁️ What is Amazon EKS?

**Amazon Elastic Kubernetes Service (EKS)** is AWS's managed Kubernetes service.

Kubernetes is a container orchestration platform that automates tasks such as:

* Deploying containers
* Scaling applications
* Restarting failed containers
* Service discovery
* Load balancing
* Rolling updates
* Managing application workloads

Amazon EKS provides a managed Kubernetes control plane while allowing applications to run on AWS compute resources such as EC2 nodes or other supported EKS compute options.

---

# 🏗️ Project Architecture

The overall architecture of this project can be represented as:

```text
                         Internet
                            │
                            ▼
                    ┌───────────────┐
                    │ AWS / EKS     │
                    │   Cluster     │
                    └───────┬───────┘
                            │
                            ▼
                    ┌───────────────┐
                    │ Kubernetes    │
                    │   Service     │
                    └───────┬───────┘
                            │
                 ┌──────────┴──────────┐
                 │                     │
                 ▼                     ▼
          ┌──────────────┐      ┌──────────────┐
          │     Pod      │      │     Pod      │
          │              │      │              │
          │ Application  │      │ Application  │
          │  Container   │      │  Container   │
          └──────────────┘      └──────────────┘
```

The application is packaged into a container and Kubernetes manages the containerized workload.

---

# 🔄 How the Project Works

The complete workflow is:

```text
Developer
    │
    ▼
Application Source Code
    │
    ▼
Docker Image
    │
    ▼
Container Registry
    │
    ▼
Amazon EKS
    │
    ▼
Kubernetes Deployment
    │
    ▼
Pods
    │
    ▼
Kubernetes Service
    │
    ▼
Users
```

The important idea is that **EKS does not directly run your source code**.

Instead:

1. The application is packaged into a container.
2. The container image is stored in a registry.
3. Kubernetes is instructed to run that image.
4. EKS schedules the workload onto available compute.
5. Kubernetes creates Pods.
6. A Service provides stable networking to the Pods.

---

# 🐳 Containerization

The application inside the `website/` directory is packaged as a container.

The basic container workflow is:

```text
Website Source
      │
      ▼
   Dockerfile
      │
      ▼
 Docker Build
      │
      ▼
 Docker Image
```

For example:

```bash
docker build -t eks-demo .
```

Run it locally:

```bash
docker run -d -p 8080:80 eks-demo
```

Then access:

```text
http://localhost:8080
```

Testing locally before deploying to EKS helps verify that the container itself works correctly.

---

# 📦 Container Registry

Before Kubernetes can start the application, the container image needs to be accessible to the cluster.

A common AWS architecture is:

```text
Docker
   │
   ▼
Amazon ECR
   │
   ▼
Amazon EKS
```

**Amazon Elastic Container Registry (ECR)** stores Docker/OCI container images.

Example:

```text
ACCOUNT_ID.dkr.ecr.us-east-1.amazonaws.com/eks-demo:latest
```

EKS can then pull the image when creating Pods.

---

# ☸️ Kubernetes

Amazon EKS uses standard Kubernetes concepts.

The most important resources used in this project are:

```text
Deployment
    │
    ▼
ReplicaSet
    │
    ▼
Pods
    │
    ▼
Containers
```

And:

```text
Service
    │
    ▼
Pods
```

---

# 1️⃣ Kubernetes Deployment

A **Deployment** describes how Kubernetes should run the application.

For example:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: eks-demo
spec:
  replicas: 2
```

The important part is:

```yaml
replicas: 2
```

This tells Kubernetes to maintain two application Pods.

Architecture:

```text
Deployment
    │
    ├── Pod 1
    │     └── Container
    │
    └── Pod 2
          └── Container
```

---

# 2️⃣ Kubernetes Pod

A **Pod** is the smallest deployable unit in Kubernetes.

For this project, a Pod contains the application container.

```text
Pod
│
└── Application Container
```

If the Deployment specifies:

```yaml
replicas: 2
```

Kubernetes attempts to maintain:

```text
Pod 1 → Running
Pod 2 → Running
```

If a Pod crashes, Kubernetes can create a replacement.

---

# 3️⃣ Kubernetes Service

Pods are temporary.

Their IP addresses can change when Pods are recreated.

Therefore, Kubernetes uses a **Service** to provide stable networking.

Conceptually:

```text
                 Service
                    │
           ┌────────┴────────┐
           │                 │
           ▼                 ▼
         Pod 1             Pod 2
```

The Service selects Pods using labels.

Example:

```yaml
selector:
  app: eks-demo
```

Any matching Pod can receive traffic.

---

# 🌐 Service Exposure

A Kubernetes Service can expose the application in different ways.

Common Service types include:

### ClusterIP

```text
Internal Kubernetes access
```

### NodePort

```text
Node IP + Port
```

### LoadBalancer

```text
Internet
    │
    ▼
AWS Load Balancer
    │
    ▼
Kubernetes Service
    │
    ▼
Pods
```

For an AWS EKS application that needs to be publicly accessible, a LoadBalancer-type Service or an Ingress-based architecture can be used depending on the desired AWS networking design.

---

# ☁️ EKS Architecture

A simplified EKS architecture looks like:

```text
                         AWS
                          │
             ┌────────────┴────────────┐
             │                         │
             ▼                         ▼
       EKS Control Plane          AWS Resources
             │
             │
             ▼
        EKS Cluster
             │
       ┌─────┴─────┐
       │           │
       ▼           ▼
    Worker       Worker
     Node          Node
       │           │
       ▼           ▼
     Pods         Pods
```

The Kubernetes control plane manages the cluster.

The compute layer provides resources where application Pods can run.

---

# 🧠 EKS Components

## EKS Control Plane

The Kubernetes control plane is managed by AWS.

It is responsible for Kubernetes control-plane functionality such as:

* API server
* Scheduling
* Cluster state management
* Kubernetes controllers

You interact with the cluster using tools such as:

```bash
kubectl
```

---

## Worker Nodes

Worker nodes provide compute resources for running Pods.

A simplified example:

```text
EKS Cluster
│
├── Node 1
│   ├── Pod
│   └── Pod
│
└── Node 2
    ├── Pod
    └── Pod
```

Nodes can be EC2 instances in traditional EKS configurations.

EKS also supports other AWS-managed compute approaches depending on the cluster configuration.

---

# 🖥️ kubectl

`kubectl` is the command-line tool used to communicate with Kubernetes.

Check the cluster:

```bash
kubectl get nodes
```

Example:

```text
NAME                         STATUS   ROLES
ip-10-0-1-101.ec2.internal   Ready    <none>
ip-10-0-2-102.ec2.internal   Ready    <none>
```

---

# 🔐 Connect kubectl to EKS

After creating an EKS cluster, configure your local kubeconfig:

```bash
aws eks update-kubeconfig \
  --region us-east-1 \
  --name my-eks-cluster
```

Verify:

```bash
kubectl get nodes
```

If the nodes are available:

```text
STATUS
------
Ready
```

your local `kubectl` is communicating with the EKS cluster.

---

# 🚀 Deploy the Application

Navigate to the Kubernetes configuration directory:

```bash
cd k8s
```

Apply the Kubernetes manifests:

```bash
kubectl apply -f .
```

Kubernetes will create the resources described by the YAML files.

Check the Deployment:

```bash
kubectl get deployments
```

Check Pods:

```bash
kubectl get pods
```

Check Services:

```bash
kubectl get services
```

---

# 🔍 Verify the Deployment

Check all resources:

```bash
kubectl get all
```

Example:

```text
NAME                           READY   STATUS
pod/eks-demo-xxxxx             1/1     Running
pod/eks-demo-yyyyy             1/1     Running

NAME                  TYPE           CLUSTER-IP
service/eks-demo      LoadBalancer   ...
```

---

# 📜 View Pod Logs

To inspect application logs:

```bash
kubectl logs <pod-name>
```

For example:

```bash
kubectl logs eks-demo-xxxxx
```

For live logs:

```bash
kubectl logs -f <pod-name>
```

---

# 🔎 Describe Kubernetes Resources

If a Pod is not starting correctly:

```bash
kubectl describe pod <pod-name>
```

For a Deployment:

```bash
kubectl describe deployment <deployment-name>
```

For a Service:

```bash
kubectl describe service <service-name>
```

These commands are particularly useful for troubleshooting.

---

# 📈 Scaling

One of Kubernetes' major advantages is the ability to scale workloads.

For example:

```bash
kubectl scale deployment eks-demo --replicas=3
```

Now Kubernetes attempts to maintain:

```text
Deployment
│
├── Pod 1
├── Pod 2
└── Pod 3
```

Check:

```bash
kubectl get pods
```

---

# 🔄 Self-Healing

Suppose there are two Pods:

```text
Pod 1 → Running
Pod 2 → Running
```

If Pod 1 fails:

```text
Pod 1 → Failed
Pod 2 → Running
```

The Deployment controller detects that the desired number of replicas is not available.

Kubernetes creates a replacement:

```text
Pod 2 → Running
Pod 3 → Running
```

This is called **self-healing**.

---

# 🔁 Rolling Updates

Kubernetes Deployments support rolling updates.

Suppose version `v1` is running:

```text
Pod 1 → v1
Pod 2 → v1
```

You deploy `v2`.

Kubernetes can gradually replace the old Pods:

```text
Pod 1 → v2
Pod 2 → v1
```

Then:

```text
Pod 1 → v2
Pod 2 → v2
```

This allows applications to be updated without manually stopping every Pod.

---

# 🏷️ Labels and Selectors

Kubernetes resources use labels to identify workloads.

Example:

```yaml
labels:
  app: eks-demo
```

The Service can then select those Pods:

```yaml
selector:
  app: eks-demo
```

The relationship is:

```text
Service
   │
   │ selector: app=eks-demo
   │
   ▼
Pods
 ├── app=eks-demo
 └── app=eks-demo
```

This is how the Service knows which Pods should receive traffic.

---

# 🔄 Complete Request Flow

When a user accesses the deployed application:

```text
                    User
                     │
                     ▼
              AWS Load Balancer
                     │
                     ▼
              Kubernetes Service
                     │
            ┌────────┴────────┐
            │                 │
            ▼                 ▼
          Pod 1             Pod 2
            │                 │
            ▼                 ▼
       Application       Application
        Container         Container
```

The Service distributes traffic among the available Pods.

If one Pod becomes unavailable, Kubernetes can replace it and the Service can continue directing traffic to available Pods.

---

# 🛠️ Useful Kubernetes Commands

## Cluster

```bash
kubectl cluster-info
```

```bash
kubectl get nodes
```

```bash
kubectl get namespaces
```

---

## Pods

```bash
kubectl get pods
```

```bash
kubectl get pods -o wide
```

```bash
kubectl describe pod <pod-name>
```

```bash
kubectl logs <pod-name>
```

---

## Deployments

```bash
kubectl get deployments
```

```bash
kubectl describe deployment <deployment-name>
```

```bash
kubectl rollout status deployment/<deployment-name>
```

---

## Services

```bash
kubectl get services
```

```bash
kubectl describe service <service-name>
```

---

## Apply Configuration

```bash
kubectl apply -f k8s/
```

---

## Delete Application

```bash
kubectl delete -f k8s/
```

---

# 🧹 Delete the EKS Cluster

If the cluster was created specifically for learning, remember that AWS resources can incur charges.

Delete the cluster using the method you used to create it.

For an `eksctl` cluster, for example:

```bash
eksctl delete cluster \
  --name my-eks-cluster \
  --region us-east-1
```

Before deleting anything, verify that the cluster name and AWS region are correct.

---

# 🔐 Security Considerations

For a production EKS deployment:

* Use IAM least-privilege permissions.
* Avoid hard-coded AWS credentials.
* Use IAM roles for workloads where appropriate.
* Use private subnets for worker workloads when appropriate.
* Restrict Security Group rules.
* Keep Kubernetes and container images updated.
* Scan container images for vulnerabilities.
* Use HTTPS for public applications.
* Store sensitive configuration in AWS Secrets Manager or Kubernetes Secrets with appropriate controls.
* Enable monitoring and logging.
* Apply Kubernetes RBAC.

---

# 📊 Monitoring

An EKS environment can be monitored using AWS and Kubernetes observability tools.

Useful areas include:

```text
EKS
 │
 ├── Cluster health
 ├── Node health
 ├── Pod health
 ├── CPU utilization
 ├── Memory utilization
 └── Application logs
```

AWS services such as CloudWatch can be integrated for monitoring and logging.

---

# 🔄 CI/CD Architecture

This project can be extended into a complete DevOps pipeline.

A typical architecture would be:

```text
Developer
    │
    ▼
GitHub
    │
    ▼
Jenkins / GitHub Actions
    │
    ├── Checkout
    ├── Build
    ├── Test
    ├── Docker Build
    ├── Security Scan
    └── Push Image
            │
            ▼
        Amazon ECR
            │
            ▼
        Amazon EKS
            │
            ▼
      Kubernetes Deployment
            │
            ▼
          Pods
            │
            ▼
        Application
```

For a GitOps implementation, tools such as Argo CD can also be introduced:

```text
GitHub
   │
   ▼
Manifest Repository
   │
   ▼
Argo CD
   │
   ▼
Amazon EKS
   │
   ▼
Kubernetes Workloads
```

---

# 🆚 EKS vs ECS

Both are AWS container services, but they use different orchestration models.

| Feature                  | Amazon EKS            | Amazon ECS          |
| ------------------------ | --------------------- | ------------------- |
| Orchestration            | Kubernetes            | AWS ECS             |
| Workload                 | Pods                  | Tasks               |
| Deployment resource      | Deployment            | ECS Service         |
| Networking               | Kubernetes networking | ECS networking      |
| CLI                      | kubectl               | AWS CLI / ECS tools |
| Kubernetes ecosystem     | Yes                   | No                  |
| AWS-native orchestration | No                    | Yes                 |
| Kubernetes manifests     | Yes                   | No                  |

This project specifically focuses on **Amazon EKS and Kubernetes**.

---

# 🎯 Learning Objectives

After completing this project, you should understand:

* What Amazon EKS is
* How Kubernetes works on AWS
* EKS control plane and worker compute
* Docker containerization
* Kubernetes Pods
* Kubernetes Deployments
* Kubernetes Services
* Labels and selectors
* Application scaling
* Self-healing
* Rolling deployments
* EKS networking
* `kubectl`
* Amazon ECR
* AWS Load Balancer integration
* Basic EKS troubleshooting
* EKS CI/CD architecture

---

# 🧠 EKS Interview Cheat Sheet

### What is EKS?

Amazon EKS is AWS's managed Kubernetes service.

### What is a Pod?

The smallest deployable unit in Kubernetes. It contains one or more containers.

### What is a Deployment?

A Kubernetes resource that manages application Pods and maintains the desired replica count.

### What is a Service?

A Kubernetes resource that provides stable networking and service discovery for Pods.

### Why do we need a Service?

Pod IP addresses can change. A Service provides a stable endpoint and routes traffic to matching Pods.

### What is ECR?

Amazon Elastic Container Registry is a managed container image registry used to store Docker/OCI images.

### How does EKS get the container image?

The Kubernetes workload references an image, commonly stored in ECR. The node/runtime pulls the image and starts the container.

### What happens if a Pod crashes?

The Kubernetes controllers work to restore the desired state, which can result in a replacement Pod being created.

### What is scaling?

Increasing or decreasing the number of application replicas.

```bash
kubectl scale deployment <deployment> --replicas=3
```

### What is rolling deployment?

Gradually replacing old application Pods with new Pods instead of replacing everything at once.

---

# 📁 Repository Structure

```text
EKS-Demo/
│
├── k8s/
│   └── Kubernetes manifests
│
└── website/
    └── Website/application files
```

The `k8s/` directory contains the Kubernetes configuration used to deploy the workload, while `website/` contains the application itself.

---

# 🚀 Future Improvements

Possible extensions for this project:

* [ ] Deploy using Amazon ECR
* [ ] Configure AWS Load Balancer Controller
* [ ] Configure HTTPS with ACM
* [ ] Add Route 53 DNS
* [ ] Add Horizontal Pod Autoscaler
* [ ] Add CloudWatch monitoring
* [ ] Add Kubernetes Secrets
* [ ] Add Helm
* [ ] Add Terraform for EKS infrastructure
* [ ] Add Jenkins CI/CD
* [ ] Add GitHub Actions
* [ ] Add Argo CD GitOps
* [ ] Add container vulnerability scanning
* [ ] Add Prometheus and Grafana
* [ ] Implement blue/green or canary deployments

---

# 👨‍💻 Author

**Navaneeth Krishna**

Cloud & DevOps | AWS | Kubernetes | Docker | Terraform | Jenkins | AI/ML

---

## ⭐ Project Summary

This project demonstrates the fundamental workflow of deploying a containerized application to **Amazon EKS**:

```text
Application
     ↓
Docker
     ↓
Container Image
     ↓
Amazon ECR
     ↓
Amazon EKS
     ↓
Kubernetes Deployment
     ↓
Pods
     ↓
Kubernetes Service
     ↓
Load Balancer
     ↓
Users
```

The main concept is:

> **ECR stores the container image, EKS provides the managed Kubernetes environment, a Deployment manages the Pods, and a Service provides stable networking to the application.**
