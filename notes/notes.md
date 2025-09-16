# Certified Kubernetes Application Development:
## Module 1: First Section
### First Section
* Exam is 2 hours, at-home
* 1 free retake within a 12 month window
* Not a multiple-choice test but an application-of-skills test, ie using the tech as it would be used IRL
* Kubernetes official documentation is allowed at all times in the exam.
* Code 30KK to get 30% off site-wide at any Linux Foundation Certification.
* Reference Documents:
    * Certified Kubernetes Application Developer: https://www.cncf.io/certification/ckad/
    * Candidate Handbook: https://www.cncf.io/certification/candidate-handbook
    * Exam Tips: https://docs.linuxfoundation.org/tc-docs/certification/tips-cka-and-ckad
* KodeKloud Discord Channel: https://discord.gg/VAfhT6ZR9E (`#ckad-sg`)

## Module 2: Core Concepts
### Recap Architecture
* Kubernetes Architecture
* Create and Configure Pods
* Nodes (Minions): 
    * A machine (physical or virtual) which Kubernetes is installed on.
    * A worker machine where containers are launched
* Cluster:
    * A set of nodes grouped together, so that if one node fails the application won’t go down.
    * Having multiple nodes helps in sharing load
* Master:
    * Another node with Kubernetes installed in it, and it’s configured as a Master node.
    * Watches over the other nodes in the cluster
    * Responsible for the orchestration of containers on the worker nodes.
* Kubernetes Components:
    * API Server
        * Acts as the frontend for Kubernetes
        * Users, management devices, command line interfaces all talk with API server to interact with Kubernetes cluster
    * etcd service
        * Key store
        * Distributed reliable key-value store used to store all data used to manage the cluster
        * Ie, when you have multiple nodes and multiple masters in your cluster, etcd stores all that information on all the nodes in a distributed manner
            * Responsible for implementing logs in the cluster to ensure there are no conflicts between the masters
    * Kubelet service
        * The agent that runs on each node in the cluster.
        * Responsible for making sure that the containers are running on the nodes as we expect.
    * Container Runtime
        * The underlying software used to run containers
        * In this case, we’re using Docker but there are other options as well.
        * Other instances might be RKT or CRI-O 
    * Controller
        * The "brain" behind the orchestration
        * Responsible for noticing and responding when nodes, containers, or endpoints go down.
        * Make decisions to bring up new containers instances.
    * Scheduler
        * Responsible to distributing work or containers across multiple nodes
        * Looks for newly created containers and assigns them to nodes.
* Master vs. Worker Nodes:
    * Worker node
        *  Where the containers are hosted, ex Docker Containers
        * To run docker containers on a system, we need a container runtime installed.
        * Has Container Runtime
        * Has Kubelet agent to provide health checks to master node and carry out actions requested by the master node.
    * Master node:
        * Has the Kubernetes API Server
        * Has etcd key-value store
        * Has the controller
        * Has the scheduler
* kubectl (CLI tool)
    * Used to deploy and manage applications on a Kubernetes cluster
    * It can be used: 
        * to get cluster information
        * To get the status of other nodes in the cluster
        * To manage "many other things"
    * `kubectl run` command is used to deploy and application on the cluster
    * `kubectl cluster info` command is used to view information about the cluster
    * `kubectl get nodes` command is used to list all nodes which are part of the cluster.

### Docker vs. Containerd
* Container Runtime Interface (CRI) 
    * Allowed any vendor to work as a container runtime for Kubernetes, as long as they adhered to the Open Container Initiative (OCI) Standards.
* Open Container Initiative (OCI)
    * Imagespec
        * Specifications to how an image should be built.
    * Runtimespec
        * Specifications to how a container runtime should be developed.
    * As long as these standards are met, anyone can build a container runtime that will work with Kubernetes.
* Because Docker was built before the CRI was introduced, it wasn’t built to support CRI standards.
* Kubernetes introduced dockershim to allow Docker to bypass the CRI standards and allow Docker to be used in Kubernetes.
* In v1.24, Kubernetes removed dockershim entirely and support for Docker was removed.
* Instead, the docker images relied on containerd to meet CRI standards and Docker was removed as a supported runtime in Kubernetes.
* Containerd
    * You can install containerd itself without having the install Docker, ie if you didn’t need all of Dockers other features.
    * `ctr` is the CLI tool to run containerd, solely made for debugging containerd as it’s not very user-friendly. Only supports limited features.
    * Ex. 
        * `ctr images pull docker.io/library/redis:alpine`
        * `ctr run docker.io/library/redis:alpine`
    * An alternative to ctr is `nerdctl`
        * Provides a docker-like CLI for Containerd
        * Supports docker compose
        * Supports newest features in containerd
            * Encrypted container images
            * Lazy pulling
            * P2P image distribution
            * Image signing and verification
            * Namespaces in Kubernetes
        * `nerdctl` used in place of `docker`, ie
            * `nerdctl run —name redis reds:alpine`
            * `nerdctl run —name webserver -p 80:80 -d nginx`
    * Another CLI tool is `crictl`
      * `crictl` provides a CLI for the CRI compatible container runtimes
      * `crictl` in installed separately
      * it's used to inspect and debug container runtimes
        * It ideally won't be used to create containers
      * The tool works across different runtimes, so long as they are OCI compatible.
      * ex
        * `crictl pull busybox`
        * `crictl images`
        * `crictl ps -a`
        * `crictl exec -it <container id> ls`
        * `crictl logs <container id>`
        * `crictl pods`
          * Unlike `docker` commands, `crictl` is aware of pods, which docker doesn't have available. 

  |                | `ctr`      | `nerdctl`       | `crictl`                     |
  |----------------|------------|-----------------|------------------------------|
  | **Purpose**    | Debugging  | General Purpose | Debugging                    |
  | **Community**  | ContainerD | ContainerD      | Kubernetes                   |
  | **Works With** | ContainerD | ContainerD      | All CRI Compatible runtimes. |

### Recap: Pods
* When discussing Pods, the assumption is that the Docker image has already been built and put on a Docker repo (ie Dockerhub) and the Kubernetes Cluster is already running.
* A pod is a single instance of an application.
* Pods are the smallest object in Kubernetes.
![pod_example.png](../images/pod_example.png)
* For scaling purposes, you create multiple pods in a node rather than multiple instances of an app inside the same pod.
* If the node cannot scale further, you would create a new node in the cluster with a new pod. 
* On the other hand, when scaling down, you would delete existing pods.
* A pod may have multiple containers inside it, but not the same container, 
  * ie 1 python image and 1 nginx image
  * Not 2 python images.
* How to deploy pods:
  * `kubectl run nginx --image nginx`
  * List pods: `kubectl get pods`

### Recap: Pods with YAML

`pod_definition.yaml`:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
    type: front-end
spec:
  containers:
    - name: nginx-container
      image: nginx
```
To deploy the pod:
```bash
kubectl apply -f pod_definition.yaml
```

To get detailed information about the pod:
```bash
kubectl describe pod myapp-pod
```

### Recap: Demo - Creating pods with YAML

### A Note on Editing Existing Pods
* If given a pod definition file, edit that file and use it to create a new pod.
* If you aren't give a pod definition file, you can extract the definition via:
```bash
kubectl get pod -o yaml > pod-definition.yaml
```
* Then, you can edit the file to make any changes, delete, and re-create the pod.
* To modify properties on the pod, you can use the following command:
```bash
kubectl edit pod 
```
* Only this list of properties is editable:
  * `spec.containers[*].image`
  * `spec.initContainers[*].image`
  * `spec.activeDeadlineSeconds`
  * `spec.tolerations`
  * `spec.terminationGracePeriodSeconds`

### Replication Controller
* Why do we need a Replication Controller?
  * High Availability
  * The Replication Controller lets us run multiple instances of a pod.
  * It can also bring up a new pod in a single pod node when the existing pod fails.
  * It ensures the specified number of pods are always running.
* Load Balancing and Scaling
  * This can be used to bring up more pods on a node, or additional pods across different nodes for scaling
  * The Replication Controller spans across multiple nodes in the cluster
  * Replication Controller vs. ReplicaSet
    * Replication Controller is the older technology that's being replaced by ReplicaSet

_rc-definition.yml:_
```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: myapp-rc
  labels:
    app: myapp
    type: front-end
spec:
  template:
    metadata:
      name: myapp-pod
      labels:
        app: myapp
        type: front-end
    spec:
      containers:
      - name: nginx-container
        image: nginx
  replicas: 3
```
```bash
kubectl create -f rc-definition.yml
```

```bash
kubectl get replicationcontroller
```
```bash
kubectl get pods
```

_replicaset-definition.yml:_

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-replicaset
  labels:
    app: myapp
    type: front-end
spec:
  template:
    metadata:
      name: myapp-pod
      labels:
        app: myapp
        type: front-end
    spec:
      containers:
        - name: nginx-container
          image: nginx
  replicas: 3
  selector:
    matchLabels:
      type: front-end
```
```bash
kubectl create -f replicaset-definition.yml
```
```bash
kubectl get replicaset
```
```bash
kubectl get pods
```

* If we want to scale up to 6 replicas, we change to `replicas: 6` and then:
```bash
kubectl create -f replicaset-definition.yml
```
```bash
kubectl scale --replicas=6 -f replicaset-definition.yml
```
(I think this last one wouldn't affect the yaml, and thus probably not ideal)
```bash
kubectl scale --replicas=6 replicaset myapp-replicaset
```
(This deletes all underlying pods as well)
```bash
kubectl delete replicaset myapp-replicaset
```

```bash
kubectl replace -f replicaset-definition.yml
```































