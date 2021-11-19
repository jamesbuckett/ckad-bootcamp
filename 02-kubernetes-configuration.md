# Kubernetes Tutorial - Configuration

In this section:

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

<details class="faq box"><summary>A Pod consuming the ConfigMap and Secret</summary>
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


_End of Section_
