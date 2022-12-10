Hirerachy - Pods >> Replicaset >> Deployment >> Services

## Services
Connect to pods from external world
- ClusterIP
- NodePort
- LoadBalancer
- Headless
- ExternalName

## Why to create services ?
for example, if the pods gets terminated and recreates again then ip's are gonna change and in container world, there is no value for ip addresses
to send traffic to backend pods we need to create a service 

`alias k=kubectl`
`k create deploy pod1 --image=chmadhus/web-server-1:delhi --replicas 5 --dry-run=client -o yaml` 

-- ClusterIP
  -  This is the default service and can be reached inside cluster only </br>
  -  `k expose deploy pod1 --port=8000 --target-port=80 --type=ClusterIP --dry-run=client -o yaml` - (service port = 8000, container-port = 80) </br>
  -  `k describe svc pod1` - this will have endpoints, how it is connected ? with labels and selector concept </br>
  -  `k get pods -l app=pod1`- this will give all the pods which has app=pod1 as label, check the deployment file </br>
  -  `k delete svc pod1` </br>

------------
-- NodePort
  - With this we can test our applications from external world </br>
  - By default, there will be some ephemeral ports like 30000 - 32767 (this can be altered by following - [Tweaking](https://stackoverflow.com/questions/54752821/how-to-set-service-node-port-range-and-then-be-able-to-deploy-services-using-the)) </br>
  - `k expose deploy pod1 --port=8000 --target-port=80 --type=NodePort --dry-run=client -o yaml` - (service port = 8000, container-port = 80) </br> 
  - `k get svc` </br>
     -   Access from browser with any `NodeIP:<NodePort>` </br>

------------
-- LoadBalancer
  - This can be used for production, this is good when we only have one service in our cluster </br>
  -  `k expose deploy pod1 --port=8000 --target-port=80 --type=LoadBalaner --dry-run=client -o yaml` - (service port = 8000, container-port = 80) </br> 
        - If you are using AWS, then this will create Classic LB, to get ELB we need to add annotation `service.beta.kubernetes.io/aws-load-balancer-type: nlb` to NodePort.yaml file </br> 
  -  If there are 50+ services in your k8s cluster then using the LB service is very costly and also in AWS we have a limit of Max 50 ELB per region, which we don't want at all :no_mouth: </br>

---------------
-- ExternalName
   - This is used when we want to connect to any db/service outside of k8s cluster. Creates a specific DNS entry for easier application access. Maps the Service to the contents of the externalName field (e.g. foo.bar.example.com), by returning a CNAME record with its value. No proxying of any kind is set up. check ExternalName.yaml
   - Sometimes if we don't have any dns name for the server but we have only ip address then in that case we will create a service with endpoints, check ServiceWithIP.yaml

--------------
**NOTE: Service will act as a loadbalancer to backend pods**

---------------
-- Headless
   - Sometimes the requirement will be like, you dont want the service not to do the loadbalance, but i want the the IP's which are connected to that service in that case we use headless svc, only change in yaml file is to give `clusterIP: None`
   - Get into pod and do `nslookup headless-1` - this will list all the pod IP's which are connected to that service, in all other cases, it'll nslookup will resolve with service ip address




