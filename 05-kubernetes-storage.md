# Kubernetes Tutorial - Kubernetes Storage

In this section:
* PersistentVolume – the low level representation of a storage volume
* PersistentVolumeClaim – the binding between a Pod and PersistentVolume
* Pod – a running container that will consume a PersistentVolume
* StorageClass – allows for dynamic provisioning of PersistentVolumes

<br />

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

> Problem Statement: I want a way abstract Storage definitions away from application developers
>
> tl;dr – Define some external storage for use by the Kubernetes cluster

![06-pv](https://user-images.githubusercontent.com/18049790/140639303-5cf1c9d9-094f-4d9b-b85d-2fce70e28afb.jpg)

kubernetes.io bookmark: [Create a PersistentVolume](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/#create-a-persistentvolume)

```yaml
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv #👈👈👈 PersistentVolume Name
  labels:
    type: local
spec:
  storageClassName: manual  #👈👈👈 NOT link to PersistentVolumeClaim
  capacity:
    storage: 5Gi #👈👈👈 Reserve 5 Gigabyte
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/my-host" #👈👈👈 Path on Linux Node 
EOF
```

</p>
</details>

<details class="faq box"><summary>Kubernetes PersistentVolumeClaim (pvc) – the binding between a Pod and PersistentVolume</summary>
<p>

> Problem Statement: I want a way for my microservices application to claim some storage
> 
> tl;dr – Now I want to claim some storage for use inside my container

![06-pvc](https://user-images.githubusercontent.com/18049790/140639304-92e5766a-82eb-4e5b-8af2-05c191fc753d.jpg)

kubernetes.io bookmark: [Create a PersistentVolumeClaim](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/#create-a-persistentvolumeclaim)

```yaml
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc #👈👈👈 PersistentVolumeClaim Name 
spec:
  storageClassName: manual #👈👈👈 NOT link to PersistentVolume
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi #👈👈👈 Reserve 2 Gigabyte
EOF
```

</p>
</details>

<details class="faq box"><summary>Kubernetes Pod consuming PersistentVolume</summary>
<p>

![06-pv-pvc-pod](https://user-images.githubusercontent.com/18049790/140639299-1b5a2c1b-139d-4b52-9a35-66d79de5fb71.jpg)


kubernetes.io bookmark: [Create a PersistentVolumeClaim](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/#create-a-persistentvolumeclaim)

```yaml
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: storage-pod 
spec:
  volumes:
    - name: my-volume
      persistentVolumeClaim:
        claimName: my-pvc #👈👈👈 Link to PersistentVolumeClaim
  containers:
    - name: my-container
      image: nginx
      ports:
        - containerPort: 80
          name: "http-server"
      volumeMounts:
        - mountPath: "/my-mount" #👈👈👈 Mount path in the container
          name: my-volume
EOF
```

```bash
kubectl exec -it storage-pod -- /bin/bash
```

```bash
echo 'Hardships often prepare ordinary people for an extraordinary destiny' >> /my-mount/hello-world.txt
```

```bash
kubectl delete pod storage-pod --force --grace-period=0
```

```yaml
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: storage-pod 
spec:
  volumes:
    - name: my-volume
      persistentVolumeClaim:
        claimName: my-pvc #👈👈👈 Link to PersistentVolumeClaim
  containers:
    - name: my-container
      image: nginx
      ports:
        - containerPort: 80
          name: "http-server"
      volumeMounts:
        - mountPath: "/my-mount" #👈👈👈 Mount path in the container
          name: my-volume
EOF
```

```bash
kubectl exec -it storage-pod -- /bin/bash
```

```bash
cat /my-mount/hello-world.txt
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
kubectl delete ns ns-bootcamp-storage --grace-period 0 --force
```

_End of Section_
