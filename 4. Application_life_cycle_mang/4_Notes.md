### 1. Rolling Updates & Roll Backs:

```
kubectl rollouts status deployment/<depl_name>
kubectl rollout history deployment/<depl_name>
```

1. Deployment Strategy
- Recreate
- Rolling Updates - default 

#### Recreate :
- To upgrade newer verion by destrying all older version of Appl intsnace.
- problem is that the application is down between older and newer version of code is deployed .

#### Rolling Update(Deafult) :
- Here we destroy 1 instance of application and get 1 instance of newer onstance each one by one.

#### How to Update:
- Using kubectl apply 
```
kubectl apply -f <dep_name_file>
kubectl set image deployment/<dep_name> <image_name>:<image_id>
```
#### How Upgrades Works:
- whenever the update is done, kube will create another Replicas Sets with same as older, and it keeps deploying new instance of appl in new replicat set. and paralally destroyes appl instace in the older replica set.
```
kubectl get replicasets
``` 
#### What something is wrong , how to rollback ?
- It rollbacks to previous version by default
```
kubectl rollback undo deployment/<dep_name>
```

- To see status of rollback?
```
kubectl rollback status deployment/<dep_name>
```

### Learn About scalling applications in kubernetes?

---
### 2. Configuring Application:

Configuring applications comprises of understanding the following concepts:
- Configuring Command and Arguments on applications
- Configuring Environment Variables
- Configuring Secrets

#### Commands:
- once the task is done, container will exits
- `CMD` is used to run the something inside container
- `CMD ["bash"]` lanch the bash command inside ubuntu container
- `CMD ["sleep 5"]` sleeps for 5 sec and then exits
- `ENTRYPOINT` Using this , the comand is appended , rather it is just ran in case of `CMD`
- `ENTRYPOINT ["sleep"]`
- `CMD ["5"]` It is command that will follow after entry point
- Inside the pod definition unser container session `COMMAND` will overrides the `ENTRYPOINT` behaviour and `ARGS` will override the `CMD`  behaviour.
```
// pod-definition.yaml
---
spec:
  containers:
    - name: ubuntu-sleeper
        image: ubuntu-sleeper

        command: ["sleep2.0"]
        args: ["100"]
---
```
- To get the containers details of all pods:
```
kubectl get pods -o=jsonpath='{.items..spec.containers}'
```
- Updating pod ubuntu-sleeper-3 to sleep for 2000 seconds.

```
---
apiVersion: v1 
kind: Pod 
metadata:
  name: ubuntu-sleeper-3 
spec:
  containers:
  - name: ubuntu
    image: ubuntu
    command:
      - "sleep"
      - "2000"
```
- Create a pod with below spec using kubectl:
    Pod Name: webapp-green
    Image: kodekloud/webapp-color
    Command line arguments: sleep 2000
```
kubectl run webapp-green --image="kodekloud/webapp-color" --command -- "sleep" "2000"
```
---
### 3. ConfigMaps:

A ConfigMap is an API object used to store non-confidential data in key-value pairs. Pods can consume ConfigMaps as environment variables, command-line arguments, or as configuration files in a volume.

A ConfigMap allows you to decouple environment-specific configuration from your container images, so that your applications are easily portable.


1. Create ConfigMaps:

- Imperative way
```
kubectl create configmap <config-name> --from-literal=<KEY>=<VALUE> --from-literal=<KEY2>=<VALUE3>

kubectl create configmap <config-name> --from-file <path_to_file>

```

- Declarative way:
```
//config-map.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_COLOR:blue
  APP_MODE:prod
```

ConfigMap:
```
APP_COLOR:blue
APP_MODE:prod
```


- To view & describe Configmaps:
```
kubectl get configmaps
kubetl describe configmaps
```

2. Injest them into the Pods
- env

pod-definition.yaml
```
---
spec:
  ---
  envFrom:
  - configMapRef:
      name: <configmap_Name>
---
```
- Single env
```
---
spec:
  ---
  env:
  - name: APP_COLOR
    valueFrom:
      configMapKeyRef:
        name: app-config
        key: APP_COLOR
---
```

- VOLUME
```
volumes:
- name: app-config-volume
  configMap:
    name: app-config
```
---
### 4. Secrets:
- storing sensitive data in config map is not a right way of storing data
- better option is to use secrets object
- used to store sensitive data, similar to configmaps except this are stored in base64

#### 1 .Creating secrets

1. Imperative way:
```
kubectl create secret generic <secrete_name> --from-literal=<key>=<value>

kubectl create seceret generic <secret_name> --from-file=<file_path>
```


2. declarative way:

```
kind: secret
data:
  BD_HOST:mysql //base64
  DB_User: root  //base64
  DB_PASSWORD: passwrd  //base64
```

How to convert to encrypted:
```
echo -n 'mysql' | base64
```


- To get secrets:
```
kubectl get secrets
kubectl describe secrets
kubectl get secrets <secret_name> -o yaml  // to see values
```

- How to decode the encrypted value:
```
echo -n 'enctyptedword' | base64 --decode

```
### 2. using it in pod

secret-data.yaml
```
spec:
  containers:
    ---
    envFrom:
    - secretRef:
      name: <secret_name>
```

1. ENV:
```
envFrom:
  - secretRef:
    name: <secret_name>
```
2. SINGLE ENV
```
env:
  - name: DB_Pass
    valueFrom:
      secretKeyRef:
        name: <secret_name>
        key: <Key_name>
```
3. VOLUME
```
volumes:
- name: app-secret-volume
  secret:
    secretName: app-secret
```

---
### Multicontainer pods:
- It means using multiple containers in a pod, so that all the continer use same life cycles and same network , storage.
How to create?
```
spec:
  containers:
  - name: 
    image:
    ports:
      - containerPort:
  - name: 
    image:
    ports:
      - containerPort:
```
- The application outputs logs to the file /log/app.log. View the logs and try to identify the user having issues with Login.
Run the command: 
```
kubectl -n elastic-stack exec -it app -- cat /log/app.log
```
---

### 5. InitContainers:

- In a multi-container pod, each container is expected to run a process that stays alive as long as the POD's lifecycle. For example in the multi-container pod that we talked about earlier that has a web application and logging agent, both the containers are expected to stay alive at all times. The process running in the log agent container is expected to stay alive as long as the web application is running. If any of them fails, the POD restarts.


- But at times you may want to run a process that runs to completion in a container. For example a process that pulls a code or binary from a repository that will be used by the main web application. That is a task that will be run only  one time when the pod is first created. Or a process that waits  for an external service or database to be up before the actual application starts. That's where initContainers comes in.


- An initContainer is configured in a pod like all other containers, except that it is specified inside a initContainers section,  like this:

```
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox
    command: ['sh', '-c', 'git clone <some-repository-that-will-be-used-by-application> ; done;']
```

- When a POD is first created the initContainer is run, and the process in the initContainer must run to a completion before the real container hosting the application starts. 

- You can configure multiple such initContainers as well, like how we did for multi-pod containers. In that case each init container is run one at a time in sequential order.

- If any of the initContainers fail to complete, Kubernetes restarts the Pod repeatedly until the Init Container succeeds.
```
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox:1.28
    command: ['sh', '-c', 'until nslookup myservice; do echo waiting for myservice; sleep 2; done;']
  - name: init-mydb
    image: busybox:1.28
    command: ['sh', '-c', 'until nslookup mydb; do echo waiting for mydb; sleep 2; done;']
```

Read more about initContainers here. And try out the upcoming practice test.

https://kubernetes.io/docs/concepts/workloads/pods/init-containers/