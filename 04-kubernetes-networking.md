# Kubernetes Tutorial - Kubernetes Networking

In this section:
- Service - A load balancer for Pods
- Ingress - Expose Kubernetes Services outside a Kubernetes cluster
- Network Policy - Software based firewall around Kubernetes Services

![00-pod](https://user-images.githubusercontent.com/18049790/135598360-f75b3c0d-1a41-4bb2-bf0e-cb983e819690.jpg)

## Kubernetes Networking

<details class="faq box"><summary>Kubernetes Namespace (ns)</summary>
<p>

### Namespace - Logical isolation for your application

kubernetes.io bookmark: [Namespaces](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)

```bash
kubectl create namespace ns-bootcamp-networking
kubectl config set-context --current --namespace=ns-bootcamp-networking
```

</p>
</details>

<details class="faq box"><summary>Kubernetes Service (svc)</summary>
<p>

### Service - A load balancer for Pods

xx

kubernetes.io bookmark: [Service](https://kubernetes.io/docs/concepts/services-networking/service/)

Create a Pod 

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

```bash
kubectl expose pod my-pod --port=8080 --target-port=80 --name=my-service
```

</p>
</details>

<details class="faq box"><summary>Kubernetes Ingress (ing)</summary>
<p>

### Ingress - Expose Kubernetes Services outside a Kubernetes cluster

kubernetes.io bookmark: [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)

```bash
cat << EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress #👈👈👈 Change: `my-ingress`
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - path: / #👈👈👈 Change
        pathType: Prefix
        backend:
          service:
            name: my-service #👈👈👈 Change: `my-service`
            port:
              number: 8080 #👈👈👈 Change: --port=8080
EOF                
```

```bash
clear
# Check your work - run a diagnostics pod
kubectl run remote-run --image=busybox --restart=Never --rm -it
# Repeat this command to see different responses
wget -qO- localhost
```

</p>
</details>

<details class="faq box"><summary>Kubernetes NetworkPolicy (netpol)</summary>
<p>

### Network Policy - Software based firewall around Kubernetes Services

Note to self: test this on Docker Desktop and Digital Ocean to see which works

kubernetes.io bookmark: [Declare Network Policy](https://kubernetes.io/docs/tasks/administer-cluster/declare-network-policy/)

```bash
kubectl create deployment nginx --image=nginx
kubectl expose deployment nginx --port=80
```

```bash
kubectl run busybox --rm -ti --image=busybox -- /bin/sh
# wget --spider --timeout=1 nginx
```

```bash
cat << EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: access-nginx
spec:
  podSelector:
    matchLabels:
      app: nginx
  ingress:
  - from:
    - podSelector:
        matchLabels:
          access: "true"
EOF         
```

```bash
kubectl run busybox --rm -ti --image=busybox -- /bin/sh
# wget --spider --timeout=1 nginx
```

</p>
</details>


### Sample CKAD Questions

* [Sample CKAD Question - Service](https://github.com/jamesbuckett/ckad-questions/blob/main/04-ckad-services-networking.md#04-02-create-a-namespace-called-service-namespace-create-a-pod-called-service-pod-using-the-nginx-image-and-exposing-port-80-label-the-pod-tierweb-create-a-service-for-the-pod-called-my-service-allowing-for-communication-inside-the-cluster-let-the-service-expose-port-8080)
* [Sample CKAD Question - Ingress](https://github.com/jamesbuckett/ckad-questions/blob/main/04-ckad-services-networking.md#04-03-create-an-ingress-called-my-ingress-to-expose-the-service-my-service-from-previous-question-outside-the-cluster)
* [Sample CKAD Question - NetworkPolicy](https://github.com/jamesbuckett/ckad-questions/blob/main/04-ckad-services-networking.md#04-01-create-a-namespace-called-netpol-namespace-create-a-pod-called-web-pod-using-the-nginx-image-and-exposing-port-80-label-the-pod-tierweb-create-a-pod-called-app-pod-using-the-nginx-image-and-exposing-port-80-label-the-pod-tierapp-create-a-pod-called-db-pod-using-the-nginx-image-and-exposing-port-80-label-the-pod-tierdb-create-a-network-policy-called-my-netpol-that-allows-the-web-pod-to-only-egress-to-app-pod-on-port-80)

</p>
</details>

_End of Section_
