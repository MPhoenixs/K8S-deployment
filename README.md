# Kubernetes MongoDB Deployment

A simple Kubernetes project demonstrating how to deploy **MongoDB and Mongo Express** using Kubernetes Deployments, Services, ConfigMaps, and Secrets.

The project is built and tested locally using **Minikube** and `kubectl`.

## Architecture

```text
                    Kubernetes Cluster
                           │
              ┌────────────┴────────────┐
              │                         │
              ▼                         ▼
       MongoDB Deployment       Mongo Express Deployment
              │                         │
              ▼                         ▼
        MongoDB Pod              Mongo Express Pod
          :27017                     :8081
              │                         │
              ▼                         ▼
     mongodb-service          mongo-express-service
       (ClusterIP)                  (NodePort)
              │
              └──────────────┐
                             │
                             ▼
                         MongoDB
```

Mongo Express connects to MongoDB through the Kubernetes Service:

```text
mongodb-service:27017
```

## Kubernetes Concepts Used

This project demonstrates:

* **Deployment** — manages MongoDB and Mongo Express Pods
* **Service** — provides stable networking between Pods
* **ConfigMap** — stores the MongoDB Service name
* **Secret** — stores MongoDB username and password
* **NodePort / LoadBalancer** — exposes Mongo Express outside the cluster
* **Minikube** — runs the Kubernetes cluster locally
* **kubectl** — manages and interacts with Kubernetes resources

## Project Structure

```text
K8S-deployment/
│
├── mongo-configmap.yaml
├── mongo-deployment.yaml
├── mongo-express.yaml
└── mongo-secret.yaml
```

### `mongo-deployment.yaml`

Creates the MongoDB Deployment and its Kubernetes Service.

MongoDB listens on:

```text
27017
```

### `mongo-express.yaml`

Creates the Mongo Express Deployment and Service.

Mongo Express listens on:

```text
8081
```

### `mongo-configmap.yaml`

Stores the MongoDB Service name:

```yaml
data:
  database_url: mongodb-service
```

Mongo Express uses this value to locate MongoDB inside the Kubernetes cluster.

### `mongo-secret.yaml`

Stores the MongoDB root username and password as Kubernetes Secret data.

> Kubernetes Secrets are Base64-encoded by default. Base64 is encoding, not encryption, so production deployments should use appropriate secret-management mechanisms.

## Prerequisites

Install:

* Docker Desktop
* Minikube
* kubectl

Verify the installations:

```bash
docker --version
minikube version
kubectl version --client
```

## Getting Started

### 1. Start Minikube

Start a local Kubernetes cluster using Docker:

```bash
minikube start --driver=docker
```

Verify the cluster:

```bash
minikube status
```

Check the node:

```bash
kubectl get nodes
```

The node should be in the `Ready` state.

## 2. Create the Secret

Apply the MongoDB Secret:

```bash
kubectl apply -f mongo-secret.yaml
```

Verify:

```bash
kubectl get secrets
```

## 3. Create the ConfigMap

```bash
kubectl apply -f mongo-configmap.yaml
```

Verify:

```bash
kubectl get configmap
```

## 4. Deploy MongoDB

```bash
kubectl apply -f mongo-deployment.yaml
```

Check the Pods:

```bash
kubectl get pods
```

Check the MongoDB Service:

```bash
kubectl get service
```

You should see the MongoDB Service exposing port `27017`.

## 5. Deploy Mongo Express

```bash
kubectl apply -f mongo-express.yaml
```

Check the Pods:

```bash
kubectl get pods
```

Wait until both MongoDB and Mongo Express show:

```text
1/1 Running
```

## 6. Access Mongo Express

For Minikube, run:

```bash
minikube service mongo-express-service
```

Minikube will provide a local URL that can be opened in the browser.

Mongo Express uses Basic Authentication. The default credentials provided by the `mongo-express` image may be:

```text
Username: admin
Password: pass
```

For production usage, these credentials should be explicitly configured and secured.

## Useful Kubernetes Commands

### View all Pods

```bash
kubectl get pods
```

### View Services

```bash
kubectl get services
```

### View Deployments

```bash
kubectl get deployments
```

### View ConfigMaps

```bash
kubectl get configmaps
```

### View Secrets

```bash
kubectl get secrets
```

### View Pod logs

```bash
kubectl logs <pod-name>
```

Example:

```bash
kubectl logs <mongo-express-pod-name>
```

### Enter a container

```bash
kubectl exec -it <pod-name> -- /bin/sh
```

### Describe a Pod

```bash
kubectl describe pod <pod-name>
```

This is useful for troubleshooting scheduling, image-pull, networking, and container-startup problems.

## Verify MongoDB Connectivity

Mongo Express should connect to MongoDB using the Kubernetes Service name:

```text
mongodb-service:27017
```

The communication flow is:

```text
Mongo Express Pod
       │
       │ mongodb-service:27017
       ▼
mongodb-service
       │
       ▼
MongoDB Pod
```

Using a Kubernetes Service instead of directly using the MongoDB Pod IP provides a stable endpoint even if the MongoDB Pod is recreated.

## Troubleshooting

### Mongo Express cannot connect to MongoDB

Check the MongoDB Service:

```bash
kubectl get service mongodb-service
```

Check its endpoints:

```bash
kubectl get endpoints mongodb-service
```

There should be an endpoint pointing to the MongoDB Pod.

Check MongoDB:

```bash
kubectl get pods
```

If MongoDB is not running, inspect its logs:

```bash
kubectl logs <mongodb-pod-name>
```

### Check Mongo Express configuration

Verify the environment variable:

```bash
kubectl exec -it <mongo-express-pod-name> -- printenv ME_CONFIG_MONGODB_SERVER
```

It should return:

```text
mongodb-service
```

## Cleanup

Delete the deployed resources:

```bash
kubectl delete -f mongo-express.yaml
kubectl delete -f mongo-deployment.yaml
kubectl delete -f mongo-configmap.yaml
kubectl delete -f mongo-secret.yaml
```

To delete the entire Minikube cluster:

```bash
minikube delete
```

## What I Learned

This project demonstrates the basic workflow of deploying a database-backed application on Kubernetes:

1. Create Kubernetes resources using YAML.
2. Store configuration using ConfigMaps.
3. Store credentials using Secrets.
4. Deploy applications using Deployments.
5. Expose applications using Services.
6. Use Kubernetes DNS for service-to-service communication.
7. Debug Pods and Services using `kubectl`.
8. Access locally deployed applications through Minikube.

## Technologies

* Kubernetes
* Minikube
* Docker
* kubectl
* MongoDB
* Mongo Express
* YAML
