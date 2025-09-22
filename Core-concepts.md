### I.	Cluster Architecture
 ![cluster_arch](/images/cluster_architecture.png)
- Controller-Manager:
  - Controller-Manager
  - Node-Controller
  - Replication-controller
- The Kube API server is responsible for orchestrating all operations within the cluster. It exposes the Kubernetes API, which is used by external users to perform management operations on the cluster, as well as the various controllers to monitor the state of the cluster necessary changes as required, and by the worker nodes to communicate with the server.
- The container runtime engine
- In Kubernetes, a Kubelet is an agent that runs on each node in a cluster. It listens for instructions from the Kube API server and deploys or destroys containers on the nodes as required. 
- The kube proxy service ensures that the necessary rules are in place on the worker nodes to allow the containers running on them to reach each other. 
 
 --> Kube Scheduler that is responsible for scheduling applications or containers on nodes, we have different controllers that take care of different functions like the node controller, replication controller, .. 

### II.	Docker vs containerD
![alt text](/images/containerD.png)
- Container Runtime Interface (CRI) allowed any vendor to work as a container runtime for Kubernetes as long as they adhere the OCI standards.
- OCI: Open Container Initiative consists of an image spec and a runtime spec.
  - Image spec: the specifications on how an image should be built.
  - Runtime spec defines the standards on how any Container runtime should be developed.
- ContainerD is CRI compatible and can work directly with K8s as all other runtimes
- From version 1.24: K8s remove support for Docker.
- Docker followed the image spec from the OCI standards so these image still can be used.
#### 1.1. ContainerD
- CLI-ctr: used for debugging containerD
- CLI-nerdctl:
  - Provides a Docker-like CLI for containerD
  - Support docker compose
  - Support newest features in contianerd
- CLI-crictl:
  - Provide a CLI for CRI compatible container runtimes
  - Installed separately
  - Used to inspect and debug container runtimes
  - Works across different runtimes

### 1. Kube-apiserver

![alt text](/images/kube-apiserver.png)

- Kube-apiserver is responsible:
  - AUthenticate User
  - Validate Request
  - retrieve data
  - update etcd
  - scheduler
  - kubelet

### 2. Kube-controller-manager
- The controller is a process that continuously monitors the state of various components within the system and works towards bringing the whole system to the desired functioning state. Example:
  - node-controller: monitoring the status of the nodes and take necessary actions to keep the app running. It does it through the kube-apiserver.

  ![node-controller](/images/node-controller.png)

- All controller are packed in the Kube-controller-manager

### 3. Kube-scheduler
- The component only decide which pod goes on which node. It doesn't actually place the pod on the nodes.That's the job of the Kubelet.

### 4. Kubelet
- Registrer Node
- Create Pods
- Monitor Node and Pods

### 5. Kube-proxy
- All pod in cluster can communicate to each others.
-  A pod network is an internal virtual network that spans across all the nodes in the cluster to which all the pods connect to.

- The service can not join the pod network because the service is not an actual thing. It can connect to other pod through Kube-proxy

- Kube-proxy is a process that runs on each node in the Kubernetes cluster.Its job is to look for new services, and every time a new service is created it creates the appropriate rules on each node to forward traffic to those services, to the bvackend pods. One way it does this is using IP tables rules. 

![kube-proxy](/images/kube-proxy.png)


### 6. YAML
- A kubernetes definition file always contains four top level fields. These are the top level or root level properties. 
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
  labels:
    app: myapp 
    type: front-end

spec:
  containers: ---- List/array
    - name: nginx-container
      image: nginx
```


- apiVersion: is the version of the Kubernetes API.


| Kind | Version |
| ---- | -------- |
| Pod | v1 |
| Service | v1 |
|ReplicaSet | apps/v1 |
| Deploymnet | apps/v1 |

- Kind: the kind refers to the type of object we are trying to create.
- Metadata: is data above the object like its name, labels, ... (Dictionary)
- Spec: we provide additional information to Kubernetes pertaining to that object. Spec is a dictionary. 

### 7. Replication Controller
- Help to run multiple instances of a single pod in Kubernetes cluster, thus providing high availability.
- Use to create multiple pods to share the load across them.
- Replication Controller is old technology that is being replaced by replica set.
- ReplicaSet can also manage pods that were not created as part of the replica set creation.

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-replicaset
  labels:
    appp: myapp
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
      - name: nginx
        image: nginx
  replicas: 3
  selector: (help the replica set identify what pods fall under it)
    matchLabels:
      type: front-end 
```

#### Labels and selectors
- Labels as the filter for the replicaset know which pods need be monitored.
- Under the selector section, we use the match labels filter and provide the same label that wee used while creating the pods.
--> This way, the replicaset knows which pods to monitor

- Scale:
  - Method 1: update replicas in the replicaset file, then use command `kubectl create -f <replicaset.yml>`
  - Method 2: Use command `kubectl scale --replicas=6 -f <replicaset.yml>`


![replicaset_command](/images/replicaset_command.png)



### 8. Deployments

![deployment](/images/depoyment.png)


- The deployment provides us with the capability to upgrade the underlying instances seamlessly using rolling updates, undo changes, pause and resume changes as required.

- File definition deployment is similar to ReplicasSet, change the kind to Deployment.
- When apply file deployment, new K8s object is created

- Some Tips:
  - Generate deployment YAML file. Don't create it and save it to file: `kubectl create deployment --image=nginx nginx --dry-run=client -o yaml > nginx-deployment.yaml`

### 9. Services

