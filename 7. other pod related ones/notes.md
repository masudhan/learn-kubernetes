**Deployments**

`k create deploy web-server-1 --image=chmadhus/web-server-1:delhi --replicas 3 --dry-run=client -o yaml`  [or]  `k apply -f deployment.yaml` - this is gonna create 3 pods

```
`k get pods`

NAME                            READY   STATUS    RESTARTS   AGE
web-server-1-7fb5bfdbbd-2sszq   1/1     Running   0          42s
web-server-1-7fb5bfdbbd-nz5nl   1/1     Running   0          42s
web-server-1-7fb5bfdbbd-t2hm6   1/1     Running   0          42s

```

Added rolling update strategy and changed the image as well in deployment-2.yaml - `k apply -f deployment-2.yaml`

```
`k get pods`

NAME                          READY   STATUS    RESTARTS   AGE
web-server-1-77677d59-gcf4w   1/1     Running   0          9s
web-server-1-77677d59-gtfrz   1/1     Running   0          8s
web-server-1-77677d59-n5czj   1/1     Running   0          10s

```

So thing that you need to observe here is, pod names were changed when we changed some configuration in manifest file. That's fine, because we are working at service level right. </br>


--------------

`k get pods -o wide` - check the NODE column pods are distributed across 2 worker nodes
```
NAME                            READY   STATUS    RESTARTS   AGE   IP               NODE       NOMINATED NODE   READINESS GATES
web-server-1-7fb5bfdbbd-r9mdt   1/1     Running   0          4s    192.168.41.141   kworker1   <none>           <none>
web-server-1-7fb5bfdbbd-vf5fj   1/1     Running   0          4s    192.168.77.138   kworker2   <none>           <none>
web-server-1-7fb5bfdbbd-wlphw   1/1     Running   0          4s    192.168.77.139   kworker2   <none>           <none>

```

`k get nodes -o wide`
```
NAME       STATUS   ROLES           AGE   VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE           KERNEL-VERSION      CONTAINER-RUNTIME
kmaster    Ready    control-plane   50m   v1.24.0   172.16.16.100   <none>        Ubuntu 22.04 LTS   5.15.0-33-generic   containerd://1.5.9-0ubuntu3
kworker1   Ready    <none>          44m   v1.24.0   172.16.16.101   <none>        Ubuntu 22.04 LTS   5.15.0-33-generic   containerd://1.5.9-0ubuntu3
kworker2   Ready    <none>          37m   v1.24.0   172.16.16.102   <none>        Ubuntu 22.04 LTS   5.15.0-33-generic   containerd://1.5.9-0ubuntu3
```
`k drain kworker2 --ignore-daemonsets` - put kworker2 node under maintainence, when we do this, pods which are running in kworker 2 will be terminated and will be shifted to other available node, in our case it's kworker1

```
_ k get nodes -o wide
NAME       STATUS                     ROLES           AGE   VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE           KERNEL-VERSION      CONTAINER-RUNTIME
kmaster    Ready                      control-plane   51m   v1.24.0   172.16.16.100   <none>        Ubuntu 22.04 LTS   5.15.0-33-generic   containerd://1.5.9-0ubuntu3
kworker1   Ready                      <none>          45m   v1.24.0   172.16.16.101   <none>        Ubuntu 22.04 LTS   5.15.0-33-generic   containerd://1.5.9-0ubuntu3
kworker2   Ready,SchedulingDisabled   <none>          38m   v1.24.0   172.16.16.102   <none>        Ubuntu 22.04 LTS   5.15.0-33-generic   containerd://1.5.9-0ubuntu3
```
`k get pods -o wide` - now if you see the NODE column, all the 3 pods are in kworker1
```
NAME                            READY   STATUS    RESTARTS   AGE   IP               NODE       NOMINATED NODE   READINESS GATES
web-server-1-7fb5bfdbbd-r9mdt   1/1     Running   0          38s   192.168.41.141   kworker1   <none>           <none>
web-server-1-7fb5bfdbbd-rcvgv   1/1     Running   0          7s    192.168.41.143   kworker1   <none>           <none>
web-server-1-7fb5bfdbbd-wws46   1/1     Running   0          7s    192.168.41.142   kworker1   <none>           <none>

```
Now, `k uncordon kworker2` - kworker2 is in ready/scheduable state
```
_ k get nodes -o wide
NAME       STATUS   ROLES           AGE   VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE           KERNEL-VERSION      CONTAINER-RUNTIME
kmaster    Ready    control-plane   60m   v1.24.0   172.16.16.100   <none>        Ubuntu 22.04 LTS   5.15.0-33-generic   containerd://1.5.9-0ubuntu3
kworker1   Ready    <none>          55m   v1.24.0   172.16.16.101   <none>        Ubuntu 22.04 LTS   5.15.0-33-generic   containerd://1.5.9-0ubuntu3
kworker2   Ready    <none>          47m   v1.24.0   172.16.16.102   <none>        Ubuntu 22.04 LTS   5.15.0-33-generic   containerd://1.5.9-0ubuntu3

_ k get pods -o wide
NAME                            READY   STATUS    RESTARTS   AGE     IP               NODE       NOMINATED NODE   READINESS GATES
web-server-1-7fb5bfdbbd-r9mdt   1/1     Running   0          6m30s   192.168.41.141   kworker1   <none>           <none>
web-server-1-7fb5bfdbbd-rcvgv   1/1     Running   0          5m59s   192.168.41.143   kworker1   <none>           <none>
web-server-1-7fb5bfdbbd-wws46   1/1     Running   0          5m59s   192.168.41.142   kworker1   <none>           <none>

So thing here is, when we made the kworker2 in ready/scheduable state, pod are still in kworker1 only, to make them distribute

`k scale deployment --replicas 6 web-server-1`

NAME                            READY   STATUS    RESTARTS   AGE     IP               NODE       NOMINATED NODE   READINESS GATES
web-server-1-7fb5bfdbbd-j85hm   1/1     Running   0          18s     192.168.77.142   kworker2   <none>           <none>
web-server-1-7fb5bfdbbd-jv9pn   1/1     Running   0          18s     192.168.77.141   kworker2   <none>           <none>
web-server-1-7fb5bfdbbd-jvbk7   1/1     Running   0          18s     192.168.77.140   kworker2   <none>           <none>
web-server-1-7fb5bfdbbd-r9mdt   1/1     Running   0          9m33s   192.168.41.141   kworker1   <none>           <none>
web-server-1-7fb5bfdbbd-rcvgv   1/1     Running   0          9m2s    192.168.41.143   kworker1   <none>           <none>
web-server-1-7fb5bfdbbd-wws46   1/1     Running   0          9m2s    192.168.41.142   kworker1   <none>           <none>

`k scale deployment --replicas 1 web-server-1`
`k scale deployment --replicas 3 web-server-1`
NAME                            READY   STATUS    RESTARTS   AGE   IP               NODE       NOMINATED NODE   READINESS GATES
web-server-1-7fb5bfdbbd-4pbd6   1/1     Running   0          43s   192.168.77.151   kworker2   <none>           <none>
web-server-1-7fb5bfdbbd-kq5pm   1/1     Running   0          43s   192.168.77.150   kworker2   <none>           <none>
web-server-1-7fb5bfdbbd-r9mdt   1/1     Running   0          16m   192.168.41.141   kworker1   <none>           <none>

```
Why are we doing all of this? :thinking: To undestand `daemonsets` </br> 

**Daemonsets:**

For general applications, it's ok to have the workloads in 1 node, but there are some instances, for example, imagine i've 3 nodes and i've done monitoring setup and my expectation is to have 1 pod in each node. To make this kind of setup done, deployments will not be the right one to choose. </br>
To have more clarity, for example, i have 3 nodes (node-1, node-2, node-3), and 3 pods (pod-1, pod-2, pod-3) are running on each node and i've used deployment strategy. what if node-3 is down/rebooted? then pod-3 will be moved to node-2. Now node-2 will have pod-2 and pod-3. Now, what if node-2 is down/rebooted ? then pod-2 and pod-3 will move to node-1. Now node-1 will have pod-1, pod-2 and pod-3. Now when node-2 and node-3 are up and running these pod-2 and pod-3 will not go back/create new pods to their respective nodes, now that is the problem with deployment when we are dealing with monitorning pods (mon-pod). `daemonsets` is our rescue, this will make sure that when the nodes are up it'll ensure to maintain one pod for each node </br>

By default when we create a cluster there will be a daemonsets running in kube-system namespace - `k get ds -n kube-system`
```
❯ k get ds -n kube-system
NAME          DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
calico-node   3         3         3       3            3           kubernetes.io/os=linux   88m
kube-proxy    3         3         3       3            3           kubernetes.io/os=linux   88m
```

`k apply -f daemonsets.yaml` - this will create 
```
_ k get pods,ds -o wide
NAME                   READY   STATUS    RESTARTS   AGE    IP               NODE       NOMINATED NODE   READINESS GATES
pod/deamon-set-gqscb   1/1     Running   0          83s    192.168.41.148   kworker1   <none>           <none>
pod/deamon-set-r57s5   1/1     Running   0          112s   192.168.77.154   kworker2   <none>           <none>

NAME                        DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE    CONTAINERS   IMAGES                            SELECTOR
daemonset.apps/deamon-set   2         2         2       2            2           <none>          4m9s   deamon-set   chmadhus/web-server-2:hyderabad   name=deamon-set

Now even if we delete any pod, it'll again recreate in the deleted node

`k delete pod/deamon-set-r57s5`

_ k get pods,ds -o wide
NAME                   READY   STATUS    RESTARTS   AGE    IP               NODE       NOMINATED NODE   READINESS GATES
pod/deamon-set-8nw6h   1/1     Running   0          2s     192.168.77.155   kworker2   <none>           <none>
pod/deamon-set-gqscb   1/1     Running   0          2m1s   192.168.41.148   kworker1   <none>           <none>

NAME                        DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE     CONTAINERS   IMAGES                            SELECTOR
daemonset.apps/deamon-set   2         2         2       2            2           <none>          4m47s   deamon-set   chmadhus/web-server-2:hyderabad   name=deamon-set


```
**Difference between deployment vs daemonsets, deployments is suitable for regular applications, whereas deamonsets is suitable for applications which needs to be run in every node, examples - logs and monitorning, if the pods are deleted in deployment or daemonsets, the pod names will be changed coz of one-to-one mapping **

------

**Statefulsets:**

Stateful uses ordinal index to manage the pods. Whatever the pods that you're going to create it'll start wih 0, 1, 2 etc. Apart from that name of the pods won't change these are mainly used when running the dbs. So in realtime, we'll have master as db-0 and slaves will be db-1 and db-2

`k apply -f statefulsets.yaml`

```
_ k get pods -o wide
NAME             READY   STATUS    RESTARTS   AGE   IP               NODE       NOMINATED NODE   READINESS GATES
web-server-1-0   1/1     Running   0          33s   192.168.77.156   kworker2   <none>           <none>
web-server-1-1   1/1     Running   0          32s   192.168.41.149   kworker1   <none>           <none>
web-server-1-2   1/1     Running   0          31s   192.168.77.157   kworker2   <none>           <none>

`k delete pod web-server-1-2`

_ k get pods
NAME             READY   STATUS    RESTARTS   AGE
web-server-1-0   1/1     Running   0          3m4s
web-server-1-1   1/1     Running   0          3m3s
web-server-1-2   1/1     Running   0          36s

```


