## Requirements
1. Ubuntu or any linux distribution
2. Atleast 12GB RAM
3. Virtualbox
4. Vagrant
6. Kubectl
7. Helm

## Setup
* Here I'm using Vagrant for setting up my kubernetes cluster, refer - https://github.com/deamonkillerM/kubernetes/tree/master/vagrant-provisioning
    clone this repo, and run `vagrant up`, it'll launch one master node and two worker nodes
    Master Node(s) has 2 CPU(s), 2048MB RAM
    Worker Node(s) has 1 CPU(s), 1024MB RAM


Once the nodes are up and running, we need to copy the kube config file from kmaster to our local so run below commands
* `mkdir ~/.kube`
* `scp root@172.16.16.100:/etc/kubernetes/admin.conf ~/.kube/config` - password - `kubeadmin`
* `kubectl get nodes -o wide`

To stop the clutser
* `cd vagrant-provisioning` </br>
* `vagrant destroy -f`
* `rm -rf ~/.ssh/*`
* `rm -rf ~/.kube/*`
* `vboxmanage list hostonlyifs | grep -q '172.16.16.1' && vboxmanage hostonlyif remove vboxnet0`
