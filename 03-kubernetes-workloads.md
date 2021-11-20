# Kubernetes Tutorial - Kubernetes Workloads

All workloads run Pods at their core.

In this section:
* Deployments - Resilient, scalable long running workload with zero downtime upgrades
* Jobs - Running a workload to guaranteed completion
* CronJobs - Running a workload to guaranteed completion on a time schedule
* Stateful Sets - Running a workload with persistent storage
* Daemon Sets - Running a workload on every Linux worker node
<br />

## Kubernetes Workloads

<details class="faq box"><summary>Kubernetes Namespace (ns) - Logical isolation for your application</summary>
<p>

```bash
kubectl create namespace ns-bootcamp-workloads
kubectl config set-context --current --namespace=ns-bootcamp-workloads
```

</p>
</details>

<details class="faq box"><summary>Kubernetes Deployment (deploy) - Resilient, scalable long running workload with zero downtime upgrades</summary>
<p>

> Problem Statement: 
> I want to have zero downtime when doing software deployments and I want a resilient microservices application

> tl;dr – The thing that controls how you do upgrades with zero-downtime, also resilience with replicas

![04-deployment](https://user-images.githubusercontent.com/18049790/140640341-b72a123b-befe-4701-8b92-130f25c062bf.jpg)

kubernetes.io bookmark: [Creating a Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#creating-a-deployment)

```yaml
cat << EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
EOF
```

</p>
</details>

<details class="faq box"><summary>Kubernetes Job - Running a workload to guaranteed completion</summary>
<p>

> Problem Statement: I want to run a Linux workload to guaranteed completion
> 
> tl;dr – Want to run a batch job? 

![04-job](https://user-images.githubusercontent.com/18049790/140640468-f448ce27-2c29-4081-b81b-00acfdcb63cb.jpg)

kubernetes.io bookmark: [Running an example Job](https://kubernetes.io/docs/concepts/workloads/controllers/job/#running-an-example-job)

* This example Job manifest computes π to 2000 places and prints it out:

```yaml
cat << EOF | kubectl apply -f -
apiVersion: batch/v1
kind: Job
metadata:
  name: pi
spec:
  template:
    spec:
      containers:
      - name: pi
        image: perl
        command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
  backoffLimit: 4
EOF
```

```bash
pods=$(kubectl get pods --selector=job-name=pi --output=jsonpath='{.items[*].metadata.name}')
kubectl logs $pods
```

</p>
</details>


<details class="faq box"><summary>Kubernetes CronJob (cj)  - Running a workload to guaranteed completion on a time schedule</summary>
<p>

> Problem Statement: I want to run a Linux workload to guaranteed completion on a particular schedule
> 
> tl;dr – Want to run a cronjob?

![04-cronjob](https://user-images.githubusercontent.com/18049790/140640514-e7eb5127-f772-4ea7-94dd-c6ddea78ee30.jpg)

kubernetes.io bookmark: [CronJob](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/#example)

* This example CronJob manifest prints the current time and a hello message every minute:

```yaml
cat << EOF | kubectl apply -f -
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox
            imagePullPolicy: IfNotPresent
            command:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
EOF
```

</p>
</details>

<details class="faq box"><summary>Kubernetes StatefulSet (sts) - Running a workload with persistent storage</summary>
<p>

StatefulSet is the workload API object used to manage stateful applications.
* Manages the deployment and scaling of a set of Pods, and provides guarantees about the ordering and uniqueness of these Pods.

kubernetes.io bookmark: [Creating a StatefulSet](https://kubernetes.io/docs/tutorials/stateful-application/basic-stateful-set/)

* This example StatefulSet manifest creates a headless Service, nginx, to publish the IP addresses of Pods in the StatefulSet, web:

```yaml
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx"
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: k8s.gcr.io/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
EOF
```

</p>
</details>

<details class="faq box"><summary>Kubernetes DaemonSet (ds) - Running a workload on every worker node</summary>
<p>

> Problem Statement: I want to run a Linux workload on every single worker node
> 
> tl;dr – Want to run everywhere in the cluster?

kubernetes.io bookmark: [Writing a DaemonSet Spec](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/#writing-a-daemonset-spec)

```yaml
cat << EOF | kubectl apply -f -
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-elasticsearch
  namespace: kube-system
  labels:
    k8s-app: fluentd-logging
spec:
  selector:
    matchLabels:
      name: fluentd-elasticsearch
  template:
    metadata:
      labels:
        name: fluentd-elasticsearch
    spec:
      tolerations:
      # this toleration is to have the daemonset runnable on master nodes
      # remove it if your masters can't run pods
      - key: node-role.kubernetes.io/master
        operator: Exists
        effect: NoSchedule
      containers:
      - name: fluentd-elasticsearch
        image: quay.io/fluentd_elasticsearch/fluentd:v2.5.2
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      terminationGracePeriodSeconds: 30
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
EOF
```

</p>
</details>
<br />

## Kubernetes Workload Best Practices 

<details class="faq box"><summary>Static Code Analysis - Deployment</summary>
<p>

* [kube-score](https://github.com/zegl/kube-score) - kube-score is a tool that performs static code analysis of your Kubernetes object definitions

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

```console
    [WARNING] Deployment has host PodAntiAffinity
      · Deployment does not have a host podAntiAffinity set
      It's recommended to set a podAntiAffinity that stops multiple pods from a deployment from being scheduled on the same node. 
      This increases availability in case the node becomes unavailable.
    [CRITICAL] Deployment has PodDisruptionBudget
      · No matching PodDisruptionBudget was found
      It's recommended to define a PodDisruptionBudget to avoid unexpected downtime during Kubernetes maintenance operations 
      Such as when draining a node.
```

</p>
</details>

<details class="faq box"><summary>Kubernetes Deployment (deploy) - PodAntiAffinity</summary>
<p>

```console
    [WARNING] Deployment has host PodAntiAffinity
      · Deployment does not have a host podAntiAffinity set
      It's recommended to set a podAntiAffinity that stops multiple pods from a deployment from being scheduled on the same node. 
      This increases availability in case the node becomes unavailable.
```

![PodAntiAffinity](https://user-images.githubusercontent.com/18049790/142724430-cd88bb28-e023-4390-9681-31a38dc956a8.jpg)

Notes:
  * tl;dr - Don't put all your eggs in the same basket 
  * podAntiAffinity stops multiple pods from a deployment from being scheduled on the same node
  * This increases availability in case a node becomes unavailable

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      affinity: ##  👈👈👈 
        podAntiAffinity: ##  👈👈👈 
          requiredDuringSchedulingIgnoredDuringExecution: ##  👈👈👈  Hard anti-affinity - guarantees the distribution
          - labelSelector:
              matchExpressions: ##  👈👈👈  Pod should not be scheduled on the node if label app=nginx already present
              - key: app ##  👈👈👈  app=nginx
                operator: In
                values:
                - nginx ##  👈👈👈 app=nginx
            topologyKey: kubernetes.io/hostname
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

</p>
</details>

<details class="faq box"><summary>Kubernetes Deployment (deploy) - PodDisruptionBudget</summary>
<p>

```console
    [CRITICAL] Deployment has PodDisruptionBudget
      · No matching PodDisruptionBudget was found
      It's recommended to define a PodDisruptionBudget to avoid unexpected downtime during Kubernetes maintenance operations
      Such as when draining a node.
```

![PodDisruptionBudget](https://user-images.githubusercontent.com/18049790/142724615-93ecefda-cfcf-45c8-bd8b-f220a2908b62.jpg)

Notes: 
* tl;dr - Please cluster administrators, cool your jets, this is a critical application and I need my application to be available
* A PodDisruptionBudget defines the budget of voluntary disruption during scheduled maintenance 
* If you do not have a PodDisruptionBudget in place your workload might go offline when a cluster maintenance event is in place
* Ensures a certain number or percentage of pods with an assigned label will not Voluntarily be evicted at any one point in time

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: my-pdb
spec:
  minAvailable: 3
  selector:
    matchLabels:
      app: nginx
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
kubectl delete ns ns-bootcamp-workloads --grace-period 0 --force
```

_End of Section_
