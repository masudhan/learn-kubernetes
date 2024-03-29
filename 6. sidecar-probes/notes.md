## Sidecar containers
* Along with main container, if we have one more container in a pod is called as sidecar container. There are 3 types of sidecar containers
    * init containers
    * adapter containers
    * ambassador containers
---------------

Usally we'll have a pod, and it'll have application container, along with that we can have additional one more container based on the business usecase. 

-- Usecase: </br>
    1.  Before starting the application container, suppose we want to check the if other containers have started or not OR check if services are up and running, best example would be imagine we have another pod which is a database pod (pgsql db) . Now the requirement is, application container should start only after the pgsql container is up or service is reachable on 5432 port number. So how we'll know if it is started or not? </br>
    So we'll keep a sidecar container in my pod along with application container. Now this sidecar container will go a test the service (5432). Once it's up, this sidecar container will be in `Completed` state and then application container will gets started </br>
    2.  If the application container is generating any logs and we want those logs, so we use sidecar container to copy the logs and store in S3 buckets or Digital ocean spaces

---------------

-- init containers </br>
    1. These are exactly like regular containers, except: </br>
        *   init containers always run to completion  </br>
        *   each init container must complete successfully before the next one starts </br>
        *   use case would be make sure db container/svc is running first then only start the application container
 
 **init containers will always start and executed and complete before the main container, we can have multiple init containers which will be executed sequentially**

 `k apply -f init-containers.yaml`

 --------------

 -- adapter container </br>
 Difference between init and adapter container is - </br>
    *   init container will do it's job and will be completed/terminated and will execute sequentially. Adapter container will run along with main/application container parallelly
    *   use case would be copy the files or logs from emptyDir and send them to AWS S3 or Elasticsearch

`k apply -f adapter-container.yaml` - so what am i doing here? i've a shared-volume which is empty-dir
    1. I have two containers which are busybox and chmadhus/web-server-1:delhi. So the busybox container will write data every 10 seconds to /httpd-data/index.html. here shared-volume will have data of /httpd-data/index.html.
    2. Now we are taking this /httpd-data/index.html and mounting to /usr/local/apache2/htdocs/ 


-- Verification

1st container - 
` k exec -it pod/httpd-webapp-7566948b8-f84hk -c busybox -- sh`
` cd /httpd-data`
` cat index.html`

2nd container - 
`k exec -it pod/httpd-webapp-7566948b8-f84hk -c adapter-container -- bash`
` cd htdocs`
` cat index.html`

--------------------------

-- ambassador container </br>
This will simplify accessing services outside the pod. When you're running applications on k8s there is a chance that you should access the data from the external services. This container will hides the complexity and provides the uniform interface to access the external services

Check this blog -https://medium.com/bb-tutorials-and-thoughts/kubernetes-learn-ambassador-container-pattern-bc2e1331bd3a



----------------------

### Probes
*   Rediness Probe
*   Liveness Probe
*   Startup Probe

------------------

* Rediness Probe
    1. This probe will decide when to allow traffic to the pods. When a Pod is not ready, it is removed from service load balancers based on this rediness probe signal
        - exec: Execute command for container status.
        - httpGet: HTTP GET Request for confirm container status.
        - tcpSocket: TCP Port check to confirm container status
    2. This will be configured under the container spec section

`k apply -f rediness-probe.yaml` </br>
Check in browser - `NodePortIPAddress<serviceporr>`

**Problem**
What if the data, in our case index.html file is deleted? 

I'm going to get into each pod and delete them, or </br>
    `k get pods --no-headers | cut -d " " -f 1` - with this you'll get only pod names
    `for pod in $(kubectl get pods --no-headers | cut -d " " -f 1) 
    do 
        kubectl exec -it $pod -- rm -f /usr/local/apache2/htdocs/index.html; 
        sleep 1; 
    done`

Now if you check in browser, Application will be down, but the pods are running. Rediness probe will help you 

----
readinessProbe:</br>
    initialDelaySeconds: 30 ==> wait for 30 seconds and then do the task </br>
    periodSeconds: 5 ==> check every 5 seconds whether the mentioned index.html file is available or not and port is listening or not </br> 
    timeoutSeconds: 10 </br>
    successThreshold: 1 ==> if the index.html file is avaiable in the first iteration then send traffic to that pod </br>
    failureThreshold: 3 ==> if the index.html file is not avaiable then do 3 iterations, if it fails for 3 times then don't send traffic to that pod </br>
    httpGet:</br> 
        path: /index.html</br>
        port: 80</br>

Now delete the index.html file inside any pod and check the status of pod.</br> 
`k describe pod <podname>` - you'll get rediness probe failed </br> 
So what happens is, since we delete the index.html file, it won't send the traffic to that pod, which is fine. But it is not self healing. I mean it'll not create a new pod. That's where liveness probe will help you</br> 

--------

* Liveness Probe
    1. This probe will decide when to restart the pod, instructions will be done by kubelet. Liveness probe could catch a deadlock, where an applicaton is running, but unable to make progress and restarting container helps in such case
        - exec: Execute command for container status.
        - httpGet: HTTP GET Request for confirm container status.
        - tcpSocket: TCP Port check to confirm container status
    2. This will be configured under the container spec section        

`k apply -f liveness_probe.yaml`

Check in browser - `NodePortIPAddress<serviceporr>`

----
livenessProbe:</br>
    initialDelaySeconds: 60 ==> this value is greater than rediness probe </br>
    periodSeconds: 5</br>
    timeoutSeconds: 10</br>
    successThreshold: 1</br>
    failureThreshold: 1</br>
    httpGet:</br>
        path: /index.html</br>
        port: 80</br>

Now delete the index.html file inside any pod and check the status of pod.</br> 
`k describe pod <podname>` </br>  

It'll automatically creates new pods

----                

**So the difference is rediness probe checks when to send the traffic to the container, when to stop the container, whereas liveness probe will make sure that when to restart the container**

**Always remember liveness probe initial delay should be greater than the rediness probe initial delay, you can give them with equal delay but that can done only by mentioning startup probe**
----


* Startup Probe
    1.  Kubelet uses startup probes to know when a container application has started. 
    2.  This probe will be used for legacy applications/slow starting ones, it disables Rediness and Liveness checks until it succeeds, making sure those probes don't interfere with the application startup.

`k apply -f startup_probe.yaml`
