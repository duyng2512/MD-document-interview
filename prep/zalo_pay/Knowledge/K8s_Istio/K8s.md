# Kubernetes

## 1. Kubernetes in action

### 1.1 Introduction

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-16-10-11-28-image.png)

**<u>SCALING MICROSERVICES</u>**

Scaling micro service can be individually scaling each one

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-16-10-16-34-image.png)

**<u>UNDERSTANDING THE DIVERGENCE OF ENVIRONMENT REQUIREMENTS</u>**

- Microservice can be individually have different dependency with each one.

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-16-10-21-54-image.png)

- Each package can develop without afraid of dependency confliction.

Providing a consistent environment to applications:

Another unavoidable fact is that the environment of a single production machine will change over time.

**<u>How container is isolated</u>**

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-16-10-59-02-image.png)

Containers, on the other hand, all perform system calls on the exact same kernel running in the host OS. This single kernel is the only one performing x86 instructions on  
the host’s CPU. The CPU doesn’t need to do any kind of virtualization the way it does  
with VMs.

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-16-11-01-48-image.png)

Remember, each VM runs its own set of system services, while containers don’t, because they all run in the same OS.

How container can isolated their service:

- The first one, **<u>Linux Namespaces</u>**, makes sure each process sees its own personal view of the system (files, processes, network interfaces, hostname, and so on).

- The second one is **<u>Linux Control Groups (cgroups)</u>**, which limit the amount of resources the process can consume (CPU, memory, network bandwidth, and so on).

**ISOLATING PROCESSES WITH LINUX NAMESPACES**

- By default, each Linux system initially has one single namespace. All system resources, such as filesystems, process IDs, user IDs, network interfaces, and others, belong to the single namespace.

- A big difference between Docker-based container images and VM images is that container images are composed of layers, which can be shared and reused across multiple images. 

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-16-11-45-06-image.png)

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-16-11-48-34-image.png)

K8s recovery:

- If one of those instances stops working properly, like when its process crashes or when it stops responding, Kubernetes will restart it automatically.

- If whole node is die, Kubernetes will select new nodes for all the containers that were running on the node and run them on the newly selected nodes.

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-16-11-50-57-image.png)

How dynamic IP can be achieve:

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-16-11-52-11-image.png)

K8s advantage:

- Better deployment

- Better machine utilization

- Health check, self healing

### 1.2 Pods

**<u>Pods in Kubernetes:</u>**

- When pods have multiple containers, all of them have to be in one node

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-16-14-18-22-image.png)

When running pod on the same machine:

- When two process require each other to run

When running pod on different machine:

- When two process don't depend on each other

Deploy pod on specific node machine.

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-16-16-47-46-image.png)

Annotation contain additional information about pod:

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-16-16-49-37-image.png)

Name space: grouping pods together

Create object within a namespace:

```bash
$ kubectl create -f kubia-manual.yaml -n custom-namespace
pod "kubia-manual" created
```

### 1.3 Replication and other controllers: deploying managed pods

Keeping pods healthy:

- Liveness probes: 
  
  - Caling GET api to <ip>:<port> of container
  
  - Try to open TCP socket
  
  - Run Exec command

Define liveness probe in container:

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-16-17-02-31-image.png)

If liveness probe return 200, pod is okay, 500 is dead

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-16-17-06-37-image.png)

```bash
Liveness: http-get http://:8080/ delay=0s timeout=1s period=10s #success=1 #failure=3
```

**<u>ReplicationControllers</u>**

- Pod A is create manually, hence no Replication Controller handle it. When node 1 fail, it does not automatically create in Node 2

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-16-17-12-58-image.png)

- Replica controller using matching selector to create perfect pod number:

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-16-17-28-39-image.png)

- Replica controller comprise of:
  
  - Label selector
  
  - Replica count
  
  - Pod definition

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-16-17-33-01-image.png)

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-16-17-37-43-image.png)

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-16-17-55-28-image.png)

### 1.4 Running exactly one pod on each node with DaemonSets

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-16-18-05-50-image.png)

Running daemon set in certain port:

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-16-18-07-38-image.png)

### 1.5 Services: enabling clients to discover and talk to pods

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-16-18-13-50-image.png)

Define service:

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-16-18-14-25-image.png)

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-16-18-26-25-image.png)

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-16-18-26-40-image.png)

**<u>Discovering services</u>**

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-16-18-50-17-image.png)

**<u>Exposing services to external clients</u>**

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-16-20-33-25-image.png)

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-16-20-43-29-image.png)

- Using a NodePort service

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-16-20-44-38-image.png)

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-16-20-49-17-image.png)

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-16-20-50-45-image.png)

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-16-20-54-06-image.png)

Using external load balancer:

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-16-20-55-11-image.png)

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-16-20-57-31-image.png)

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-16-20-56-14-image.png)

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-16-20-57-09-image.png)

**<u>Exposing services externally through an Ingress resource</u>**

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-16-21-04-29-image.png)

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-16-21-12-17-image.png)

Multiple service in ingress:

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-16-21-13-05-image.png)

### 1.6 Signaling when a pod is ready to accept connections

**<u>Readiness probes</u>**

The readiness probe is invoked periodically and determines whether the specific pod should receive client requests or not.

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-16-21-17-22-image.png)

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-16-21-16-50-image.png)

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-16-21-22-51-image.png)

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-16-21-23-13-image.png)

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-16-21-24-28-image.png)

### 1.7 ConfigMaps and Secrets: configuring applications

#### 1.7.1 ConfigMap

```yaml
kind: ConfigMap
apiVersion: v1
metadata:
  name: example-configmap
  namespace: default
data:
  # Configuration Values are stored as key-value pairs
  system.data.name: "app-name"
  system.data.url: "https://app-name.com"
  system.data.type_one: "app-type-xxx"
  system.data.value: "3"
  # File like Keys
  system.interface.properties: |
    ui.type=2
    ui.color1=red
    ui.color2=green
```

Create and reference to config Map from pod

```yaml
kubectl create configmap app-basic-configmap  
--from-file=configmap-example/app-basic.properties
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configmap-example-pod
spec:
  containers:
    - name: configmap-example-busybox
      image: k8s.gcr.io/busybox
      command: [ "/bin/sh", "-c", "env" ]
      envFrom:
        # Load the Complete ConfigMap
        - configMapRef:
            name: app-basic-configmap
  restartPolicy: Never
```

#### 1.7.2 Secret

Define secret:

```yaml
# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file, it will be
# reopened with the relevant failures.
#
apiVersion: v1
data:
  username: YWRtaW4=
  password: MWYyZDFlMmU2N2Rm
kind: Secret
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: { ... }
  creationTimestamp: 2020-01-22T18:41:56Z
  name: mysecret
  namespace: default
  resourceVersion: "164619"
  uid: cfee02d6-c137-11e5-8d73-42010af00002
type: Opaque
```

Using secret:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: redis
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
      readOnly: true
  volumes:
  - name: foo
    secret:
      secretName: mysecret
      optional: false # default setting; "mysecret" must exist
```

### 1.8 Deployments: updating applications declaratively

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-16-22-16-51-image.png)

Spinning up new pods and then deleting the old ones:

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-16-22-20-43-image.png)

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-16-22-21-08-image.png)

```bash
kubectl rolling-update kubia-v1 kubia-v2 --image=luksa/kubia:v2
```

### 1.9 StatefulSets: deploying replicated stateful applications

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-16-22-30-10-image.png)

**<u>Running multiple replicas with separate storage for each</u>**

Instead of using a ReplicaSet to run these types of pods, you create a StatefulSet resource, which is specifically tailored to applications where instances of the application must be treated as non-fungible individuals, with each one having a stable name and state.

## 2. Mastering Kubernetes

**<u>Reliability K8s best practice</u>**

![HA-Kubernetes-Cluster-Setup](https://www.linuxtechi.com/wp-content/uploads/2020/07/HA-Kubernetes-Cluster-Setup-1024x567.png?ezimgfmt=rs:723x400/rscb22/ng:webp/ngcb22)

- Step 1: set host name /etc/hosts

- Step 2: install HA proxy and keep alive in all master
  
  - In keep alive configuration, state in worker is SLAVE

- Step 3: Disable SWAP & config firewall

- Step 4: Install Container Run Time (CRI) Docker on Master & Worker Nodes

- Step 5: Install kubeadm kubelet and kubectl

- Step 6: Initialize the Kubernetes Cluster from first master node
  
  - sudo kubeadm init --control-plane-endpoint "vip-k8s-master:8443" --upload-certs

- Step 7: Join Worker nodes to Kubernetes cluster

## 3. Official Kubernetes documentation

### 3.1 Stateful set

- It is important not to configure other applications to connect to Pods in a StatefulSet by IP address.

- Scaling stateful set:
  
  - ```shell
    kubectl scale sts web --replicas=5
    ```

- Update stateful set:
  
  - spec.updateStrategy:
    
    - **RollingUpdate**: the StatefulSet controller will delete and recreate each Pod in the StatefulSet. It will proceed in the same order as Pod termination (from the largest ordinal to the smallest), updating each Pod one at a time. It will wait until an updated Pod is Running and Ready prior to updating its predecessor.
    
    - **OnDelete**: the StatefulSet controller will not automatically update the Pods in a StatefulSet. Users must manually delete Pods to cause the controller to create new Pods that reflect modifications made to a StatefulSet's

### 3.2 Nginx ingress controller

![ingress nginx controller](https://raw.githubusercontent.com/xuanthulabnet/learn-kubernetes/master/imgs/kubernetes057.png)
