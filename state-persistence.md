
# State Persistence (8%)

## Practice questions based on these concepts

* Understand PersistentVolumeClaims for Storage

## Questions


<details><summary>List Persistent Volumes in the cluster</summary>
<p>
   
```
kubectl get pv
```
</p>
</details>


<details><summary>Create a hostPath PersistentVolume named task-pv-volume with storage 10Gi, access modes ReadWriteOnce, storageClassName manual, and volume at /mnt/data and verify</summary>
<p>
   
```
// task-pv-volume.yaml

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
    
kubectl create -f task-pv-volume.yaml

kubectl get pv
```
</p>
</details>



<details><summary>Create a PersistentVolumeClaim of at least 3Gi storage and access mode ReadWriteOnce and verify status is Bound</summary>
<p>
   
```
// task-pv-claim.yaml

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
      
kubectl create -f task-pv-claim.yaml

kubectl get pvc
```
</p>
</details>


<details><summary>Delete persistent volume and PersistentVolumeClaim we just created</summary>
<p>
   
```
kubectl delete pvc task-pv-claim
kubectl delete pv task-pv-volume
```
</p>
</details>


<details><summary>Create a Pod with an image Redis and configure a volume that lasts for the lifetime of the Pod</summary>
<p>
   
```
// emptyDir is the volume that lasts for the life of the pod

apiVersion: v1
kind: Pod
metadata:
  name: redis
spec:
  containers:
  - name: redis
    image: redis
    volumeMounts:
    - name: redis-storage
      mountPath: /data/redis
  volumes:
  - name: redis-storage
    emptyDir: {}
    
kubectl create -f redis-storage.yaml
```
</p>
</details>


<details><summary>Exec into the above pod and create a file named file.txt with the text ‘This is called the file’ in the path /data/redis and open another tab and exec again with the same pod and verifies file exist in the same path.</summary>
<p>
   
```
// first terminal
kubectl exec -it redis-storage /bin/sh
cd /data/redis
echo 'This is called the file' > file.txt

//open another tab
kubectl exec -it redis-storage /bin/sh
cat /data/redis/file.txt
```
</p>
</details>


<details><summary>Delete the above pod and create again from the same yaml file and verifies there is no file.txt in the path /data/redis.</summary>
<p>
   
```
kubectl delete pod redis

kubectl create -f redis-storage.yaml
kubectl exec -it redis-storage /bin/sh
cat /data/redis/file.txt // file doesn't exist
```
</p>
</details>


<details><summary>Create PersistentVolume named task-pv-volume with storage 10Gi, access modes ReadWriteOnce, storageClassName manual, and volume at /mnt/data and Create a PersistentVolumeClaim of at least 3Gi storage and access mode ReadWriteOnce and verify status is Bound</summary>
<p>
   
```
kubectl create -f task-pv-volume.yaml
kubectl create -f task-pv-claim.yaml

kubectl get pv
kubectl get pvc
```
</p>
</details>


<details><summary> Create an nginx pod with containerPort 80 and with a PersistentVolumeClaim task-pv-claim and has a mouth path "/usr/share/nginx/html"</summary>
<p>
   
```
// task-pv-pod.yaml

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
          
kubectl create -f task-pv-pod.yaml
```
</p>
</details>

