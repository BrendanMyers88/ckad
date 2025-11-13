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
        * Users, management devices, command line interfaces all talk with API server to interact with Kubernetes
          cluster
    * etcd service
        * Key store
        * Distributed reliable key-value store used to store all data used to manage the cluster
        * Ie, when you have multiple nodes and multiple masters in your cluster, etcd stores all that information on all
          the nodes in a distributed manner
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
        * Where the containers are hosted, ex Docker Containers
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
    * Allowed any vendor to work as a container runtime for Kubernetes, as long as they adhered to the Open Container
      Initiative (OCI) Standards.
* Open Container Initiative (OCI)
    * Imagespec
        * Specifications to how an image should be built.
    * Runtimespec
        * Specifications to how a container runtime should be developed.
    * As long as these standards are met, anyone can build a container runtime that will work with Kubernetes.
* Because Docker was built before the CRI was introduced, it wasn’t built to support CRI standards.
* Kubernetes introduced dockershim to allow Docker to bypass the CRI standards and allow Docker to be used in
  Kubernetes.
* In v1.24, Kubernetes removed dockershim entirely and support for Docker was removed.
* Instead, the docker images relied on containerd to meet CRI standards and Docker was removed as a supported runtime in
  Kubernetes.
* Containerd
    * You can install containerd itself without having the install Docker, ie if you didn’t need all of Dockers other
      features.
    * `ctr` is the CLI tool to run containerd, solely made for debugging containerd as it’s not very user-friendly. Only
      supports limited features.
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

* When discussing Pods, the assumption is that the Docker image has already been built and put on a Docker repo (ie
  Dockerhub) and the Kubernetes Cluster is already running.
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

### Deployment

_deployment-definition.yml:_

```yaml
apiVersion: apps/v1
kind: Deployment
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
kubectl create -f deployment-definition.yaml
```

```bash
kubectl get deployments
```

To see all created objects at once, use the following command:

```bash
kubectl get all
```

### Namespaces

To see all the pods in a non-default namespace

```bash
kubectl get pods --namespace=kube-system
```

To create a pod in a separate namespace

```bash
kubectl create -f definition.yaml --namespace=dev
```

Alternately, you can add the namespace into the `definition.yaml` file under the `metadata.namespace` attribute

To create a Namespace

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev
```

```bash
kubectl create -f namespace-def.yaml
```

To change the default namespace to dev:

```bash
kubectl config set-context $(kubectl config current-context) --namespace=dev
```

To show the contents of all namespaces:

```bash
kubectl get pods --all-namespaces
```

To limit resources in a namespace, a ResourceQuota must be made:

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: dev
spec:
  hard:
    pods: "10"
    requests.cpu: "4"
    requests.memory: 5Gi
    limits.cpu: "10"
    limits.memory: 5Gi
```

### Services

Services help us connect applications together with other applications or users

* NodePort service can act similarly to a reverse proxy in that you can access an IP on a separate network, ie external
  accessibility.
    * There are 3 ports involved
        1. Target Port: 80, the port where the webserver is running and where the service forwards the request to.
        2. Port: 80, the port on the service itself, think host port in docker for the Service Object
        3. NodePort: 30000-32767, The port which we use to access the service externally

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: NodePort
  ports:
    - targetPort: 80
      port: 80
      nodePort: 30008
  selector:
    app: myapp
    type: front-end
```

port is the only required field under ports. the `spec.ports` is also an array, so multiple NodePorts can be listed here

```bash
kubectl create -f service-def.yaml
```

```bash
kubectl get services
```

* ClusterIP service creates a virtual IP inside the cluster to enable communication between different services, such as
  between frontend and backend servers

```yaml
apiVersion: v1
kind: Service
metadata:
  name: back-end
spec:
  type: ClusterIP
  ports:
    - targetPort: 80
      port: 80
  selector:
    app: myapp
    type: back-end
```

* LoadBalancer service provisions a load balancer for the application in supported cloud providers, ex to distribute
  load across different web servers in the frontend tier.

Create a Service named redis-service of type ClusterIP to expose pod redis on port 6379:

```bash
kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml
or
kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml
```

```bash
kubectl expose pod nginx --port=80 --name nginx-service --type=NodePort --dry-run=client -o -yaml
or
kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o -yaml
```

The `-o` command has many different human-readable formats

1. `-o json` Output a json formatted API object

```bash
master $ kubectl create namespace test-123 --dry-run -o json
{
    "kind": "Namespace",
    "apiVersion": "v1",
    "metadata": {
        "name": "test-123",
        "creationTimestamp": null
    },
    "spec": {},
    "status": {}
}
```

2. `-o name` Print only the resource name and nothing else

```bash
 
```

3. `-o wide` Output in plain-text format w/ additional info

```bash
master $ kubectl get pods -o wide
NAME      READY   STATUS    RESTARTS   AGE     IP          NODE     NOMINATED NODE   READINESS GATES
busybox   1/1     Running   0          3m39s   10.36.0.2   node01          <​none​>         <​none​>  
ningx     1/1     Running   0          7m32s   10.44.0.1   node03          <​none​>         <​none​> 
redis     1/1     Running   0          3m59s   10.36.0.1   node01          <​none​>         <​none​> 
 
```

4. `-o yaml` Output in a yaml formatted API object

```bash
master $ kubectl create namespace test-123 --dry-run -o yaml
apiVersion: v1
kind: Namespace
metadata:
  creationTimestamp: null
  name: test-123
spec: {}
status: {}
```

For additional details: https://kubernetes.io/docs/reference/kubectl/

### kubectl explain command

To list all resources:

#### Command:

```bash
kubectl api-resources
```

#### Output:

```bash
NAME                      SHORTNAMES    APIVERSION     NAMESPACED   KIND
bindings                                v1             true         Binding
componentstatuses         cs            v1             false        ComponentStatus
configmaps                cm            v1             true         ConfigMap
endpoints                 ep            v1             true         Endpoints
events                    ev            v1             true         Event
limitranges               limits        v1             true         LimitRange
namespaces                ns            v1             false        Namespace
nodes                     no            v1             false        Node
persistentvolumeclaims    pvc           v1             true         PersistentVolumeClaim
persistentvolumes         pv            v1             false        PersistentVolume
pods                      po            v1             true         Pod
podtemplates                            v1             true         PodTemplate
replicationcontrollers    rc            v1             true         ReplicationController
resourcequotas            quota         v1             true         ResourceQuota
secrets                                 v1             true         Secret
serviceaccounts           sa            v1             true         ServiceAccount
services                  svc           v1             true         Service
```

#### Command:

```bash
kubectl explain pods
```

#### Output:

```bash
KIND:       Pod
VERSION:    v1

DESCRIPTION:
    Pod is a collection of containers that can run on a host. This resource is
    created by clients and scheduled onto hosts.

FIELDS:
  apiVersion    <string>
    APIVersion defines the versioned schema of this representation of an object.
    Servers should convert recognized schemas to the latest internal value, and
    may reject unrecognized values. More info:
    https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources

  kind  <string>
    Kind is a string value representing the REST resource this object
    represents. Servers may infer this from the endpoint the client submits
    requests to. Cannot be updated. In CamelCase. More info:
    https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds

  metadata      <ObjectMeta>
    Standard object\'s metadata. More info:
    https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata

  spec  <PodSpec>
    Specification of the desired behavior of the pod. More info:
    https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status

  status        <PodStatus>
    Most recently observed status of the pod. This data may not be up to date.
    Populated by the system. Read-only. More info:
    https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status
```

To go deeper into a specific field (This is not comprehensive w/ all specific fields):

#### Command:

```bash
kubectl explain pods.spec
```

#### Output:

```bash
KIND:       Pod
VERSION:    v1

FIELD: spec <PodSpec>


DESCRIPTION:
    Specification of the desired behavior of the pod. More info:
    https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status
    PodSpec is a description of a pod.

FIELDS:
  activeDeadlineSeconds <integer>
    Optional duration in seconds the pod may be active on the node relative to
    StartTime before the system will actively try to mark it failed and kill
    associated containers. Value must be a positive integer.

  affinity      <Affinity>
    If specified, the pod\'s scheduling constraints

  automountServiceAccountToken  <boolean>
    AutomountServiceAccountToken indicates whether a service account token
    should be automatically mounted.
 
  etc...
```

To output ALL fields (Comprehensive list):

#### Command:

```bash
kubectl explain pods --recursive
```

#### Output:

```bash
KIND:       Pod
VERSION:    v1

DESCRIPTION:
    Pod is a collection of containers that can run on a host. This resource is
    created by clients and scheduled onto hosts.

FIELDS:
  apiVersion    <string>
  kind  <string>
  metadata      <ObjectMeta>
    annotations <map[string]string>
    creationTimestamp   <string>
    deletionGracePeriodSeconds  <integer>
    deletionTimestamp   <string>
    finalizers  <[]string>
    generateName        <string>
    generation  <integer>
    labels      <map[string]string>
    managedFields       <[]ManagedFieldsEntry>
      apiVersion        <string>
      fieldsType        <string>
      fieldsV1  <FieldsV1>
      manager   <string>
      operation <string>
      subresource       <string>
      time      <string>
    name        <string>
    namespace   <string>
    ownerReferences     <[]OwnerReference>
      apiVersion        <string> -required-
      blockOwnerDeletion        <boolean>
      controller        <boolean>
      kind      <string> -required-
      name      <string> -required-
      uid       <string> -required-
    resourceVersion     <string>
    selfLink    <string>
    uid <string>
  
  etc...
```

Create a namespace called 'mynamespace' and a pod with image nginx called nginx on this namespace:

```bash
kubectl create namespace mynamespace
kubectl run nginx --image=nginx --namespace=mynamespace
```

Write out the yaml format of the previous pod

```bash
kubectl run nginx --image=nginx --namespace=mynamespace --dry-run=client -o yaml > nginx-def.yaml
```

```bash
cat nginx-def.yaml
```

```bash
kubectl create -f nginx-def.yaml
```

Create a busybox pod (using kubectl command) that runs the command "env". Run it and see the output

```bash
kubectl run busybox --image=busybox --command --restart=Never --rm env
```

```bash
kubectl run busybox --image=busybox --command --restart=Never env -o yaml > busybox.yaml
kubectl create -f busybox.yaml
```

Get the YAML for a new namespace called 'myns' _without_ creating it

```bash
kubectl create namespace myns --dry-run=client -o yaml
```

## Configuration

### Define, build, and modify container images

All Dockerfiles must begin with the FROM command for an operating system or an image built off an OS.
Dockerfiles follow an Instruction Argument format

```bash
docker build Dockerfile -t username/appname
```

```aiignore
INSTRUCTION ARGUMENT
```

```Dockerfile
FROM Ubuntu

RUN apt-get update
RUN apt-get install python

RUN pip install flask
RUN pip install flask-mysql

COPY . /opt/source-code

ENTRYPOINT FLASK_APP=/opt/source-code/app.py flask run
```

```Dockerfile
FROM Ubuntu

ENTRYPOINT ["sleep"]
CMD ["5"]
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper-pod
spec:
  containers:
    - name: ubuntu-sleeper
      image: ubuntu-sleeper
      command: [ "sleep2.0" ]
      args: [ "10" ]
```

`ENTRYPOINT` (Dockerfile) corresponds to `command` (container)
`CMD` (Dockerfile) corresponds to `args` (container)

### ENV Variables in Kubernetes

Environment variables are a key/value pair under the container. It can be called via -e on docker run.

```bash
docker run -e APP_COLOR=pink simple-webapp-color
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
spec:
  containers:
    - name: simple-webapp-color
      image: simple-webapp-color
      ports:
        - containerPort: 8080
      env:
        - name: APP_COLOR
          value: pink
```

Environment Variables can be plain Key/Value pairs, a ConfigMap, or Secrets

#### Plain Key Value:

```yaml
  env:
    - name: APP_COLOR
      value: pink
```

#### ConfigMap:

```yaml
  env:
    - name: APP_COLOR
      valueFrom:
        configMapKeyRef:
```

#### Secrets:

```yaml
  env:
    - name: APP_COLOR
      valueFrom:
        secretKeyRef:
```

### ConfigMaps

* Rather than defining the key-value pairs in the pod definition, we can store the key-value pair in a ConfigMap and
  inject it into the pod

1. Create the ConfigMap
2. Inject the ConfigMap into the Pod

```yaml
  envFrom:
    - configMapRef:
      name: app-config
```

ConfigMaps can be created in two ways, imperative or declarative using a definition file.

#### Imperative ConfigMap

```bash
kubectl create configmap

kubectl create configmap <config-name> --from-literal=<key>=<value>

kubectl create configmap app-config --from-literal=APP_COLOR=blue --from-literal=APP_MOD=prod

kubectl create configmap <config-name> --from-file=<path-to-file>

kubectl create configmap app-config --from-file=app_config.properties
```

#### Declarative ConfigMap

```bash
kubectl create -f
```

#### config-map.yaml

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_COLOR: blue
  APP_MODE: prod
```

```bash
kubectl create -f config-map.yaml
```

### Creating ConfigMaps

#### app-config

```yaml
APP_COLOR: blue
APP_MODE: prod
```

#### mysql-config

```yaml
port: 3306
max_allowed_packet: 128M
```

#### redis-config

```yaml
port: 6379
rdb-compression: yes
```

### Viewing ConfigMaps

```bash
kubectl get configmaps
kubectl describe configmaps
```

### config-map.yaml

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_COLOR: blue
  APP_MODE: prod
```

### pod-definition.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
spec:
  containers:
    - name: simple-webapp-color
      image: simple-webapp-color
      ports:
        containerPort: 8080
      envFrom:
        - configMapRef:
            name: app-config
```

### Create pod

```bash
kubectl create -f pod-definition.yaml
```

#### ConfigMaps in Pods

### ENV

```yaml
envFrom:
  - configMapRef:
      name: app-config
```

### Single ENV

```yaml
env:
  - name: APP_COLOR
    valueFrom:
      configMapKeyRef:
        name: app-config
        key: APP_COLOR
```

### Volume

```yaml
volumes:
  - name: app-config-volume
    configMap:
      name: app-config
```
