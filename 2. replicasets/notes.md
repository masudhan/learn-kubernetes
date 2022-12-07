## Replica Sets (Replication Controller)
*  It's responsibility is to maintain the # of replicas of the pod, this is used along with deployments
*  When we create deployment it'll automatically creates replica set as well..

*Difference b/w RS vs RC is, RC supports equality based selectors whereas RS supports both equality based as well as set based selectors* </br>
    Ex:</br>
    RC: env: prod , tier: fronend</br>
    RS: env: prod , tier: fronend, { key: tier, operator: In, values: [frontend] }</br>

## Commands to run
- `alias k=kubectl`</br>
- `k apply -f rs.yaml`</br>
- `k get rs`</br>
- `k get pods`</br>

name of the pods will be `webserver1-replicaset-6gm2p` (replicasetname-randomchars)
`k delete rs webserver1-replicaset`

If we do `k delete pod <any pod name>` , it'll create one more pod to maintain the replicas in our case it'll make sure to maintain 3 pods

`k get pods -o wide` - to check pods are in which nodes

## Problem with using only replicaset
1. Expose the replicaset - `k expose rs webserver1-replicaset --port=8000 --target-port=80 --type=NodePort
2. Now change the image name from `chmadhus/web-server-1:delhi` to `chmadhus/web-server-2:hyderabad` and apply that change, replicaset will be updated but pod will not be terminated and will have the latest change - this is the problem, it's responsibility is only to maintain the # of replicas, it doesn't know/care what's running behind and are there any changes made, in techinical terms, it doesn't know about any rollout happening behind the screen

## Solution 
Use Deployment