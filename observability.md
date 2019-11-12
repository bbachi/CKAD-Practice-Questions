# Observability (18%)

## Practice questions based on these concepts

* Understand LivenessProbes and ReadinessProbes
* Understand Container Logging
* Understand how to monitor applications in kubernetes
* Understand Debugging in Kubernetes

## Questions

<details><summary>Create an nginx pod with containerPort 80 and it should only receive traffic only it checks the endpoint / on port 80 and verify and delete the pod</summary>
<p>

```
kubectl run nginx --image=nginx --restart=Never --port=80 --dry-run -o yaml > nginx-pod.yaml

// add the readinessProbe section and create
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    ports:
    - containerPort: 80
    readinessProbe:
      httpGet:
        path: /
        port: 80
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}

kubectl create -f nginx-pod.yaml

// verify
kubectl describe pod nginx | grep -i readiness
kubectl delete po nginx
```
</p>
</details>


<details><summary>Create an nginx pod with containerPort 80 and it should check the pod running at endpoint / healthz on port 80 and verify and delete the pod</summary>
<p>

```
kubectl run nginx --image=nginx --restart=Never --port=80 --dry-run -o yaml > nginx-pod.yaml

// add the livenessProbe section and create
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    ports:
    - containerPort: 80
    livenessProbe:
      httpGet:
        path: /healthz
        port: 80
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}

kubectl create -f nginx-pod.yaml

// verify
kubectl describe pod nginx | grep -i readiness
kubectl delete po nginx
```
</p>
</details>


<details><summary>Create an nginx pod with containerPort 80 and it should check the pod running at endpoint /healthz on port 80 and it should only receive traffic only it checks the endpoint / on port 80. verify the pod</summary>
<p>

```
kubectl run nginx --image=nginx --restart=Never --port=80 --dry-run -o yaml > nginx-pod.yaml

// add the livenessProbe and readiness section and create
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    ports:
    - containerPort: 80
    livenessProbe:
      httpGet:
        path: /healthz
        port: 80
    readinessProbe:
      httpGet:
        path: /
        port: 80
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}

kubectl create -f nginx-pod.yaml

// verify
kubectl describe pod nginx | grep -i readiness
kubectl describe pod nginx | grep -i liveness
```
</p>
</details>


<details><summary>Check what all are the options that we can configure with readiness and liveness probes</summary>
<p>

```
kubectl explain Pod.spec.containers.livenessProbe
kubectl explain Pod.spec.containers.readinessProbe
```
</p>
</details>


<details><summary>Create the pod nginx with the above liveness and readiness probes so that it should wait for 20 seconds before it checks liveness and readiness probes and it should check every 25 seconds.</summary>
<p>

```
// nginx-pod.yaml

apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    ports:
    - containerPort: 80
    livenessProbe:
      initialDelaySeconds: 20
      periodSeconds: 25
      httpGet:
        path: /healthz
        port: 80
    readinessProbe:
      initialDelaySeconds: 20
      periodSeconds: 25
      httpGet:
        path: /
        port: 80
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}

kubectl create -f nginx-pod.yaml
```
</p>
</details>


<details><summary>Create a busybox pod with this command “echo I am from busybox pod; sleep 3600;” and verify the logs</summary>
<p>

```
kubectl run busybox --image=busybox --restart=Never -- /bin/sh -c "echo I am from busybox pod; sleep 3600;"

kubectl logs busybox
```
</p>
</details>


<details><summary>copy the logs of the above pod to the busybox-logs.txt and verify</summary>
<p>

```
kubectl logs busybox > busybox-logs.txt

cat busybox-logs.txt
```
</p>
</details>


<details><summary>List all the events sorted by timestamp and put them into file.log and verify</summary>
<p>

```
kubectl get events --sort-by=.metadata.creationTimestamp

// putting them into file.log
kubectl get events --sort-by=.metadata.creationTimestamp > file.log

cat file.log
```
</p>
</details>


<details><summary>Create a pod with an image alpine which executes this command ”while true; do echo ‘Hi I am from alpine’; sleep 5; done” and verify and follow the logs of the pod</summary>
<p>

```
// create the pod
kubectl run hello --image=alpine --restart=Never  -- /bin/sh -c "while true; do echo 'Hi I am from Alpine'; sleep 5;done"

// verify and follow the logs
kubectl logs --follow hello
```
</p>
</details>


<details><summary>Create the pod with this kubectl create -f https://gist.githubusercontent.com/bbachi/212168375b39e36e2e2984c097167b00/raw/1fd63509c3ae3a3d3da844640fb4cca744543c1c/not-running.yml. The pod is not in the running state. Debug it.</summary>
<p>

```
// create the pod
kubectl create -f https://gist.githubusercontent.com/bbachi/212168375b39e36e2e2984c097167b00/raw/1fd63509c3ae3a3d3da844640fb4cca744543c1c/not-running.yml

// get the pod
kubectl get pod not-running
kubectl describe po not-running

// it clearly says ImagePullBackOff something wrong with image
kubectl edit pod not-running // it will open vim editor
                     or
kubectl set image pod/not-running not-running=nginx
```
</p>
</details>


<details><summary>This following yaml creates 4 namespaces and 4 pods. One of the pod in one of the namespaces are not in the running state. Debug and fix it. `https://gist.githubusercontent.com/bbachi/1f001f10337234d46806929d12245397/raw/84b7295fb077f15de979fec5b3f7a13fc69c6d83/problem-pod.yaml`</summary>
<p>

```
kubectl create -f https://gist.githubusercontent.com/bbachi/1f001f10337234d46806929d12245397/raw/84b7295fb077f15de979fec5b3f7a13fc69c6d83/problem-pod.yaml

// get all the pods in all namespaces
kubectl get po --all-namespaces

// find out which pod is not running
kubectl get po -n namespace2

// update the image
kubectl set image pod/pod2 pod2=nginx -n namespace2

// verify again
kubectl get po -n namespace2
```
</p>
</details>


<details><summary>Get the memory and CPU usage of all the pods and find out top 3 pods which have the highest usage and put them into the cpu-usage.txt file</summary>
<p>

```
// get the top 3 hungry pods
kubectl top pod --all-namespaces | sort --reverse --key 3 --numeric | head -3

// putting into file
kubectl top pod --all-namespaces | sort --reverse --key 3 --numeric | head -3 > cpu-usage.txt

// verify
cat cpu-usage.txt
```
</p>
</details>
