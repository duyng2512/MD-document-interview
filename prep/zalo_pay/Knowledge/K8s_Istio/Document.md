# Kubernets

## 1. Overview

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-19-14-59-47-image.png)

- **kube-apiserver**: expose k8s api, can be scale horizontally.

- **etcd**: Consistent and highly-available key value store used as Kubernetes' backing store for all cluster data.

- **kube-scheduler**: checking newly created or terminated pod.

- **kube-controller-manager**



**<u>Node Components </u>**:

- kubelet: An agent that runs on each node in the cluster. It makes sure that containers are running in a Pod.

- kube-proxy: manage service and networking



**<u>The Kubernetes API</u>**:

The core of Kubernetes' control plane is the API server. The API server exposes an HTTP API that lets end users, different parts of your cluster, and external components communicate with one another.

The Kubernetes API lets you query and manipulate the state of API objects in Kubernetes (for example: Pods, Namespaces, ConfigMaps, and Events).

**<u>Recommend labels:</u>**

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-19-15-44-55-image.png)

## 2. Cluster architecture

**<u>Nodes:</u>**

Kubernetes runs your workload by placing containers into Pods to run on *Nodes*. A node may be a virtual or physical machine, depending on the cluster. 

Each node is managed by the [control plane](https://kubernetes.io/docs/reference/glossary/?all=true#term-control-plane) and contains the services necessary to run [Pods](https://kubernetes.io/docs/concepts/workloads/pods/).

The components on a node include: 

- the kubelet

- a container runtime

- the kube-proxy.

Node info:

- HostName

- External IP: IP address of the node that is externally routable (available from outside the cluster).

- Internal IP: IP address of the node that is routable only within the cluster.





**<u>Pods:</u>**

*Pods* are the smallest deployable units of computing that you can create and manage in Kubernetes.



A *Pod* (as in a pod of whales or pea pod) is a group of one or more [containers](https://kubernetes.io/docs/concepts/containers/), with shared storage and network resources, and a specification for how to run the containers.



Within a Pod, containers share an IP address and port space, and can find each other via `localhost`![Pod creation diagram](https://d33wubrfki0l68.cloudfront.net/aecab1f649bc640ebef1f05581bfcc91a48038c4/728d6/images/docs/pod.svg)



**<u>Deployments</u>**:

A *Deployment* provides declarative updates for [Pods](https://kubernetes.io/docs/concepts/workloads/pods/) and [ReplicaSets](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/).

- Replicate set: group of pods, support self healing, scalable, desired state

- Deployment: support update and roll backs



Updating a Deployment:

```bash
kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1

kubectl rollout status deployment/nginx-deploymentWaiting for rollout to finish: 2 out of 3 new replicas have been updated...



```



Rolling Back a Deployment:

```bash
kubectl rollout undo deployment/nginx-deployment

```

Scaling a Deployment:

```bash
kubectl scale deployment/nginx-deployment --replicas=10

```

**<u>Stateful sets</u>**:

StatefulSet is the workload API object used to manage stateful applications.



Manages the deployment and scaling of a set of [Pods](https://kubernetes.io/docs/concepts/workloads/pods/), *and provides guarantees about the ordering and uniqueness* of these Pods.



Pod Identity:

The domain managed by this Service takes the form: `$(service name).$(namespace).svc.cluster.local`, where "cluster.local" is the cluster domain.



**<u>Service:</u>**

- `ClusterIP`: Exposes the Service on a cluster-internal IP. Choosing this value makes the Service only reachable from within the cluster.

- [`NodePort`](https://kubernetes.io/docs/concepts/services-networking/service/#type-nodeport): Exposes the Service on each Node's IP at a static port (the `NodePort`). A `ClusterIP` Service, to which the `NodePort` Service routes, is automatically created. You'll be able to contact the `NodePort` Service, from outside the cluster, by requesting `<NodeIP>:<NodePort>`.

- [`LoadBalancer`](https://kubernetes.io/docs/concepts/services-networking/service/#loadbalancer): Exposes the Service externally using a cloud provider's load balancer. `NodePort` and `ClusterIP` Services, to which the external load balancer routes, are automatically created.



**<u>Ingress:</u>**

An API object that manages external access to the services in a cluster, typically HTTP.

Ingress may provide load balancing, SSL termination and name-based virtual hosting



[Ingress](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.24/#ingress-v1-networking-k8s-io) exposes HTTP and HTTPS routes from outside the cluster to [services](https://kubernetes.io/docs/concepts/services-networking/service/) within the cluster. Traffic routing is controlled by rules defined on the Ingress resource.



![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-19-20-04-19-image.png)

An Ingress does not expose arbitrary ports or protocols. Exposing services other than HTTP and HTTPS to the internet typically uses a service of type [Service.Type=NodePort](https://kubernetes.io/docs/concepts/services-networking/service/#type-nodeport) or [Service.Type=LoadBalancer](https://kubernetes.io/docs/concepts/services-networking/service/#loadbalancer).



Ingress rules:

- A list of paths (for example, `/testpath`), each of which has an associated backend defined with a `service.name` and a `service.port.name` or `service.port.number`. Both the host and path must match the content of an incoming request before the load balancer directs traffic to the referenced Service.



![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-19-20-17-02-image.png)

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-19-20-17-21-image.png)





**<u>Volumes:</u>**

- **Persistent Volumes**: Kubernetes does not destroy persistent volumes. For any kind of volume in a given pod, data is preserved across container restarts.

- **Ephemeral volume**: Ephemeral volume types have a lifetime of a pod, but persistent volumes exist beyond the lifetime of a pod.





## 3. Istio

**<u>Architect:</u>**

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-19-20-46-31-image.png)

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-19-20-46-44-image.png)

**<u>Metric collection:</u>**

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-19-21-33-06-image.png)



![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-19-21-48-13-image.png)

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-19-21-48-27-image.png)


