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

kubernetes.io bookmark: [Namespaces](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)

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

> Problem Statement: I want to run immutable and resilient Linux workloads 
>
> tl;dr – Compute, Network, Storage and Monitoring around your application

![02-basic-pod](https://user-images.githubusercontent.com/18049790/140636726-0c08ffb0-e520-42ba-8807-8928da6c53e7.jpg)

kubernetes.io bookmark: [Using Pods](https://kubernetes.io/docs/concepts/workloads/pods/#using-pods)

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

</p>
</details>

<details class="faq box"><summary>Limits and Requests - CPU and Memory reservation for a Pod</summary>
<p>

> Problem Solving: I want to guarantee CPU and RAM for my microservice application
>
> tl;dr – Let me make a CPU and RAM reservation

![02-cpu-ram](https://user-images.githubusercontent.com/18049790/140636727-9b6ceba7-5bfa-4f24-bca0-7f40c73181b2.jpg)

kubernetes.io bookmark: [Meaning of Memory](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#meaning-of-memory)

```bash
kubectl delete pod my-pod --grace-period 0 --force
clear
```

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
        cpu: "500m" #👈👈👈 500m = 500 milliCPU (½ CPU)
EOF
```
The Relationship between Resources and Quality of Service

![02-qos](https://user-images.githubusercontent.com/18049790/140636729-64c34a54-38e3-4057-92ca-3ffd82c4fbb6.jpg)

</p>
</details>

<details class="faq box"><summary>Liveness and Readiness Probes - Health Checks for a Pod</summary>
<p>

> Problem Solving: I want a way to check the health of my microservices application

> tl;dr – How to perform Health Checks on my microservices application 

![probes](https://user-images.githubusercontent.com/18049790/140636733-846c09eb-2e81-467a-8ad0-331e45b9b4fd.jpg)

kubernetes.io bookmark: [Define a liveness HTTP request](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/#define-a-liveness-http-request)

```bash
kubectl delete pod my-pod --grace-period 0 --force
clear
```

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

</p>
</details>

<details class="faq box"><summary>ServiceAccount (sa) - Functional ID inside the Pod to connect to the API server</summary>
<p>

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

Pass:
```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: my-pod
  name: my-pod
spec:
  securityContext:
    runAsUser: 10000 ## 👈👈👈 A userid above 10 000 is recommended to avoid conflicts with the host. Set securityContext.runAsUser to a value > 10000
    runAsGroup: 30000 ## 👈👈👈 A groupid above 10 000 is recommended to avoid conflicts with the host. Set securityContext.runAsGroup to a value > 10000
    fsGroup: 2000
  containers:
  - image: nginx:1.20.0
    name: my-pod
    ports:
    - containerPort: 80
    securityContext:
      readOnlyRootFilesystem: true ##  👈👈👈 Container Security Context ReadOnlyRootFilesystem
    resources:
      requests:
        memory: "64Mi" ## 👈👈👈 Resource requests are recommended to make sure that the application can start and run without crashing. Set resources.requests.memory
        cpu: "32m" ## 👈👈👈 Resource requests are recommended to make sure that the application can start and run without crashing. Set resources.requests.cpu
      limits:
        memory: "64Mi" ## 👈👈👈 Resource limits are recommended to avoid resource DDOS. Set resources.limits.memory
        cpu: "32m" ## 👈👈👈 Resource limits are recommended to avoid resource DDOS. Set resources.limits.cpu
    imagePullPolicy: Always ## 👈👈👈 It's recommended to always set the ImagePullPolicy to Always. To make sure that the imagePullSecrets are always correct, and to always get the image you want.
    livenessProbe: ## 👈👈👈 Missing property object `livenessProbe` - add a properly configured livenessProbe to catch possible deadlocks
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 10
      periodSeconds: 5
    readinessProbe: ## 👈👈👈 Missing property object `readinessProbe` - Add a properly configured readinessProbe to notify kubelet your Pods are ready for traffic
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

_End of Section_
