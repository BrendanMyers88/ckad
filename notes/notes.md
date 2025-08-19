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
* 





































