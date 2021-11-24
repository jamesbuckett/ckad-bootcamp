# Kubernetes Tutorial - Kubernetes Security

 In this section:
- ServiceAccount - Functional ID inside the Pod to connect to the API server
- Role - Specify which verbs can be performed on which resources in a namespace
- RoleBinding - Bind roles to specific users, groups, or ServiceAccounts in a namespace
- ClusterRole -  Specify which verbs can be performed on which resources in a cluster
- ClusterRoleBinding - Bind roles to specific users, groups, or ServiceAccounts in a cluster
<br />

![07-01-k8s-bootcamp-security-overview](https://user-images.githubusercontent.com/18049790/142753738-acbde4b0-c02f-40f2-8e79-4030c7b3c5a2.jpg)
<br />

## Kubernetes Security

<details class="faq box"><summary>Enable RBAC on Docker Desktop</summary>
<p>

```bash
kubectl delete clusterrolebinding docker-for-desktop-binding
```

```bash
cat << EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: docker-for-desktop-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: Group
  name: system:serviceaccounts:kube-system
EOF  
```

Notes:

- Docker Desktop has a `ClusterRoleBinding` called [docker-for-desktop-binding](https://www.portainer.io/blog/docker-desktop-kubernetes-not-enforcing-rbac-rules) that gives `cluster-admin` privileges to all ServiceAccounts
- This means that any Pod running on Docker Desktop has cluster-admin privileges

</p>
</details>

<details class="faq box"><summary>Kubernetes Namespace (ns) - Logical isolation for your application</summary>
<p>

```bash
kubectl create namespace ns-bootcamp-sec
kubectl config set-context --current --namespace=ns-bootcamp-sec
```

</p>
</details>

<details class="faq box"><summary>ServiceAccount (sa) - Functional ID inside the Pod to connect to the API server</summary>
<p>

![07-02-k8s-bootcamp-security-sa](https://user-images.githubusercontent.com/18049790/142753742-6f209245-f3e1-4316-ba9d-45cb92f415c2.jpg)

kubernetes.io bookmark: [Configure Service Accounts for Pods](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)

```yaml
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-service-account
imagePullSecrets:
- name: my-image-pull-secret
EOF
```  

Notes:
- ServiceAccounts are namespace scoped
- A default serviceAccount is automatically created for each namespace
- ServiceAccounts are nothing more than a way for an application to authenticate itself with the Kubernetes API server
- ServiceAccount use [JSON Web Tokens](https://jwt.io/introduction) (JWT) to authenticate with the Kubernetes API server
- A ServiceAccount can contain a list of imagePullSecrets
- This saves you from having to include the imagePullSecret with each Pod

</p>
</details>

<details class="faq box"><summary>Role (Role) - specify which verbs can be performed on which resources</summary>
<p>

![07-03-k8s-bootcamp-security-role](https://user-images.githubusercontent.com/18049790/142753743-f17b0b0c-ba14-4555-bfe5-58e0c756a917.jpg)

```yaml
cat << EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: my-role
rules:
- apiGroups: [""]                    
  verbs: ["get", "list"]             
  resources: ["services"] 
EOF
```  

Notes:
* Roles are namespaced, if the namespace is omitted, the current namespace is used
* Services are resources in the core apiGroup, which has no name - hence the “”
* Getting individual Services by name and listing all of them is allowed
* This rule pertains to services, the plural name must be used

</p>
</details>

<details class="faq box"><summary>RoleBinding (RoleBinding) - bind roles to specific users, groups, or ServiceAccounts</summary>
<p>

![07-04-k8s-bootcamp-security-rolebinding](https://user-images.githubusercontent.com/18049790/142753744-898263f4-11e0-4850-83bf-caa83382aa5a.jpg)

```yaml
cat << EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  creationTimestamp: null
  name: my-rolebinding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: my-role
subjects:
- kind: ServiceAccount
  name: my-service-account  
EOF
```  

Notes: 
* This RoleBinding references the `my-role` Role
* And binds it to the `my-service-account` ServiceAccount 

</p>
</details>

<details class="faq box"><summary>kubectl auth can-i</summary>
<p>

```bash
kubectl describe role my-role
```

```console
Name:         my-role
Labels:       <none>
Annotations:  <none>
PolicyRule:
  Resources  Non-Resource URLs  Resource Names  Verbs
  ---------  -----------------  --------------  -----
  services   []                 []              [get list]
```

```bash
kubectl run service-pod --image=nginx --port=80  --labels="tier=web"
kubectl expose pod service-pod --port=8080 --target-port=80 --name=my-service
```

```bash
# Check the permission of the service account: my-service-account 
kubectl auth can-i --list --as=system:serviceaccount:ns-bootcamp-sec:my-service-account
```

```console
Resources                                       Non-Resource URLs                     Resource Names   Verbs
selfsubjectaccessreviews.authorization.k8s.io   []                                    []               [create]
selfsubjectrulesreviews.authorization.k8s.io    []                                    []               [create]
services                                        []                                    []               [get list] #👈👈👈
                                                [/.well-known/openid-configuration]   []               [get]
                                                [/api/*]                              []               [get]
                                                [/api]                                []               [get]
                                                [/apis/*]                             []               [get]
                                                [/apis]                               []               [get]
                                                [/healthz]                            []               [get]
                                                [/healthz]                            []               [get]
                                                [/livez]                              []               [get]
                                                [/livez]                              []               [get]
                                                [/openapi/*]                          []               [get]
                                                [/openapi]                            []               [get]
                                                [/openid/v1/jwks]                     []               [get]
                                                [/readyz]                             []               [get]
                                                [/readyz]                             []               [get]
                                                [/version/]                           []               [get]
                                                [/version/]                           []               [get]
                                                [/version]                            []               [get]
                                                [/version]                            []               [get]
```


```bash
# Permitted RBAC Action
kubectl auth can-i get services --as=system:serviceaccount:ns-bootcamp-sec:my-service-account
```

```console
yes
```

```bash
# Illegal RBAC Action
kubectl auth can-i get pods --as=system:serviceaccount:ns-bootcamp-sec:my-service-account
```

```console
no
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
kubectl delete ns ns-bootcamp-sec
kubectl delete sa my-serviceaccount
```

</p>
</details>

_End of Section_
