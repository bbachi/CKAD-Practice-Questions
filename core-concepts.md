# Core Concepts (13%)

## Practice questions based on these concepts

* Understand Kubernetes API Primitives
* Create and Configure Basic Pods

## Questions

<details><summary>List all the namespaces in the cluster</summary>
<p>

```
kubectl get namespaces

kubectl get ns
```
</p>
</details>

<details><summary>List all the pods in all namespaces</summary>
<p>

```
kubectl get po --all-namespaces
```
</p>
</details>

<details><summary>List all the pods in the particular namespace</summary>
<p>

```
kubectl get po -n <namespace name>
```
</p>
</details>

<details><summary>List all the services in the particular namespace</summary>
<p>

```
kubectl get svc -n <namespace name>
```
</p>
</details>

<details><summary>List all the pods showing name and namespace with a json path expression</summary>
<p>

```
kubectl get pods -o=jsonpath="{.items[*]['metadata.name', 'metadata.namespace']}"
```
</p>
</details>

<details><summary>Create an nginx pod in a default namespace and verify the pod running</summary>
<p>

```
// creating a pod
kubectl run nginx --image=nginx --restart=Never

// List the pod
kubectl get po
```
</p>
</details>


<details><summary>Create the same nginx pod with a yaml file</summary>
<p>

```
// get the yaml file with --dry-run flag
kubectl run nginx --image=nginx --restart=Never --dry-run -o yaml > nginx-pod.yaml

// cat nginx-pod.yaml
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
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}

// create a pod 
kubectl create -f nginx-pod.yaml
```
</p>
</details>


<details><summary>Output the yaml file of the pod you just created</summary>
<p>

```
kubectl get po nginx -o yaml
```
</p>
</details>


<details><summary>Output the yaml file of the pod you just created without the cluster-specific information</summary>
<p>

```
kubectl get po nginx -o yaml --export
```
</p>
</details>


<details><summary>Get the complete details of the pod you just created</summary>
<p>

```
kubectl describe pod nginx
```
</p>
</details>


<details><summary>Delete the pod you just created</summary>
<p>

```
kubectl delete po nginx

kubectl delete -f nginx-pod.yaml
```
</p>
</details>


<details><summary>Delete the pod you just created without any delay (force delete)</summary>
<p>

```
kubectl delete po nginx --grace-period=0 --force
```
</p>
</details>


<details><summary>Create the nginx pod with version 1.17.4 and expose it on port 80</summary>
<p>

```
kubectl run nginx --image=nginx:1.17.4 --restart=Never --port=80
```
</p>
</details>


<details><summary>Change the Image version to 1.15-alpine for the pod you just created and verify the image version is updated</summary>
<p>

```
kubectl set image pod/nginx nginx=nginx:1.15-alpine

kubectl describe po nginx

// another way it will open vi editor and change the version
kubeclt edit po nginx

kubectl describe po nginx
```
</p>
</details>


<details><summary>Change the Image version back to 1.17.1 for the pod you just updated and observe the changes</summary>
<p>

```
kubectl set image pod/nginx nginx=nginx:1.17.1

kubectl describe po nginx

kubectl get po nginx -w # watch it
```
</p>
</details>


<details><summary>Check the Image version without the describe command</summary>
<p>

```
kubectl get po nginx -o jsonpath='{.spec.containers[].image}{"\n"}'
```
</p>
</details>


<details><summary>Create the nginx pod and execute the simple shell on the pod</summary>
<p>

```
// creating a pod
kubectl run nginx --image=nginx --restart=Never

// exec into the pod
kubectl exec -it nginx /bin/sh
```
</p>
</details>


<details><summary>Get the IP Address of the pod you just created</summary>
<p>

```
kubectl get po nginx -o wide
```
</p>
</details>

<details><summary>Create a busybox pod and run command ls while creating it and check the logs</summary>
<p>

```
kubectl run busybox --image=busybox --restart=Never -- ls

kubectl logs busybox
```
</p>
</details>


<details><summary>If pod crashed check the previous logs of the pod</summary>
<p>

```
kubectl logs busybox -p
```
</p>
</details>


<details><summary>Create a busybox pod with command sleep 3600</summary>
<p>

```
kubectl run busybox --image=busybox --restart=Never -- /bin/sh -c "sleep 3600"
```
</p>
</details>


<details><summary>Check the connection of the nginx pod from the busybox pod</summary>
<p>

```
kubectl get po nginx -o wide

// check the connection
kubectl exec -it busybox -- wget -o- <IP Address>
```
</p>
</details>


<details><summary>Create a busybox pod and echo message ‘How are you’ and delete it manually</summary>
<p>

```
kubectl run busybox --image=nginx --restart=Never -it -- echo "How are you"

kubectl delete po busybox
```
</p>
</details>


<details><summary>Create a busybox pod and echo message ‘How are you’ and have it deleted immediately</summary>
<p>

```
// notice the --rm flag
kubectl run busybox --image=nginx --restart=Never -it --rm -- echo "How are you"
```
</p>
</details>


<details><summary>Create an nginx pod and list the pod with different levels of verbosity</summary>
<p>

```
// create a pod
kubectl run nginx --image=nginx --restart=Never --port=80

// List the pod with different verbosity
kubectl get po nginx --v=7
kubectl get po nginx --v=8
kubectl get po nginx --v=9
```
</p>
</details>


<details><summary>List the nginx pod with custom columns POD_NAME and POD_STATUS</summary>
<p>

```
kubectl get po -o=custom-columns="POD_NAME:.metadata.name, POD_STATUS:.status.containerStatuses[].state"
```
</p>
</details>


<details><summary>List all the pods sorted by name</summary>
<p>

```
kubectl get pods --sort-by=.metadata.name
```
</p>
</details>


<details><summary>List all the pods sorted by created timestamp</summary>
<p>

```
kubectl get pods--sort-by=.metadata.creationTimestamp
```
</p>
</details>
