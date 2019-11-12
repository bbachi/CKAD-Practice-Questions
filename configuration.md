# Configuration (18%)

## Practice questions based on these concepts

* Understand ConfigMaps
* Understand SecurityContexts
* Define an application’s resource requirements
* Create & Consume Secrets
* Understand ServiceAccounts

## Questions

### TBD

<details><summary>List all the configmaps in the cluster</summary>
<p>
   
```
kubectl get cm
     or
kubectl get configmap
```
</p>
</details>


<details><summary>Create a configmap called myconfigmap with literal value appname=myapp</summary>
<p>
   
```
kubectl create cm myconfigmap --from-literal=appname=myapp
```
</p>
</details>


<details><summary>Verify the configmap we just created has this data</summary>
<p>
   
```
// you will see under data
kubectl get cm -o yaml
         or
kubectl describe cm
```
</p>
</details>


<details><summary>delete the configmap myconfigmap we just created</summary>
<p>
   
```
kubectl delete cm myconfigmap
```
</p>
</details>


<details><summary>Create a file called config.txt with two values key1=value1 and key2=value2 and verify the file</summary>
<p>
   
```
cat >> config.txt << EOF
key1=value1
key2=value2
EOF

cat config.txt
```
</p>
</details>


<details><summary>Create a configmap named keyvalcfgmap and read data from the file config.txt and verify that configmap is created correctly</summary>
<p>
   
```
kubectl create cm keyvalcfgmap --from-file=config.txt

kubectl get cm keyvalcfgmap -o yaml
```
</p>
</details>



<details><summary>Create an nginx pod and load environment values from the above configmap keyvalcfgmap and exec into the pod and verify the environment variables and delete the pod</summary>
<p>
   
```
// first run this command to save the pod yml
kubectl run nginx --image=nginx --restart=Never --dry-run -o yaml > nginx-pod.yml

// edit the yml to below file and create
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
    envFrom:
    - configMapRef:
        name: keyvalcfgmap
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}

kubectl create -f nginx-pod.yml

// verify
kubectl exec -it nginx -- env
kubectl delete po nginx
```
</p>
</details>


<details><summary>Create an env file file.env with var1=val1 and create a configmap envcfgmap from this env file and verify the configmap</summary>
<p>
   
```
echo var1=val1 > file.env
cat file.env

kubectl create cm envcfgmap --from-env-file=file.env
kubectl get cm envcfgmap -o yaml --export
```
</p>
</details>


<details><summary>Create an nginx pod and load environment values from the above configmap envcfgmap and exec into the pod and verify the environment variables and delete the pod</summary>
<p>
   
```
// first run this command to save the pod yml
kubectl run nginx --image=nginx --restart=Never --dry-run -o yaml > nginx-pod.yml

// edit the yml to below file and create
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
    env:
    - name: ENVIRONMENT
      valueFrom:
        configMapKeyRef:
          name: envcfgmap
          key: environment
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}

kubectl create -f nginx-pod.yml

// verify
kubectl exec -it nginx -- env
kubectl delete po nginx
```
</p>
</details>


<details><summary>Create a configmap called cfgvolume with values var1=val1, var2=val2 and create an nginx pod with volume nginx-volume which reads data from this configmap cfgvolume and put it on the path /etc/cfg</summary>
<p>
   
```
// first create a configmap cfgvolume
kubectl create cm cfgvolume --from-literal=var1=val1 --from-literal=var2=val2

// verify the configmap
kubectl describe cm cfgvolume

// create the config map 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  volumes:
  - name: nginx-volume
    configMap:
      name: cfgvolume
  containers:
  - image: nginx
    name: nginx
    resources: {}
    volumeMounts:
    - name: nginx-volume
      mountPath: /etc/cfg
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}

kubectl create -f nginx-volume.yml

// exec into the pod
kubectl exec -it nginx -- /bin/sh

// check the path
cd /etc/cfg
ls
```
</p>
</details>


<details><summary>Create a pod called secbusybox with the image busybox which executes command sleep 3600 and makes sure any Containers in the Pod, all processes run with user ID 1000 and with group id 2000 and verify.</summary>
<p>
   
```
// create yml file with dry-run
kubectl run secbusybox --image=busybox --restart=Never --dry-run -o yaml -- /bin/sh -c "sleep 3600;" > busybox.yml

// edit the pod like below and create
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: secbusybox
  name: secbusybox
spec:
  securityContext: # add security context
    runAsUser: 1000
    runAsGroup: 2000
  containers:
  - args:
    - /bin/sh
    - -c
    - sleep 3600;
    image: busybox
    name: secbusybox
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}

kubectl create -f busybox.yml

// verify
kubectl exec -it secbusybox -- sh
id // it will show the id and group
```
</p>
</details>


<details><summary>Create the same pod as above this time set the securityContext for the container as well and verify that the securityContext of container overrides the Pod level securityContext.</summary>
<p>
   
```
// create yml file with dry-run
kubectl run secbusybox --image=busybox --restart=Never --dry-run -o yaml -- /bin/sh -c "sleep 3600;" > busybox.yml

// edit the pod like below and create
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: secbusybox
  name: secbusybox
spec:
  securityContext:
    runAsUser: 1000
  containers:
  - args:
    - /bin/sh
    - -c
    - sleep 3600;
    image: busybox
    securityContext:
      runAsUser: 2000
    name: secbusybox
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}

kubectl create -f busybox.yml

// verify
kubectl exec -it secbusybox -- sh
id // you can see container securityContext overides the Pod level
```
</p>
</details>


<details><summary>Create pod with an nginx image and configure the pod with capabilities NET_ADMIN and SYS_TIME verify the capabilities</summary>
<p>
   
```
// create the yaml file
kubectl run nginx --image=nginx --restart=Never --dry-run -o yaml > nginx.yml

// edit as below and create pod
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
    securityContext:
      capabilities:
        add: ["SYS_TIME", "NET_ADMIN"]
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}

kubectl create -f nginx.yml

// exec and verify
kubectl exec -it nginx -- sh
cd /proc/1
cat status

// you should see these values
CapPrm: 00000000aa0435fb
CapEff: 00000000aa0435fb
```
</p>
</details>


<details><summary>Create a Pod nginx and specify a memory request and a memory limit of 100Mi and 200Mi respectively.</summary>
<p>
   
```
// create a yml file
kubectl run nginx --image=nginx --restart=Never --dry-run -o yaml > nginx.yml

// add the resources section and create
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
    resources: 
      requests:
        memory: "100Mi"
      limits:
        memory: "200Mi"
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}

kubectl create -f nginx.yml

// verify
kubectl top pod
```
</p>
</details>


<details><summary>Create a Pod nginx and specify a CPU request and a CPU limit of 0.5 and 1 respectively.</summary>
<p>
   
```
// create a yml file
kubectl run nginx --image=nginx --restart=Never --dry-run -o yaml > nginx.yml

// add the resources section and create
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
    resources:
      requests:
        cpu: "0.5"
      limits:
        cpu: "1"
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}

kubectl create -f nginx.yml

// verify
kubectl top pod
```
</p>
</details>


<details><summary>Create a Pod nginx and specify both CPU, memory requests and limits together and verify.</summary>
<p>
   
```
// create a yml file
kubectl run nginx --image=nginx --restart=Never --dry-run -o yaml > nginx.yml

// add the resources section and create
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
    resources:
      requests:
        memory: "100Mi"
        cpu: "0.5"
      limits:
        memory: "200Mi"
        cpu: "1"
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}

kubectl create -f nginx.yml

// verify
kubectl top pod
```
</p>
</details>


<details><summary>Create a Pod nginx and specify a memory request and a memory limit of 100Gi and 200Gi respectively which is too big for the nodes and verify pod fails to start because of insufficient memory</summary>
<p>
   
```
// create a yml file
kubectl run nginx --image=nginx --restart=Never --dry-run -o yaml > nginx.yml

// add the resources section and create
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
    resources:
      requests:
        memory: "100Gi"
        cpu: "0.5"
      limits:
        memory: "200Gi"
        cpu: "1"
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}

kubectl create -f nginx.yml

// verify
kubectl describe po nginx // you can see pending state
```
</p>
</details>


<details><summary>Create a secret mysecret with values user=myuser and password=mypassword</summary>
<p>
   
```
kubectl create secret generic my-secret --from-literal=username=user --from-literal=password=mypassword
```
</p>
</details>


<details><summary>List the secrets in all namespaces</summary>
<p>
   
```
kubectl get secret --all-namespaces
```
</p>
</details>


<details><summary>Output the yaml of the secret created above</summary>
<p>
   
```
kubectl get secret my-secret -o yaml
```
</p>
</details>


<details><summary>Create an nginx pod which reads username as the environment variable</summary>
<p>
   
```
// create a yml file
kubectl run nginx --image=nginx --restart=Never --dry-run -o yaml > nginx.yml

// add env section below and create
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
    env:
    - name: USER_NAME
      valueFrom:
        secretKeyRef:
          name: my-secret
          key: username
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}

kubectl create -f nginx.yml

//verify
kubectl exec -it nginx -- env
```
</p>
</details>


<details><summary>Create an nginx pod which loads the secret as environment variables</summary>
<p>
   
```
// create a yml file
kubectl run nginx --image=nginx --restart=Never --dry-run -o yaml > nginx.yml

// add env section below and create
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
    envFrom:
    - secretRef:
        name: my-secret
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}

kubectl create -f nginx.yml

//verify
kubectl exec -it nginx -- env
```
</p>
</details>


<details><summary>List all the service accounts in the default namespace</summary>
<p>
   
```
kubectl get sa
```
</p>
</details>


<details><summary>List all the service accounts in all namespaces</summary>
<p>
   
```
kubectl get sa --all-namespaces
```
</p>
</details>


<details><summary>Create a service account called admin</summary>
<p>
   
```
kubectl create sa admin
```
</p>
</details>


<details><summary>Output the YAML file for the service account we just created</summary>
<p>
   
```
kubectl get sa admin -o yaml
```
</p>
</details>


<details><summary>Create a busybox pod which executes this command sleep 3600 with the service account admin and verify</summary>
<p>
   
```
kubectl run busybox --image=busybox --restart=Never --dry-run -o yaml -- /bin/sh -c "sleep 3600" > busybox.yml

kubectl create -f busybox.yml

apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  serviceAccountName: admin
  containers:
  - args:
    - /bin/sh
    - -c
    - sleep 3600
    image: busybox
    name: busybox
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}

// verify
kubectl describe po busybox
```
</p>
</details>

