**Network Policies**

Till now we haven't concentrated on security and pod-to-pod communication..

As we know, namespaces are like virtual clusters. In that we can do,

  * Apply Resources
  * Apply Limits
  * RBAC
  
Namespace is not a network boundary/isolation.

Ex: lets say i've 3 namespaces

delhi
hyderabad
bangalore

if i keep 3 pods, there will be communication in between them..so to test them

I'll create 3 namespaces and create pods inside those ones and we'll test the communication

`alias k=kubectl`

`k create ns delhi`

`k create ns hyderabad`

`k create ns bangalore`

*Labeling*

`k label ns delhi nsp=delhi`

`k label ns hyderabad nsp=hyderabad`

`k label ns bangalore nsp=bangalore`


*Creating a pod*

`k run -n delhi web-server-1 --image chmadhus/web-server-1:delhi -l ns=delhi`

`k run -n hyderabad web-server-2 --image chmadhus/web-server-2:hyderabad -l ns=hyderabad`

`k run -n bangalore web-server-3 --image chmadhus/web-server-3:bangalore -l ns=bangalore`


```
❯ k get pods -o wide -n delhi
NAME           READY   STATUS    RESTARTS   AGE    IP               NODE       NOMINATED NODE   READINESS GATES
web-server-1   1/1     Running   0          2m1s   192.168.77.146   kworker2   <none>           <none>

❯ k get pods -o wide -n hyderabad
NAME           READY   STATUS    RESTARTS   AGE    IP               NODE       NOMINATED NODE   READINESS GATES
web-server-2   1/1     Running   0          104s   192.168.41.146   kworker1   <none>           <none>

❯ k get pods -o wide -n bangalore
NAME           READY   STATUS    RESTARTS   AGE    IP               NODE       NOMINATED NODE   READINESS GATES
web-server-3   1/1     Running   0          117s   192.168.77.148   kworker2   <none>           <none>

```


*Checking pod communication*

web-server-1 pod which is running in delhi namespace, should ping the pods which are running in hyderabad and bangalore namespace

web-server-2 pod which is running in hyderabad namespace, should ping the pods which are running in delhi and bangalore namespace

web-server-3 pod which is running in bangalore namespace, should ping the pods which are running in hyderabad and delhi namespace

get into each pod and do `apt update && apt install iputils-ping -y`

`k exec -it web-server-1 -n delhi --  ping -c 3 192.168.41.146`

`k exec -it web-server-1 -n delhi --  ping -c 3 192.168.77.148`

`k exec -it web-server-2 -n hyderabad --  ping -c 3 192.168.77.146`

`k exec -it web-server-2 -n hyderabad --  ping -c 3 192.168.77.148`

`k exec -it web-server-3 -n bangalore --  ping -c 3 192.168.77.146`

`k exec -it web-server-3 -n bangalore --  ping -c 3 192.168.41.146`

We'll be able to ping to all the pods which are in different namespaces

This is proving that even when you have applied RBAC for namespaces, but because all of them are in the same network then it's possible that pod to pod communication will happen and we can't block that. Now to block them we need to use network policies

So using network policies, we can block access between namespaces/pods

First i'll create one more in each namespace,

`k run -n delhi web-server-1-1 --image chmadhus/web-server-1:delhi -l ns=delhi`

`k run -n hyderabad web-server-2-1 --image chmadhus/web-server-2:hyderabad -l ns=hyderabad`

`k run -n bangalore web-server-3-1 --image chmadhus/web-server-3:bangalore -l ns=bangalore`

**Block traffic in between the pods in a namespace**

`k get netpol -n delhi`

```
❯ k get netpol -n delhi
No resources found in delhi namespace.
```

`k apply -f delhi-deny-all.yaml`

```
❯ k get netpol -n delhi
NAME             POD-SELECTOR   AGE
delhi-deny-all   nsp=delhi      6s

❯ k get pods -n delhi -o wide
NAME             READY   STATUS    RESTARTS   AGE   IP               NODE       NOMINATED NODE   READINESS GATES
web-server-1     1/1     Running   0          44m   192.168.77.146   kworker2   <none>           <none>
web-server-1-1   1/1     Running   0          24m   192.168.41.147   kworker1   <none>           <none>

❯ k exec -it web-server-1-1 -n delhi --  ping -c 3 192.168.77.146
PING 192.168.77.146 (192.168.77.146) 56(84) bytes of data.

❯ k exec -it web-server-1 -n delhi --  ping -c 3 192.168.41.147
PING 192.168.41.147 (192.168.41.147) 56(84) bytes of data.


```
Ping is not working when both the pods are in same namespace 

Now same thing we'll apply to hyderabad and bangalore

`k apply -f bangalore-deny-all.yaml`
`k apply -f hyderabad-deny-all.yaml`


**Allow traffic from one pod to another in a namespace**

`k apply -f delhi-allow-pods-in-namespace.yaml` - we are telling, pods which are having labels as ns=delhi can communicate to the pods whose labels are app=delhinginx01..

**Allow/Deny pods from namespace level**

`k create ns mysqldb`

`k label ns mysqldb nsp=mysqldb`

`k create -f mysqldb.yaml`

Install [kubens](https://github.com/ahmetb/kubectx#manual-installation-macos-and-linux:~:text=Example%20installation%20steps%3A) 

`kubens mysqldb`


`k run -n delhi web-server-1 --image chmadhus/web-server-1:delhi -l ns=delhi`

`k run -n hyderabad web-server-2 --image chmadhus/web-server-2:hyderabad -l ns=hyderabad`

check if web-server-2 and web-server-1 are able to connect to mysqldb on port 3306, before that you need to install telnet so enter into web-server-2 and install telnet

`k exec -it web-server-2 -n hyderabad -- telnet "<mysqldb ip>" 3306` - it'll say connected 

Now first i don't want any one to ping/telent mysqldb so i'll create an ingress to deny all

`k apply -f ingress-deny-mysqldb.yaml`

```
❯ k exec -it web-server-2 -n hyderabad -- telnet  192.168.41.150 3306
Trying 192.168.41.150...

❯ k exec -it web-server-1 -n delhi -- telnet  192.168.41.150 3306
Trying 192.168.41.150...

```
Now we'll allow hyderabad namespace to connect to mysqldb

```
❯ k get ns --show-labels | grep hyd
hyderabad         Active   163m   kubernetes.io/metadata.name=hyderabad,nsp=hyderabad

```

`k apply -f allow-pods-from-hyderabad-to-mysqldb.yaml` - here we have mentioned the `namespaceSelector as nsp=hyderabad` and `podSelector as app: mysql` which means namespace which is having label as nsp=hyderabad, they (pods) can be connect to the pods which are having label as app: mysql in mysqldb namespace

```
❯ k exec -it web-server-2 -n hyderabad -- telnet  192.168.41.150 3306
Trying 192.168.41.150...
Connected to 192.168.41.150.

❯ k exec -it web-server-1 -n delhi -- telnet  192.168.41.150 3306 - this is not possible because we haven't applied network policy from delhi namespace
Trying 192.168.41.150...

❯ k exec -it web-server-2 -n hyderabad -- ping 192.168.41.150
PING 192.168.41.150 (192.168.41.150) 56(84) bytes of data.
64 bytes from 192.168.41.150: icmp_seq=1 ttl=62 time=0.600 ms
64 bytes from 192.168.41.150: icmp_seq=2 ttl=62 time=1.29 ms

```

As you can see above, we are passing all the traffic to mysqldb. Our requirement is only to connect to 3306 port 

`k apply -f allow-pods-from-hyderabad-to-mysqldb-to-port-3306-only.yaml`

```
❯ k exec -it web-server-2 -n hyderabad -- telnet  192.168.41.150 3306
Trying 192.168.41.150...
Connected to 192.168.41.150.

❯ k exec -it web-server-2 -n hyderabad -- ping  192.168.41.150
PING 192.168.41.150 (192.168.41.150) 56(84) bytes of data.

```

Cool..

Now i don't want mysqldb to connect to external world, i want to restrict that.. To do that we need to use Egress

`k apply -f mysql-egress-deny-all.yaml`


We can also egress to specific namespace on specific port

`k apply -f mysql-egress-specific-ns-port.yaml`


Reference - https://reuvenharrison.medium.com/an-introduction-to-kubernetes-network-policies-for-security-people-ba92dd4c809d