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
* The default kube-proxy mode for rule-based IP management is `iptables`
*  The `iptables` mode native method for load distribution is random selection
* In English - No round robin load balancing for Kubernetes Service, it is random selection

Create a Pod 

```bash
kubectl run service-pod --image=nginx --port=80  --labels="tier=web"
```

Create the Service

```bash
kubectl expose pod service-pod --port=8080 --target-port=80 --name=my-service --type=ClusterIP
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

<details class="faq box"><summary>Kubernetes Service (svc) -  Types of Service</summary>
<p>

> tl;dr – Kubernetes always respects the Law of Three, sometimes Four

There are four types of Kubernetes service:

<details class="faq box"><summary>Kubernetes Service (svc) - ClusterIP (default)</summary>
<p>

* [ClusterIP](https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types): #👈👈👈 Part of CKAD exam
  * Exposes the Service on a cluster-internal IP
  * Choosing this value makes the Service only reachable from within the cluster 
  * This is the default ServiceType

</p>
</details>

<details class="faq box"><summary>Kubernetes Service (svc) -  NodePort (insecure)</summary>
<p>

* [NodePort](https://kubernetes.io/docs/concepts/services-networking/service/#type-nodeport): #👈👈👈 Part of CKAD exam
  * A NodePort is an open port on every node of your cluster 
  * When traffic is received on that open port, it directs it to a specific port on the ClusterIP for the service it is representing
  * You will  be able to contact the NodePort Service, from outside the cluster, by requesting `NodeIP:NodePort`
  * Do NOT do this in Production

```yaml
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: my-nodeport-service
  namespace: ns-bootcamp-networking
spec:
  type: NodePort #👈👈👈
  selector:
    tier: web
  ports:    
    - port: 8080
      targetPort: 80
      # By default and for convenience, the Kubernetes control plane will allocate a port from a range (default: 30000-32767)
      nodePort: 30007 #👈👈👈
EOF
```

```bash
# NodeIP:NodePort
# NodeIP = kubectl get nodes -o wide 
# NodePort = nodePort: 30007
wget -qO- localhost:30007
```

</p>
</details>

<details class="faq box"><summary>Kubernetes Service (svc) -  LoadBalancer (expensive)</summary>
<p>

* [LoadBalancer](https://kubernetes.io/docs/concepts/services-networking/service/#loadbalancer): 
  * Expensive if you deploy a Cloud Load Balancer for each Service
  * Exposes the Service externally using a cloud provider's load balancer
  * Quickly went out of fashion and was addressed by [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) which consolidates services and routes to a single Cloud Load Balancer

</p>
</details>

<details class="faq box"><summary>Kubernetes Service (svc) -  ExternalName (DNS)</summary>
<p>

* [ExternalName](https://kubernetes.io/docs/concepts/services-networking/service/#externalname): 
  * Services of type ExternalName map a Service to a DNS name
  * Maps the Service to the contents of the externalName field (e.g. foo.bar.example.com), by returning a CNAME record with its value
  * No proxying of any kind is set up

```yaml
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: my-externalname-service
  namespace: ns-bootcamp-networking
spec:
  type: ExternalName #👈👈👈
  externalName: www.google.com
EOF
```

</p>
</details>

</p>
</details>

<details class="faq box"><summary>Kubernetes Ingress (ing) - Expose Kubernetes Services outside a Kubernetes cluster</summary>
<p>

> Problem Statement: I want a way to expose my application outside the Kubernetes cluster

> tl;dr – Think Layer 7 Load balancer for individual microservices

![05-network-ing](https://user-images.githubusercontent.com/18049790/140637548-d1a9ced9-7c66-406c-86d3-1a7001de2e75.jpg)

kubernetes.io bookmark: [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)

* Ingress operates using three constructs:
  * Ingress Controller 
    * Control Plane for Ingress
  * Ingress Resources
    * Ingress Traffic Rules #👈👈👈 These are the YAML files that you will work with
  * Ingress DaemonSet
    * Execution Plane for Ingress
    * Cluster wide pods that apply the traffic rules
    
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

Notes on [rewrite-target](https://kubernetes.github.io/ingress-nginx/examples/rewrite/)

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
* Network policies do not conflict; they are additive
* If any policy or policies select a pod, the pod is restricted to what is allowed by the union of those policies' ingress/egress rules
* Thus, order of evaluation does not affect the policy result


```diff
Please NOTE:
- Docker Desktop does not support CNI (container network interface) so the NetworkPolicy's define are ignored.
- The commands work but the NetworkPolicy's are not enforced
- Perform this on any cluster that enforces Network Policies
```

[Sample CKAD Question - NetworkPolicy](https://github.com/jamesbuckett/ckad-questions/blob/main/04-ckad-services-networking.md#04-01-create-a-namespace-called-netpol-namespace-create-a-pod-called-web-pod-using-the-nginx-image-and-exposing-port-80-label-the-pod-tierweb-create-a-pod-called-app-pod-using-the-nginx-image-and-exposing-port-80-label-the-pod-tierapp-create-a-pod-called-db-pod-using-the-nginx-image-and-exposing-port-80-label-the-pod-tierdb-create-a-network-policy-called-my-netpol-that-allows-the-web-pod-to-only-egress-to-app-pod-on-port-80)

</p>
</details>


<details class="faq box"><summary>Kubernetes NetworkPolicy (netpol) - Types of Selector</summary>
<p>

> tl;dr – Kubernetes always respects the Law of Three 

* [podSelector ](https://kubernetes.io/docs/concepts/services-networking/network-policies/#behavior-of-to-and-from-selectors)
  * This selects particular Pods in the same namespace as the NetworkPolicy which should be allowed as ingress sources or egress destinations
* [namespaceSelector](https://kubernetes.io/docs/concepts/services-networking/network-policies/#behavior-of-to-and-from-selectors)
  * This selects particular namespaces for which all Pods should be allowed as ingress sources or egress destinations
* [ipBlock](https://kubernetes.io/docs/concepts/services-networking/network-policies/#behavior-of-to-and-from-selectors)
  * This selects particular IP CIDR ranges to allow as ingress sources or egress destinations
  * These should be cluster-external IPs, since Pod IPs are ephemeral and unpredictable.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector: #👈👈👈 To which pod does this Network Policy apply: label = role: db
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - ipBlock: #👈👈👈This selects particular IP CIDR ranges
        cidr: 172.17.0.0/16
        except:
        - 172.17.1.0/24
    - namespaceSelector: #👈👈👈This selects particular namespaces
        matchLabels:
          project: myproject
    - podSelector: #👈👈👈This selects particular Pods
        matchLabels:
          role: frontend
    ports:
    - protocol: TCP
      port: 6379
  egress:
  - to:
    - ipBlock:
        cidr: 10.0.0.0/24
    ports:
    - protocol: TCP
      port: 5978
```

</p>
</details>

<details class="faq box"><summary>Kubernetes NetworkPolicy (netpol) - AND & OR Rules</summary>
<p>

OR Rule

```yaml
  ingress:
  - from:
    - ipBlock: #👈👈👈- = First Rule OR 
        cidr: 172.17.0.0/16
        except:
        - 172.17.1.0/24
    - namespaceSelector: #👈👈👈- = Second Rule OR
        matchLabels:
          project: myproject
    - podSelector: #👈👈👈- = Third Rule
        matchLabels:
          role: frontend
```

AND Rule

```yaml
  ingress:
  - from:
    - ipBlock: #👈👈👈- = First Rule First Element AND  
        cidr: 172.17.0.0/16
        except:
        - 172.17.1.0/24
      namespaceSelector: #👈👈👈 First Rule Second Element OR
        matchLabels:
          project: myproject
    - podSelector: #👈👈👈- = Second Rule
        matchLabels:
          role: frontend
```

DNS Rule

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: internal-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      name: internal #👈👈👈To which Pod does this Policy apply 
  policyTypes:
  - Egress
  - Ingress
  ingress:
    - {}
  egress:
  - to: #👈👈👈 First Rule Egress to mysql on port 3306
    - podSelector:
        matchLabels:
          name: mysql
    ports:
    - protocol: TCP
      port: 3306

  - to: #👈👈👈 Second Rule Egress to payroll on port 8080
    - podSelector:
        matchLabels:
          name: payroll
    ports:
    - protocol: TCP
      port: 8080

  - ports: #👈👈👈 Third Rule Egress to DNS on port 53
    - port: 53
      protocol: UDP
    - port: 53
      protocol: TCP
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
kubectl delete ns ns-bootcamp-networking --now
```

</p>
</details>
<br />

Next [Kubernetes Tutorial - Kubernetes Storage](https://github.com/jamesbuckett/ckad-bootcamp/blob/master/05-kubernetes-storage.md)

_End of Section_
