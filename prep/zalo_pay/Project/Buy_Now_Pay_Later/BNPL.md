# 1. Architecture

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-18-11-40-27-image.png)

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-18-18-08-20-image.png)

1. From merchant choosing BNPL

2. Fill in basic info

3. Authentication:
   
   1. Using CMND: AWS Rekognition.
   
   2. Face detection

4. System calling API to CIC check.

5. Using decision engine to Select credit amount.

6. Pay Later System sends request to WAY4 to create new client and BNPL contract and also stores customer data and e-signing contract. And then send notification to customer.

7. **WAY4**: Create Contract request from BNPL system.

8. Customer approve transaction. First API Send to **WAY4**, **WAY4** Generate OTP. **<u>(Explain Architect later)</u>**

9. **WAY4** Create a message to database. **WAY4** Polling, query from database periodically and push message to RabbitMQ. **<u>(Explain Architect later)</u>**

10. **WAY4** Checking Limit and create transaction.

**Transaction Hanlder:**

Is a stateful replica set:

- Note about Replica Set:
  
  - Require scale down to 0 before terminate service
  
  - StatefulSets currently require a [Headless Service](https://kubernetes.io/docs/concepts/services-networking/service/#headless-services) to be responsible for the network identity of the Pods. You are responsible for creating this Service.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  selector:
    matchLabels:
      app: nginx # has to match .spec.template.metadata.labels
  serviceName: "nginx"
  replicas: 3 # by default is 1
  minReadySeconds: 10 # by default is 0
  template:
    metadata:
      labels:
        app: nginx # has to match .spec.selector.matchLabels
    spec:
      terminationGracePeriodSeconds: 10 # wait 10 second from TERMINATING state to terminated
      containers:
      - name: nginx
        image: k8s.gcr.io/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "my-storage-class"
      resources:
        requests:
          storage: 1Gi
```

**NOTE**: In order to using sticky session each API, we use `service.spec.sessionAffinity` to "ClientIP". service.spec.sessionAffinityConfig.clientIP.timeoutSeconds

**Queue gurantee delivery:**

**<u>Not losing your exchanges and queues</u>**:

- `durable=true`, `auto-delete=false` 

**<u>Turn off auto ack:</u>**

- Requeue message that are not acknowledge

**<u>When Consumers Fail or Lose Connection: Automatic Requeueing</u>**

    When consumer down, it automatically requeue, we must decrease time to detect unvailable client.


