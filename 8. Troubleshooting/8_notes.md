## Troubleshooting:

###  Application failures:

#### Check Accessibility:
- Users reported some kind of access issue ?
    - imagine entire flow (from backend -> frontend)
    - Steps:
        - check if web server is accessable using ip address:
            ```
            curl http://web-server-ip:node-port        // o/p failure
            ```
        - Check the service description:
            ```
            kubectl describe svc <service-name>
            ```
        - now check the pod and service connectors using `selectors` in the pod, if it is mapped correctly
        - Now check POD if it is running
            ```
            kubectl get pods
            kubectl describe pod <pod_name>
            kubectl logs <pod> -f                   // -f for watching logs
            ```
        - Now check the dependend application(in this case its DB part) and looks for describe , logs, status etc

#### Tips:
- Alays go with bottom up apporoch
   
         
### Cotrol plane failures:
- Check node status
- check the status of pod running in the cluster
- Check control plane components:
    - Check control plane if deployed as Pods 
        ```
        kubectl get pods -n kube-system
        ```
    - Check control plane if deployed as service:
        ```
        service kubelet status
        service kube-proxy status
        ```
    - Check the service logs    
        ```
        kubectl logs kube-apiserver-master -n kube-system
        sudo journalctl -u kube-apiserver                           //to view the kubeapi server logs
        ```
- when fixing control plane components always edit manifest file instead editing pod directly
```
cd /etc/kubenetes/manifest/
```
### Worker node failues:


### Networking trouble shooting:
- Check node status
```
kubectl get nodes
kubectl describe node <worker_node_name>
// watch condition section and see its status 
```
- Check Node 
```
top
df -h
```
- check status of kubelet and its logs
```
- ssh into  that node sna dcheck kubelet status
ssh <nodeName> "service kubelet status"
service kubelet status
sudo journalctl -u kubelet
```
- check kubelet certificate and part of right group
```
openssl x509 -in /var/lib/kubelet/<worker-node-name>.crt -text
```

Issues:
>> journalctl -u kubelet
- Certificate failure  -> /var/lib/kubelet/config
- kubelet trying to connect kubeapi server not rechable  -> /etc/kubernelets/kubelet.conf
- config file mistakes -> /var/lib/kubelet/config