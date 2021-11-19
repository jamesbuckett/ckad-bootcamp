# Kubernetes Tutorial - Kubernetes Networking

In this section:
- Service - A load balancer for Pods
- Ingress - Expose Kubernetes Services outside a Kubernetes cluster
- Network Policy - Software based firewall around Kubernetes Pods
<br />

![05-network-basic](https://user-images.githubusercontent.com/18049790/140637546-e3535aa3-cbd6-4dd9-bd39-37c466b6566a.jpg)
<br />

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

![05-network-svc](https://user-images.githubusercontent.com/18049790/140637642-5ef46de4-4867-41a6-ba44-6333cd9441af.jpg)

kubernetes.io bookmark: [Service](https://kubernetes.io/docs/concepts/services-networking/service/)

Notes
* The default kube-proxy mode for rule-based IP management is iptables, and the iptables mode native method for load distribution is random selection. 


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

![05-network-ing](https://user-images.githubusercontent.com/18049790/140637548-d1a9ced9-7c66-406c-86d3-1a7001de2e75.jpg)

kubernetes.io bookmark: [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)

Notes
* Ingress operates using a controller with an Ingress resource and a daemon. 
* The Ingress resource is a set of rules governing traffic. 
  * kind: Ingress
* The daemon applies the rules inside a specialized Kubernetes pod. 
  * Envoy DaemonSet

Prerequisite Software for this example to work:
```bash
kubectl apply -f https://projectcontour.io/quickstart/contour.yaml
```

```yaml
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

![05-netpol](https://user-images.githubusercontent.com/18049790/140638229-62871b17-bc71-4e51-a71c-4c75c178a78f.jpg)

GUI for explaining and generating Network Policies: [editor.cilium.io](https://editor.cilium.io/)

kubernetes.io bookmark: [Declare Network Policy](https://kubernetes.io/docs/tasks/administer-cluster/declare-network-policy/)

Notes 
* Network policies do not conflict; they are additive. 
* If any policy or policies select a pod, the pod is restricted to what is allowed by the union of those policies' ingress/egress rules. 
* Thus, order of evaluation does not affect the policy result.


```diff
Please NOTE:
- Docker Desktop does not support CNI (container network interface) so the NetworkPolicy's define are ignored.
- The commands work but the NetworkPolicy's are not enforced
- Perform this on any cluster that enforces Network Policies
```

* [Sample CKAD Question - NetworkPolicy](https://github.com/jamesbuckett/ckad-questions/blob/main/04-ckad-services-networking.md#04-01-create-a-namespace-called-netpol-namespace-create-a-pod-called-web-pod-using-the-nginx-image-and-exposing-port-80-label-the-pod-tierweb-create-a-pod-called-app-pod-using-the-nginx-image-and-exposing-port-80-label-the-pod-tierapp-create-a-pod-called-db-pod-using-the-nginx-image-and-exposing-port-80-label-the-pod-tierdb-create-a-network-policy-called-my-netpol-that-allows-the-web-pod-to-only-egress-to-app-pod-on-port-80)


</p>
</details>
<br />

## Clean Up

<details class="faq box"><summary>Clean Up</summary>
<p>

```bash
cd
yes | rm -R ~/ckad/
kubectl delete ns ns-bootcamp-networking --now
```

_End of Section_
