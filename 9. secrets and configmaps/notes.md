**How to manage sensitive data in k8s**

Sensitive data can be </br>
- Passwords
- Access keys and Secret access keys
- SSL/TLS Certificates
- Container Registry (GCR, ECR private registry)

```
Ex - 
If your application container wants to connect to mongodb, you can either hardcode mongodb username and password in your application container, which will be visible to everyone who checks out your docker continainer image. Best way would be to create a secret resource where you put your username and password and then when creating the  application container you can pull those secrets from secret file

Say for example, i want to encode my username and password and then put them into secrets manifest file 

cd secrets
vi secrets.yaml

for username - `echo -n "deamonkillerM" | base64`
for password - `echo -n "password@123" | base64`

save the file (:wq)

`k apply -f secrets.yaml`

❯ k get secrets
NAME          TYPE     DATA   AGE
secret-demo   Opaque   2      3s

❯ k describe secret secret-demo
Name:         secret-demo
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
password:  12 bytes
username:  13 bytes

❯ k get secret secret-demo -o yaml
apiVersion: v1
data:
  password: cGFzc3dvcmRAMTIz
  username: ZGVhbW9ua2lsbGVyTQ==
kind: Secret
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","data":{"password":"cGFzc3dvcmRAMTIz","username":"ZGVhbW9ua2lsbGVyTQ=="},"kind":"Secret","metadata":{"annotations":{},"name":"secret-demo","namespace":"default"},"type":"Opaque"}
  creationTimestamp: "2022-12-13T13:11:55Z"
  name: secret-demo
  namespace: default
  resourceVersion: "40609"
  uid: 919d68a1-9ecb-4371-adae-684288324610
type: Opaque
```

```
What if you want to create secrets via the command line?

First delete the existing one,

`k delete secret secret-demo`

`k create secret generic secret-demo --from-literal=username=deamonkillerM --from-literal=password=password@123`

❯ k get secrets
NAME          TYPE     DATA   AGE
secret-demo   Opaque   2      4s


Also checkout, `k create secret --help`

you can also create username in one file and password in another file and can give it as,
`k create secret generic secret-demo --from-file./<filename>`

```

```
How to use the secrets in your pods

There are couple of ways you can do
1. Use secret as an env variable inside your container
2. Mount it as a volume - secret will mounted as individual files inside your container which you can then reference

1. create secret username as env variable
`k apply -f env-reference.yaml`

Once the pod gets created, get into the pod

❯ k exec -it busybox -- sh
/ # env | grep myusername
myusername=deamonkillerM
/ # echo $myusername
deamonkillerM
/ # 

2. Mount it as a volume
`k create -f volume-reference.yaml`
❯ k exec -it busybox -- sh
/ # cd mydata/
/mydata # ls -la
lrwxrwxrwx    1 root     root            15 Dec 13 15:35 password -> ..data/password
lrwxrwxrwx    1 root     root            15 Dec 13 15:35 username -> ..data/username

```

```
It is also possible that, if you want to update your username or password or add any extra secrets you can add those changes and apply them, it'll be reflected in existing pods and newly created pods as well.

Note - It'll take couple of minutes to reflect in existing pods

To test that, i'm going to update my password and add another key-value pair
❯ echo -n 'MadhuSudhan' | base64
TWFkaHVTdWRoYW4=
❯ echo -n 'newvalue' | base64
bmV3dmFsdWU=

`k apply -f secrets-2.yaml` - secret name will be same

busybox pod is already running,

❯ k describe secret secret-demo
Name:         secret-demo
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
newkey:    8 bytes
password:  12 bytes
username:  11 bytes

/mydata # ls -ltr
total 0
lrwxrwxrwx    1 root     root            15 Dec 13 15:35 username -> ..data/username
lrwxrwxrwx    1 root     root            15 Dec 13 15:35 password -> ..data/password
lrwxrwxrwx    1 root     root            13 Dec 13 15:49 newkey -> ..data/newkey

/mydata # cat username 
MadhuSudhan
/mydata # cat password 
password@123
/mydata # cat newkey 
newvalue



```


**ConfigMap** 

```
This can be used when you've non-sensitive or application configuration information

cd configmaps
`k apply -f configmap.yaml`

❯ k get configmap
NAME               DATA   AGE
configmap-demo     2      17s
kube-root-ca.crt   1      10h

❯ k describe configmap configmap-demo
Name:         configmap-demo
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
packagename:
----
httpd
version:
----
2.4

Now, how to refer these in pod?
1. As an env variable
2. As a volume mount

`k apply -f env-reference.yaml`

❯ k exec -it busybox -- sh
/ # env | grep package
package=httpd

Now as volume reference, 
`k delete pod busybox`
`k apply -f volume-reference.yaml`

❯ k exec -it busybox -- sh
/ # cd mydata/
/mydata # ls -ltr
total 0
lrwxrwxrwx    1 root     root            14 Dec 13 16:13 version -> ..data/version
lrwxrwxrwx    1 root     root            18 Dec 13 16:13 packagename -> ..data/packagename

Now same as secrets, It is also possible that, if you want to update/add any new config data you can add those changes and apply them, it'll be reflected in existing pods and newly created pods as well.

`k apply -f configmap-2.yaml`

❯ k describe configmap configmap-demo
Name:         configmap-demo
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
packagename:
----
apache2
version:
----
2.3
maintainer:
----
deamonkillerM

Now get into pod and check,
❯ k exec -it busybox -- sh
/ # cd mydata/
/mydata # ls
maintainer   packagename  version
/mydata # 

```

*It's also possible to use secrets and configmap in single manifest file as well*
*We also have secrets which are immutable (values cannot be changed) to do this add `immutable: true` under `kind` in your secrets manifest files*