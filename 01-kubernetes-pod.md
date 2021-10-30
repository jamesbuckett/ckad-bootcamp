# Kubernetes Tutorial - Kubernetes Pod & Friends

In this section:

- Pod - A pod is the smallest execution unit in Kubernetes
- Limits and Requests - CPU and Memory reservation for a Pod
- Liveness and Readiness - Health Checks for a Pod
- ServiceAccount - Functional ID inside the Pod to connect to the API server
- Security Context - Linux privilege and access control settings for a Pod or Container

![00-pod](https://user-images.githubusercontent.com/18049790/135598360-f75b3c0d-1a41-4bb2-bf0e-cb983e819690.jpg)

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

xx

> Problem Statement: I want to run immutable and resilient Linux workloads 
>
> tl;dr – Compute, Network, Storage and Monitoring around your application

kubernetes.io bookmark: [Using Pods](https://kubernetes.io/docs/concepts/workloads/pods/#using-pods)

```bash
clear
# Create the pod via the command line imperatively
kubectl run my-pod –image=nginx:1.20.0 –port=80
```

Output:

```bash
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

xx

> Problem Solving: I want to guarantee CPU and RAM for my microservice
>
> tl;dr – Give me guaranteed CPU and RAM 


kubernetes.io bookmark: [Meaning of Memory](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#meaning-of-memory)

```bash
mkdir -p ~/ckad/
vi ~/ckad/01-k8s-bootcamp-pod-resources.yml
```

```bash
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

```bash
kubectl apply -f ~/ckad/01-k8s-bootcamp-pod-resources.yml
```

</p>
</details>

<details class="faq box"><summary>Liveness and Readiness Probes - Health Checks for a Pod</summary>
<p>

xx

> Problem Solving: I want a way to check the health of my microservices application

> tl;dr – How to perform Health Checks on my microservices application 


kubernetes.io bookmark: [Define a liveness HTTP request](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/#define-a-liveness-http-request)

```bash
mkdir -p ~/ckad/
vi ~/ckad/02-k8s-bootcamp-pod-probes.yml
```

```bash
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

```bash
mkdir -p ~/ckad/
vi ~/ckad/02-k8s-bootcamp-pod-probes.yml
```

</p>
</details>

<details class="faq box"><summary>ServiceAccount (sa) - Functional ID inside the Pod to connect to the API server</summary>
<p>

xx

kubernetes.io bookmark: [Configure Service Accounts for Pods](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)

```bash
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

## Sample CKAD Questions

* [Sample CKAD Question - Container Name](https://github.com/jamesbuckett/ckad-questions/blob/main/01-ckad-design-build.md#01-02-create-a-namespace-called-pod-namespace-create-a-pod-called-pod-1-using-nginx-image-the-container-in-the-pod-should-be-named-container-1)

* [Sample CKAD Question - ServiceAccount](https://github.com/jamesbuckett/ckad-questions/blob/main/02-ckad-env-configuration-security.md#02-04-create-a-namespace-called-serviceaccount-namespace-create-a-pod-called-serviceaccount-pod-using-nginx-image-create-a-seviceaccount-called-my-serviceaccount-update-the-pod-to-use-the-new-serviceaccount-display-the-token-for-the-new-serviceaccount)


_End of Section_
