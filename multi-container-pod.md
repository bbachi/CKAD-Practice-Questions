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
