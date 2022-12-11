## Generate Fake SSL/TLS Certificates

We've already deployed Nginx Ingress Controller 

1. Cert-Manager
    `kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.10.1/cert-manager.yaml` - this will deploy huge # of resources in cert-manager namespace
    `k get all -n cert-manager`
    `k get crds | grep cert-manager` - this will show all the custom resources which are created by cert-manager out of them we need clusterissuer

Now we need to deploy cluster-issuer.yaml - give your email id

Now go back to 4. Ingressess/ingress-resource-2.yaml


Reference - https://www.digitalocean.com/community/tutorials/how-to-set-up-an-nginx-ingress-with-cert-manager-on-digitalocean-kubernetes

