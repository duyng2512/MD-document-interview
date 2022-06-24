



# Microservice

## 1. Introduction to micro service

- Central log and alarm

- Central configuration

- Service discovery

- Edge server 

- Reactive microservice

- Distributed tracing

- Circuit breaker



**<u>Service discovery</u>**

- Problems: How can clients find microservices and their instances? Microservices instances are typically assigned dynamically allocated IP addresses when they start up, for example, when running in containers.



<img title="" src="file:///C:/Users/duyng/AppData/Roaming/marktext/images/2022-06-15-02-13-39-image.png" alt="" width="357" data-align="center">

- Solution: Add a new component – a service discovery service – to the system landscape, which keeps track of currently available microservices and the IP addresses of its instances.
  
  - Client side routing: client ask ip
  
  - Server side routing: using reverse proxy



**<u>Edge server</u>**

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-15-02-15-38-image.png)

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-15-02-15-57-image.png)



**<u>Reactive microservices</u>**

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-15-02-16-39-image.png)

**<u>Central configuration</u>**

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-15-02-17-01-image.png)

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-15-02-17-11-image.png)



**<u>Centralized log analysis</u>**

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-15-02-18-20-image.png)

**<u>Distributed tracing</u>**

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-15-02-19-14-image.png)

**<u>Circuit breaker</u>**

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-15-02-20-25-image.png)

**Centralized monitoring and alarms**





## 2. Deploy our microservices on Docker

Using Docker with one microservice:

- To build a Docker image, we need a Dockerfile, so we will start with that.



```docker
FROM openjdk:16
EXPOSE 8080
ADD ./build/libs/*.jar app.jar
ENTRYPOINT ["java","-jar","/app.jar"]
```



## 3. Developing event-driven asynchronous services

Retries and dead-letter queues:

```java
spring.cloud.stream.bindings.messageProcessor-in-0.consumer:
maxAttempts: 3
backOffInitialInterval: 500
backOffMaxInterval: 1000
backOffMultiplier: 2.0
spring.cloud.stream.rabbit.bindings.messageProcessor-in-0.consumer:
autoBindDlq: true
republishToDlq: true
spring.cloud.stream.kafka.bindings.messageProcessor-in-0.consumer:
enableDlq: true
```

## 4. Spring Cloud

<u>Using Netflix Eureka for service discovery</u>

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-15-05-28-43-image.png)

In client define Load balancing aware class

```java
@Bean
@LoadBalanced
public WebClient.Builder loadBalancedWebClientBuilder() {
return WebClient.builder();
}
```

Setting up the configuration for development use

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-15-05-41-26-image.png)





**<u>Using Spring Cloud Gateway as an edge server</u>**

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-15-05-42-32-image.png)







**<u>Using Spring Cloud Config for centralized configuration</u>**

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-15-05-44-21-image.png)



**<u>Using Resilience4j for improved resilience</u>**

Configuration:

- slidingWindowType: To determine if a circuit breaker needs to be opened, time or number of call

- slidingWindowSize: 

- failureRateThreshold

- automaticTransitionFromOpenToHalfOpenEnabled

- waitDurationInOpenState

- permittedNumberOfCallsInHalfOpenState

Time limiter:

- timeoutDuration: Specifies how long a TimeLimiter instance waits for a call  
  to complete before it throws a timeout exception. We will set it to 2s.

```java
@TimeLimiter(name = "product")
@CircuitBreaker(
name = "product", fallbackMethod = "getProductFallbackValue")
public Mono<Product> getProduct(
int productId, int delay, int faultPercent) {
...
}
```

```java
private Mono<Product> getProductFallbackValue(int productId,
int delay, int faultPercent, CallNotPermittedException ex) {
```

```yaml
resilience4j.circuitbreaker:
instances:
product:
allowHealthIndicatorToFail: false
registerHealthIndicator: true
slidingWindowType: COUNT_BASED
slidingWindowSize: 5
failureRateThreshold: 50
waitDurationInOpenState: 10000
permittedNumberOfCallsInHalfOpenState: 3
automaticTransitionFromOpenToHalfOpenEnabled: true
```



## 5. Kubernetes

### 5.1 Introduction to Kubernetes

K8s object:

- Node: a server

- Pod: A Pod represents the smallest possible deployable component in Kubernetes, consisting of one or more co-located containers.

- Deployment: A Deployment is used to deploy and upgrade Pods.

- ReplicaSet: A ReplicaSet is used to ensure that a specified number of Pods are running at all times. If a Pod is deleted, it will be replaced with a new Pod by the ReplicaSet

- Service: A Service is a stable network endpoint that you can use to connect to one or multiple Pods. A Service is assigned an IP address and a DNS name in the internal network of the Kubernetes cluster.

- Name space

- Config map

- Ingress: Ingress can manage external access to Services in a Kubernetes cluster, typically using HTTP or HTTPS.

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-15-06-06-25-image.png)

**<u>Kubernetes runtime components</u>**

- Control plane (master node):
  
  - api-server
  
  - etcd: database for k8s
  
  - controller manger
  
  - scheduler

- Components that run on all the nodes, constituting the data plane:
  
  - kubelet, a node agent that executes as a process directly in the nodes' operating system and not as a container. communicate between master and pod
  
  - kube-proxy: load balacing to pod
  
  - container runtime

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-15-06-11-17-image.png)

**<u>Deploying Our Microservices to Kubernetes</u>**

- Helm, a package manager for Kubernetes

Using Spring Boot's support for graceful shutdown and probes for liveness and readiness:

- Graceful shutdown

```ini
server.shutdown: graceful
spring.lifecycle.timeout-per-shutdown-phase: 10
```

- Liveness and readiness probes



**<u>Introducing Helm</u>**

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-15-06-39-41-image.png)



![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-15-06-40-29-image.png)

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-15-06-41-10-image.png)





Structure of helm folder:

<img src="file:///C:/Users/duyng/AppData/Roaming/marktext/images/2022-06-15-06-30-38-image.png" title="" alt="" width="183">

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-15-06-32-38-image.png)

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-15-06-34-38-image.png)

```bash
helm dependency update .
helm template . -s templates/configmap_from_file.yaml
```

**<u>Using a Service Mesh to Improve Observability and Management</u>**

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-15-07-19-47-image.png)

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-15-07-24-45-image.png)

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-15-07-25-10-image.png)

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-15-07-25-24-image.png)

```bash
kubectl get deployment sample-deployment -o yaml | istioctl kube-inject
-f - | kubectl apply -f -
```



Isito set up:

- Deploy sample k8s service

- Create isito ingress gateway

- Set ip and port



Routing:

- Apply virtual service

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
  - reviews
  http:
  - route:
    - destination:
        host: reviews
        subset: v1
```

Querying Metrics from Prometheus:

- Install promteous plugin 

- Open promtheus: istioctl dashboard prometheus


