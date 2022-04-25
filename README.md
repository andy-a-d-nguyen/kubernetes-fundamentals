# Kubernetes

- [Kubernetes](#kubernetes)
  - [Key Features](#key-features)
  - [Overview](#overview)
  - [Key Container Benefits](#key-container-benefits)
  - [Key Kubernetes Benefits](#key-kubernetes-benefits)
  - [Developer Use Cases](#developer-use-cases)
  - [Basic kubectl Commands](#basic-kubectl-commands)
    - [Enabling Kubernetes Web UI Dashboard](#enabling-kubernetes-web-ui-dashboard)
  - [Pods](#pods)
    - [Creating a Pod](#creating-a-pod)
    - [Exposing a Pod Port](#exposing-a-pod-port)
    - [Deleting a Pod](#deleting-a-pod)
  - [YAML Fundamentals](#yaml-fundamentals)
    - [Creating a Pod Using YAML](#creating-a-pod-using-yaml)
    - [Creating or Applying a Pod Using YAML](#creating-or-applying-a-pod-using-yaml)
    - [Inspecting Pods](#inspecting-pods)
    - [Opening a shell in a Pod](#opening-a-shell-in-a-pod)
    - [Editing a Pod configuration](#editing-a-pod-configuration)
  - [Pod Health](#pod-health)
    - [Defining an HTTP Liveness Probe](#defining-an-http-liveness-probe)
    - [Defining an ExecAction Liveness Probe](#defining-an-execaction-liveness-probe)
    - [Defining a Readiness Probe](#defining-a-readiness-probe)
  - [Deployments](#deployments)
    - [Defining a Deployment](#defining-a-deployment)
    - [Defining Probes in a Deployment](#defining-probes-in-a-deployment)
    - [Creating a Deployment](#creating-a-deployment)
    - [Creating or Applying Changes to a Deployment](#creating-or-applying-changes-to-a-deployment)
    - [Getting Deployments](#getting-deployments)
      - [Deployments and Labels](#deployments-and-labels)
    - [Deleting a Deployment](#deleting-a-deployment)
    - [Scaling Pods Horizontally](#scaling-pods-horizontally)
  - [Deployment Options](#deployment-options)
    - [Updating a Deployment (Rolling Update)](#updating-a-deployment-rolling-update)
  - [Services](#services)

## Key Features

- Service Discovery/Load Balancing
- Storage Orchestration
- Automate Rollouts/Rollbacks
- Self-healing Containers
- Secret and Configuration Management
- Horizontal Scaling

## Overview

- Move from current state to desired state
- Master node that manages worker nodes (physical servers, virtual machines, etc.)
  - Together, worker nodes create a cluster
  - In each worker node, the master node can start pods
  - A master node contains a store (etcd), which tracks a cluster
  - A master node contains a Controller Manager that can receive requests
  - The Controller Manager can schedule requests using a Scheduler
    - The Scheduler determines when a node or pods in a node come to live or go away
- A node can run one or more pods
  - Each node has an agent called a kubelet that registers the node with the cluster and reports to and from the Controller Manager
  - Each node contains a Container Runtime to run the pods
  - Each node has a Kube-Proxy that gives each pod a unique IP address
- A pod contains a container (e.g. a box contains actual stuff)
  - Pods are created through deployments
  - Pods connect to the outside world through Kubernetes Services
- Developers interact with the master node through kubectl to the API Server in the master node
- The API Server is a RESTful service

## Key Container Benefits

- Accelerate Developer Onboarding
- Eliminate App Conflicts
- Environment Consistency
- Ship Software Faster

## Key Kubernetes Benefits

- Orchestrate Containers
- Zero-Downtime Deployments
- Self Healing

## Developer Use Cases

- Emulate production locally
- Create an end-to-end testing environment
- Ensure application scales properly
- Ensure secrets/config are working properly
- Performance testing scenarios
- Workload scenarios (CI/CD and more)
- Learn how to leverage deployment options
- Help DevOps create resources and solve problems

## Basic kubectl Commands

```sh
kubectl version # Check Kubernetes version
kubectl cluster-info # View cluster information
kubectl get all # Retrieve information about Kubernetes Pods, Deployments, Services, etc.
kubectl run [container-name] --image=[image-name] # Simple way to create a Pod (on Kubernetes version < 1.18, run will create a deployment)
kubectl port-forward [pod] [ports] # Forward a port to allow external access
kubectl expose ... # Expose a port for a Deployment/Pod
kubectl create [resource] # Create a resource (Deployment, Service, etc.)
kubectl apply [resource] # Create or modify a resource
```

### Enabling Kubernetes Web UI Dashboard

```sh
kubectl apply [dashboard-yaml-url]
kubectl describe secret -n kube-system
kubectl proxy
```

## Pods

- A Pod is the basic execution unit of a Kubernetes application
- Smallest object of the Kubernetes object model
- Environment for containers
- Organize application "parts" into Pods (server, caching, APIs, database, etc.)
- Pod IP, memory, volumes, etc. shared across containers
- Scale horizontally by adding Pod replicas
- Pods live and die but never come back to life
- Replicas:
  - Copies or clones of a Pod
  - Kubernetes can load-balance between them
- A sick Pod is removed from a node and replaced with a new healthy one
- Pods have unique IPs called ClusterIP by default
- Containers within a Pod have unique ports
- Pod containers share the same Network namespace (share IP/port)
- Pod containers have the same loopback network interface (localhost)
- Container processes need to bind to different ports within a Pod
- Ports can be reused by containers in separate Pods
- Pods do not span nodes

### Creating a Pod

- Ways to create a pod:
  - `kubectl run`
    - Ex: `kubectl run [pod-name] --image=nginx:alpine`
  - `kubectl create/apply` with a yaml file
- To list pods: `kubectl get pods`
- To list all resources: `kubectl get all`

### Exposing a Pod Port

- Pods and containers are only accessible within the Kubernetes cluster by default
- A way to expose a container port externally: `kubectl port-forward`
  - Ex: `kubectl port-forward [pod-name] 8080:80` where 8080 is the external port and 80 is the internal port

### Deleting a Pod

- Running a Pod will cause a Deployment to be created on Kubernetes version 1.18+, but not < 1.18
- To delete a Pod: `kubectl delete pod` (will cause a new Pod to be created to replace the deleted one to maintain deployment state tracked by Kubernetes)
  - Ex: `kubectl delete pod [pod-name]`
- To delete a deployment: `kubectl delete deployment`
  - Ex: `kubectl delete deployment [deployment-name]`
- To delete a Pod using a YAML file that created it
  - Ex: `kubectl delete -f file.pod.yml`

## YAML Fundamentals

- YAML files are composed of maps and lists
- Indentation matters
- Always use spaces
- Maps:
  - key-value pairs
  - Maps can contain other maps for more complex data structures
- Lists:
  - Sequence of items
  - Multiple maps can be defined in a list

Ex:

```yml
key: value
complexMap:
  key1: value
  key2:
    subKey: value
items:
  - item1
  - item2
itemsMap:
  - map1: value
    map1Prop: value
  - map2: value
    map2Prop: value
```

Ex:

```yml
apiVersion: v1 # Kubernetes API version
kind: Pod # Type of Kubernetes resource
metadata: # Metadata about the Pod
  name: my-nginx
spec: # The spec/blueprint for the Pod
  containers: # Info about the containers that will run in the Pod
  - name: my-nginx
    image: nginx:alpine
```

### Creating a Pod Using YAML

- To create a pod using YAML, use the `kubectl create` command along with the `--filename` or `-f` switch

```bash
# Perform a "trial" create and also validate the YAML
# --validate defaults to true unless specified otherwise
kubectl create -f file.pod.yml --dry-run --validate=true

# Create a Pod from a YAML file
# Will error if Pod already exists
kubectl create -f file.pod.yml
```

### Creating or Applying a Pod Using YAML

- To create or apply changes to a pod using YAML, use the `kubectl apply` command along with the `--filename` or `-f` switch

```bash
# Alternate way to create or apply changes to a Pod from YAML
kubectl apply -f file.pod.yml
```

```bash
# Use --save-config when you want to use kubectl apply in the future
# --save-config stores current properties/configuration settings in resource's annotations
kubectl create -f file.pod.yml --save-config
```

Ex using `kubectl create --save-config`:

```yml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: # Allows in-place changes to be made to a Pod in the future using kubectl apply
    { "apiVersion": "v1",
      "kind": "Pod",
      "metadata": {
        "name": "my-ngninx"
        ...
      }
    }
...
```

- In-place/non-disruptive changes can also be made to a Pod using `kubectl edit` or `kubectl patch`

### Inspecting Pods

```bash
kubectl describe pod [pod-name]
```

### Opening a shell in a Pod

```bash
kubectl exec [pod-name] -it sh
```

### Editing a Pod configuration

```bash
kubectl edit -f file.pod.yml
```

## Pod Health

- Kubernetes relies on Probes to determine the health of a Pod container
- Types of Probes:
  - Liveness Probe:
    - Can be used to determine if a Pod is healthy and running as expected
    - When should a container restart?
  - Readiness Probe:
    - Can be used to determine if a Pod should receive requests
    - When should a container start receiving traffic?
- Failed Pod containers are recreated by default (restartPolicy defaults to Always)
- ExecAction: Executes an action inside the container
- TCPSocketAction: TCP check against the container's IP address on a specified port
- HTTPGetAction: HTTP GET request against the container
- Probes can have the following results:
  - Success
  - Failure
  - Unknown

### Defining an HTTP Liveness Probe

```yml
apiVersion: v1
kind: Pod
...
spec:
  containers:
  - name: my-nginx
    image: nginx:alpine
    livenessProbe: # Define liveness probe
      httpGet:
        path: /index.html # Check /index.html on port 80
        port: 80
      initialDelaySeconds: 15 # Wait 15 seconds for this pod to come up
      timeoutSeconds: 2 # Timeout after 2 seconds
      periodSeconds: 5 # Check every 5 seconds
      failureThreshold: 1 # Allow 1 failure before failing Pod
```

### Defining an ExecAction Liveness Probe

```yml
apiVersion: v1
kind: Pod
...
spec:
  containers:
  - name: liveness
    image: k8s.gcr.io/busybox
    args: # Define args for container
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30;
      rm -rf /tmp/healthy; sleep 600
    livenessProbe: # Define liveness probe
      exec:
        command: # Define action/command to execute
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5
```

### Defining a Readiness Probe

```yml
apiVersion: v1
kind: Pod
...
spec:
  containers:
  - name: my-nginx
    image: nginx:alpine
    readinessProbe: # Define readiness probe
      httpGet:
        path: /index.html # Check /index.html on port 80
        port: 80
      initialDelaySeconds: 2 # Wait 2 seconds
      periodSeconds: 5 # Check every 5 seconds
```

## Deployments

- A ReplicaSet is a declarative way to manage Pods
- A Deployment is a declarative way to manage Pods using a ReplicaSet
- ReplicaSets act as a Pod controller:
  - Self-healing mechanism
  - Ensure the requested number of Pods are available
  - Provide fault-tolerance
  - Can be used to scale Pods
  - Relies on a Pod template
  - No need to create Pods directly
  - Used by Deployments
- A Deployment manages Pods:
  - Pods are managed using ReplicaSets
  - Scales ReplicaSets, which scales Pods
  - Supports zero-downtime updates by creating and destroying ReplicaSets
  - Provides rollback functionality
  - Creates a unique label that is assigned to the ReplicaSet and generated Pods
  - YAML is very similar to a ReplicaSet

### Defining a Deployment

```yml
apiVersion: apps/v1 # Kubernetes API version and resource type (Deployment)
kind: Deployment
metadata: # Metadata about the Deployment
spec:
  selector: # Select Pod template label(s)
  template: # Template used to create the Pods
    spec:
      containers: # Containers that will run in the Pod
      - name: my-nginx
        image: nginx:alpine
```

```yml
apiVersion: apps/v1 # Kubernetes API version and resource type (Deployment)
kind: Deployment
metadata: # Metadata about the Deployment
  name: frontend
  labels:
    app: my-nginx
    tier: frontend
spec:
  selector: # The selector is used to "select" the template to use (based on labels)
    matchLabels:
      tier: frontend
  template: # Template to use to create the Pod/Containers (note that the selector matches the label)
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: my-nginx
        image: nginx:alpine
```

### Defining Probes in a Deployment

```yml
apiVersion: apps/v1
kind: Deployment
...
template:
  spec:
    containers:
    - name: my-nginx
      image: nginx:alpine
      livenessProbe: # Define liveness probe (readiness probes can also be defined)
        httpGet:
          path: /index.html
          port: 80
        initialDelaySeconds: 15
        timeoutSeconds: 2
        periodSeconds: 5
        failureThreshold: 1
```

### Creating a Deployment

- Use the `kubectl create` command along with the `--filename` or `-f` switch

Ex:

```bash
kubectl create -f file.deployment.yml
```

### Creating or Applying Changes to a Deployment

- Use the `kubectl apply` command along with the `--filename` or `-f` switch

Ex:

```bash
# Alternate way to create or apply changes to a Deployment from YAML
kubectl apply -f file.deployment.yml

# Use --save-config when you want to use kubectl apply in the future
kubectl create -f file.deployment.yml --save-config
```

### Getting Deployments

```bash
kubectl get deployments
```

#### Deployments and Labels

- List the labels for all Deployments using the `--show-labels` switch
- To get information about a Deployment with a specified label, use the `-l` switch

```bash
# List all Deployments and their labels
kubectl get deployment --show-labels

# Get all Deployments with a specified label
kubectl get deployment -l app=nginx
```

### Deleting a Deployment

- To delete a Deployment, use `kubectl delete`
- Will delete the Deployment and all associated Pods/Containers

```bash
# Delete Deployment
kubectl delete deployment [deployment-name]
```

### Scaling Pods Horizontally

- Update the YAML file or use the `kubectl scale` command

```bash
# Scale the Deployment Pods to 5
kubectl scale deployment [deployment-name] --replicas=5

# Scale by referencing the YAML file
kubectl scale -f file.deployment.yaml --replicas=5
```

```yml
spec:
  replicas: 3
  selector:
    tier: frontend
```

## Deployment Options

- One of the strengths of Kubernetes is zero downtime deployments
- Update an application's Pods without impacting end users
- Several options are available:
  - Rolling updates
  - Blue-green deployments
  - Canary deployments
  - Rollbacks

### Updating a Deployment (Rolling Update)

- Update a Deployment by changing the YAML and applying changes to the cluster with `kubectl apply`

```bash
# Apply changes made in a YAML file
kubectl apply -f file.deployment.yaml
```

## Services

- A Service provides a single point of entry for accessing one or more Pods
- Since Pods live and die, you can't rely on their IPs because they change
- The life of Pods:
  - Pods are "mortal" and may only live a short time (ephemeral)
  - Pods can be horizontally scaled so each Pod gets its own IP address
  - A Pod gets an IP address after it has been scheduled (no way for clients to know IP ahead of time)
- The role of Services:
  - Services abstract Pod IP addresses from consumers
  - Load balances between Pods
  - Relies on labels to associate a Service with a Pod
  - Node's kube-proxy creates a virtual IP for Services
  - Layer 4 (TCP/UPD over IP)
  - Services are not ephemeral
  - Creates endpoints which sit between a Service and Pod

Ex:

External caller -> Service (label: frontend, IP: 10.0.0.1:80) -> Pod (label: frontend, IP: 10.0.0.43:8080) -> Service (label: backend, IP: 10.2.0.1:9000) -> Pod (label: backend, IP: 10.2.0.10:27017)
