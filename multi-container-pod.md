# Multi-Container Pods (10%)

## Practice questions based on these concepts

* Understand Multi-container pod design patterns (eg: ambassador, adaptor, sidecar)

## Questions

<details><summary>Create a Pod with three busy box containers with commands “ls; sleep 3600;”, “echo Hello World; sleep 3600;” and “echo this is the third container; sleep 3600” respectively and check the status</summary>
<p>

```
// first create single container pod with dry run flag
kubectl run busybox --image=busybox --restart=Never --dry-run -o yaml -- bin/sh -c "sleep 3600; ls" > multi-container.yaml

// edit the pod like below

apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  containers:
  - args:
    - bin/sh
    - -c
    - ls; sleep 3600
    image: busybox
    name: busybox1
    resources: {}
  - args:
    - bin/sh
    - -c
    - echo Hello world; sleep 3600
    image: busybox
    name: busybox2
    resources: {}
  - args:
    - bin/sh
    - -c
    - echo this is third container; sleep 3600
    image: busybox
    name: busybox3
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}

// create it
kubectl create -f multi-container.yaml

kubectl get po busybox
```
</p>
</details>
   
<details><summary>Check the logs of each container that you just created</summary>
<p>
   
```
kubectl logs busybox -c busybox1
kubectl logs busybox -c busybox2
kubectl logs busybox -c busybox3
```
</p>
</details>

<details><summary>Check the previous logs of the second container busybox2 if any</summary>
<p>
   
```
kubectl logs busybox -c busybox2 --previous
```
</p>
</details>

<details><summary>Run command ls in the third container busybox3 of the above pod</summary>
<p>
   
```
kubectl exec busybox -c busybox3 -- ls
```
</p>
</details>

<details><summary>Show metrics of the above pod containers and puts them into the file.log and verify</summary>
<p>
   
```
kubectl top pod busybox --containers

// putting them into file
kubectl top pod busybox --containers > file.log
cat file.log
```
</p>
</details>


<details><summary>Create a Pod with main container busybox and which executes this “while true; do echo ‘Hi I am from Main container’ >> /var/log/index.html; sleep 5; done” and with sidecar container with nginx image which exposes on port 80. Use emptyDir Volume and mount this volume on path /var/log for busybox and on path /usr/share/nginx/html for nginx container. Verify both containers are running.</summary>
<p>
   
```
// create an initial yaml file with this
kubectl run multi-cont-pod --image=busbox --restart=Never --dry-run -o yaml > multi-container.yaml

// edit the yml as below and create it
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: multi-cont-pod
  name: multi-cont-pod
spec:
  volumes:
  - name: var-logs
    emptyDir: {}
  containers:
  - image: busybox
    command: ["/bin/sh"]
    args: ["-c", "while true; do echo 'Hi I am from Main container' >> /var/log/index.html; sleep 5;done"]
    name: main-container
    resources: {}
    volumeMounts:
    - name: var-logs
      mountPath: /var/log
  - image: nginx
    name: sidecar-container
    resources: {}
    ports:
      - containerPort: 80
    volumeMounts:
    - name: var-logs
      mountPath: /usr/share/nginx/html
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}

kubectl create -f multi-container.yaml

kubectl get po multi-cont-pod
```
</p>
</details>


<details><summary>Exec into both containers and verify that main.txt exist and query the main.txt from sidecar container with curl localhost</summary>
<p>
   
```
// exec into main container
kubectl exec -it  multi-cont-pod -c main-container -- sh
cat /var/log/main.txt

// exec into sidecar container
kubectl exec -it  multi-cont-pod -c sidecar-container -- sh
cat /usr/share/nginx/html/index.html

// install curl and get default page
kubectl exec -it  multi-cont-pod -c sidecar-container -- sh
# apt-get update && apt-get install -y curl
# curl localhost
```
</p>
</details>
