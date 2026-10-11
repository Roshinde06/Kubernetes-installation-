# Minikube Installation on AWS EC2 --- Step-by-Step Summary

## Environment

-   Cloud: AWS EC2
-   Operating system: Amazon Linux 2023
-   Container runtime / Minikube driver: Docker
-   Kubernetes tool: `kubectl`
-   Minikube cluster name: `minikube`

## 1. Install Docker and start the service

If Docker is not already installed:

``` bash
sudo dnf update -y
sudo dnf install -y docker
sudo systemctl enable --now docker
```

Check Docker:

``` bash
sudo systemctl status docker
sudo docker ps
```

## 2. Allow `ec2-user` to run Docker

Add the user to the Docker group:

``` bash
sudo usermod -aG docker ec2-user
```

Then exit SSH completely and reconnect. Verify:

``` bash
id
docker ps
```

The `id` output should include `docker`, and `docker ps` should work
without `sudo`.

**Security:** The Docker group effectively grants root-level privileges.
Never fix socket access with `chmod 666 /var/run/docker.sock`.

## 3. Install Minikube

Download and install the latest x86-64 Linux binary:

``` bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
rm minikube-linux-amd64
```

Verify:

``` bash
minikube version
```

## 4. Install kubectl

Download and install the current stable x86-64 Linux client:

``` bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
rm kubectl
```

Verify:

``` bash
kubectl version --client
```

## 5. Start the Minikube cluster

Set Docker as the driver and start the cluster:

``` bash
minikube config set driver docker
minikube start --driver=docker
```

The first start can take a few minutes because Minikube downloads
Kubernetes images.

## 6. Verify cluster health

Run each command separately:

``` bash
minikube status
kubectl get nodes
kubectl get pods -A
```

Expected: - Minikube components report `Running`. - The node status
becomes `Ready`. - Kubernetes system pods gradually become `Running`.

If `kubectl` cannot connect, check `minikube status` first. The API
server must be running.

## 7. Check available memory

The EC2 screenshot showed a warning that allocating 1909 MiB leaves
little memory for the operating system. Check:

``` bash
free -h
df -h
```

A small EC2 instance may become unstable if Minikube uses nearly all
available RAM. If needed, stop Minikube and try a lower allocation,
provided enough memory remains for Kubernetes:

``` bash
minikube stop
minikube start --driver=docker --memory=1500
```

If the node or system pods fail with low memory, use a larger instance
rather than continually lowering memory.

## 8. Deploy a test Nginx application

After `kubectl get nodes` shows `Ready`, create a deployment:

``` bash
kubectl create deployment nginx --image=nginx
kubectl rollout status deployment/nginx
kubectl get deployments
kubectl get pods
```

Optional: expose Nginx inside the cluster:

``` bash
kubectl expose deployment nginx --port=80 --type=ClusterIP
kubectl get services
```

A `ClusterIP` service is reachable inside the cluster. It does not
automatically make Nginx publicly accessible from the internet.

## 9. Useful day-to-day commands

``` bash
minikube status
minikube stop
minikube start
kubectl get nodes
kubectl get pods -A
kubectl describe pod <pod-name>
kubectl logs <pod-name>
kubectl delete deployment nginx
kubectl delete service nginx
```

Replace `<pod-name>` with the actual pod name from `kubectl get pods`.

## 10. AWS cost reminders

-   `minikube stop` stops the local Kubernetes cluster but does **not**
    stop EC2 instance billing.
-   Stop the EC2 instance when you are finished using it if you want to
    stop compute charges.
-   EBS volumes, snapshots, Elastic IPs, and other resources may still
    incur charges.
-   Review AWS Billing and Cost Management and remove resources you no
    longer need.

## Official documentation

-   Minikube start: https://minikube.sigs.k8s.io/docs/start/
-   Minikube Docker driver:
    https://minikube.sigs.k8s.io/docs/drivers/docker/
-   Docker post-installation:
    https://docs.docker.com/engine/install/linux-postinstall/
