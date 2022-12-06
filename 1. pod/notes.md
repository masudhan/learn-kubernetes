## POD
*   POD is a wrapper which can contain one or more containers.If we have multiple containers in a pod, which means one container will serve the application code and other container will be for internal purpose like get the metrics of the pods. This second container is called as `sidecar container`
*   By default, when we create a pod, there will be a volume gets created which is called as `emptyDir`. This can be mounted to the container at different locations
    for ex: In main container, we can mount as /var/logs(emptyDir)
            In sidecard container, we can mount as /tmp/logs(emptyDir)

## Creating a pod using cmd
- `alias k=kubectl` </br> 
- `k run pod1 --image=chmadhus/web-server-1:delhi` This will create a pod, it was running on port 80 </br>
- `k run pod2 --image=chmadhus/web-server-2:hyderabad --dry-run=client -o yaml` - This will print the yaml manifest as stdout</br>
- `k run pod2 --image=chmadhus/web-server-1 --dry-run=client -o yaml > pod.yaml` - This will store the output in pod.yaml file</br>
- `k logs <podname>`</br>
- `k logs <podname> -c <pod1>` - if there are multiple containers and to check logs of particular container</br>