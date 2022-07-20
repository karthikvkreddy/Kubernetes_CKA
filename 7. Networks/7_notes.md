# 7. Networking

## 1. Networking Prerequisite:

### a. Switching:
- To connect the 2 systems/ Host, we need interface
    ```
    // To see interface of host
    ip link
    ```
    o/p: eth0 , eth2 etc
    - ip addr
    - We will have a network to connect the host, eg ip addr: 192.168.1.0 
    - to add this address to each of this host to communicate with each other
    ```
    ip addr add <ip_address_of_network> <host1> <host2>
    ```
- here Switch helps communicating in only same network, but if we want to comminicate in different network.

### B. Routing:
- helps connect 2 networks together
- get 2 ips assigned

### C. Gateway


### D. Ports:
- kubelet : 10250
- kube-api : 6443
- kube-scheduler : 10251
- kube-controller-manager : 10252
- services: 30000 - 32767
- ETCD : 2379

#### Commands:
```
>> ip link
>> ip addr
>> ip addr add 192.168.1.10/24 dev eth0
>> ip route
>> ip route add 192.168.1.0/24 via 192.168.2.1
>> cat /proc/sys/net/ipv4/ip_forward
>> arp
>> netstat -pint
```


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


### POD networking:
- How does communication happends between the containers ?
- Networking Model:
    - Every POD shoul dhave ip address
    - every pod shoul dbe able to communicate with every other pod in the same node
    - every pod should be able to communicate with every other POD on other nodes without NAT
- Solutions: 
    - different plugins are already available (weave, fannel etc.)
- Can we do it manually ? YES

### CNI(container network interface) In kubernetes:
- responsible for container runtime 
- runtime shoud must create network namespace
- container runtime to invoke plugin (bridge) when container is Added
- identify network the container must attach to
- container runtime to invoke plugin(bridge) when cotainer is added
- JSON format of th enetwork configuration

- Configuring CNI:
    - configured in kubelet service
        ```
        //kubelet.service
        --network-plugin=cni \
        --cni-bin-dir=/opt/cni/bin  \
        --cni-conf-dir=/etc/cni/net.d
        ```
    - view kubelet options:
    ```
    >> ps -aux | grep kubelet
    >> ls /opt/cni/bin
    >> ls /etc/cni/net.d
        - 10-bridgt.conf
    >> cat /etc/cni/net.d/10-bridgr.conf
        - shows list of the configurations
        - gateways
    ```

### Weave-works :
- this plugins are like a agent , where they present in all the nodes and when some packets arives , it knows where exactly needs to be delivers as it keeps a track of all its agents in other nodes.
- Deploy Weave
    - deploy it as a doemonset(makes sure atleat 1 set of of pod of this kin dis present)
    - if already configured by k8s, deployed it as a pod
    - kubectl apply -f 'PATH'
- weave peers:(deployed as a pods):
    - `kubectl get pods -n kube-systems`

- Replace the default IP address and subnet of weave-net to the 10.50.0.0/16. Please check the official weave installation and configuration guide which is available at the top right panel.?
    - deploy weave-net with extra configuration:  So we need to change the default IP address by adding `&env.IPALLOC_RANGE=10.50.0.0/16` option at the end of the manifest file
    ```
    kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')&env.IPALLOC_RANGE=10.50.0.0/16"
    ```

### IP address management:
- CNI uses plugins that invokes the free IPs from the list that is , host-local or DHCP .
- it invoked by `ip = get_free_ip_from_host_local()`
- to do this task dynamically, CNI has a section call IPAM where it specifies which plugins to use for this task
- `cat /etc/cni/cni.net/net-script.conf`
```
---
'ipam':
{
    type: 'host-local',
    'subnet': '10.244.0.0/16'
    'routes':[{'dst':"0.0.0.0/0"}]
}
---
```
- For exmple weave plugin allocates the ips with 10.32..0.1/12 => 10.32.0.1 > 10.47.255.254 ~ 1048574 ips 

- How to check the name of the interface created by the weave plugin:
```
ip link
```
- To check ip range set by weave for pod assignment
```
ip addr show weave
```

### service networks:
- To access the other pod in kubernetes we normally use `service`
- The service that can be accessable only within the cluster is known as `ClusterIP`
- If we want to access the servce outside the cluster , then we call it as `NodePort` . eg. web app. to access outside the cluster.
    - it get ip address where all pods can access it and also exposes the port of the all node.so, externals users can access to the service.
- every node contains `kubelet` process which is responsible for the pod creation.
- each kubelet watches the changes in the cluster through kube-api service, if any pod is creates it invokes the CNI to configure the network to that pod. 
- `kube-proxy` watches the services in the nodes: as services is cluster wide or node wise, everytime services created `kube-proxy` will be invoked.
- when service is created it gets pre-defined ip is assigned.
- so, when services gets the ip addr as `10.99.13.278` , the kube-proxy will create a forward To tables (iptables), all the ips that reaches this pre-defined ip will be forwarded to the pod ip.
```
IP:Port => Forward To:
```
- To see the default ip range for the service in the kube-api-server
```
ps aux | grep kube-api-server
```
- 10.96.0.0/12   => 10.96.0.0 -> 10.111.255.255
- so, the pod and service will never have same ip address
- To see iptables of the particular service:
```
iptables -L -t nat| grep <service_name>
```
- See the kube-proxy logs on how it see the table:
```
cat /var/log/kube-proxy.log
```
-  What is the range of IP addresses configured for PODs on this cluster?
    - The network is configured with weave. Check the weave pods logs using command `kubectl logs <weave-pod-name> weave -n kube-system` and look for ipalloc-range
- What is the IP Range configured for the services within the cluster?
    - Inspect the setting on kube-api server by running on command `cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep cluster-ip-range`
- What type of proxy is the kube-proxy configured to use?
    - Check the logs of the kube-proxy pods. Run the command: `kubectl logs <kube-proxy-pod-name> -n kube-system`

### DNS in Kubernetes: ****
- Pre-requisite:
    - DNS?
    - HOST/NSLookup , Dig utility
    - Recorded types - A, CNAME
    - Domain Name hierarchy
- Objectives:
    - what names are assigned to what objects?
    - service DNS Records
    - POD DNS Records

### CoreDNS in kubernetes: ***
- When 2 pods are available, how communication is done?
    - we can do this by adding ip addr in host file of the pod
        1. POD1 `/etc/hosts` => `web 10.244.2.5`
        2. POD2 `/etc/hosts` => `test 10.244.1.5`   
    - when we have 100s / 1000s in our cluster this approach will not be easier.

    - Now we can to do this by adding DNS server
        - add entry to each pod `/etc/resolve`
            POD1 `/etc/resolve.conf` => `nameserver 10.96.0.10`
            POD2 `/etc/resolve.conf` => `nameserver 10.96.0.10`

        - DNS Server in cluster:
            ```
            10-244-2-5 10.244.2.5
            10-244-1-5 10.244.1.5
            10-244-2-15 10.244.2.15
            ----
            ```
        - for each entry it replaces pod name with ip address with - .
        - each time pod is created its entry is added to the DNS server and it is pointed by each pod using resolv.conf file

    - #### with k8s > 1.12 ~~ `coreDNS`
        - deployed as POD - replicaSet
        - runs `./coredns`
        - uses file `cat /etc/coredns/Corefile`
            - Plugins: handling errors
            - cluster.local -> top level domain name of cluster is set
            - every pods falls under this domain
            - a record is formed for each pod
    - this config file can be found in below, so if we want to modify it can be done here.
    ```
    kubectl get configMap -n kube-system
    ```
    - it watches the cluster for pod creation, when it founds, it creates an entry in the coreDNS server
    ```
    // ./coredns
    Name        IP address
    10-244-1-5  10.244.1.4
    web-service 10.107.37.188

    ```
    - get the service of kube-dns
    ```
    kubectl get service -n kube-system
    ```
    - to see the ip address of the codeDNS server:
    ```
    cat /var/lib/kubelet/config.yaml
    o/p: clusterDNS:
          - 10.96.0.10
         clusterDomain: cluster.local
    ```
    - so we found that kubelet is assigns the ip assignment 
    - when we see the host name of the server, it always returns the FQDN :
        ```
        host web-service
        web-service.default.svc.cluster.local has address 10.97.206.196
        ```
    - `NAME     NAMESPACE     OBJECT-TYPE      ROOT-DOMAIN`  

- From the hr pod nslookup the mysql service and redirect the output to a file /root/CKA/nslookup.out
    - Run the command: `kubectl exec -it hr -- nslookup mysql.payroll > /root/CKA/nslookup.out`

### Ingress ***
- application contains database eg. mysql that can be accessable inside the cluster , so we need clusterIP to access this pod.
- To make app accessible to outsode world - nodePort service (<ip>:38080)
- when traffic increases we increase the no of replicas pods and service take scare of splitting the traffics to the pod
- thr exists many things in splitting the traffic
- we dont want users to use ip or along with ports to access the applications
- so, here we bring another layer as proxy server , that it will forwards 38080:80
- now users can access the application using hostName

- what happens when we deploy this app in public cloud platform ?
    - when we set type as `loadbalancer` , kubernetes will do eevrything same as nodeport
    - provisions the port and also  deploys network load balancer ,automatically configures route traffic to all the nodes and returns its info to kubernetes.
    - we set the dns to point to the ip: <my-online-store.com>

    - now we need to make accesses : <host-name>/watches  /home /logins
    - each services will get seperate load-balancers .
    - Now how to direct traffice to these loadbalancers based on the url path
    - now we need another loadbalancer and mark it as
        ```
        /home   loadbalancer1
        /watches    loadbalancer2
        ```
    - need SSL to access this , where to configure it, can be done at loadbalancer / applications
    - we see we need lot of works to do this , when we have to manage all this by ourself when we have large applications 

- WHAT IF WE CAN MANAGE ALL THIS INSIDE OUR CLUSTER ?
    - Solution : `Ingress`
    - helps users to access application using a single externally accessable url, and can be configured route to diff services within our cluster based on URL and at the same it implements SSL certficate as well.
    - it is as layer 7 built in kubernetes cluster
    - even with the ingress we still have to expose it outside so we need nodePort or cloud native load balancer
    - HOW?
        1. deploy `ingress controller`
        2. configure `ingree resources`
    - By default k8s wont come with the inress controller
    - How you deploy ?
        -  many controller available: HAPROXY, CONTOUR, Trafik, GCP HTTP(s), load balancer, istio, nginx etc.
        - here lets take NGINX as a example
        - it deployed as deployment object
            - use image (nginx-ingress-contoller) etc / args: -/nginx-ingress-controller / stores logs etc 
        - create configMaps object and add this name in configMaps args
        - env: namespace 
        - ports
        - As `service` to type `nodePort` to export ingress-controller
        - ingress-controller needs additional info to perform activities for that it needs add. permission , so we need correct role
        - in overAll for ingress we need :  deployment(Pods) - service(Network) - Auth(ServiceAccount) - configMap(conf)
        
        - `INGRESS RESOURCES` is nothing but a set of rules  . eg. forwards all incoming the traffices to a single application or route traffice based on the diff path
        - route users based on the domain name sitself eg. wear.myonline.com / watch.myonline.com
        - reourves can be created using the yaml file
        ```
        // ingress-wear.yaml
        kind: networking.k8s.io/v1
        kind:ingress
        metadata:
            name: ingress-wear
        spec:
            backend:   // session where service is routed
                serviceName: wear-service
                servicePort: 80 
        ``` 
        ```
        kubectl create -f ingress-wear.yaml
        kubectle get ingress
        ```

        - Now how to create RULES:
            ```
            --
            --
            spec:
                rules:
                - http:
                    paths:
                    - path: /wear                       // <domain_name>.com/wear
                    backend:                            // session where service is routed
                        service: 
                            name: wear-service
                            port: 
                                number: 80 
                    - path: /watch                       // <domain_name>.com/watch
                    backend:                            // session where service is routed
                     backend:                            // session where service is routed
                        service: 
                            name: wear-service
                            port: 
                                number: 80 

            ```
        - Now check describe - to see the rules in it.
        - how to route based on the domain name: eg. wear.domain_name>.com
        ```
            spec:
                rules:
                - host: wear.<domain_name>.com
                  http: 
                    paths:
                        backend:                            // session where service is routed
                            serviceName: wear-service
                            servicePort: 80 
                - host: watch.<domain_name>.com
                  http: 
                    paths:
                        backend:                            // session where service is routed
                            serviceName: watch-service
                            servicePort: 80 
        ```

### Reference:
Now, in k8s version 1.20+ we can create an Ingress resource from the imperative way like this:-
```
Format - kubectl create ingress <ingress-name> --rule="host/path=service:port"
```

Example - 
```
kubectl create ingress ingress-test --rule="wear.my-online-store.com/wear*=wear-service:80"
```
Find more information and examples in the below reference link:-

https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#-em-ingress-em-

References:

https://kubernetes.io/docs/concepts/services-networking/ingress
https://kubernetes.io/docs/concepts/services-networking/ingress/#path-types


### Ingress - Annotations and rewrite-target
- Different ingress controllers have different options that can be used to customise the way it works. NGINX Ingress controller has many options that can be seen here. I would like to explain one such option that we will use in our labs. The Rewrite target option.

- Our watch app displays the video streaming webpage at `http://<watch-service>:<port>/`

- Our wear app displays the apparel webpage at `http://<wear-service>:<port>/`

- We must configure Ingress to achieve the below. When user visits the URL on the left, his request should be forwarded internally to the URL on the right. Note that the /watch and /wear URL path are what we configure on the ingress controller so we can forwarded users to the appropriate application in the backend. The applications don't have this URL/Path configured on them:
```
http://<ingress-service>:<ingress-port>/watch --> http://<watch-service>:<port>/
http://<ingress-service>:<ingress-port>/wear --> http://<wear-service>:<port>/
```

- Without the rewrite-target option, this is what would happen:
```
http://<ingress-service>:<ingress-port>/watch --> http://<watch-service>:<port>/watch

http://<ingress-service>:<ingress-port>/wear --> http://<wear-service>:<port>/wear
```

- Notice watch and wear at the end of the target URLs. The target applications are not configured with /watch or /wear paths. They are different applications built specifically for their purpose, so they don't expect /watch or /wear in the URLs. And as such the requests would fail and throw a 404 not found error.

- To fix that we want to "ReWrite" the URL when the request is passed on to the watch or wear applications. We don't want to pass in the same path that user typed in. So we specify the rewrite-target option. This rewrites the URL by replacing whatever is under rules->http->paths->path which happens to be /pay in this case with the value in rewrite-target. This works just like a search and replace function.

- For example: replace(path, rewrite-target)

In our case: replace("/path","/")
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: test-ingress
  namespace: critical-space
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - path: /pay
        backend:
          serviceName: pay-service
          servicePort: 8282
```

- In another example given here, this could also be:

replace("/something(/|$)(.*)", "/$2")
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$2
  name: rewrite
  namespace: default
spec:
  rules:
  - host: rewrite.bar.com
    http:
      paths:
      - backend:
          serviceName: http-svc
          servicePort: 80
        path: /something(/|$)(.*)
```