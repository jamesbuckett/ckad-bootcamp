# Kubernetes Tutorial - Kubernetes Storage

In this section:
* PersistentVolume – the low level representation of a storage volume
* PersistentVolumeClaim – the binding between a Pod and PersistentVolume
* Pod – a running container that will consume a PersistentVolume
* StorageClass – allows for dynamic provisioning of PersistentVolumes

## Kubernetes Storage

<details class="faq box"><summary>Kubernetes Namespace (ns) - Logical isolation for your application</summary>
<p>

```bash
kubectl create namespace ns-bootcamp-storage
kubectl config set-context --current --namespace=ns-bootcamp-storage
```

</p>
</details>

<details class="faq box"><summary>Kubernetes PersistentVolume (pv) – the low level representation of a storage volume</summary>
<p>

kubernetes.io bookmark: [Create a PersistentVolume](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/#create-a-persistentvolume)

```bash
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolume
metadata:
  name: task-pv-volume
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
EOF      
```

</p>
</details>


<details class="faq box"><summary>Kubernetes PersistentVolumeClaim (pvc) – the binding between a Pod and PersistentVolume</summary>
<p>

kubernetes.io bookmark: [Create a PersistentVolumeClaim](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/#create-a-persistentvolumeclaim)

```bash
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: task-pv-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
EOF      
```

<details class="faq box"><summary>Kubernetes Pod consuming PersistentVolume</summary>
<p>

kubernetes.io bookmark: [Create a PersistentVolumeClaim](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/#create-a-persistentvolumeclaim)

```bash
apiVersion: v1
kind: Pod
metadata:
  name: task-pv-pod
spec:
  volumes:
    - name: task-pv-storage
      persistentVolumeClaim:
        claimName: task-pv-claim
  containers:
    - name: task-pv-container
      image: nginx
      ports:
        - containerPort: 80
          name: "http-server"
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: task-pv-storage
EOF      
```

```bash
kubectl exec -it task-pv-pod -- /bin/bash
```

```bash
apt update
apt install curl
curl http://localhost/
```

</p>
</details>

## Sample CKAD Questions

* [Sample CKAD Question - PersistentVolume & PersistentVolumeClaim](https://github.com/jamesbuckett/ckad-questions/blob/main/01-ckad-design-build.md#01-03-create-a-namespace-called-storage-namespace-create-a-persistent-volume-called-my-pv-with-5gi-storage-using-hostpath-mntmy-host-create-a-persistent-volume-claim-called-my-pvc-with-2gi-storage-create-a-pod-called-storage-pod-using-the-nginx-image-mount-the-persistent-volume-claim-onto-my-mount-in-storage-pod)

_End of Section_
