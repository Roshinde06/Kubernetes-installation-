# Kubernetes Cluster Setup Using KIND on AWS EC2

## 📌 Project Overview

This project focuses on setting up a local Kubernetes cluster using **KIND (Kubernetes IN Docker)** on an AWS EC2 instance running Amazon Linux 2023.

KIND allows Kubernetes clusters to run inside Docker containers, making it useful for learning Kubernetes concepts, deploying applications, and practising container orchestration.

## 🛠️ Technologies Used

- **AWS EC2** – Cloud-based virtual machine
- **Amazon Linux 2023** – Operating system
- **Docker** – Container runtime
- **KIND** – Kubernetes cluster management
- **kubectl** – Kubernetes command-line tool
- **YAML** – Cluster configuration

## 🏗️ Architecture

```text
AWS Cloud
└── EC2 Instance
    └── Amazon Linux 2023
        └── Docker
            └── KIND Kubernetes Cluster
                ├── Control Plane Node
                └── Worker Node(s)
```

## ⚙️ Project Setup

### 1. Launch an EC2 Instance

- Launch an EC2 instance with Amazon Linux 2023.
- Configure SSH access through a security group.
- Connect to the instance using SSH.

### 2. Install and Start Docker

```bash
sudo dnf update -y
sudo dnf install docker -y
sudo systemctl enable --now docker
```

Add the EC2 user to the Docker group:

```bash
sudo usermod -aG docker ec2-user
```

Reconnect to SSH and verify Docker:

```bash
docker ps
```

### 3. Install KIND

```bash
curl -Lo kind https://kind.sigs.k8s.io/dl/v0.30.0/kind-linux-amd64
chmod +x kind
sudo mv kind /usr/local/bin/kind
kind version
```

### 4. Install kubectl

Install the Kubernetes command-line tool and verify it:

```bash
kubectl version --client
```

### 5. Configure the KIND Cluster

Create a file named `config.yml`:

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4

nodes:
  - role: control-plane
    image: kindest/node:v1.33.1
    extraPortMappings:
      - containerPort: 80
        hostPort: 80
        protocol: TCP

  - role: worker
    image: kindest/node:v1.33.1
```

### 6. Create the Kubernetes Cluster

```bash
kind create cluster --name my-cluster --config config.yml
```

### 7. Verify the Cluster

```bash
kubectl cluster-info
kubectl get nodes
kubectl get pods -A
```

## 📚 Key Learnings

- Setting up a cloud-based Linux environment on AWS EC2.
- Installing and configuring Docker on Amazon Linux.
- Creating Kubernetes clusters using KIND.
- Configuring Kubernetes nodes using YAML.
- Using kubectl to inspect nodes and system pods.
- Troubleshooting Docker daemon permissions and cluster creation errors.
- Understanding port mappings between the EC2 host and KIND nodes.

## 🔐 Security Considerations

- Restrict SSH access to your own IP address.
- Avoid exposing the Kubernetes API server publicly.
- Allow inbound HTTP traffic only when required for application access.
- Remember that membership in the Docker group grants highly privileged access to the host.

## 💰 Cost Considerations

- Check AWS Free Tier eligibility and available credits before launching EC2.
- Stop or terminate unused EC2 instances.
- Remember that EBS volumes and other resources may continue to incur charges.
- Monitor AWS billing regularly.

## 🚀 Future Improvements

- Deploy a sample application to the cluster.
- Configure Kubernetes Services and Ingress.
- Practise rolling updates and scaling deployments.
- Integrate a CI/CD pipeline using GitHub Actions.
- Explore Helm and Kubernetes monitoring.

## 👩‍💻 Author

**Roshani Shinde**

Aspiring Cloud & DevOps Engineer | AWS | Linux | Docker | Kubernetes | CI/CD | Terraform
