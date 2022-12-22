We'll learn, 

- Lables and annotations
- Advanced Scheduling
  - Node selctors
  - Taints and Tolerations
  - Node Affinity and Anti Affinity
  - Pod Affininty

I'm going to directly use commands to deploy the resources instead of using the `kubectl apply` command

**Labels**

```
Why labels and annotations? Where are we using?

We can use labels for scheduling or executing arbitary commands and also narrow down the search

For ex:
1.  Replicaset will be managed by lables
2.  In Deployment, if we update an image and apply those again, then based on labels it knows which all pods that i should delete and recreate again
3.  In Services, for which pods i should be sending traffic too, that's based on labels
4.  With lables, we can use them to sort

`alias k=kubectl`
`k create deploy web-server-1 --image=chmadhus/web-server-1:delhi --replicas 4`
`k create deploy web-server-2 --image=chmadhus/web-server-2:hyderabad --replicas 4`

❯ k get pods --show-labels
NAME                            READY   STATUS    RESTARTS   AGE   LABELS
web-server-1-7fb5bfdbbd-9svh2   1/1     Running   0          72s   app=web-server-1,pod-template-hash=7fb5bfdbbd
web-server-1-7fb5bfdbbd-bl8bg   1/1     Running   0          72s   app=web-server-1,pod-template-hash=7fb5bfdbbd
web-server-1-7fb5bfdbbd-rhmpg   1/1     Running   0          72s   app=web-server-1,pod-template-hash=7fb5bfdbbd
web-server-1-7fb5bfdbbd-zdzvw   1/1     Running   0          72s   app=web-server-1,pod-template-hash=7fb5bfdbbd
web-server-2-7599b869ff-d7qsr   1/1     Running   0          48s   app=web-server-2,pod-template-hash=7599b869ff
web-server-2-7599b869ff-l9qh8   1/1     Running   0          48s   app=web-server-2,pod-template-hash=7599b869ff
web-server-2-7599b869ff-nkszp   1/1     Running   0          48s   app=web-server-2,pod-template-hash=7599b869ff
web-server-2-7599b869ff-tg9lc   1/1     Running   0          48s   app=web-server-2,pod-template-hash=7599b869ff

So here we haven't added any lables while creating the deployment, by default k8s created them..

Now based on these labels we can sort the output
`k get pods -l app=web-server-1`


We can also give labels to the nodes, 
`k get nodes -o wide`

❯ k get nodes --show-labels
NAME       STATUS   ROLES           AGE   VERSION   LABELS
kmaster    Ready    control-plane   12h   v1.24.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=kmaster,kubernetes.io/os=linux,node-role.kubernetes.io/control-plane=,node.kubernetes.io/exclude-from-external-load-balancers=
kworker1   Ready    <none>          12h   v1.24.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=kworker1,kubernetes.io/os=linux
kworker2   Ready    <none>          12h   v1.24.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=kworker2,kubernetes.io/os=linux
3

for example, consider kworker1 as high performace node and kworker2 as not so high performance node

So we can assign custom labels to each node and use them when doing the deployment

for kworker1 - label is, `perf=high`
for kworker2 - label is, `perf=low`

`k label node kworker1 perf=high`

❯ k get nodes -l perf=high
NAME       STATUS   ROLES    AGE   VERSION
kworker1   Ready    <none>   12h   v1.24.0


`k label node kworker1 perf=low`

❯ k get nodes -l perf=low
NAME       STATUS   ROLES    AGE   VERSION
kworker2   Ready    <none>   12h   v1.24.0

Also based on the labels we can delete the deployments/pods etc
`k delete pods -l app=web-server-1`

*Maximum length of label is only 63 characters*

```

**Annotations**

```
Annoations are used to provide additional information for the resources and also to enable additional features

1st usecase:
ex: if i want to give some information about the deployment so i can give an annotation like

Deployed by : deamonkillerM, he is learning kubernetes and show casing them on the github for the outside world --> this is 111 characters

`k annotate node kworker2 info='Deployed by : deamonkillerM, he is learning kubernetes and show casing them on the github for the outside world'`

❯ k describe node kworker2
Name:               kworker2
Roles:              <none>
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=kworker2
                    kubernetes.io/os=linux
                    perf=low
Annotations:        info: Deployed by : deamonkillerM, he is learning kubernetes and show casing them on the github for the outside world

2nd usecase:

When we are exposing deployment using the service as LoadBalancer in AWS, it's going to create a Class Load Balancer, but if we add anotation like `service.beta.kubernetes.io/aws-load-balancer-type: "nlb"` then i'll create Network Load Balancer. Also when using nginx as an ingress resource, if we are using path based routing then we need to add an annotation like `nginx.ingress.kubernetes.io/rewrite-target: /`

```

**Advanced Scheduling**

    * Node Selector
        Earlier we've added label to node, perf=high and perf=low for kworker1 and kworker2 right.. Now using this label we can place containers on those nodes
          ❯ k get pods -o wide
            No resources found in default namespace.
          
           `k create -f web-server-1-perf-high.yaml`
           `k get pods -o wide` - you'll see all the pods will be in kworker1 because we've added a label in the manifest file under containers

           `k create -f web-server-2-perf-low.yaml`
           `k get pods -o wide`

        Here there is a problem, what if developers create the pods vice-versa, ex: if they do any deployment with 100 replicas and not given any nodeSelector field- pods will be distribued across all the nodes. So nodeselector is not that effective.. So to solve this problem we use taints and tolerations

    * Taints and Tolerations
        In real life,
          Imagine Taints is bad smell and if we are ok with that, then it's toleration
        Taints will be applied to Nodes
        Toleration will be applied to Pods

        Before doing this, i'll remove the labels which we have given earlier,
        `kubectl label node $(kubectl get nodes --no-headers | cut -d " " -f 1) perf-`

        Demo:
        ----

        `k create deploy web-server-1 --image=chmadhus/web-server-1:delhi --replicas 4`

        ❯ k get pods -o wide
          NAME                            READY   STATUS    RESTARTS   AGE   IP               NODE       NOMINATED NODE   READINESS GATES
          web-server-1-7fb5bfdbbd-9hvs8   1/1     Running   0          12s   192.168.77.148   kworker2   <none>           <none>
          web-server-1-7fb5bfdbbd-hwqf5   1/1     Running   0          12s   192.168.77.147   kworker2   <none>           <none>
          web-server-1-7fb5bfdbbd-nngcg   1/1     Running   0          12s   192.168.41.142   kworker1   <none>           <none>
          web-server-1-7fb5bfdbbd-z56k9   1/1     Running   0          12s   192.168.41.143   kworker1   <none>           <none>

        Now if we apply taint NoSchedule on kworker1, kworker2

        ❯ k taint node kworker1 perf=high:NoSchedule && k taint node kworker2 perf=low:NoSchedule
        node/kworker1 tainted
        ❯ k get pods -o wide - `existing ones won't get any impact`
        NAME                            READY   STATUS    RESTARTS   AGE    IP               NODE       NOMINATED NODE   READINESS GATES
        web-server-1-7fb5bfdbbd-9hvs8   1/1     Running   0          107s   192.168.77.148   kworker2   <none>           <none>
        web-server-1-7fb5bfdbbd-hwqf5   1/1     Running   0          107s   192.168.77.147   kworker2   <none>           <none>
        web-server-1-7fb5bfdbbd-nngcg   1/1     Running   0          107s   192.168.41.142   kworker1   <none>           <none>
        web-server-1-7fb5bfdbbd-z56k9   1/1     Running   0          107s   192.168.41.143   kworker1   <none>           <none>

        ************************************************************************************************************************************************************
        Now we'll create new deployment,
        
        ❯ k create deploy web-server-2 --image=chmadhus/web-server-2:hyderabad --replicas 4 or `k apply -f without-node-selector.yaml`
          deployment.apps/web-server-2 created
        ❯ k get pods -o wide
        NAME                            READY   STATUS    RESTARTS   AGE   IP               NODE       NOMINATED NODE   READINESS GATES
        web-server-1-7fb5bfdbbd-9hvs8   1/1     Running   0          11m   192.168.77.148   kworker2   <none>           <none>
        web-server-1-7fb5bfdbbd-hwqf5   1/1     Running   0          11m   192.168.77.147   kworker2   <none>           <none>
        web-server-1-7fb5bfdbbd-nngcg   1/1     Running   0          11m   192.168.41.142   kworker1   <none>           <none>
        web-server-1-7fb5bfdbbd-z56k9   1/1     Running   0          11m   192.168.41.143   kworker1   <none>           <none>
        web-server-2-7599b869ff-45rr8   0/1     Pending   0          6s    <none>           <none>     <none>           <none>
        web-server-2-7599b869ff-5tlhh   0/1     Pending   0          6s    <none>           <none>     <none>           <none>
        web-server-2-7599b869ff-brjn9   0/1     Pending   0          6s    <none>           <none>     <none>           <none>
        web-server-2-7599b869ff-tnk7j   0/1     Pending   0          6s    <none>           <none>     <none>           <none>
        
        Now the new pods (web-server-2) are all in pending state coz we have added taints(perf=high:NoSchedule and perf=low:NoSchedule) on both kworker1 and kworker2

        ❯ k describe pod web-server-2-7599b869ff-brjn9
        Events:
        Type     Reason            Age    From               Message
        ----     ------            ----   ----               -------
        Warning  FailedScheduling  2m45s  default-scheduler  0/3 nodes are available: 1 node(s) had untolerated taint {node-role.kubernetes.io/master: }, 1 node(s) had untolerated taint {perf: high}, 1 node(s) had untolerated taint {perf: low}. preemption: 0/3 nodes are available: 3 Preemption is not helpful for scheduling.

        ----------------------------------------
        We need to add tolerations to the containers, so that scheduler will place them on the respective nodes
        `k apply -f tolerations-perf-high.yaml`

        ❯ k get pods -o wide
          NAME                            READY   STATUS    RESTARTS   AGE   IP               NODE       NOMINATED NODE   READINESS GATES
          web-server-2-74b6498c6f-8rvrp   1/1     Running   0          4s    192.168.41.144   kworker1   <none>           <none>
          web-server-2-74b6498c6f-ldc92   1/1     Running   0          4s    192.168.41.145   kworker1   <none>           <none>
          web-server-2-74b6498c6f-td66v   1/1     Running   0          4s    192.168.41.146   kworker1   <none>           <none>

        Similarily, i'll apply toleration as perf=low and deploy - `k apply -f tolerations-perf-low.yaml`
        ❯ k get pods -o wide
          NAME                            READY   STATUS    RESTARTS   AGE    IP               NODE       NOMINATED NODE   READINESS GATES
          web-server-1-86f8d5585d-7zxp4   1/1     Running   0          4s     192.168.77.153   kworker2   <none>           <none>
          web-server-1-86f8d5585d-mfc7r   1/1     Running   0          4s     192.168.77.154   kworker2   <none>           <none>
          web-server-1-86f8d5585d-rp5m4   1/1     Running   0          4s     192.168.77.155   kworker2   <none>           <none>

        So by this, you're forcing developer to use the toleration. Here `NoSchedule` will be applied only to the newly created pods and we need to give tolerations to the containers, it'll not be applied to already running pods

        So first reemove taint on all the nodes and delete all the deploymets
        `kubectl taint node $(kubectl get nodes --no-headers | cut -d " " -f 1) perf-`

        If we give `NoExecute` taint on the nodes then already running pods will be terminated and will be recreated - this is very dangerous if we don't apply the toleration of existing containers

        `k create deploy web-server-1 --image=chmadhus/web-server-1:delhi --replicas 4`
          ❯ k get pods -o wide
          NAME                            READY   STATUS    RESTARTS   AGE   IP               NODE       NOMINATED NODE   READINESS GATES
          web-server-1-7fb5bfdbbd-47m52   1/1     Running   0          30s   192.168.77.157   kworker2   <none>           <none>
          web-server-1-7fb5bfdbbd-bz4wt   1/1     Running   0          30s   192.168.41.148   kworker1   <none>           <none>
          web-server-1-7fb5bfdbbd-kbgzr   1/1     Running   0          30s   192.168.41.147   kworker1   <none>           <none>
          web-server-1-7fb5bfdbbd-wzrbt   1/1     Running   0          30s   192.168.77.156   kworker2   <none>           <none>

        `k taint node kworker1 perf=high:NoExecute && k taint node kworker2 perf=low:NoExecute`

          ❯ k get pods -o wide 
          NAME                            READY   STATUS        RESTARTS   AGE    IP               NODE       NOMINATED NODE   READINESS GATES
          web-server-1-7fb5bfdbbd-47m52   1/1     Terminating   0          111s   192.168.77.157   kworker2   <none>           <none>
          web-server-1-7fb5bfdbbd-4v4q5   0/1     Pending       0          1s     <none>           <none>     <none>           <none>
          web-server-1-7fb5bfdbbd-bz4wt   1/1     Terminating   0          111s   192.168.41.148   kworker1   <none>           <none>
          web-server-1-7fb5bfdbbd-jflkh   0/1     Pending       0          1s     <none>           <none>     <none>           <none>
          web-server-1-7fb5bfdbbd-kbgzr   1/1     Terminating   0          111s   192.168.41.147   kworker1   <none>           <none>
          web-server-1-7fb5bfdbbd-lwdp2   0/1     Pending       0          1s     <none>           <none>     <none>           <none>
          web-server-1-7fb5bfdbbd-wz4j9   0/1     Pending       0          1s     <none>           <none>     <none>           <none>
          web-server-1-7fb5bfdbbd-wzrbt   1/1     Terminating   0          111s   192.168.77.156   kworker2   <none>           <none>
          
          ❯ k get pods -o wide
          NAME                            READY   STATUS    RESTARTS   AGE   IP       NODE     NOMINATED NODE   READINESS GATES
          web-server-1-7fb5bfdbbd-4v4q5   0/1     Pending   0          75s   <none>   <none>   <none>           <none>
          web-server-1-7fb5bfdbbd-jflkh   0/1     Pending   0          75s   <none>   <none>   <none>           <none>
          web-server-1-7fb5bfdbbd-lwdp2   0/1     Pending   0          75s   <none>   <none>   <none>           <none>
          web-server-1-7fb5bfdbbd-wz4j9   0/1     Pending   0          75s   <none>   <none>   <none>           <none>

        Now to make them schedule on kworker2
          `k apply -f tolerations-NoExecute.yaml`

        Or you can also do the patch on existing pods as well please refer this - (Update API Objects in Place Using kubectl patch) [https://kubernetes.io/docs/tasks/manage-kubernetes-objects/update-api-object-kubectl-patch/#:~:text=Create%20a%20file%20named%20patch%2Dfile%2Dtolerations.yaml%20that%20has%20this%20content]
        
        Or by command line - `k patch deployment web-server-1 -n default -p '{"spec": { "template": { "spec": {"tolerations": [{"effect": "NoExecute","key": "perf","value": "high" }]}}}}'`
        ************************************************************************************************************************************************************************

        * Node Affinity & Anti Affinity

        Node Affinity will attract the pods to come, where as taint will be applicable only when you have toleration
        We define affinity and nodeAffinity under spec section

        Remove all the taints which are there, 
        `kubectl taint node $(kubectl get nodes --no-headers | cut -d " " -f 1) perf-`

        `nodeSelector` is the simplest way to constrain Pods to nodes with specific labels. Affinity and anti-affinity expands the types of constraints you can define. Some of the benefits of affinity and anti-affinity include:

        1. The affinity/anti-affinity language is more expressive. `nodeSelector` only selects nodes with all the specified labels. Affinity/anti-affinity gives you more control over the selection logic.
        2. You can indicate that a rule is soft or preferred, so that the scheduler still schedules the Pod even if it can't find a matching node.
        3. You can constrain a Pod using labels on other Pods running on the node (or other topological domain), instead of just node labels, which allows you to define rules for which Pods can be co-located on a node.    

        first, we'll add some labels the node like
        `k label node kworker1 env=prod application=app1`
        `k label node kworker2 env=dev application=app2`    

         cd 'nodeaffinity & anti affinity'
        `k apply -f node-affinity-required.yaml`

        `k apply -f node-affinity-preferred.yaml`

        ❯ k get pods -l app=web-server-2 -o wide - here weightage has been given to kworker1 and also to kworker2
          NAME                            READY   STATUS    RESTARTS   AGE     IP               NODE       NOMINATED NODE   READINESS GATES
          web-server-2-6b9d89b455-5hvkm   1/1     Running   0          2m47s   192.168.41.139   kworker1   <none>           <none>
          web-server-2-6b9d89b455-fnzgw   1/1     Running   0          2m47s   192.168.41.138   kworker1   <none>           <none>
          web-server-2-6b9d89b455-glzt2   1/1     Running   0          2m47s   192.168.41.133   kworker1   <none>           <none>
          web-server-2-6b9d89b455-mnq4q   1/1     Running   0          2m47s   192.168.77.129   kworker2   <none>           <none>
          web-server-2-6b9d89b455-nd7pl   1/1     Running   0          2m47s   192.168.41.134   kworker1   <none>           <none>
          web-server-2-6b9d89b455-nrvc7   1/1     Running   0          2m47s   192.168.41.136   kworker1   <none>           <none>
          web-server-2-6b9d89b455-vnqpk   1/1     Running   0          2m47s   192.168.41.137   kworker1   <none>           <none>
          web-server-2-6b9d89b455-wk4qc   1/1     Running   0          2m47s   192.168.41.135   kworker1   <none>           <none>

          * Anti Affinity
            `NotIn` and `DoesNotExist` allow you to define node anti-affinity behavior. Alternatively, you can use node taints to repel Pods from specific nodes.
            `k apply -f node-anti-affinity-required.yaml` 
        ************************************************************************************************************************************
        * Pod Affinity

          Remove all the taints and labels first and add below topology
          `k label node kworker2 topology.kubernetes.io/zone=kworker2`
          `k apply -f pod-affinity-required.yaml`
          ❯ k get pods
            NAME                            READY   STATUS    RESTARTS   AGE
            web-server-1-57c96cc58f-bdk7f   0/1     Pending   0          2s
            web-server-1-57c96cc58f-h86xc   0/1     Pending   0          2s
            web-server-1-57c96cc58f-kk78j   0/1     Pending   0          2s

          `k run hello --image nginx:latest`
          `k label pod hello env=prod`

          As soon as we do this, all the pods will be moved to node in which hello pods running
          ❯ k get pods -o wide
            NAME                            READY   STATUS    RESTARTS   AGE     IP               NODE       NOMINATED NODE   READINESS GATES
            hello                           1/1     Running   0          35s     192.168.77.142   kworker2   <none>           <none>
            web-server-1-57c96cc58f-bdk7f   1/1     Running   0          2m19s   192.168.77.144   kworker2   <none>           <none>
            web-server-1-57c96cc58f-h86xc   1/1     Running   0          2m19s   192.168.77.143   kworker2   <none>           <none>
            web-server-1-57c96cc58f-kk78j   1/1     Running   0          2m19s   192.168.77.145   kworker2   <none>           <none>





        
          












        




        





