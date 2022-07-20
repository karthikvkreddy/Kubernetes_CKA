## 2. Scheduling
---

### 1. Manual Scheduling:
- specify the node name on which pod should run .
```
spec:
    ---
    nodeName: <Node_Name>
    ---
```
- Using binding object
```
---
kind: Binding
---
target:
    apiVersion:v1
    kind:Node
    name:<Node_Name>
```

### 2. Labels & Selectors:
- To filter Objects
- eg 1
```
---
matadata:
    labels:
        <key>:<value>
---
```
- eg 2 : The set of pods that a service targets is defined with a label selector. Similarly, the population of pods that a replicationcontroller should manage is also defined with a label selector.
```
---
spec:
    ---
    selectors:
        matchLabels:
            <key>:<value>
    ---
---
```
- eg 3, here's the configuration file for a Pod that has two labels environment: production and app: nginx :

```
apiVersion: v1
kind: Pod
metadata:
  name: label-demo
  labels:
    environment: production
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```

### 3. Taints & Tolerations:
- Tainints is condition applies on node , so that any pods that has toleration to that taints can be scheduledin that Node.

- Setting taints on Node?
```
kubectl taint nodes <Node_Name> <KEY>=<VALUE>:<EFFECT>
```
- `<EFFECT>` : NoSchedule | PreferredSchedule | NoExecute

- applying toleration to Pods ?
```
---
spec:
    ---
    tolerations:
        key:<KEY_NAME>
        Operator:<Operator_name> # Exists | Equal 
        value:<Value_Name>
        effect:<effect_name>
    ---
```
- Assignment
    - explain diff effects & operators

- Untaint a node using `-` .
```
kubectl taint nodes <Node_Name> <KEY>=<VALUE>:<EFFECT> -
```

### 4. Node Selector:
- Labels in : Node & Selector in : Pods
- Labels in node:
```
kubectl bales node <Node_Name> <key>=<value>
```
- Selector in Pods:
```
spec:
    ---
    nodeSelector:
        <key>:<value>
    ---
```

#### NOTE: What if we want to place a pod with multiple conditions ? Soln: `Node Affinity`

### 5. Node Affinity:

- required node affinity 
    - This manifest describes a Pod that has a requiredDuringSchedulingIgnoredDuringExecution node affinity,`<KEY>:<VALUE>` This means that the pod will get scheduled only on a node that has a `<KEY>:<VALUE>` label.

    ```
    ---
    spec:
    affinity:
        nodeAffinity:
            requiredDuringSchedulingIgnoredDuringExecution:
                nodeSelectorTerms:
                - matchExpressions:
                    - key: <KEY_NAME>
                        operator: In
                        values:
                        - <VALUE_NAME>
    ---
    ```

- preferred node affinity :
    - This manifest describes a Pod that has a preferredDuringSchedulingIgnoredDuringExecution node affinity,disktype: ssd. This means that the pod will prefer a node that has a disktype=ssd label.
    ```
    ---
    affinity:
        nodeAffinity:
            preferredDuringSchedulingIgnoredDuringExecution:
            - weight: 1
                preference:
                    matchExpressions:
                    - key: <KEY_NAME>
                        operator: In
                        values:
                        - <VALUES>    
    ```


```
>> kubectl get nodes --show-labels
>> kubectl label nodes <your-node-name> disktype=ssd


```







