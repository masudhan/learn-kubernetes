Hirerachy - Pods >> Replicaset >> Deployment

## Deployment
This will know if there are any changes to the manifest file and apply those changes

`alias k=kubectl`</br>
`k create deploy web-server-2 --image=chmadhus/web-server-2:hyderabad --replicas 3 --dry-run=client -o yaml`</br>
`k expose deploy web-server-2 --port=8000 --target-port=80 --type=NodePort`</br>
*output*</br>
    `web-server-2   NodePort    10.97.186.58   <none>        8000:31763/TCP   5s`</br>

To access from browser, `nodeIP<NodePort>` </br>

Now change the image from `chmadhus/web-server-2:hyderabad` to `chmadhus/web-server-1:delhi` and refresh the browser

If you want to scale the # of replcias -
`k scale deployment --replicas=6 <deployment>`

For deployment-v2.yaml</br>
Reference - https://www.bluematador.com/blog/kubernetes-deployments-rolling-update-configuration
Added maxSurge and maxUnavailable under replicas >> strategy
* maxSurge 
    - if maxSurge: 1, then if there are 4 replicas which are already running, then it'll create one more replica, then terminate old pod, once the new pods are up and running,
    along with this if we give minReadySeconds: 60, which means, after the new pod is up and running, it'll wait for 60 seconds then will terminate old pod and at the same time another new pod will be created
* maxUnavailable
    - Already there are 4 replicas running, from that how many that i can delete

- `k rollout history deployment apache-deployment` - this will give us the history, but it'll not show *CHANGE-CAUSE* to get that only using cmd line
- `k set image dployment apache-deployment apache=apache:latest --record` - this is callled *record set* now check the rollout history