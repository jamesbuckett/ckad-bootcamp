# Kubernetes Tutorial - Kubernetes Pod & Friends

 A pod should be considered as the smallest unit of execution in Kubernetes and all Kubernetes workloads run Pods at their core.

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

```bash
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

> Problem Solving: I want to guarantee CPU and RAM for my microservice
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

Tools
* [kube-score](https://github.com/zegl/kube-score) - kube-score is a tool that performs static code analysis of your Kubernetes object definitions.
* [datree](https://www.datree.io/) - Prevent Kubernetes Misconfigurations From Reaching Production

Usage
* `kubes-core score ~/ckad/01-kubernetes-pod-basic-pod.yml`
* `datree test  ~/ckad/01-kubernetes-pod-basic-pod.yml`

<details class="faq box"><summary>Pod Best Practices</summary>
<p>

```bash
clear
mkdir -p ~/ckad/
kubectl run my-pod --image=nginx:1.20.0 --port=80 --dry-run=client -o yaml > ~/ckad/01-kubernetes-pod-basic-pod.yml
```

Before:
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
  - image: nginx:1.20.0
    name: my-pod
    ports:
    - containerPort: 80
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

After:
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
    imagePullPolicy: Always ## 👈👈👈 It's recommended to always set the ImagePullPolicy to Always.
                            ## 👈👈👈 To make sure that the imagePullSecrets are always correct, and to always get the image you want.
    livenessProbe: ## 👈👈👈 Missing property object `livenessProbe` - add a properly configured livenessProbe to catch possible deadlocks
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 10
      periodSeconds: 5
    readinessProbe: ## 👈👈👈 Missing property object `readinessProbe` - add a properly configured readinessProbe to notify kubelet your Pods are ready for traffic
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 10
      periodSeconds: 5
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

</p>
</details>
<br />

## Sample CKAD Questions

* [Sample CKAD Question - Container Name](https://github.com/jamesbuckett/ckad-questions/blob/main/01-ckad-design-build.md#01-02-create-a-namespace-called-pod-namespace-create-a-pod-called-pod-1-using-nginx-image-the-container-in-the-pod-should-be-named-container-1)

* [Sample CKAD Question - ServiceAccount](https://github.com/jamesbuckett/ckad-questions/blob/main/02-ckad-env-configuration-security.md#02-04-create-a-namespace-called-serviceaccount-namespace-create-a-pod-called-serviceaccount-pod-using-nginx-image-create-a-seviceaccount-called-my-serviceaccount-update-the-pod-to-use-the-new-serviceaccount-display-the-token-for-the-new-serviceaccount)
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
