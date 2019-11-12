
# Pod Design (20%)

## Practice questions based on these concepts

* Understand how to use Labels, Selectors and Annotations
* Understand Deployments and how to perform rolling updates
* Understand Deployments and how to perform rollbacks
* Understand Jobs and CronJobs

## Questions

<details><summary>Get the pods with label information</summary>
<p>
   
```
kubectl get pods --show-labels
```
</p>
</details>


<details><summary>Create 5 nginx pods in which two of them is labeled env=prod and three of them is labeled env=dev</summary>
<p>
   
```
kubectl run nginx-dev1 --image=nginx --restart=Never --labels=env=dev
kubectl run nginx-dev2 --image=nginx --restart=Never --labels=env=dev
kubectl run nginx-dev3 --image=nginx --restart=Never --labels=env=dev
kubectl run nginx-prod1 --image=nginx --restart=Never --labels=env=prod
kubectl run nginx-prod2 --image=nginx --restart=Never --labels=env=prod
```
</p>
</details>


<details><summary>Verify all the pods are created with correct labels</summary>
<p>
   
```
kubeclt get pods --show-labels
```
</p>
</details>

<details><summary>Get the pods with label env=dev</summary>
<p>
   
```
kubectl get pods -l env=dev
```
</p>
</details>

<details><summary>Get the pods with label env=dev and also output the labels</summary>
<p>
   
```
kubectl get pods -l env=dev --show-labels
```
</p>
</details>

<details><summary>Get the pods with label env=prod</summary>
<p>
   
```
kubectl get pods -l env=prod
```
</p>
</details>

<details><summary>Get the pods with label env=prod and also output the labels</summary>
<p>
   
```
kubectl get pods -l env=prod --show-labels
```
</p>
</details>


<details><summary>Get the pods with label env</summary>
<p>
   
```
kubectl get pods -L env
```
</p>
</details>


<details><summary>Get the pods with labels env=dev and env=prod</summary>
<p>
   
```
kubectl get pods -l 'env in (dev,prod)'
```
</p>
</details>


<details><summary>Get the pods with labels env=dev and env=prod and output the labels as well</summary>
<p>
   
```
kubectl get pods -l 'env in (dev,prod)' --show-labels
```
</p>
</details>


<details><summary>Change the label for one of the pod to env=uat and list all the pods to verify</summary>
<p>
   
```
kubectl label pod/nginx-dev3 env=uat --overwrite

kubectl get pods --show-labels
```
</p>
</details>


<details><summary>Remove the labels for the pods that we created now and verify all the labels are removed</summary>
<p>
   
```
kubectl label pod nginx-dev{1..3} env-
kubectl label pod nginx-prod{1..2} env-

kubectl get po --show-labels
```
</p>
</details>


<details><summary>Let’s add the label app=nginx for all the pods and verify</summary>
<p>
   
```
kubectl label pod nginx-dev{1..3} app=nginx
kubectl label pod nginx-prod{1..2} app=nginx

kubectl get po --show-labels
```
</p>
</details>


<details><summary>Get all the nodes with labels (if using minikube you would get only master node)</summary>
<p>
   
```
kubectl get nodes --show-labels
```
</p>
</details>


<details><summary>Label the node (minikube if you are using) nodeName=nginxnode</summary>
<p>
   
```
kubectl label node minikube nodeName=nginxnode
```
</p>
</details>


<details><summary>Create a Pod that will be deployed on this node with the label nodeName=nginxnode</summary>
<p>
   
```
kubectl run nginx --image=nginx --restart=Never --dry-run -o yaml > pod.yaml

// add the nodeSelector like below and create the pod

apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  nodeSelector:
    nodeName: nginxnode
  containers:
  - image: nginx
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}

kubectl create -f pod.yaml
```
</p>
</details>


<details><summary>Verify the pod that it is scheduled with the node selector</summary>
<p>
   
```
kubectl describe po nginx | grep Node-Selectors
```
</p>
</details>


<details><summary>Verify the pod nginx that we just created has this label</summary>
<p>
   
```
kubectl describe po nginx | grep Labels
```
</p>
</details>


<details><summary>Annotate the pods with name=webapp</summary>
<p>
   
```
kubectl annotate pod nginx-dev{1..3} name=webapp
kubectl annotate pod nginx-prod{1..2} name=webapp
```
</p>
</details>


<details><summary>Verify the pods that have been annotated correctly</summary>
<p>
   
```
kubectl describe po nginx-dev{1..3} | grep -i annotations
kubectl describe po nginx-prod{1..2} | grep -i annotations
```
</p>
</details>



<details><summary>Remove the annotations on the pods and verify</summary>
<p>
   
```
kubectl annotate pod nginx-dev{1..3} name-
kubectl annotate pod nginx-prod{1..2} name-

kubectl describe po nginx-dev{1..3} | grep -i annotations
kubectl describe po nginx-prod{1..2} | grep -i annotations
```
</p>
</details>


<details><summary>Remove all the pods that we created so far</summary>
<p>
   
```
kubectl delete po --all
```
</p>
</details>


<details><summary>Create a deployment called webapp with image nginx with 5 replicas</summary>
<p>
   
```
kubectl create deploy webapp --image=nginx --dry-run -o yaml > webapp.yaml

// change the replicas to 5 in the yaml and create it

apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: webapp
  name: webapp
spec:
  replicas: 5
  selector:
    matchLabels:
      app: webapp
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: webapp
    spec:
      containers:
      - image: nginx
        name: nginx
        resources: {}
status: {}

kubectl create -f webapp.yaml
```
</p>
</details>


<details><summary>Get the deployment you just created with labels</summary>
<p>
   
```
kubectl get deploy webapp --show-labels
```
</p>
</details>


<details><summary>Output the yaml file of the deployment you just created</summary>
<p>
   
```
kubectl get deploy webapp -o yaml
```
</p>
</details>


<details><summary>Get the pods of this deployment</summary>
<p>
   
```
// get the label of the deployment
kubectl get deploy --show-labels

// get the pods with that label
kubectl get pods -l app=webapp
```
</p>
</details>


<details><summary>Scale the deployment from 5 replicas to 20 replicas and verify</summary>
<p>
   
```
kubectl scale deploy webapp --replicas=20

kubectl get po -l app=webapp
```
</p>
</details>


<details><summary>Get the deployment rollout status</summary>
<p>
   
```
kubectl rollout status deploy webapp
```
</p>
</details>


<details><summary>Get the replicaset that created with this deployment</summary>
<p>
   
```
kubectl get rs -l app=webapp
```
</p>
</details>


<details><summary>Get the yaml of the replicaset and pods of this deployment</summary>
<p>
   
```
kubectl get rs -l app=webapp -o yaml

kubectl get po -l app=webapp -o yaml
```
</p>
</details>


<details><summary>Delete the deployment you just created and watch all the pods are also being deleted</summary>
<p>
   
```
kubectl delete deploy webapp

kubectl get po -l app=webapp -w
```
</p>
</details>


<details><summary>Create a deployment of webapp with image nginx:1.17.1 with container port 80 and verify the image version</summary>
<p>
   
```
kubectl create deploy webapp --image=nginx:1.17.1 --dry-run -o yaml > webapp.yaml

// add the port section and create the deployment

apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: webapp
  name: webapp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: webapp
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: webapp
    spec:
      containers:
      - image: nginx:1.17.1
        name: nginx
        ports:
        - containerPort: 80
        resources: {}
status: {}

kubectl create -f webapp.yaml

// verify
kubectl describe deploy webapp | grep Image
```
</p>
</details>


<details><summary>Update the deployment with the image version 1.17.4 and verify</summary>
<p>
   
```
kubectl set image deploy/webapp nginx=nginx:1.17.4

kubectl describe deploy webapp | grep Image
```
</p>
</details>


<details><summary>Check the rollout history and make sure everything is ok after the update</summary>
<p>
   
```
kubectl rollout history deploy webapp

kubectl get deploy webapp --show-labels
kubectl get rs -l app=webapp
kubectl get po -l app=webapp
```
</p>
</details>


<details><summary>Undo the deployment to the previous version 1.17.1 and verify Image has the previous version</summary>
<p>
   
```
kubectl rollout undo deploy webapp

kubectl describe deploy webapp | grep Image
```
</p>
</details>



<details><summary>Update the deployment with the image version 1.16.1 and verify the image and also check the rollout history</summary>
<p>
   
```
kubectl set image deploy/webapp nginx=nginx:1.16.1

kubectl describe deploy webapp | grep Image

kubectl rollout history deploy webapp
```
</p>
</details>



<details><summary>Update the deployment to the Image 1.17.1 and verify everything is ok</summary>
<p>
   
```
kubectl rollout undo deploy webapp --to-revision=3

kubectl describe deploy webapp | grep Image

kubectl rollout status deploy webapp
```
</p>
</details>


<details><summary>Update the deployment with the wrong image version 1.100 and verify something is wrong with the deployment</summary>
<p>
   
```
kubectl set image deploy/webapp nginx=nginx:1.100

kubectl rollout status deploy webapp (still pending state)

kubectl get pods (ImagePullErr)
```
</p>
</details>


<details><summary>Undo the deployment with the previous version and verify everything is Ok</summary>
<p>
   
```
kubectl rollout undo deploy webapp
kubectl rollout status deploy webapp

kubectl get pods
```
</p>
</details>


<details><summary>Check the history of the specific revision of that deployment</summary>
<p>
   
```
kubectl rollout history deploy webapp --revision=7
```
</p>
</details>



<details><summary>Pause the rollout of the deployment</summary>
<p>
   
```
kubectl rollout pause deploy webapp
```
</p>
</details>


<details><summary>Update the deployment with the image version latest and check the history and verify nothing is going on</summary>
<p>
   
```
kubectl set image deploy/webapp nginx=nginx:latest

kubectl rollout history deploy webapp (No new revision)
```
</p>
</details>



<details><summary>Resume the rollout of the deployment</summary>
<p>
   
```
kubectl rollout resume deploy webapp
```
</p>
</details>


<details><summary>Check the rollout history and verify it has the new version
</summary>
<p>
   
```
kubectl rollout history deploy webapp

kubectl rollout history deploy webapp --revision=9
```
</p>
</details>


<details><summary>Apply the autoscaling to this deployment with minimum 10 and maximum 20 replicas and target CPU of 85% and verify hpa is created and replicas are increased to 10 from 1
</summary>
<p>
   
```
kubectl autoscale deploy webapp --min=10 --max=20 --cpu-percent=85

kubectl get hpa

kubectl get pod -l app=webapp
```
</p>
</details>


<details><summary>Clean the cluster by deleting deployment and hpa you just created</summary>
<p>
   
```
kubectl delete deploy webapp

kubectl delete hpa webapp
```
</p>
</details>


<details><summary>Create a Job with an image node which prints node version and also verifies there is a pod created for this job</summary>
<p>
   
```
kubectl create job nodeversion --image=node -- node -v

kubectl get job -w
kubectl get pod
```
</p>
</details>



<details><summary>Get the logs of the job just created</summary>
<p>
   
```
kubectl logs <pod name> // created from the job
```
</p>
</details>



<details><summary>Output the yaml file for the Job with the image busybox which echos “Hello I am from job”</summary>
<p>
   
```
kubectl create job hello-job --image=busybox --dry-run -o yaml -- echo "Hello I am from job"
```
</p>
</details>


<details><summary>Copy the above YAML file to hello-job.yaml file and create the job</summary>
<p>
   
```
kubectl create job hello-job --image=busybox --dry-run -o yaml -- echo "Hello I am from job" > hello-job.yaml

kubectl create -f hello-job.yaml
```
</p>
</details>


<details><summary>Verify the job and the associated pod is created and check the logs as well</summary>
<p>
   
```
kubectl get job
kubectl get po

kubectl logs hello-job-*
```
</p>
</details>



<details><summary>Delete the job we just created</summary>
<p>
   
```
kubectl delete job hello-job
```
</p>
</details>


<details><summary>Create the same job and make it run 10 times one after one</summary>
<p>
   
```
kubectl create job hello-job --image=busybox --dry-run -o yaml -- echo "Hello I am from job" > hello-job.yaml

// edit the yaml file to add completions: 10

apiVersion: batch/v1
kind: Job
metadata:
  creationTimestamp: null
  name: hello-job
spec:
  completions: 10
  template:
    metadata:
      creationTimestamp: null
    spec:
      containers:
      - command:
        - echo
        - Hello I am from job
        image: busybox
        name: hello-job
        resources: {}
      restartPolicy: Never
status: {}

kubectl create -f hello-job.yaml
```
</p>
</details>


<details><summary>Watch the job that runs 10 times one by one and verify 10 pods are created and delete those after it’s completed</summary>
<p>
   
```
kubectl get job -w
kubectl get po

kubectl delete job hello-job
```
</p>
</details>



<details><summary>Create the same job and make it run 10 times parallel</summary>
<p>
   
```
kubectl create job hello-job --image=busybox --dry-run -o yaml -- echo "Hello I am from job" > hello-job.yaml

// edit the yaml file to add parallelism: 10

apiVersion: batch/v1
kind: Job
metadata:
  creationTimestamp: null
  name: hello-job
spec:
  parallelism: 10
  template:
    metadata:
      creationTimestamp: null
    spec:
      containers:
      - command:
        - echo
        - Hello I am from job
        image: busybox
        name: hello-job
        resources: {}
      restartPolicy: Never
status: {}

kubectl create -f hello-job.yaml
```
</p>
</details>



<details><summary>Watch the job that runs 10 times parallelly and verify 10 pods are created and delete those after it’s completed</summary>
<p>
   
```
kubectl get job -w
kubectl get po

kubectl delete job hello-job
```
</p>
</details>



<details><summary>Create a Cronjob with busybox image that prints date and hello from kubernetes cluster message for every minute</summary>
<p>
   
```
kubectl create cronjob date-job --image=busybox --schedule="*/1 * * * *" -- bin/sh -c "date; echo Hello from kubernetes cluster"
```
</p>
</details>



<details><summary>Output the YAML file of the above cronjob</summary>
<p>
   
```
kubectl get cj date-job -o yaml
```
</p>
</details>



<details><summary>Verify that CronJob creating a separate job and pods for every minute to run and verify the logs of the pod</summary>
<p>
   
```
kubectl get job
kubectl get po

kubectl logs date-job-<jobid>-<pod>
```
</p>
</details>



<details><summary>Delete the CronJob and verify all the associated jobs and pods are also deleted</summary>
<p>
   
```
kubectl delete cj date-job

// verify pods and jobs
kubectl get po
kubectl get job
```
</p>
</details>


