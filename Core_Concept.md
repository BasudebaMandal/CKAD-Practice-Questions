![](https://gaforgithub.azurewebsites.net/api?repo=CKAD-exercises/core_concepts&empty)
# Core Concepts (13%)

Kubernetes Resources

1) [Kubernetes Pods](https://kubernetes.io/docs/concepts/workloads/pods/pod/)
2) [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
3) [Shell to a Running Container](https://kubernetes.io/docs/tasks/debug-application-cluster/get-shell-running-container/)
4) [Kubernetes Namespace](https://kubernetes.io/docs/tasks/administer-cluster/namespaces-walkthrough/)

### 1)List out all the namespaces in the kubernetes cluster. List all the pods in all the namespaces.

<details><summary>Answer</summary>
<p>

```bash
kubectl get mynamespace
kubectl get po --all-namespaces
```

</p>
</details>

### 2)Create a namespace with name 'customns' and a pod with image nginx called webserver on it.

<details><summary>Answer</summary>
<p>

```bash
kubectl create namespace customns
kubectl run webserver --image=nginx --restart=Never -n mynamespace
```

</p>
</details>

### 3)Create the pod that was just described using YAML

<details><summary>Answer</summary>
<p>

Easily generate YAML with:

```bash
kubectl run webserver --image=nginx --restart=Never --dry-run=client -o yaml > pod.yaml
```

```bash
cat pod.yaml
```

```yaml
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
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl create -f pod.yaml -n customns
```

Alternatively, you can run in one line

```bash
kubectl run nginx --image=nginx --restart=Never --dry-run=client -o yaml | kubectl create -n customns -f -
```

</p>
</details>

### 4)Get the complete details of the pod you just created

<details><summary>Answer</summary>
<p>

```bash
kubectl describe pod nginx
```

</p>
</details>

### 5)List out all pods in customns namespace

<details><summary>Answer</summary>
<p>

```bash
kubectl get po -n customns
```

</p>
</details>

### 6)Create a busybox pod (using kubectl command) that runs the command "ls". Run it and see the output

<details><summary>Answer</summary>
<p>

```bash
kubectl run busybox --image=busybox --command --restart=Never -it -- ls # -it will help in seeing the output
# or, just run it without -it
kubectl run busybox --image=busybox --command --restart=Never -- ls
# and then, check its logs
kubectl logs busybox
```

</p>
</details>

### 7)Create a busybox pod (using YAML) that runs the command "env". Run it and see the output

<details><summary>Answer</summary>
<p>

```bash
# create a  YAML template with this command
kubectl run busybox --image=busybox --restart=Never --dry-run -o yaml --command -- env > envpod.yaml
# see it
cat envpod.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  containers:
  - command:
    - env
    image: busybox
    name: busybox
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
# apply it and then see the logs
kubectl apply -f envpod.yaml
kubectl logs busybox
```

</p>
</details>

### 8)Get the YAML for a new namespace called 'myns' without creating it

<details><summary>Answer</summary>
<p>

```bash
kubectl create namespace myns -o yaml --dry-run
```

</p>
</details>

### 9)Get the YAML for a new ResourceQuota called 'myrq' with hard limits of 1 CPU, 1G memory and 2 pods without creating it

<details><summary>Answer</summary>
<p>

```bash
kubectl create quota myrq --hard=cpu=1,memory=1G,pods=2 --dry-run -o yaml
```

</p>
</details>

### 10)Create the nginx pod with version 1.17.4 and allow traffic on port 80

<details><summary>Answer</summary>
<p>

```bash
kubectl run nginx --image=nginx:1.17.4 --restart=Never --port=80
```

</p>
</details>

### 11)Change pod's image to nginx:1.7.1. Observe that the pod will be killed and recreated as soon as the image gets pulled

<details><summary>Answer</summary>
<p>

```bash
# kubectl set image POD/POD_NAME CONTAINER_NAME=IMAGE_NAME:TAG
kubectl set image pod/nginx nginx=nginx:1.7.1
kubectl describe po nginx # you will see an event 'Container will be killed and recreated'
kubectl get po nginx -w # watch it
```
*Note*: you can check pod's image by running

```bash
kubectl get po nginx -o jsonpath='{.spec.containers[].image}{"\n"}'
```

</p>
</details>

### 12)Check the Image version without the describe command

<details><summary>Answer</summary>
<p>

```bash
kubectl get po nginx -o jsonpath='{.spec.containers[].image}{"\n"}'
```

</p>
</details>

### 13)Create the nginx pod and execute the simple shell on the pod

<details><summary>Answer</summary>
<p>

```bash
kubectl run nginx --image=nginx --restart=Never
```

</p>
</details>

### 14)Get the IP Address of the pod you just created

<details><summary>Answer</summary>
<p>

```bash
kubectl get po nginx -o wide
```

</p>
</details>

### 15)If pod crashed check the previous logs of the pod

<details><summary>Answer</summary>
<p>

```bash
kubectl logs busybox -p
```

</p>
</details>

### 16)Get nginx pod's ip created in previous step, use a temp busybox image to wget its '/'

<details><summary>Answer</summary>
<p>

```bash
kubectl get po -o wide # get the IP, will be something like '10.1.1.131'
# create a temp busybox pod
kubectl run busybox --image=busybox --rm -it --restart=Never -- wget -O- 10.1.1.131:80
```

Alternatively you can also try a more advanced option:

```bash
# Get IP of the nginx pod
NGINX_IP=$(kubectl get pod nginx -o jsonpath='{.status.podIP}')
# create a temp busybox pod
kubectl run busybox --image=busybox --env="NGINX_IP=$NGINX_IP" --rm -it --restart=Never -- sh -c 'wget -O- $NGINX_IP:80'
``` 

Or just in one line:

```bash
kubectl run busybox --image=busybox --rm -it --restart=Never -- wget -O- $(kubectl get pod nginx -o jsonpath='{.status.podIP}:{.spec.containers[0].ports[0].containerPort}')
```

</p>
</details>

### 17)Get pod's YAML

<details><summary>Answer</summary>
<p>

```bash
kubectl get po nginx -o yaml
# or
kubectl get po nginx -oyaml
# or
kubectl get po nginx --output yaml
# or
kubectl get po nginx --output=yaml
```

</p>
</details>

### 18)Get information about the pod, including details about potential issues (e.g. pod hasn't started)

<details><summary>Answer</summary>
<p>

```bash
kubectl describe po nginx
```

</p>
</details>

### 19)Get pod logs

<details><summary>Answer</summary>
<p>

```bash
kubectl logs nginx
```

</p>
</details>

### 20)Create a busybox pod that echoes 'hello world' and then exits

<details><summary>Answer</summary>
<p>

```bash
kubectl run busybox --image=busybox -it --restart=Never -- echo 'hello world'
# or
kubectl run busybox --image=busybox -it --restart=Never -- /bin/sh -c 'echo hello world'
```

</p>
</details>

### 21)Do the same, but have the pod deleted automatically when it's completed

<details><summary>Answer</summary>
<p>

```bash
kubectl run busybox --image=busybox -it --rm --restart=Never -- /bin/sh -c 'echo hello world'
kubectl get po # nowhere to be found :)
```

</p>
</details>

### 22)Create an nginx pod and set an env value as 'var1=val1'. Check the env value existence within the pod

<details><summary>Answer</summary>
<p>

```bash
kubectl run nginx --image=nginx --restart=Never --env=var1=val1
# then
kubectl exec -it nginx -- env
# or
kubectl exec -it nginx -- sh -c 'echo $var1'
# or
kubectl describe po nginx | grep val1
# or
kubectl run nginx --restart=Never --image=nginx --env=var1=val1 -it --rm -- env
```

</p>
</details>

### 23)Create a busybox pod with command sleep 3600

<details><summary>Answer</summary>
<p>

```bash
kubectl run busybox --image=busybox --restart=Never -- /bin/sh -c "sleep 3600"
```

</p>
</details>

### 24)Create an nginx pod and list the pod with different levels of verbosity

<details><summary>Answer</summary>
<p>

```bash
kubectl run nginx --image=nginx --restart=Never --port=80
// List the pod with different verbosity
kubectl get po nginx --v=7
kubectl get po nginx --v=8
kubectl get po nginx --v=9
```

</p>
</details>

### 25)Create a new namespace 'myns' and change the current context to new namespace and create a buxybox pod and list the pods

<details><summary>Answer</summary>
<p>

```bash
kubectl create ns myns
kubectl config set-context --current --namespace=myns
kubectl run busybox --image=busybox --restart=Never
kubectl get po
```

</p>
</details>


### 26)Get all running pods in the namespace

<details><summary>Answer</summary>
<p>

```bash
kubectl get pods --field-selector=status.phase=Running
```

</p>
</details>

List the nginx pod with custom columns POD_NAME and POD_STATUS

### 27)List the nginx pod with custom columns POD_NAME and POD_STATUS

<details><summary>Answer</summary>
<p>

```bash
kubectl get po -o=custom-columns="POD_NAME:.metadata.name, POD_STATUS:.status.containerStatuses[].state"
```

</p>
</details>

### 28)List all the pods sorted by name

<details><summary>Answer</summary>
<p>

```bash
kubectl get pods --sort-by=.metadata.name
```

</p>
</details>

### 29)List all the pods sorted by created timestamp

<details><summary>Answer</summary>
<p>

```bash
kubectl get pods--sort-by=.metadata.creationTimestamp
```

</p>
</details>