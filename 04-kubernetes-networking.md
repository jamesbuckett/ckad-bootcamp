# Kubernetes Tutorial - Kubernetes Networking

In this section:
- Service - A load balancer for Pods
- Ingress - Expose Kubernetes Services outside a Kubernetes cluster
- Network Policy - Software based firewall around Kubernetes Pods

![04-network](https://user-images.githubusercontent.com/18049790/135599284-67c0bc4c-b880-4b84-9a85-9c45497b02c7.jpg)

## Kubernetes Networking

<details class="faq box"><summary>Kubernetes Namespace (ns) - Logical isolation for your application</summary>
<p>

kubernetes.io bookmark: [Namespaces](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)

```bash
kubectl create namespace ns-bootcamp-networking
kubectl config set-context --current --namespace=ns-bootcamp-networking
```

</p>
</details>

<details class="faq box"><summary>Kubernetes Service (svc) - A load balancer for Pods</summary>
<p>

> Problem Statement: I want a stable network entry point into my application
> 
> tl;dr – Think Load balancer for individual microservices

![04-pod-svc](https://user-images.githubusercontent.com/18049790/139566952-602bacc9-03b3-4109-a85e-2ebd582024a7.jpg)

kubernetes.io bookmark: [Service](https://kubernetes.io/docs/concepts/services-networking/service/)

Create a Pod 

```bash
kubectl run service-pod --image=nginx --port=80  --labels="tier=web"
```

Create the Service

```bash
kubectl expose pod service-pod --port=8080 --target-port=80 --name=my-service
```
```bash
clear
# Check your work - run a diagnostics pod
kubectl run remote-run --image=busybox --restart=Never --rm -it
# Repeat this command to see different responses
wget -qO- my-service:8080
```


</p>
</details>

<details class="faq box"><summary>Kubernetes Ingress (ing) - Expose Kubernetes Services outside a Kubernetes cluster</summary>
<p>

> Problem Statement: I want a way to expose my application outside the Kubernetes cluster

> tl;dr – Want to open up your microservice application to the Internet with a fancy URL?

![04-pod-svc-ing](https://user-images.githubusercontent.com/18049790/139566958-21ce4c6b-aef0-4839-9628-a6f56ed67f8f.jpg)

kubernetes.io bookmark: [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)

Prerequisite Software for this example to work:
```bash
kubectl apply -f https://projectcontour.io/quickstart/contour.yaml
```

```bash
cat << EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress #👈👈👈 Ingress Name
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
            name: my-service #👈👈👈 Service Name
            port:
              number: 8080 #👈👈👈 Change: --port=8080
EOF
```

```bash
curl localhost
```

</p>
</details>

<details class="faq box"><summary>Kubernetes NetworkPolicy (netpol) - Software based firewall around Kubernetes Pods</summary>
<p>

> Problem Statement: I want a way to deny all network traffic around pods unless explicitly allowed
>
> tl;dr – Trust no one, explicitly define who talks to who with my software based firewall 

GUI for explaining and generating Network Policies: [editor.cilium.io](https://editor.cilium.io/)

kubernetes.io bookmark: [Declare Network Policy](https://kubernetes.io/docs/tasks/administer-cluster/declare-network-policy/)

Notes
* This does not work on Docker Desktop
* Network Policies are not enforced on Docker Desktop
* Run on alternative Cloud Provider

```bash
kubectl create deployment nginx --image=nginx
kubectl expose deployment nginx --port=80
```

```bash
kubectl run busybox --rm -ti --image=busybox -- /bin/sh
# wget --spider --timeout=1 nginx
```

Leave the busybox active in a shell. Open another shell and apply the network policy.

Deny All Traffic in Namespace
```bash
cat << EOF | kubectl apply -f -
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: default-deny-all
  namespace: default
spec:
  podSelector: {}
  ingress: []
  EOF
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

Come back to the busybox shell and check connectivity.

```bash
kubectl run busybox --rm -ti --image=busybox -- /bin/sh
# wget --spider --timeout=1 nginx
```

</p>
</details>

## Sample CKAD Questions

* [Sample CKAD Question - Service](https://github.com/jamesbuckett/ckad-questions/blob/main/04-ckad-services-networking.md#04-02-create-a-namespace-called-service-namespace-create-a-pod-called-service-pod-using-the-nginx-image-and-exposing-port-80-label-the-pod-tierweb-create-a-service-for-the-pod-called-my-service-allowing-for-communication-inside-the-cluster-let-the-service-expose-port-8080)
* [Sample CKAD Question - Ingress](https://github.com/jamesbuckett/ckad-questions/blob/main/04-ckad-services-networking.md#04-03-create-an-ingress-called-my-ingress-to-expose-the-service-my-service-from-previous-question-outside-the-cluster)
* [Sample CKAD Question - NetworkPolicy](https://github.com/jamesbuckett/ckad-questions/blob/main/04-ckad-services-networking.md#04-01-create-a-namespace-called-netpol-namespace-create-a-pod-called-web-pod-using-the-nginx-image-and-exposing-port-80-label-the-pod-tierweb-create-a-pod-called-app-pod-using-the-nginx-image-and-exposing-port-80-label-the-pod-tierapp-create-a-pod-called-db-pod-using-the-nginx-image-and-exposing-port-80-label-the-pod-tierdb-create-a-network-policy-called-my-netpol-that-allows-the-web-pod-to-only-egress-to-app-pod-on-port-80)

## Clean Up

<details class="faq box"><summary>Clean Up</summary>
<p>

```bash
cd
yes | rm -R ~/ckad/
kubectl delete ns ns-bootcamp-networking --grace-period 0 --force
```

_End of Section_
