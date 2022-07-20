### A quick note on editing PODs and Deployments:

#### 1. Edit a POD
    Remember, you CANNOT edit specifications of an existing POD other than the below.

- spec.containers[*].image
- spec.initContainers[*].image
- spec.activeDeadlineSeconds
- spec.tolerations

1. Run the ```kubectl edit pod <pod name>``` command.  
    This will open the pod specification in an editor (vi editor). 
    Then edit the required properties. When you try to save it, you will be denied. 
    This is because you are attempting to edit a field on the pod that is not editable.

    A copy of the file with your changes is saved in a temporary location as shown above.

    You can then delete the existing pod by running the command:
    ```kubectl delete pod webapp```

    Then create a new pod with your changes using the temporary file
    ```kubectl create -f /tmp/kubectl-edit-ccvrq.yaml```

2. The second option is to extract the pod definition in YAML format to a file using the command

    ``` kubectl get pod webapp -o yaml > my-new-pod.yaml```

    Then make the changes to the exported file using an editor (vi editor). Save the changes
    ```vi my-new-pod.yaml```

    Then delete the existing pod
    ```kubectl delete pod webapp```

    Then create a new pod with the edited file
   ``` kubectl create -f my-new-pod.yaml```

#### 2. Edit Deployments

    With Deployments you can easily edit any field/property of the POD template. Since the pod template is a child of the deployment specification,  with every change the deployment will automatically delete and create a new pod with the new changes.
    So if you are asked to edit a property of a POD part of a deployment you may do that simply by running the command. 
    
 ```kubectl edit deployment my-deployment```

#### 3 How to get the manifest file of running pods/deployments ?
```
kubectl get deploy <DEPLOYMENT_NAME> -o yaml

kubectl get pod <POD_NAME> -o yaml

```
#### 4 How to check the document on terminal to get the syntaxs:
```
kubectl explain <object_name> --recursive | less
kubectl explain pods --recursive | less
kubectl explain deployment.spec.strategy              // to show with explanation
kubectl explain pod.spec.containers.envFrom --recursive | less   // to shpw all syntax 
```

- The application outputs logs to the file /log/app.log. View the logs and try to identify the user having issues with Login.
Run the command: 
```
kubectl -n elastic-stack exec -it app -- cat /log/app.log
```

#### Here's a quick tip. 

In the exam, you won't know if what you did is correct or not as in the practice tests in this course. You must verify your work yourself. For example, if the question is to create a pod with a specific image, you must run the the `kubectl describe pod` command to verify the pod is created with the correct name and correct image.

#### 5. Use the alias only if it is necessary. 
You can create one with dry-run:
```
alias kdr='kubectl run --dry-run=client -o yaml'
```
Then you can use it to create a new pod template like this:
```
kdr <pod_name> --image=nignx > pod.yaml
```

#### 6. Important Note about CNI and CKA Exam
An important tip about deploying Network Addons in a Kubernetes cluster.

In the upcoming labs, we will work with Network Addons. This includes installing a network plugin in the cluster. While we have used weave-net as an example, please bear in mind that you can use any of the plugins which are described here:

https://kubernetes.io/docs/concepts/cluster-administration/addons/

https://kubernetes.io/docs/concepts/cluster-administration/networking/#how-to-implement-the-kubernetes-networking-model


In the CKA exam, for a question that requires you to deploy a network addon, unless specifically directed, you may use any of the solutions described in the link above.

However, the documentation currently does not contain a direct reference to the exact command to be used to deploy a third party network addon.

The links above redirect to third party/ vendor sites or GitHub repositories which cannot be used in the exam. This has been intentionally done to keep the content in the Kubernetes documentation vendor-neutral.

At this moment in time, there is still one place within the documentation where you can find the exact command to deploy weave network addon:

https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/#steps-for-the-first-control-plane-node (step 2)

### 7. Network related questions Tips:

#### How to check if network interface is used for node-to-node communication?
- check for `kubectl get nodes -o wide`
- now see internal ip address 
- run  `ip a`
- see the internal ip addr and see its network interface name

#### How to check the MAC address:
- `ip link show eth0`
- and follow the same process as 7 question and check of mac adrress

#### What is the MAC address assigned to node01?
- `arp <node_name>`

#### If you were to ping google from the master node, which route does it take? What is the IP address of the Default Gateway?
- `ip route show default`

#### What is the port the kube-scheduler is listening on in the controlplane node?
- `netstat -nplt`
- shows all the service port
