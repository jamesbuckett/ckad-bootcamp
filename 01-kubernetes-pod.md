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

> Problem Statement: I want to run immutable and resilient Linux workloads 
>
> tl;dr – Compute, Network, Storage and Monitoring around your application

![01-pod-basic](https://user-images.githubusercontent.com/18049790/139566637-f641afe6-67b1-4066-88e3-b4762e8b0ebf.jpg)

kubernetes.io bookmark: [Using Pods](https://kubernetes.io/docs/concepts/workloads/pods/#using-pods)

```bash
clear
# Create the pod via the command line imperatively
kubectl run my-pod --image=nginx:1.20.0 --port=80
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

> Problem Solving: I want to guarantee CPU and RAM for my microservice
>
> tl;dr – Let me make a CPU and RAM reservation

![01-pod-cpu-ram](https://user-images.githubusercontent.com/18049790/139566107-f1c831ff-2d97-48bb-beb3-a45df7bb8371.jpg)

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
Relationship between Resources and Quality of Service

![01-pod-qos](https://user-images.githubusercontent.com/18049790/139566111-74dcc981-a0d6-47a1-900a-02c65314314d.jpg)

</p>
</details>

<details class="faq box"><summary>Liveness and Readiness Probes - Health Checks for a Pod</summary>
<p>

> Problem Solving: I want a way to check the health of my microservices application

> tl;dr – How to perform Health Checks on my microservices application 

![01-pod-probes](https://user-images.githubusercontent.com/18049790/139566110-9b9b79ef-225e-49ec-8a8d-af630ced639e.jpg)

kubernetes.io bookmark: [Define a liveness HTTP request](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/#define-a-liveness-http-request)

```bash
kubectl delete pod my-pod --grace-period 0 --force
clear
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

</p>
</details>

<details class="faq box"><summary>ServiceAccount (sa) - Functional ID inside the Pod to connect to the API server</summary>
<p>

![01-pod-sa](https://user-images.githubusercontent.com/18049790/139566115-4d03c529-8db7-49e9-9a35-ef54136ade72.jpg)

kubernetes.io bookmark: [Configure Service Accounts for Pods](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)

```bash
kubectl delete pod my-pod --grace-period 0 --force
clear
```

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
