# Kubernetes Tutorial - Kubernetes Workloads

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

![03-deployment](https://user-images.githubusercontent.com/18049790/140038711-2ff33916-cd52-4d11-bd0c-812118517a68.jpg)

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
* Like a Deployment, a StatefulSet manages Pods that are based on an identical container spec. 
* Unlike a Deployment, a StatefulSet maintains a sticky identity for each of their Pods. 
* These pods are created from the same spec, but are not interchangeable: each has a persistent identifier that it maintains across any rescheduling.
* If you want to use storage volumes to provide persistence for your workload, you can use a StatefulSet as part of the solution. 
* Although individual Pods in a StatefulSet are susceptible to failure, the persistent Pod identifiers make it easier to match existing volumes to the new Pods that replace any that have failed.

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

## Sample CKAD Questions

* [Sample CKAD Question - Blue-Green-Canary](https://github.com/jamesbuckett/ckad-questions/blob/main/03-ckad-deployment.md#03-05-create-a-namespace-called-blue-green-namespace-create-a-deployment-called-blue-deployment-with-10-replicas-using-the-nginx-image-inside-the-namespace-expose-port-80-for-the-nginx-containers-label-the-pods-versionblue-and-tierweb-create-a-service-called-bsg-service-to-route-traffic-to-blue-deployment-verify-that-traffic-is-flowing-from-the-service-to-the-deployment-create-a-new-deployment-called-green-deployment--with-10-replicas-using-the-nginx-image-inside-the-namespace-expose-port-80-for-the-nginx-containers-label-the-pods-versiongreen-and-tierweb-once-the-green-deployment-is-active-split-traffic-between-blue-deployment70-and-green-deployment30)
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
