# Kubernetes Tutorial - Kubernetes Pod & Friends

 A pod should be considered as the smallest unit of execution in Kubernetes and all Kubernetes workload types run Pods at their core.

In this section:
- Pod - A pod is the smallest execution unit in Kubernetes
- Limits and Requests - CPU and Memory reservation for a Pod
- Liveness and Readiness - Health Checks for a Pod
- ServiceAccount - Functional ID inside the Pod to connect to the API server
- Security Context - Linux privilege and access control settings for a Pod or Container
<br />

![00-pod](https://user-images.githubusercontent.com/18049790/135598360-f75b3c0d-1a41-4bb2-bf0e-cb983e819690.jpg)
<br />

## Kubernetes Pod

<details class="faq box"><summary>Kubernetes Namespace (ns)  - Logical isolation for your application</summary>
<p>

> Problem Statement: I want logical separation and isolation for my application
>
> tl;dr – This is the holder for your application

What is a Kubernetes Namespace?
* In Kubernetes, a namespace is a way to divide cluster resources between multiple users (via resource quotas). 
* When you create a namespace, you can specify resource quotas for that namespace, which can help ensure that one team or set of users doesn't use too many resources and negatively impact the other users in the cluster.
* Namespaces are also used to isolate resources within a cluster, so that different teams or projects can use the same names for resources without conflicting with each other. 
* For example, you could have a Deployment named "frontend" in the "dev" namespace, and another Deployment also named "frontend" in the "prod" namespace, and they would not conflict with each other.
* In short, namespaces are a way to organize and divide resources in a Kubernetes cluster, and can be used to isolate resources and set resource quotas.

kubernetes.io bookmark: [Namespaces](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)

Create a namespace with the name: `ns-bootcamp-pod`
```bash
kubectl create namespace ns-bootcamp-pod
kubectl config set-context --current --namespace=ns-bootcamp-pod
```

Output:

```yaml
apiVersion: v1
kind: Namespace
metadata:  
  name: ns-bootcamp-pod
```

</p>
</details>

<details class="faq box"><summary>Kubernetes Pod (po) - A pod is the smallest execution unit in Kubernetes</summary>
<p>

> Problem Statement: I want to run immutable and resilient Linux or Windows workloads 
>
> tl;dr – Compute, Network, Storage and Monitoring around your application

What is a Kubernetes Pod?
* In Kubernetes, a pod is the smallest deployable unit. 
* It is the basic execution unit of a Kubernetes application–the smallest and simplest unit in the Kubernetes object model that you create or deploy.
* A pod consists of one or more containers, and is used to host the containers that make up an application. 
* The containers in a pod are designed to work together and share resources, such as storage and networking. 
* They can also communicate with each other using localhost.
* Pods are ephemeral, meaning that they are not meant to be long-lived. 
* When a pod is deleted, all of the containers in the pod are also deleted. 
* This is in contrast to more long-lived objects like Deployments, which create and manage ReplicaSets, which in turn create and manage pods.
* In summary, a pod is a deployable unit in Kubernetes that consists of one or more containers and is used to host the containers that make up an application.

![02-basic-pod](https://user-images.githubusercontent.com/18049790/140636726-0c08ffb0-e520-42ba-8807-8928da6c53e7.jpg)

kubernetes.io bookmark: [Using Pods](https://kubernetes.io/docs/concepts/workloads/pods/#using-pods)

Run a Pod called `my-pod` using the `nginx` image exposing port 80
```bash
clear
# Create the pod via the command line imperatively
kubectl run my-pod --image=nginx:1.20.0 --port=80
```

Output:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-container #👈👈👈 Altered to call out container spec
    image: nginx:1.20.0
    ports:
    - containerPort: 80
```

Delete the Pod called `my-pod`
```bash
kubectl delete pod my-pod --grace-period 0 --force
clear
```

<details class="faq box"><summary>The Laws of Three - Container Types</summary>
<p>

> tl;dr – Kubernetes always respects the Law of Three

There are three container types:
* [containers](https://kubernetes.io/docs/concepts/containers/) #👈👈👈 Part of CKAD exam
  * A container image is a ready-to-run software package, containing everything needed to run an application: 
    * the code and any runtime it requires, application and system libraries, and default values for any essential settings
* [initContainers](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/) #👈👈👈 Part of CKAD exam
  * Specialized containers that run before app containers in a Pod
  * Init containers can contain utilities or setup scripts not present in an app image
* [ephemeralcontainers](https://kubernetes.io/docs/concepts/workloads/pods/ephemeral-containers/)
  * A special type of container that runs temporarily in an existing Pod to accomplish user-initiated actions such as [troubleshooting](https://github.com/jamesbuckett/ckad-advanced-topics/blob/master/05-kubernetes-pod-ephemeral.md)

</p>
</details>

</p>
</details>

<details class="faq box"><summary>Limits and Requests - CPU and Memory reservation for a Pod</summary>
<p>

> Problem Solving: I want to guarantee CPU and RAM for my microservice application
>
> tl;dr – Let me make a CPU and RAM reservation

What is a Kubernetes Limit and Request?
* In Kubernetes, a limit is the maximum amount of resources (such as CPU or memory) that a container is allowed to use. 
* A request is the minimum amount of resources that a container is guaranteed to be allocated.
* When you create a pod in Kubernetes, you can specify resource limits and requests for the containers in the pod. 
* The resource limits define the maximum amount of resources that a container is allowed to use. 
* If a container exceeds its resource limits, it may be terminated by the Kubernetes system.
* The resource requests define the minimum amount of resources that a container is guaranteed to be allocated. 
* When the Kubernetes scheduler places a pod on a node, it ensures that the node has enough resources available to meet the resource requests of all of the containers in the pod.
* By setting resource limits and requests, you can ensure that your containers have the resources they need to run properly, and that the resources in your cluster are being used efficiently.

![02-cpu-ram](https://user-images.githubusercontent.com/18049790/140636727-9b6ceba7-5bfa-4f24-bca0-7f40c73181b2.jpg)

kubernetes.io bookmark: [Meaning of Memory](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#meaning-of-memory)

Create a Pod with Limits and Requests
```yaml
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-container
    image: nginx:1.20.0
    ports:
    - containerPort: 80
    resources: #👈👈👈 CPU & RAM Resources
      requests: #👈👈👈 Minimum “Request”
        memory: "64Mi"  #👈👈👈 64Mi = 64 Megabyte
        cpu: "250m" #👈👈👈 250m = 250 milliCPU
      limits:  #👈👈👈 Maximum “Limit”
        memory: "128Mi" #👈👈👈 128Mi = 128 Megabyte
        # cpu: "500m" #👈👈👈 500m = 500 milliCPU (½ CPU)
EOF
```

Delete the Pod called `my-pod`
```bash
kubectl delete pod my-pod --grace-period 0 --force
clear
```

[CPU resource units](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#meaning-of-cpu)
* Limits and requests for CPU resources are measured in cpu units. 
* In Kubernetes, 1 CPU unit is equivalent to 1 physical CPU core, or 1 virtual core 
  * Depending on whether the node is a physical host or a virtual machine running inside a physical machine.
* Fractional requests are allowed. 
* When you define a container with spec.containers[].resources.requests.cpu set to 0.5 
  * You are requesting half as much CPU time compared to if you asked for 1.0 CPU
* For CPU resource units, the quantity expression 0.1 is equivalent to the expression 100m 
  * Which can be read as "one hundred millicpu"
  * Some people say "one hundred millicores", and this is understood to mean the same thing

The Relationship between Resources and Quality of Service

What is Kubernetes Quality of Service?
* In Kubernetes, Quality of Service (QoS) refers to the relative priority of pods and the resources they are allowed to consume. 
* There are three QoS classes in Kubernetes:
  * BestEffort: 
    * This is the default QoS class, and it means that the pod is not guaranteed any specific amount of resources. 
    * The pod will only receive resources if they are available after all other pods have been given their required resources.
  * Burstable: 
    * This QoS class means that the pod is guaranteed a certain minimum amount of resources, but is allowed to use additional resources if they are available.
  * Guaranteed: 
    * This QoS class means that the pod is guaranteed a certain amount of resources, and will always receive that amount of resources regardless of the usage of other pods in the cluster.
* By setting the QoS class for a pod, you can control the priority and resource allocation for that pod relative to other pods in the cluster. 
* This can be useful in cases where you have critical workloads that need to be guaranteed a certain amount of resources, or if you want to ensure that certain pods always have access to the resources they need to function properly.

![02-qos](https://user-images.githubusercontent.com/18049790/140636729-64c34a54-38e3-4057-92ca-3ffd82c4fbb6.jpg)

[CPU limits on Kubernetes are an antipattern](https://home.robusta.dev/blog/stop-using-cpu-limits?s=03)

</p>
</details>

<details class="faq box"><summary>Liveness and Readiness Probes - Health Checks for a Pod</summary>
<p>

> Problem Solving: I want a way to check the health of my microservices application

> tl;dr – How to perform Health Checks on my microservices application 

What are Kubernetes Probes?
* In Kubernetes, a probe is a mechanism for checking the health of a container. 
* There are two types of probes: liveness probes and readiness probes.
  * Liveness probes are used to determine if a container is still running. 
    * If a liveness probe fails, the Kubernetes system will restart the container in an attempt to recover it.
  * Readiness probes are used to determine if a container is ready to start accepting traffic. 
    * If a readiness probe fails, the Kubernetes system will not send traffic to the container until it becomes ready.
* Probes are implemented as HTTP requests or command executions, and can be configured to specify the initial delay, the period between checks, and the number of failures allowed before marking the container as failed.
* By configuring liveness and readiness probes for your containers, you can ensure that your applications are running properly and are able to handle traffic. 
* This can help improve the reliability and availability of your applications.

![probes](https://user-images.githubusercontent.com/18049790/140636733-846c09eb-2e81-467a-8ad0-331e45b9b4fd.jpg)

kubernetes.io bookmark: [Define a liveness HTTP request](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/#define-a-liveness-http-request)

Create a Pod with Probes
```yaml
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-container
    image: nginx:1.20.0
    ports:
    - containerPort: 80
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
    livenessProbe: #👈👈👈 Are you alive?
      httpGet: #👈👈👈 Execute a HTTP request 
        path: / #👈👈👈 Check for an HTTP response between 200 and 400
        port: 80 #👈👈👈 Run the probe against port 80
      initialDelaySeconds: 3  #👈👈👈 Start probe after initial wait of 3 seconds
      periodSeconds: 3 #👈👈👈 Run probe every 3 seconds
    readinessProbe: #👈👈👈 Are you ready for traffic?
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 3
      periodSeconds: 3
EOF
```

Observation:
* Start Octant
* Go to the `ns-bootcamp-pod` namespace
* Go to `Workloads`...`Pods`...`my-pod`
* Go to the `Logs` tab

```console
my-container 10.1.0.1 - - [16/Jan/2022:04:55:15 +0000] "GET / HTTP/1.1" 200 612 "-" "kube-probe/1.21" "-" #👈👈👈 Probe entries in STDOUT
my-container 10.1.0.1 - - [16/Jan/2022:04:55:15 +0000] "GET / HTTP/1.1" 200 612 "-" "kube-probe/1.21" "-"
my-container 10.1.0.1 - - [16/Jan/2022:04:55:18 +0000] "GET / HTTP/1.1" 200 612 "-" "kube-probe/1.21" "-"
my-container 10.1.0.1 - - [16/Jan/2022:04:55:18 +0000] "GET / HTTP/1.1" 200 612 "-" "kube-probe/1.21" "-"
my-container 10.1.0.1 - - [16/Jan/2022:04:55:21 +0000] "GET / HTTP/1.1" 200 612 "-" "kube-probe/1.21" "-"
my-container 10.1.0.1 - - [16/Jan/2022:04:55:21 +0000] "GET / HTTP/1.1" 200 612 "-" "kube-probe/1.21" "-"
my-container 10.1.0.1 - - [16/Jan/2022:04:55:24 +0000] "GET / HTTP/1.1" 200 612 "-" "kube-probe/1.21" "-"
my-container 10.1.0.1 - - [16/Jan/2022:04:55:24 +0000] "GET / HTTP/1.1" 200 612 "-" "kube-probe/1.21" "-"
my-container 10.1.0.1 - - [16/Jan/2022:04:55:27 +0000] "GET / HTTP/1.1" 200 612 "-" "kube-probe/1.21" "-"
my-container 10.1.0.1 - - [16/Jan/2022:04:55:27 +0000] "GET / HTTP/1.1" 200 612 "-" "kube-probe/1.21" "-"
my-container 10.1.0.1 - - [16/Jan/2022:04:55:30 +0000] "GET / HTTP/1.1" 200 612 "-" "kube-probe/1.21" "-"
my-container 10.1.0.1 - - [16/Jan/2022:04:55:30 +0000] "GET / HTTP/1.1" 200 612 "-" "kube-probe/1.21" "-"
my-container 10.1.0.1 - - [16/Jan/2022:04:55:33 +0000] "GET / HTTP/1.1" 200 612 "-" "kube-probe/1.21" "-"
my-container 10.1.0.1 - - [16/Jan/2022:04:55:33 +0000] "GET / HTTP/1.1" 200 612 "-" "kube-probe/1.21" "-"
my-container 10.1.0.1 - - [16/Jan/2022:04:55:36 +0000] "GET / HTTP/1.1" 200 612 "-" "kube-probe/1.21" "-"
```

</p>
</details>

<details class="faq box"><summary>Liveness and Readiness Probes - The Laws of Three</summary>
<p>

<details class="faq box"><summary>Liveness and Readiness Probes - Probe Types</summary>
<p>

> tl;dr – Kubernetes always respects the Law of Three

There are three probe types:
* [livenessProbe](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#types-of-probe) #👈👈👈 Part of CKAD exam
  * Kubernetes uses liveness probes to know when to restart a container 
  * If a container is unresponsive, perhaps the application is deadlocked due to a multi-threading defect, restarting the container can make the application more available, despite the defect
  * For example, liveness probes could catch a deadlock, where an application is running, but unable to make progress
  * The kubelet uses liveness probes to know when to restart a container
    * If the liveness probe fails, the kubelet kills the container, and the container is subjected to its restart policy  
* [readinessProbe](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#types-of-probe) #👈👈👈 Part of CKAD exam
  * Kubernetes uses readiness probes to decide when the container is available for accepting traffic 
  * The readiness probe is used to control which pods are used as the backends for a service 
  * A pod is considered ready when all of its containers are ready 
  * If a pod is not ready, it is removed from service load balancers 
  * For example, if a container loads a large cache at startup and takes minutes to start, you do not want to send requests to this container until it is ready, or the requests will fail—you want to route requests to other pods, which are capable of servicing requests.  
* [startupProbe](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#types-of-probe)
  * Indicates whether the application within the container is started 
  * All other probes are disabled if a startup probe is provided, until it succeeds
  * If the startup probe fails, the kubelet kills the container, and the container is subjected to its restart policy

</p>
</details>

<details class="faq box"><summary>Liveness and Readiness Probes - Probe Checks</summary>
<p>

> tl;dr – Kubernetes always respects the Law of Three, sometimes Four

There are three probe checks:
* [exec](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#probe-check-methods) 
  * Executes a specified command inside the container. 
  * The diagnostic is considered successful if the command exits with a status code of 0.
* [httpGet](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#probe-check-methods) #👈👈👈 Part of CKAD exam
  * Performs an HTTP GET request against the Pod's IP address on a specified port and path
  * The diagnostic is considered successful if the response has a status code greater than or equal to 200 and less than 400
* [tcpSocket](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#probe-check-methods)
  * Performs a TCP check against the Pod's IP address on a specified port
  * The diagnostic is considered successful if the port is open 
  * If the remote system (the container) closes the connection immediately after it opens, this counts as healthy

I lied there is one extra probe check:
* [grpc](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#probe-check-methods) `alpha feature`
  *   Performs a remote procedure call using gRPC
  *   The target should implement gRPC health checks
  *   The diagnostic is considered successful if the status of the response is SERVING
  *   gRPC probes are an alpha feature and are only available if you enable the GRPCContainerProbe feature gate

</p>
</details>

<details class="faq box"><summary>Liveness and Readiness Probes - Probe Results</summary>
<p>

> tl;dr – Kubernetes always respects the Law of Three

There are three probe results:
* Success
  * The container passed the diagnostic
* Failure
  * The container failed the diagnostic
* Unknown
  * The diagnostic failed (no action should be taken, and the kubelet will make further checks)

</p>
</details>

</p>
</details>

<details class="faq box"><summary>ServiceAccount (sa) - Functional ID inside the Pod to connect to the API server</summary>
<p>

What is a Kubernetes Service Account?
* In Kubernetes, a service account provides an identity for processes that run in a cluster. 
* When you create a service account, you can specify the permissions that the service account has within the cluster. 
* This allows you to finely control the actions that the service account can perform.
* Service accounts are used to grant permissions to processes that need to interact with the Kubernetes API. 
* For example, if you want to deploy an application to a cluster, you might use a service account to authenticate the deployment process with the API server.
* By default, every Kubernetes cluster includes a set of built-in service accounts, including the `default` service account that is used by pods that do not specify a service account. 
* You can also create custom service accounts and grant them the permissions that are required for your specific use case.

![02-sa](https://user-images.githubusercontent.com/18049790/140636731-77ba1689-b901-4ad9-bb2a-885fdddfe3ca.jpg)

kubernetes.io bookmark: [Configure Service Accounts for Pods](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)

```bash
kubectl delete pod my-pod --now
clear
```

```yaml
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-serviceaccount  
---
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  serviceAccountName: my-serviceaccount #👈👈👈 give an identity to your pod
  containers:
  - name: my-container
    image: nginx:1.20.0
    ports:
    - containerPort: 80
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
    livenessProbe: 
      httpGet:  
        path: / 
        port: 80 
      initialDelaySeconds: 3  
      periodSeconds: 3 
    readinessProbe: 
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 3
      periodSeconds: 3
EOF
```

</p>
</details>
<br />

## Kubernetes Pod Best Practices 

<details class="faq box"><summary>Static Code Analysis - CRITICAL</summary>
<p>

> tl;dr I do not want you to get hacked straight out of the gate so here are the security best practices for securing a Pod

* [kube-score](https://github.com/zegl/kube-score) - kube-score is a tool that performs static code analysis of your Kubernetes object definitions

```bash
clear
mkdir -p ~/ckad/
kubectl run my-pod --image=nginx --port=80 --dry-run=client -o yaml > ~/ckad/01-kubernetes-pod-basic-pod.yml
```

Output:
```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: my-pod
  name: my-pod
spec:
  containers:
  - image: nginx
    name: my-pod
    ports:
    - containerPort: 80
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```bash
kube-score score ~/ckad/01-kubernetes-pod-basic-pod.yml
```
```console
v1/Pod my-pod                                                                 💥
    [CRITICAL] Container Security Context User Group ID
        · my-pod -> Container has no configured security context
            Set securityContext to run the container in a more secure context.

    [CRITICAL] Container Security Context ReadOnlyRootFilesystem
        · my-pod -> Container has no configured security context
            Set securityContext to run the container in a more secure context.

    [CRITICAL] Container Resources
        · my-pod -> CPU limit is not set
            Resource limits are recommended to avoid resource DDOS. Set resources.limits.cpu
        · my-pod -> Memory limit is not set
            Resource limits are recommended to avoid resource DDOS. Set resources.limits.memory
        · my-pod -> CPU request is not set
            Resource requests are recommended to make sure that the application can start and run without crashing. Set resources.requests.cpu
        · my-pod -> Memory request is not set
            Resource requests are recommended to make sure that the application can start and run without crashing. Set resources.requests.memory

    [CRITICAL] Container Image Tag
        · my-pod -> Image with latest tag
            Using a fixed tag is recommended to avoid accidental upgrades
    [CRITICAL] Container Image Pull Policy
       · my-pod -> ImagePullPolicy is not set to Always
           It's recommended to always set the ImagePullPolicy to Always, to make sure that the imagePullSecrets are always correct, and to always get the image you want.
           
    [CRITICAL] Pod NetworkPolicy
        · The pod does not have a matching NetworkPolicy
            Create a NetworkPolicy that targets this pod to control who/what can communicate with this pod. Note, this feature needs to be supported by the CNI
            implementation used in the Kubernetes cluster to have an effect.
```

</p>
</details>

<details class="faq box"><summary>Container Image Tag</summary>
<p>

```Console
Image with latest tag
ImagePullPolicy is not set to Always
```

```yaml
#Before
  - image: nginx
  - image: nginx:latest
#After  
  - image: nginx:1.20.0 ## 👈👈👈 
    imagePullPolicy: Always ## 👈👈👈 
```

Notes:
* tl;dr - Hard code which container version you want to run
* Using a fixed tag is recommended to avoid accidental upgrades
* It's recommended to always set the ImagePullPolicy to Always. 
* To make sure that the imagePullSecrets are always correct (from private repositories), and to always get the image you want.

</p>
</details>


<details class="faq box"><summary>Container Security Context User Group ID</summary>
<p>

```Console
Set securityContext to run the container in a more secure context.
```

```yaml
  securityContext:
    runAsUser: 10000 ## 👈👈👈 
    runAsGroup: 30000 ## 👈👈👈 
    fsGroup: 2000
```

Notes
* A userid above 10 000 is recommended to avoid conflicts with the host. 
  * Set securityContext.runAsUser to a value > 10000
* A groupid above 10 000 is recommended to avoid conflicts with the host. 
  * Set securityContext.runAsGroup to a value > 10000

</p>
</details>

<details class="faq box"><summary>Container Security Context ReadOnlyRootFilesystem</summary>
<p>

```Console
Set securityContext to run the container in a more secure context.
```

```yaml
    securityContext:
      readOnlyRootFilesystem: true ##  👈👈👈 
```

Notes:
* tl;dr - Make the container filesystem immutable
* Requiring the use of a read only root file system
* readOnlyRootFilesystem is one setting that controls whether a container is able to write into its filesystem 
* It’s a feature most want enabled in the event of a hack
* If an attacker gets in, they won’t be able to tamper with the application or write foreign executables to disk

</p>
</details>

<details class="faq box"><summary>Container Resources</summary>
<p>

```Console
Resource limits are recommended to avoid resource DDOS. Set resources.limits.cpu
Resource limits are recommended to avoid resource DDOS. Set resources.limits.memory
Resource requests are recommended to make sure that the application can start and run without crashing. Set resources.requests.cpu
Resource requests are recommended to make sure that the application can start and run without crashing. Set resources.requests.memory
```

```yaml
    resources:
      limits:
        cpu: "32m" ## 👈👈👈 Set resources.limits.cpu          
        memory: "64Mi" ## 👈👈👈 Set resources.limits.memory
      requests:
        cpu: "32m" ## 👈👈👈 Set resources.requests.cpu      
        memory: "64Mi" ## 👈👈👈 Set resources.requests.memory
```

</p>
</details>

<details class="faq box"><summary>Pod Probes</summary>
<p>

```Console
    [CRITICAL] Pod Probes
        · Container has the same readiness and liveness probe
            Using the same probe for liveness and readiness is very likely dangerous. 
            Generally it's better to avoid the livenessProbe than re-using the readinessProbe.
            More information: https://github.com/zegl/kube-score/blob/master/README_PROBES.md
```

Please read [Readiness and Liveness Probes](https://github.com/zegl/kube-score/blob/master/README_PROBES.md)

Notes:
* Container has the same readiness and liveness probe
* Using the same probe for liveness and readiness is very likely dangerous
* Generally it's better to avoid the livenessProbe than re-using the readinessProbe
* Set interval (default: 10s), timeout (default: 1s), successThreshold (default: 1), failureThreshold (default: 3) to your needs. 
* In the default configuration, your application will fail for 30s (+ the time it takes for the network to react), for clients to stop sending traffic to your application.

livenessProbe:
* tl;dr: Is the container healthy right now, or do we need to restart it?
* It can be used to let Kubernetes know if your application is deadlocked, and needs to be restarted. 
* Only the container with the failing probe will be restarted, other containers in the same Pod will be unaffected.
* Do not use livenessProbe unless you have a clear use case

readinessProbe:
* tl;dr: Is it a good idea to send traffic to this Pod right now?
* Without a readinessProbe you're risking that:
  * Traffic is sent to the Pod before the server has started.
  * Traffic is still sent to the Pod after the Pod has stopped.
* Use for applications to decide when a pod should receive traffic  

```yaml
    readinessProbe: ## 👈👈👈 Add a properly configured readinessProbe to notify kubelet your Pods are ready for traffic
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 10 ## 👈👈👈 Probes start running after initialDelaySeconds after container is started (default: 0 seconds)
      periodSeconds: 20 ## 👈👈👈 How often probe should run (default: 10 seconds)
      timeoutSeconds: 5  ## 👈👈👈 Probe timeout (default: 1 second)
      successThreshold: 1   ## 👈👈👈 Required number of successful probes to mark container healthy/ready (default: 1 iteration)
      failureThreshold: 3   ## 👈👈👈 When a probe fails, it will try failureThreshold times before deeming unhealthy/not ready (default: 3 iterations)
```

</p>
</details>

<details class="faq box"><summary>Pod NetworkPolicy</summary>
<p>

```Console
Create a NetworkPolicy that targets this pod to control who/what can communicate with this pod. 
Note, this feature needs to be supported by the CNI implementation used in the Kubernetes cluster to have an effect.
```

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: my-netpol  
spec:
  podSelector:
    matchLabels:
      run: my-pod 
  ingress: #👈👈👈 Ingress
    - from:
        - podSelector:
            matchLabels:
              tier: web 
      ports:
        - port: 80
  egress: #👈👈👈 Egress
    - to:
        - podSelector:
            matchLabels:
              tier: app 
      ports:
        - port: 80        
```

Notes:
* Allow communications FROM any pod with label: `tier=web`
* Allow communications TO any pod with label: `tier=app`
* Apply this network policy to any pod with label: `run=my-pod`
* [Network Policy Editor for Kubernetes](https://editor.cilium.io)
* [Kubernetes Tutorial - Kubernetes Networking](https://github.com/jamesbuckett/ckad-bootcamp/blob/master/04-kubernetes-networking.md)

</p>
</details>

<details class="faq box"><summary>Static Code Analysis - PASS</summary>
<p>

```yaml
echo "
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: my-pod
  name: my-pod
  namespace: ns-bootcamp-pod 
spec:
  securityContext: ## 👈👈👈 securityContext at the Pod Level
    runAsUser: 10000 ## 👈👈👈 A userid above 10 000 is recommended to avoid conflicts with the host. Set securityContext.runAsUser to a value > 10000
    runAsGroup: 30000 ## 👈👈👈 A groupid above 10 000 is recommended to avoid conflicts with the host. Set securityContext.runAsGroup to a value > 10000
    fsGroup: 2000    
  volumes:
  - name: cache-volume ##  👈👈👈 nginx needs to write to these directories
    emptyDir: {}
  - name: runtime-volume ##  👈👈👈 nginx needs to write to these directories
    emptyDir: {}  
  containers:
  - image: nginx:1.20.0
    name: my-pod
    ports:
    - containerPort: 80
    securityContext: ## 👈👈👈 securityContext at the container level
      readOnlyRootFilesystem: true ##  👈👈👈 Container Security Context ReadOnlyRootFilesystem    
    volumeMounts:
      - name: cache-volume 
        mountPath: /var/cache/nginx ##  👈👈👈 nginx needs to write to these directories
      - name: runtime-volume
        mountPath: /var/run ##  👈👈👈 nginx needs to write to these directories
    resources:
      requests:
        memory: "64Mi" ## 👈👈👈 Resource requests are recommended to make sure that the application can start and run without crashing. Set resources.requests.memory
        cpu: "32m" ## 👈👈👈 Resource requests are recommended to make sure that the application can start and run without crashing. Set resources.requests.cpu
      limits:
        memory: "64Mi" ## 👈👈👈 Resource limits are recommended to avoid resource DDOS. Set resources.limits.memory
        cpu: "32m" ## 👈👈👈 Resource limits are recommended to avoid resource DDOS. Set resources.limits.cpu
    imagePullPolicy: Always ## 👈👈👈 It's recommended to always set the ImagePullPolicy to Always. To make sure that the imagePullSecrets are always correct, and to always get the image you want
    readinessProbe: ## 👈👈👈 Missing property object readinessProbe
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 10
      periodSeconds: 5
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: my-netpol
  namespace: ns-bootcamp-pod 
spec:
  podSelector:
    matchLabels:
      run: my-pod #👈👈👈 Change - Which pod does this Network Policy Apply to
  ingress:
    - from:
        - podSelector:
            matchLabels:
              tier: web #👈👈👈 Ingress - Traffic from pod with label: tier=web
      ports:
        - port: 80
  egress:
    - to:
        - podSelector:
            matchLabels:
              tier: app #👈👈👈 Egress - Traffic to pod with label: tier=app
      ports:
        - port: 80
"  > ~/ckad/01-kubernetes-pod-pass-pod.yml        
```

```bash
kube-score score ~/ckad/01-kubernetes-pod-pass-pod.yml
```

```console
networking.k8s.io/v1/NetworkPolicy my-netpol                                  ✅
v1/Pod my-pod                                                                 ✅
```

```bash
kubectl apply -f ~/ckad/01-kubernetes-pod-pass-pod.yml
```

</p>
</details>
<br />

## Clean Up

<details class="faq box"><summary>Clean Up</summary>
<p>

```bash
cd
yes | rm -R ~/ckad/
kubectl delete ns ns-bootcamp-pod 
kubectl delete sa my-serviceaccount
```

</p>
</details>
<br />

Next [Kubernetes Tutorial - Configuration](https://github.com/jamesbuckett/ckad-bootcamp/blob/master/02-kubernetes-configuration.md)

_End of Section_
