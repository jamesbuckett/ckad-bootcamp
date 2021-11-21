# Kubernetes Tutorial - Kubernetes Security

 In this section:
- ServiceAccount - Functional ID inside the Pod to connect to the API server
- Roles and RoleBindings - Who can do what inside a namespace
- ClusterRoles and ClusterRoleBindings - Who can do what on the cluster
<br />

Security North Star
<br />

## Kubernetes Security

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

xxx

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
* ServiceAccounts are namespace scoped
* A default serviceAccount is automatically created for each namespace
* ServiceAccounts are nothing more than a way for an application to authenticate itself with the Kubernetes API server
* ServiceAccount use JSON Web Tokens to authenticate with the Kubernetes API server
* A ServiceAccount can contain a list of imagePullSecrets
* This saves you from having to include the imagePullSecret with each Pod

</p>
</details>

<details class="faq box"><summary>Role - specify which verbs can be performed on which resources</summary>
<p>

xxx

```yaml
cat << EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: my-service-reader-role
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


<details class="faq box"><summary>RoleBinding - bind roles to specific users, groups, or ServiceAccounts</summary>
<p>

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

</p>
</details>



<details class="faq box"><summary>Example Pod with RBAC</summary>
<p>


</p>
</details>
<br />

```bash
kubectl delete pod my-pod --now
clear
```








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
