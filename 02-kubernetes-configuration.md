# Kubernetes Tutorial - Configuration

- Resource Quota - Namespace restriction on total resource usage
- Limit Range - A policy to constrain resource allocations (to Pods or Containers) in a namespace
- Configuration Map - Storing environmental variables per namespace
- Secret  - Storing obfuscated environmental variables per namespace
<br />

![01-config](https://user-images.githubusercontent.com/18049790/135598375-34b557bc-96fd-499b-bc02-c2bb0c743199.jpg)
<br />

## Kubernetes Configuration

<details class="faq box"><summary>Kubernetes Namespace (ns) - Logical isolation for your application</summary>
<p>

```bash
kubectl create namespace ns-bootcamp-configuration
kubectl config set-context --current --namespace=ns-bootcamp-configuration
```

</p>
</details>

<details class="faq box"><summary>Kubernetes ResourceQuota (quota) - Namespace restriction on total resource usage</summary>
<p>

> Problem Statement:  I want to prevent resource contention and "land grabs" by controlling how much CPU or memory an application can consume.
>
> tl;dr – CPU and Memory constraints for the namespace

![03-quota](https://user-images.githubusercontent.com/18049790/140637139-550aec0f-1e2e-4988-9d31-b0b62efaf77b.jpg)

kubernetes.io bookmark: [Create a ResourceQuota](https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/quota-memory-cpu-namespace/#create-a-resourcequota)

```yaml
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: ResourceQuota
metadata:
  name: my-quota  
spec:
  hard:
    cpu: 500Mi
    memory: 2G
EOF
```

Observation:
* Start Octant
* Go to  `Cluster Overview` on left side
* Go to `Namespaces`
* Scroll down to `Resource Quotas`

</p>
</details>

<details class="faq box"><summary>Kubernetes LimitRange (limits) A policy to constrain resource allocations (to Pods or Containers) in a namespace</summary>
<p>

![03-quota](https://user-images.githubusercontent.com/18049790/140637139-550aec0f-1e2e-4988-9d31-b0b62efaf77b.jpg)

> Problem Statement: I want to set default CPU and Memory allocations for my microservices if missing
>
> tl;dr – Forgot to define CPU and Memory in your Pod spec, no worries let me set some defaults for you

```yaml
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: LimitRange
metadata:
  name: my-limit-range  
spec:
  limits:
  - default:
      cpu: 250m
      memory: 256Mi
    defaultRequest:
      cpu: 100m
      memory: 128Mi
    type: Container 
EOF
```

Observation:
* Start Octant
* Go to  `Cluster Overview` on left side
* Go to `Namespaces`
* Scroll down to `Resource Limits`

</p>
</details>

<details class="faq box"><summary>Kubernetes ConfigMap (cm) - Storing environmental variables per namespace</summary>
<p>

> Problem Statement: I want to store configuration (environmental variables) in the environment
>
> tl;dr – All configuration data should be stored separately from the code

![03-cm](https://user-images.githubusercontent.com/18049790/140637134-aa560ccb-cba8-47be-9f3a-3e8cc19d719a.jpg)

kubernetes.io bookmark: [Define a container environment variable with data from a single ConfigMap](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#define-a-container-environment-variable-with-data-from-a-single-configmap)

```yaml
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-configmap  
data:
  error-log-level: notice
EOF
```

</p>
</details>

<details class="faq box"><summary>Kubernetes Secret  - Storing obfuscated environmental variables per namespace</summary>
<p>

> Problem Statement: I want a way to obfuscate my environmental variables
> 
> tl;dr – base64 encoded environment variables in memory

![03-secret](https://user-images.githubusercontent.com/18049790/140637140-709ae658-74e4-4a76-aa5f-b2751cb1e4c7.jpg)

kubernetes.io bookmark: [Using Secrets as environment variables](https://kubernetes.io/docs/concepts/configuration/secret/#using-secrets-as-environment-variables)

```yaml
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: Secret
metadata:
  name: my-secret  
data:
  password: MTIzNDU2
  user: Ym9i
kind: Secret
metadata:
  name: my-secret
EOF
```

</p>
</details>

<details class="faq box"><summary>Kubernetes Secret  - The Laws of Three</summary>
<p>

> tl;dr – Kubernetes always respects the Law of Three

<details class="faq box"><summary>Kubernetes Secret  - Types of Secret</summary>
<p>

There are three types of secret: (TYPES)
* [generic](https://kubernetes.io/docs/concepts/configuration/secret/#opaque-secrets) #👈👈👈 Part of CKAD exam
  * Create a secret from a local file, directory, or literal value
* [tls](https://kubernetes.io/docs/concepts/configuration/secret/#tls-secrets) 
  * Create a TLS secret
* [docker-registry](https://kubernetes.io/docs/concepts/configuration/secret/#docker-config-secrets)
  * Create a secret for use with a Docker registry

</p>
</details>


<details class="faq box"><summary>Kubernetes Secret  - Create a Secret</summary>
<p>

There are three ways to create a secret: (CREATION)
* [create Secret using kubectl command](https://kubernetes.io/docs/tasks/configmap-secret/managing-secret-using-kubectl/) #👈👈👈 Part of CKAD exam
* [create Secret from config file](https://kubernetes.io/docs/tasks/configmap-secret/managing-secret-using-config-file/)
* [create Secret using kustomize](https://kubernetes.io/docs/tasks/configmap-secret/managing-secret-using-kustomize/)

</p>
</details>


<details class="faq box"><summary>Kubernetes Secret  - Consume a Secret</summary>
<p>

There are three ways to use a secret: (CONSUMPTION)
* As files in a volume mounted on one or more of its containers
* As container environment variable #👈👈👈 Part of CKAD exam
* By the `kubelet` when pulling images for the Pod

</p>
</details>

</p>
</details>

<details class="faq box"><summary>Example: A Pod consuming the ConfigMap and Secret</summary>
<p>

![03-pod-cm-sec](https://user-images.githubusercontent.com/18049790/140637137-09125908-0320-49c2-aafd-c91b956bddbf.jpg)

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
    env: #👈👈👈 Environment Variable section
      - name: error-log-level
        valueFrom:
          configMapKeyRef:
            name: my-configmap  #👈👈👈 Configuration Map
            key: error-log-level
      - name: SECRET-ENV-USER
        valueFrom:
          secretKeyRef:
            name: my-secret  #👈👈👈 Secret
            key: user
      - name: SECRET-ENV-PASSWORD
        valueFrom:
          secretKeyRef:
            name: my-secret  #👈👈👈 Secret
            key: password
EOF
```

Observation
* Start Octant
* Go to  `Workloads`...`Pods`
* Select `my-pod`...`Terminal`...type `env`

```bash
# kubectl exec --stdin --tty my-pod -- /bin/bash

root@my-pod:/# env
error-log-level=notice #👈👈👈
SECRET-ENV-USER=bob #👈👈👈
SECRET-ENV-PASSWORD=123456 #👈👈👈
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_SERVICE_PORT=443
HOSTNAME=my-pod
PWD=/
PKG_RELEASE=1~buster
HOME=/root
KUBERNETES_PORT_443_TCP=tcp://10.245.0.1:443
NJS_VERSION=0.5.3
TERM=xterm
SHLVL=1
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP_ADDR=10.245.0.1
KUBERNETES_SERVICE_HOST=10.245.0.1
KUBERNETES_PORT=tcp://10.245.0.1:443
KUBERNETES_PORT_443_TCP_PORT=443
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
NGINX_VERSION=1.20.0
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
kubectl delete ns ns-bootcamp-configuration --grace-period 0 --force
```

</p>
</details>
<br />

Next [Kubernetes Tutorial - Kubernetes Workloads](https://github.com/jamesbuckett/ckad-bootcamp/blob/master/03-kubernetes-workloads.md)

_End of Section_
