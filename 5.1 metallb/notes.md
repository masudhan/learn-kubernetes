Hirerachy - Pods >> Replicaset >> Deployment >> Services >> Ingress & Ingress Controller >> Load Balancer (Metallb)

## Installation By Manifest
* `kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.7/config/manifests/metallb-native.yaml`

This will create controller and speaker pods in metallb-system namespace

## Configuration
* We need to apply Layer 2 Configuration, check metallb-ipaddress-pool.yaml

`k apply -f metallb-ipaddress-pool.yaml`

Now go back to 5. ingress

