Q1  ############### JSON PATH ####################

### Get the list of nodes in JSON format and store it in a file at /opt/outputs/nodes.json

### ANSWER
kubectl get nodes -o json > /opt/outputs/nodes.json

### CHECK
vim /opt/outputs/nodes.json


Q2 ############### JASON PATH #########################

### Get the details of the node node01 in json format and store it in the file /opt/outputs/node01.json

### ANSWER
kubectl get node node01 -o json > /opt/outputs/node01.json

### CHECK

vim /opt/outputs/node01.json

Q3 ################# JSON PATH #######################

### Use JSON PATH query to fetch node names and store them in /opt/outputs/node_names.txt

### ANSWER

kubectl get nodes -o json > /opt/outputs/nodes.json

Let us figure out the jason path looking at the the /opt/outputs/nodes.json using below command:

vim /opt/outputs/nodes.jso

The json path is figured out to be: $.items[*].metadata.name

kubectl get nodes -o=jsonpath='{.items[*].metadata.name}' > /opt/outputs/node_names.txt


Q4 ################## JASON PATH ##################################

### Use JSON PATH query to retrieve the osImages of all the nodes and store it in a file /opt/outputs/nodes_os.txt . The osImages are under the nodeInfo section under status of each node.

### ANSWER

kubectl get nodes -o json > /opt/outputs/nodes.json

Let us figure out the jason path looking at the the /opt/outputs/nodes.json using below command:

vim /opt/outputs/nodes.jso

The json path is figured out to be: $.items[*].status.nodeInfo.osImage

kubectl get nodes -o jsonpath='{.items[*].status.nodeInfo.osImage}' > /opt/outputs/nodes_os.txt


Q5 ################## JSON PATH ########################################

### A kube-config file is present at /root/my-kube-config. Get the user names from it and store it in a file /opt/outputs/users.txt. Use the command kubectl config view --kubeconfig=/root/my-kube-config to view the custom kube-config.

### ANSWER

Let us have the json file for this kubeconfig file usinbg the below command;

kubectl config view --kubeconfig=/root/my-kube-config -o json > mytestkubeconfig.json

Now let us figure out the json path from the file mytestkubeconfig.json using the below command:

vim mytestkubeconfig.json

The json path is figured out to be: $.users[*].name

kubectl config view --kubeconfig=/root/my-kube-config -o jsonpath="{.users[*].name}" > /opt/outputs/users.txt


Q6 ################################## JSON PATH #########################################

### A set of Persistent Volumes are available. Sort them based on their capacity and store the result in the file /opt/outputs/storage-capacity-sorted.txt

### ANSWER

Let us find the PV in json format and figure out the needful using the below commands:

kubectl get pv -o json > mypvsort.json

vim mypvsort.json

The json path is figured out to be: $.items[*].spec.capacity.storage

kubectl get pv --sort-by=.spec.capacity.storage > /opt/outputs/storage-capacity-sorted.txt


Q7 ########################## JSON PATH ##################################################

### That was good, but we don't need all the extra details. Retrieve just the first 2 columns of output and store it in /opt/outputs/pv-and-capacity-sorted.txt. The columns should be named NAME and CAPACITY. Use the custom-columns option. And remember it should still be sorted as in the previous question.

### ANSWER

kubectl get pv --sort-by=.spec.capacity.storage -o=custom-columns=NAME:.metadata.name,CAPACITY:.spec.capacity.storage > /opt/outputs/pv-and-capacity-sorted.txt


Q8 ########################## JSON PATH #################################################

### Use a JSON PATH query to identify the context configured for the aws-user in the my-kube-config context file and store the result in /opt/outputs/aws-context-name.

### ANSWER

Let us have the json file for this kubeconfig file usinbg the below command;

kubectl config view --kubeconfig=/root/my-kube-config -o json > mytestkubeconfig.json

Now let us figure out the json path from the file mytestkubeconfig.json using the below command:

vim mytestkubeconfig.json

The json path is figured out to be: $.contexts[?(@.context.user=='aws-user')].name

kubectl config view --kubeconfig=my-kube-config -o jsonpath="{.contexts[?(@.context.user=='aws-user')].name}" > /opt/outputs/aws-context-name


The mytestkubeconfig.json file is as given below:

{
    "kind": "Config",
    "apiVersion": "v1",
    "preferences": {},
    "clusters": [
        {
            "name": "development",
            "cluster": {
                "server": "KUBE_ADDRESS",
                "certificate-authority": "/etc/kubernetes/pki/ca.crt"
            }
        },
        {
            "name": "kubernetes-on-aws",
            "cluster": {
                "server": "KUBE_ADDRESS",
                "certificate-authority": "/etc/kubernetes/pki/ca.crt"
            }
        },
        {
            "name": "production",
            "cluster": {
                "server": "KUBE_ADDRESS",
                "certificate-authority": "/etc/kubernetes/pki/ca.crt"
            }
        },
        {
            "name": "test-cluster-1",
            "cluster": {
                "server": "KUBE_ADDRESS",
                "certificate-authority": "/etc/kubernetes/pki/ca.crt"
            }
        }
    ],
    "users": [
        {
            "name": "aws-user",
            "user": {
                "client-certificate": "/etc/kubernetes/pki/users/aws-user/aws-user.crt",
                "client-key": "/etc/kubernetes/pki/users/aws-user/aws-user.key"
            }
        },
        {
            "name": "dev-user",
            "user": {
                "client-certificate": "/etc/kubernetes/pki/users/dev-user/developer-user.crt",
                "client-key": "/etc/kubernetes/pki/users/dev-user/dev-user.key"
            }
        },
        {
            "name": "test-user",
            "user": {
                "client-certificate": "/etc/kubernetes/pki/users/test-user/test-user.crt",
                "client-key": "/etc/kubernetes/pki/users/test-user/test-user.key"
            }
        }
    ],
    "contexts": [
        {
            "name": "aws-user@kubernetes-on-aws",
            "context": {
                "cluster": "kubernetes-on-aws",
                "user": "aws-user"
            }
        },
        {
            "name": "research",
            "context": {
                "cluster": "test-cluster-1",
                "user": "dev-user"
            }
        },
        {
            "name": "test-user@development",
            "context": {
                "cluster": "development",
                "user": "test-user"
            }
        },
        {
            "name": "test-user@production",
            "context": {
                "cluster": "production",
                "user": "test-user"
            }
        }
    ],
    "current-context": "test-user@development"
}


Q9 ############################## PV & VOLUME CLAIM ###################################

### The application webapp stores logs at location /log/app.log. View the logs.

### ANSWER

kubectl exec -ti pod/webapp sh

vim /log/app.log


Q10 ######################## PV & VOLUME CLAIM ###########################################

### Configure a volume to store these logs at /var/log/webapp on the host. Use the spec below.

Name: webapp
Image Name: kodekloud/event-simulator
Volume HostPath: /var/log/webapp
Volume Mount: /log

### ANSWER

Let us create a manifest file named volume.yaml as given below:

Official documentation: https://kubernetes.io/docs/concepts/storage/volumes/

apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
  containers:
  - image: kodekloud/event-simulator
    name: test-container
    volumeMounts:
    - mountPath: /log
      name: test-volume
  volumes:
  - name: test-volume
    hostPath:
      # directory location on host
      path: /var/log/webapp
      # this field is optional
      type: Directory

Let us delte the existing pod and create a new one with the help of the above volume.yaml.

kubectl delete pod/webapp

kubectl create -f volume.yaml

Check:

kubectl get all

ssh node01

cd /var/log/webapp

ls


Q11 ########################## PV & VOLUME CLAIM #########################################


### Create a 'Persistent Volume' with the given specification.

Volume Name: pv-log
Storage: 100Mi
Access modes: ReadWriteMany
Host Path: /pv/log

### ANSWER

Official documentation: https://kubernetes.io/docs/concepts/storage/persistent-volumes/

apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-log
spec:
  capacity:
    storage: 100Mi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Recycle
  hostPath:
    path: /pv/log

kubectl create -f persistentvolume.yaml

Check: kubectl get persistentvolume


Q12 ######################## PV & VOLUME CLAIM ########################################

### Let us claim some of that storage for our application. Create a 'Persistent Volume Claim' with the given specification.

Volume Name: claim-log-1
Storage Request: 50Mi
Access modes: ReadWriteOnce

### ANSWER

Official documentation: https://kubernetes.io/docs/concepts/storage/persistent-volumes/

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: claim-log-1
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 50Mi

kubectl create -f persistentvolumeclaim.yaml

Check: kubectl get persistentvolumeclaim/claim-log-1


Q13 ####################### PV & VOLUME CLAIM ######################################

### Update the Access Mode on the claim to bind it to the PVC. Delete and recreate the claim-log-1.

### ANSWER

kubectl delete persistentvolumeclaim/claim-log-1

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: claim-log-1
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 50Mi


kubectl create -f persistentvolumeclaim.yaml

Check: kubectl get persistentvolumeclaim/claim-log-1


Q14 ####################### PV & VOLUME CLAIM #############################################

### Update the webapp pod to use the persistent volume claim as its storage. Replace hostPath configured earlier with the newly created PersistentVolumeClaim.

Name: webapp
Image Name: kodekloud/event-simulator
Volume: PersistentVolumeClaim=claim-log-1
Volume Mount: /log

### ANSWER

kubectl delete pod/webapp

Let us edit the pod file as given below:

apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
  containers:
  - name: event-simulator
    image: kodekloud/event-simulator
    env:
    - name: LOG_HANDLERS
      value: file
    volumeMounts:
    - mountPath: /log
      name: pv-log

  volumes:
  - name: pv-log
    persistentVolumeClaim:
      claimName: claim-log-1


kubectl create -f webapp-pod-hostpath.yaml



Q15 ####################### SECURITY #################################################

### Kubectl suddenly stops responding to your commands. Check it out! Someone recently modified the /etc/kubernetes/manifests/etcd.yaml file. You are asked to investigate and fix the issue. Once you fix the issue wait for sometime for kubectl to respond. Check the logs of the ETCD container.

### ANSWER

We checked the options in /etc/kubernetes/manifests/etcd.yaml file and found that the --cert= option contains etcd server certificate was set as server-certificate.crt instead of server.crt.

Modified the file the problem was over.


Q16 ######################### SECURITY #################################################

### The kube-api server stopped again! Check it out. Inspect the kube-api server logs and identify the root cause and fix the issue. Run docker ps -a command to identify the kube-api server container. Run docker logs container-id command to view the logs.

### ANSWER

docker ps -a

docker logs -t f482214fca84 | grep error


The below error is found:

2020-07-13T18:34:36.136748442Z W0713 18:34:36.136599       1 clientconn.go:1208] grpc: addrConn.createTransport failed to connect to {https://127.0.0.1:2379  <nil> 0 <nil>}. Err :connection error: desc = "transport: authentication handshake failed: x509: certificate signed by unknown authority". Reconnecting...

cat /etc/kubernetes/manifests/kube-apiserver.yaml

Found that the ca for the etcd was not set to the proper path:

    ---etcd-cafile=/etc/kubernetes/pki/ca.crt

We had to change as given below:

    ---etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt

The we created the pod as given below:

kubectl create -f /etc/kubernetes/manifests/kube-apiserver.yaml


Q17 ####################### SECURITY ###########################################################

### What is the location to find different services in a kubernetes cluster deployed without the kubeadm way.


### ANSWER

In non kubeadm kubernetes replicaset/installation the service files are located in /etc/systemd/system/ on the master node.


vagrant@laba-master-1:~$ ls -la /etc/systemd/system/
total 76
drwxr-xr-x 15 root root 4096 Jul  1 23:15 .
drwxr-xr-x  5 root root 4096 Jun 29 08:49 ..
drwxr-xr-x  2 root root 4096 Jun 18 15:51 cloud-final.service.wants
drwxr-xr-x  2 root root 4096 Jun 18 15:51 cloud-init.target.wants
lrwxrwxrwx  1 root root   44 Jun 18 15:49 dbus-org.freedesktop.resolve1.service -> /lib/systemd/system/systemd-resolved.service
drwxr-xr-x  2 root root 4096 Jun 18 15:51 default.target.wants
-rw-r--r--  1 root root  967 Jul  1 20:04 etcd.service
drwxr-xr-x  2 root root 4096 Jun 18 15:51 final.target.wants
drwxr-xr-x  2 root root 4096 Jun 18 15:49 getty.target.wants
drwxr-xr-x  2 root root 4096 Jun 18 15:51 graphical.target.wants
lrwxrwxrwx  1 root root   38 Jun 18 15:51 iscsi.service -> /lib/systemd/system/open-iscsi.service
-rw-r--r--  1 root root 1577 Jul  1 23:15 kube-apiserver.service
-rw-r--r--  1 root root  765 Jul  1 22:10 kube-controller-manager.service
-rw-r--r--  1 root root  342 Jul  1 22:11 kube-scheduler.service
drwxr-xr-x  2 root root 4096 Jul  1 22:11 multi-user.target.wants
drwxr-xr-x  2 root root 4096 Jun 18 15:51 network-online.target.wants
drwxr-xr-x  2 root root 4096 Jun 18 15:51 open-vm-tools.service.requires
drwxr-xr-x  2 root root 4096 Jun 18 15:51 paths.target.wants
drwxr-xr-x  2 root root 4096 Jun 18 15:51 sockets.target.wants
lrwxrwxrwx  1 root root   31 Jun 18 15:51 sshd.service -> /lib/systemd/system/ssh.service
drwxr-xr-x  2 root root 4096 Jun 18 15:51 sysinit.target.wants
lrwxrwxrwx  1 root root   35 Jun 18 15:49 syslog.service -> /lib/systemd/system/rsyslog.service
drwxr-xr-x  2 root root 4096 Jun 18 15:51 timers.target.wants
lrwxrwxrwx  1 root root   41 Jun 18 15:51 vmtoolsd.service -> /lib/systemd/system/open-vm-tools.service

We can look in to the .service files using cat or vi/vim.

We can also see in to the services using the below command:

ps -aux | grep kube-apiserver

Output:

root      1014  2.9 20.9 567108 320092 ?       Ssl  Jul07 258:42 /usr/local/bin/kube-apiserver --advertise-address=192.168.5.6 --allow-privileged=true --apiserver-count=3 --audit-log-maxage=30 --audit-log-maxbackup=3 --audit-log-maxsize=100 --audit-log-path=/var/log/audit.log --authorization-mode=Node,RBAC --bind-address=0.0.0.0 --client-ca-file=/var/lib/kubernetes/ca.crt --enable-admission-plugins=NodeRestriction,ServiceAccount --enable-swagger-ui=true --enable-bootstrap-token-auth=true --etcd-cafile=/var/lib/kubernetes/ca.crt --etcd-certfile=/var/lib/kubernetes/laba-etcd-server.crt --etcd-keyfile=/var/lib/kubernetes/laba-etcd-server.key --etcd-servers=https://192.168.5.6:2379,https://192.168.5.7:2379 --event-ttl=1h --encryption-provider-config=/var/lib/kubernetes/laba-encryption-config.yaml --kubelet-certificate-authority=/var/lib/kubernetes/ca.crt --kubelet-client-certificate=/var/lib/kubernetes/laba-kube-apiserver.crt --kubelet-client-key=/var/lib/kubernetes/laba-kube-apiserver.key --kubelet-https=true --runtime-config=api/all=true --service-account-key-file=/var/lib/kubernetes/laba-service-account.crt --service-cluster-ip-range=10.96.0.0/24 --service-node-port-range=30000-32767 --tls-cert-file=/var/lib/kubernetes/laba-kube-apiserver.crt --tls-private-key-file=/var/lib/kubernetes/laba-kube-apiserver.key --v=2
vagrant   7748  0.0  0.0  14856  1004 pts/0    S+   21:31   0:00 grep --color=auto kube-apiserver

Likewise:
ps -aux | grep kube-controller-manager

ps -aux | grep kube-scheduler

ps -aux | grep etcd

ps -aux | grep kubelet

On Worker nodes:

ps -aux | grep kubelet

ps -aux | grep kube-proxy


Q18 ####################### SECURITY ###########################################################

### What is the location to find different services in a kubernetes cluster deployed with the kubeadm way.


### ANSWER


Let us go to the below directory on the kubeadm master node:

cd /etc/kubernetes/manifests/

Now let us type ls -la:

total 24
drwxr-xr-x 2 root root 4096 Jul 11 22:15 .
drwxr-xr-x 4 root root 4096 Jul 11 22:14 ..
-rw------- 1 root root 1858 Jul 11 22:15 etcd.yaml
-rw------- 1 root root 3225 Jul 11 22:15 kube-apiserver.yaml
-rw------- 1 root root 3081 Jul 11 22:15 kube-controller-manager.yaml
-rw------- 1 root root 1120 Jul 11 22:15 kube-scheduler.yaml

Again let us look in to the cd /etc/kubernetes/pki/ on the kubeadm master node:

And then type ls -la:

total 68
drwxr-xr-x 3 root root 4096 Jul 11 22:14 .
drwxr-xr-x 4 root root 4096 Jul 11 22:14 ..
-rw-r--r-- 1 root root 1090 Jul 11 22:14 apiserver-etcd-client.crt
-rw------- 1 root root 1679 Jul 11 22:14 apiserver-etcd-client.key
-rw-r--r-- 1 root root 1099 Jul 11 22:14 apiserver-kubelet-client.crt
-rw------- 1 root root 1675 Jul 11 22:14 apiserver-kubelet-client.key
-rw-r--r-- 1 root root 1224 Jul 11 22:14 apiserver.crt
-rw------- 1 root root 1679 Jul 11 22:14 apiserver.key
-rw-r--r-- 1 root root 1025 Jul 11 22:14 ca.crt
-rw------- 1 root root 1679 Jul 11 22:14 ca.key
drwxr-xr-x 2 root root 4096 Jul 11 22:14 etcd
-rw-r--r-- 1 root root 1038 Jul 11 22:14 front-proxy-ca.crt
-rw------- 1 root root 1675 Jul 11 22:14 front-proxy-ca.key
-rw-r--r-- 1 root root 1058 Jul 11 22:14 front-proxy-client.crt
-rw------- 1 root root 1675 Jul 11 22:14 front-proxy-client.key
-rw------- 1 root root 1679 Jul 11 22:14 sa.key
-rw------- 1 root root  451 Jul 11 22:14 sa.pub


Q19a ############################ POD ##########################################################

### Create a new pod with the NGINX image

### ANSWER

kubectl run mynginx --image=nginx

Check: kubectl get pods or
       kubectl get all


Q19b ############################# POD #########################################################

### There is a POD named newpod. What is the image used to create this?

### ANSWER

kubectl get all

kubectl describe pod/newpods-5lwtm

Let us check in the container section:

Containers:
  busybox:
    Container ID:  docker://9961e916debc8684adb3434c6bcb0f974ed020d5807481f53bd1244f9471014f
    Image:         busybox
    Image ID:      docker-pullable://busybox@sha256:9ddee63a712cea977267342e8750ecbc60d3aab25f04ceacfa795e6fce341793
    Port:          <none>
    Host Port:     <none>
    Command:
      sleep

Q19c ########################## POD ##############################################################

### Delete the pod webapp.

### ANSWER

kubectl delete pod webapp

Q19d ########################### POD ##############################################################

### Create a new pod with the name 'redis' and with the image 'redis123'. Use a pod-definition YAML file. And yes the image name is wrong!

### ANSWER

Let us type the below command :

kubectl run redis --image=redis123 --dry-run=client -oyaml > redispod.yaml

Then run the below command:

kubectl create -f redispod.yaml

Check: kubectl get all

Q19e ########################## POD ###############################################################

### Now fix the image on the pod to 'redis'. Update the pod-definition file and use 'kubectl apply' command or use 'kubectl edit pod redis' command.

### ANSWER

We can change the redispod yaml file and the apply but let us try and use the kubectl edit pod command:

kubectl edit pod redis

Then after editing the file for the image in the container section let us check using the below command:

kubectl get all

Output:

NAME              READY   STATUS    RESTARTS   AGE
pod/redis         1/1     Running   0          67s


Q19f ######################### POD #################################################################

### Deploy a redis pod using the redis:alpine image with the labels set to tier=db. Either use imperative commands to create the pod with the labels. Or else use imperative commands to generate the pod definition file, then add the labels before creating the pod using the file.

### ANSWER

kubectl run redis --image=redis:alpine -l tier=db

Check: kubectl get all

Q19g ############################ POD ####################################################

### Create a new pod called custom-nginx using the nginx image and expose it on container port 8080.

### ANSWER

Let us check the command for creatinbg pod:

kubectl run --help

kubectl run custom-nginx --image=nginx --port=8080


Q20a ########################## REPLICASET ##########################################################

### Find the version of replicaset and use the yaml file 'replicaset-definition-1.yaml to create a rs after rectifying the apiversion.

### ANSWER

kubectl explain replicaset | grep VERSION

Output: VERSION:  apps/v1

vim replicaset-definition-1.yaml

apiVersion: v1
kind: ReplicaSet
metadata:
  name: replicaset-1
spec:
  replicas: 2
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: nginx
        image: nginx

Let us now change the apiversion and create the rs.

kubectl create -f replicaset-definition-1.yaml


Q20b ######################### REPLICASET #################################################

### Fix the issue in the replicaset-definition-2.yaml file and create a ReplicaSet using it. This file is located at /root/.

### ANSWER

Let us check the yaml file:

vim replicaset-definition-2.yaml

apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replicaset-2
spec:
  replicas: 2
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: nginx
    spec:
      containers:
      - name: nginx
        image: nginx

The labels under template is not correct and let us fix it and create the rs.

kubectl create -f replicaset-definition-2.yaml

Check: kubectl get all


Q20c ################################ REPLICASET ############################################

### Delete the two newly created ReplicaSets - replicaset-1 and replicaset-2

### ANSWER

kubectl get all

kubectl delete replicaset.apps/replicaset-1

kubectl delete replicaset.apps/replicaset-2

Check: kubectl get all


Q20d ################################ REPLICASET ############################################

### Fix the original replica set 'new-replica-set' to use the correct 'busybox' image. Either delete and re-create the ReplicaSet or Update the existing ReplicSet and then delete all PODs, so new ones with the correct image will be created.

### ANSWER

kubectl get all

kubectl edit replicaset.apps/new-replica-set

Let us change the template.spec.image under spec.

Now let us delete all the existing pods under the replicaset of our concern.

kubectl delete pod/new-replica-set-cs7dr pod/new-replica-set-jvfm7 pod/new-replica-set-pw5xh pod/new-replica-set-rv7c7

Check: kubectl get all

Output:

NAME                        READY   STATUS    RESTARTS   AGE
pod/new-replica-set-96mj8   1/1     Running   0          65s
pod/new-replica-set-97nzr   1/1     Running   0          65s
pod/new-replica-set-fb47g   1/1     Running   0          65s
pod/new-replica-set-qc7wk   1/1     Running   0          65s


Q20e ###################### REPLICASET #########################################################

### Scale the ReplicaSet to 5 PODs. Use 'kubectl scale' command or edit the replicaset using 'kubectl edit replicaset'.

### ANSWER

kubectl get all

kubectl edit replicaset.apps/new-replica-set

Let us change the replicas to 5 and save the file.

Check:

kubectl get all

Output:

NAME                        READY   STATUS    RESTARTS   AGE
pod/new-replica-set-96mj8   1/1     Running   0          4m9s
pod/new-replica-set-97nzr   1/1     Running   0          4m9s
pod/new-replica-set-fb47g   1/1     Running   0          4m9s
pod/new-replica-set-gkzk8   1/1     Running   0          8s
pod/new-replica-set-qc7wk   1/1     Running   0          4m9s

Q20f ############################## REPLICASET ##################################################

### Now scale the ReplicaSet down to 2 PODs. Use 'kubectl scale' command or edit the replicaset using 'kubectl edit replicaset'.

### ANSWER

kubectl get all

kubectl edit replicaset.apps/new-replica-set

Let us change the replicas to 5 and save the file

Let us wait a bit and check:

kubectl get all

NAME                        READY   STATUS    RESTARTS   AGE
pod/new-replica-set-96mj8   1/1     Running   0          9m9s
pod/new-replica-set-fb47g   1/1     Running   0          9m9s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   110m

NAME                              DESIRED   CURRENT   READY   AGE
replicaset.apps/new-replica-set   2         2         2       32m


Q20g ############################ REPLICASET ######################################################

### Create a replicaset named mohammd-rs with the image of nginx and with 3 replicast.

### ANSWER

Let us try to create in imperative way. But thesre is no direct command for creating replicaset using the command and hence let us try to produce a yaml file for replicaset with nginx image and then change the file to fit for the replicaset.

kubectl create replicaset mynginxreplicaset --image=nginx --dry-run=client -oyaml > mynginxreplicaset.yaml

Let us rename the mynginxreplicaset.yaml as mynginxreplicaset.yaml

mv mynginxreplicaset.yaml mynginxrelicaset.yaml

vim mynginxrelicaset.yaml

Output:

apiVersion: apps/v1
kind: ReplicaSet
metadata:
  creationTimestamp: null
  labels:
    app: mynginxreplicaset
  name: mynginxreplicaset
spec:
  replicas: 3
  selector:
    matchLabels:
      app: mynginxreplicaset
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: mynginxreplicaset
    spec:
      containers:
      - image: nginx
        name: nginx
        resources: {}
status: {}

Let us check the apiverion for replicaset using the below command:

vagrant@kubemaster:~$ kubectl explain replicaset | grep VERSION
VERSION:  apps/v1

Now let us change the kind, replicas and other fields in the above yaml file as given below. Let us remove all the unnecessary fields especially like strategy: {}, creationTimestamp: null, resources: {}, status: {}.

apiVersion: apps/v1
kind: ReplicaSet
metadata:
  labels:
    app: mynginxrs
  name: mynginxrs
spec:
  replicas: 3
  selector:
    matchLabels:
      app: mynginxrs
  template:
    metadata:
      labels:
        app: mynginxrs
    spec:
      containers:
      - image: nginx
        name: nginx

kubectl create -f mynginxrelicaset.yaml 

check: kubectl get all

Output:

vagrant@kubemaster:~$ kubectl get all
NAME                  READY   STATUS    RESTARTS   AGE
pod/mynginxrs-hbgkq   1/1     Running   0          4m44s
pod/mynginxrs-llwv2   1/1     Running   0          4m44s
pod/mynginxrs-p7fn9   1/1     Running   0          4m44s
pod/redis             1/1     Running   0          3h9m
pod/static-pod1       1/1     Running   1          3d22h
pod/static-pod2       1/1     Running   1          3d22h
pod/static-pod3       1/1     Running   1          3d22h
pod/test-nginx        1/1     Running   1          4d5h

NAME                      TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)        AGE
service/kubernetes        ClusterIP      10.96.0.1        <none>          443/TCP        4d5h
service/my-service-lb     LoadBalancer   10.105.12.191    192.168.56.51   80:30092/TCP   4d5h
service/my-service-lb-1   LoadBalancer   10.109.121.141   192.168.56.52   80:30146/TCP   4d4h

NAME                        DESIRED   CURRENT   READY   AGE
replicaset.apps/mynginxrs   3         3         3       4m44s


Q21a ############################# DEPLOYMENT #################################################

### Create a new Deployment using the 'deployment-definition-1.yaml' file located at /root/. There is an issue with the file, so try to fix it.

### ANSWER

vim deployment-definition-1.yaml

Output:

apiVersion: apps/v1
kind: deployment
metadata:
  name: deployment-1
spec:
  replicas: 2
  selector:
    matchLabels:
      name: busybox-pod
  template:
    metadata:
      labels:
        name: busybox-pod
    spec:
      containers:
      - name: busybox-container
        image: busybox888
        command:
        - sh
        - "-c"
        - echo Hello Kubernetes! && sleep 3600

Let us check the apiversion for deployment using the below command:

kubectl explain deployment | grep VERSION

Version is found to be alright.

Let us change the kind from deployment to Deployment, let us change the image from busybox888 to busybox and then run the below command after saving the deployment-definition-1.yaml file:

kubectl create -f deployment-definition-1.yaml

kubectl get all

Output:

NAME                                       READY   STATUS             RESTARTS   AGE
pod/deployment-1-66dc945c47-phmgq          1/1     Running            0          36s
pod/deployment-1-66dc945c47-qw6dh          1/1     Running            0          36s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   27m

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/deployment-1          2/2     2            2           36s

NAME                                             DESIRED   CURRENT   READY   AGE
replicaset.apps/deployment-1-66dc945c47          2         2         2       36s


Q21b ############################## DEPLOYMENT ##################################################

### Create a new Deployment with the below attributes using your own deployment definition file. Name: httpd-frontend; Replicas: 3; Image: httpd:2.4-alpine.

### ANSWER

I will use the below command to generate the yaml definition file:

kubectl create deployment deployment-httpd-frontend --image=httpd:2.4-alpine --dry-run=client -oyaml > deployment-httpd-frontend.yaml

Now, let us edit the deployment-httpd-frontend.yaml file to adjust the replicas to 3 and exclude the options that are not really necessary.


apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: deployment-httpd-frontend
  name: httpd-frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: deployment-httpd-frontend
  template:
    metadata:
      labels:
        app: deployment-httpd-frontend
    spec:
      containers:
      - image: httpd:2.4-alpine
        name: httpd

Now, let us run the commnad below:

kubectl create -f deployment-httpd-frontend.yaml

Check: kubectl get all

NAME                                             READY   STATUS             RESTARTS   AGE
pod/httpd-frontend-7599cd8d96-6bpnn   1/1     Running            0          13s
pod/httpd-frontend-7599cd8d96-87975   1/1     Running            0          13s
pod/httpd-frontend-7599cd8d96-9jstj   1/1     Running            0          13spod/

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   37m

NAME                                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/httpd-frontend   3/3     3            3           13s

NAME                                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/httpd-frontend-7599cd8d96   3         3         3       13s

Q21c ############################## DEPLOYMENT ################################################

### Create a deployment named webapp using the image kodekloud/webapp-color with 3 replicas. Try to use imperative commands only. Do not create definition files.

### ANSWER

kubectl create deployment webapp --image=kodekloud/webapp-color

Let us look into the command using the below command:

kubectl scale --help

Now, let us scale the deployment webapp using the below command:

kubectl scale --replicas=3 deployment/webapp

Check: kubectl get all

Q21d ############################## DEPLOYMENT ################################################

### Create a new deployment called redis-deploy in the dev-ns namespace with the redis image. It should have 2 replicas. Use imperative commands.

### ANSWER

kubectl create --help

kubectl create deployment redis-deploy --image=redis --namespace=dev-ns

kubectl get all -n dev-ns

kubectl scale deployment.apps/redis-deploy --replicas=2 --namespace=dev-ns

Check: kubectl get all -n dev-ns


Q22a ############################## NAMESPACE ##################################################

### Create a pod in the name space finance with name redis and image redis.

### ANSWER

kubectl run redis --image=redis --namespace=finance

When I tried to dry-run using the below command, the namespace option did not automatically appeared in the yaml file but the namespace should be included under metadata like name and labels.

kubectl run redis --image=redis --namespace=finance --dry-run=client -oyaml > redis.yaml


Q22b ############################### NAMESPACE ##############################################

### Commands to see resources in all namespaces?

### ANSWER

kubectl get all --all-namespaces

kubectl get pods --all-namespaces

etc



Q23a ############################### SERVICES ##############################################

### Create a service redis-service to expose the redis application within the cluster on port 6379. Use imperative commands.

### ANSWER

We can have the command for exposing typing the below command:

kubectl expose --help

kubectl expose pod redis --port=6379 --name redis-service

Check: kubectl describe service/redis-service



Q23c ########################### SERVICE ########################################################

### Create a pod called httpd using the image httpd:alpine in the default namespace. Next, create a service of type ClusterIP by the same name (httpd). The target port for the service should be 80. Try to do this with as few steps as possible.

### ANSWER

kubectl run httpd --image=httpd:alpine --port=80

kubectl expose pod httpd --port=80 --name=httpd

But the above two commands can be shrinked down to the below one:

kubectl run httpd --image=httpd:alpine --port=80 --expose


Q24a ############################ SCHEDULING/STATIC POD ####################################################

### Create a static pod named static-pod with gninx image on kubenode01.

### ANSWER

kubectl run static-pod --image=nginx --dry-run=client -oyaml > static-pod.yaml

vim static-pod.yaml

apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: static-pod
  name: static-pod
spec:
  containers:
  - image: nginx
    name: static-pod
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}

Let us edit the yaml file as given below:

apiVersion: v1
kind: Pod
metadata:
  labels:
    run: static-pod
  name: static-pod
spec:
  containers:
  - image: nginx
    name: static-pod
  nodeName: kubenode02

Reference: https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/

kubectl create -f static-pod.yaml

Check: kubectl get pods -o wide

Output:

NAME              READY   STATUS    RESTARTS   AGE     IP          NODE         NOMINATED NODE   READINESS
static-pod        1/1     Running   0          7s      10.44.0.3   kubenode02   <none>           <none>


Q24b ############################ SCHEDULING/STATIC POD ####################################################

### Create a static pod named static-busybox that uses the busybox image and the command sleep 1000.

### ANSWER

kubectl run static-busybox --image=busybox --dry-run=client -o yaml --command -- sleep 1000 > /etc/kubernetes/manifests/static-busybox.yaml

Check: kubectl get all

Output:

NAME                        READY   STATUS    RESTARTS   AGE
pod/static-busybox-master   1/1     Running   0          49s

Q24c ############################ SCHEDULING/STATIC POD ####################################################

### We just created a new static pod named static-greenbox. Find it and delete it.

### ANSWER

We try to find the pod definition file in /etc/kubernetes/manifest/ folder on the master node but could not find.

We tried to delte the pod using the below command but it did not work, the pod got created again.

kubectl delete pod static-greenbox-node01

Then we tried to find the location of the pod using the below command:

kubectl get all -o wide

The pod was created on the node01 but we could not ssh in to node01 using the hostname.

We tried to find out the ip of node01 using below command:

kubectl get nodes -o wide

Then we ssh in to node01 and check in the /var/lib/kubelet/config.yaml file for the path for the static pod.

The path for the static pos was found to be /etc/just-to-mess-with-you/ and inside this folder we could find the pod definition file and we removed it to delete the pod.


Q25a ############################## SELECTOR ###################################################

### Create pod in the default namespace with the labels env=prod, bu=finance and tier=frontend. Name the pods as selector-pod and use image of nginx.

### ANSWER

kubectl run selector-pod --image=nginx --labels env=prod,bu=finance,tier=frontend

Check:

kubectl get pods --selector env=prod,bu=finance,tier=frontend


Q26a ############################## TAINT ########################################################

### Create a taint on node01 with key of 'spray', value of 'mortein' and effect of 'NoSchedule'

### ANSWER

Let us check the help by below command:

kubectl taint --help

Then let us type the below command:

kubectl taint nodes node01 spray=mortein:NoSchedule

Check: kubectl describe node node01

Q26b ############################## TAINT ########################################################

### Create another pod named 'bee' with the NGINX image, which has a toleration set to the taint Mortein.

### ANSWER

Let us chek the documentation for tolerations:

https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/

The example found and edited as given below:

apiVersion: v1
kind: Pod
metadata:
  name: bee
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
  tolerations:
  - key: "spray"
    operator: "Equal"
    value: "mortein"
    effect: "NoSchedule"

kubectl get all

Q26c ############################## TAINT ########################################################

### Remove taints from kubemaster so that pods can be scheduled.

### ANSWER

Let us see the taints for kubemaster using the below command:

kubectl describe node kubemaster

The below taint is found:

Taints:             node-role.kubernetes.io/master:NoSchedule

Now let us see the help for taints:

kubectl taint --help

From the hlep, we find the below example:

 # Remove from node 'foo' the taint with key 'dedicated' and effect 'NoSchedule' if one exists.
  kubectl taint nodes foo dedicated:NoSchedule-

Now let us run the below command for removing the taint from kubemaster:

kubectl taint nodes kubemaster node-role.kubernetes.io/master:NoSchedule-

Check: kubectl describe node kubemaster

Output:

Taints:             <none>
Unschedulable:      false


Q27a ############################## LABELS ########################################################

### Apply a label color=blue to node kubenode01.

### ANSWER

kubectl label node kubenode01 color=blue

kubectl describe node kubenode01

Q28a ############################## NODE AFFINITY ##################################################

### Set Node Affinity to the ruzlan-deploy deployment to place the PODs on kubenode01 only with 3 replicas. It is worth mentioning that for this to be accomplished we have to add some label to kubenode01 first. We have a label color=blue already set up. Use image of nginx.

### ANSWER

Now, let us create a yaml file for deployment:

kubectl create deployment ruzlan-deploy --image=nginx --dry-run=client -oyaml > ruzlan-deploy.yaml

Now, let us change the replicas to 3 and also accommodate change for the node affinity. Let us consult the below official documentation:

https://kubernetes.io/docs/tasks/configure-pod-container/assign-pods-nodes-using-node-affinity/

vim ruzlan-deploy.yaml:

apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: ruzlan-deploy
  name: ruzlan-deploy
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ruzlan-deploy
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: ruzlan-deploy
    spec:
      containers:
      - image: nginx
        name: nginx
        resources: {}
status: {}

Let us change as given below. We have to add the part for node affinity in the section spec under template:

apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: ruzlan-deploy
  name: ruzlan-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: ruzlan-deploy
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: ruzlan-deploy
    spec:
      containers:
      - image: nginx
        name: nginx
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: color
                operator: In
                values:
                - blue  


kubectl create -f ruzlan-deploy.yaml



Q28b ############################## NODE AFFINITY ##################################################

### Create a new deployment named 'red' with the NGINX image and 3 replicas, and ensure it gets placed on the master node only. Use the label - node-role.kubernetes.io/master - set on the master node.

### ANSWER

kubectl create deployment red --image=nginx --dry-run=client -oyaml > red.yaml

Let us edit the red.yaml file appropriately:

apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: red
  name: red
spec:
  replicas: 3
  selector:
    matchLabels:
      app: red
  template:
    metadata:
      labels:
        app: red
    spec:
      containers:
      - image: nginx
        name: nginx
      affinity:
          nodeAffinity:
            requiredDuringSchedulingIgnoredDuringExecution:
              nodeSelectorTerms:
              - matchExpressions:
                - key: node-role.kubernetes.io/master
                  operator: Exists
~                                 
Then----

kubectl create -f red.yaml

Check:

kubectl get all -o wide

Q29a ########################## DAEMONSET ##########################################################

### Create a daemonset named ruzdaemon with NGINX image that will run on all the nodes including the master node.

### ANSWER

Let us go to the officela documentation at the below link and have the example manifest file:

https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/

apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: ruzdaemon
  labels:
    k8s-app: fluentd-logging
spec:
  selector:
    matchLabels:
      name: ruzdaemon
  template:
    metadata:
      labels:
        name: ruzdaemon
    spec:
      tolerations:
      # this toleration is to have the daemonset runnable on master nodes
      # remove it if your masters can't run pods
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      containers:
      - name: ruzdaemon
        image: mginx
        
kubectl create -f my-daemonset-1.yaml

Check: kubectl get all -o wide

N.B. This manifest file for daemonset covers almost all the features that might be required for pods, deployments, volumes and resoureces and hence we could use this to edit and prepare required manifest files.

Q29a ########################## DAEMONSET ##########################################################

### Deploy a pod named mynginx-pod with nginx image in the default namespace ane using the scheduler named my-scheduler.

### ANSWER

Let us first check the schedulers in the kube-system namespace in the kube adm cluster.

kubectl get pods - kube-system

Output:

NAME                                      READY   STATUS             RESTARTS   AGE
coredns-66bff467f8-77dkz                  1/1     Running            0          43m
coredns-66bff467f8-92lzm                  1/1     Running            0          43m
etcd-master                               1/1     Running            0          43m
kube-apiserver-master                     1/1     Running            0          43m
kube-controller-manager-master            1/1     Running            0          43m
kube-flannel-ds-amd64-5sppb               1/1     Running            1          43m
kube-flannel-ds-amd64-8srvs               1/1     Running            0          43m
kube-keepalived-vip-j427v                 1/1     Running            0          42m
kube-proxy-8h2xx                          1/1     Running            0          43m
kube-proxy-gjz97                          1/1     Running            0          43m
kube-scheduler-master                     1/1     Running            0          43m
my-scheduler-master                       1/1     Running            0          18s

Now let us look in to the official documentation in the below link to find how to accommodate the custom scheduler in the pod definition file.


https://kubernetes.io/docs/tasks/extend-kubernetes/configure-multiple-schedulers/

apiVersion: v1
kind: Pod
metadata:
  name: annotation-default-scheduler
  labels:
    name: multischeduler-example
spec:
  schedulerName: default-scheduler
  containers:
  - name: pod-with-default-annotation-container
    image: k8s.gcr.io/pause:2.0

Let us modify the above pod definition to match our requirement and then save as mynginx-pod.yaml.


apiVersion: v1
kind: Pod
metadata:
  name: mynginx-pod
  labels:
    name: mynginx-pod-label
spec:
  schedulerName: my-scheduler
  containers:
  - name: mynginx-pod-container
    image: nginx

kubectl create -f mynginx-pod.yaml

Check:

kubectl get pods


Q30a ########################## METRIC SERVER ##########################################################

### Deploy a metric server on kubemaster in the kubeadm kubernetes cluster.

### ANSWER

Let us download by cloning from the below github location.

git clone https://github.com/kodekloudhub/kubernetes-metrics-server.git

cd kubernetes-metrics-server

kubectl create -f .

Output of the command:

clusterrole.rbac.authorization.k8s.io/system:aggregated-metrics-reader created
clusterrolebinding.rbac.authorization.k8s.io/metrics-server:system:auth-delegator created
rolebinding.rbac.authorization.k8s.io/metrics-server-auth-reader created
apiservice.apiregistration.k8s.io/v1beta1.metrics.k8s.io created
serviceaccount/metrics-server created
deployment.apps/metrics-server created
service/metrics-server created
clusterrole.rbac.authorization.k8s.io/system:metrics-server created
clusterrolebinding.rbac.authorization.k8s.io/system:metrics-server created

Check:

kubectl top node
kubectl top pod

Q31a ########################## LOG MONITOR ##########################################################

### We have an application named webapp-1. A user - 'USER5' - has expressed concerns accessing the application. Identify the cause of the issue. Inspect the logs of the POD.

### ANSWER

Let us look in to the pod to dind out how many containers are there in the pod.

kubectl get pod

NAME       READY   STATUS    RESTARTS   AGE
webapp-1   1/1     Running   0          2m25s

From the READY column we find that the app/pod has only one container.

To find the content in log for the USER5 let us run the below command:

kubectl logs pod/webapp-1 | grep USER5

Output:

[2020-07-18 02:56:43,319] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.

Q31b ########################## LOG MONITOR ##########################################################

### We have an application named webapp-2. A user is reporting issues while trying to purchase an item. Identify the user and the cause of the issue. Inspect the logs of the webapp in the POD.

### ANSWER

Let us look in to the pod to dind out how many containers are there in the pod.

kubectl get pod

NAME       READY   STATUS    RESTARTS   AGE
webapp-2   2/2     Running   0          115s

From the above output and READY column we can clearly see that the app webapp-2 has two containers in it.

So, let us inspect into the app using the below command:

kubectl describe pod webapp-2

We can have the below output in the container section:

Containers:
  simple-webapp:
    Container ID:   docker://76ec51086f950b5da5ddf2afc0dd63d6c4ff9510f67af67ab763580eb6eb5840
    Image:          kodekloud/event-simulator
    Image ID:       docker-pullable://kodekloud/event-simulator@sha256:1e3e9c72136bbc76c96dd98f29c04f298c3ae241c7d44e2bf70bcc209b030bf9
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Sat, 18 Jul 2020 02:57:28 +0000
    Ready:          True
    Restart Count:  0
    Environment:
      OVERRIDE_USER:  USER30
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-5gsrn (ro)
  db:
    Container ID:  docker://d098c4486037a9501cfe082f7e947fbca4e8c890632e260dfc8d71eccb503a71
    Image:         busybox
    Image ID:      docker-pullable://busybox@sha256:9ddee63a712cea977267342e8750ecbc60d3aab25f04ceacfa795e6fce341793
    Port:          <none>
    Host Port:     <none>
    Command:
      sleep
      5000
    State:          Running
      Started:      Sat, 18 Jul 2020 02:57:30 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-5gsrn (ro)

So, we will have to check logs for the container simple-webapp to find out of the root cause of the problem stated in the question.

kubectl logs webapp-2 -c simple-webapp | grep USER1 OR kubectl logs pod/webapp-2 -c simple-webapp | grep USER1

kubectl logs webapp-2 -c simple-webapp | grep USER1 OR kubectl logs pod/webapp-2 -c simple-webapp | grep USER2

kubectl logs webapp-2 -c simple-webapp | grep USER1 OR kubectl logs pod/webapp-2 -c simple-webapp | grep USER30

kubectl logs webapp-2 -c simple-webapp | grep USER1 OR kubectl logs pod/webapp-2 -c simple-webapp | grep USER4

The below output gives the correct reason for the issue:

master $ kubectl logs pod/webapp-2 -c simple-webapp | grep USER30
[2020-07-18 03:12:52,987] WARNING in event-simulator: USER30 Order failed as the item is OUT OF STOCK.
[2020-07-18 03:13:01,024] WARNING in event-simulator: USER30 Order failed as the item is OUT OF STOCK.
[2020-07-18 03:13:09,055] WARNING in event-simulator: USER30 Order failed as the item is OUT OF STOCK

Q32a ########################## LIFECYCLE MANAGEMENT ####################################################

### 

### ANSWER


Q33a ######################### NEW USER ACCESS IN KUBERNETES ###########################################

### You have to provide a user named developer using a machine developer_machine on your same network the access to the kubernetes cluster in the namespace named developer and with full access on that particular namespace. The developer would like to start just typing kubectl commands and no background works can be allocated to him. Assume that as the administrator you have access to the developer machine and developer account.

### ANSWER

Since the developer is not going to do any of the background works, let us login to the machine that developer uses (name developer_machine and with IP address 10.9.108.252). My management machine has IP address 10.9.108.253. It is worth mentioning that both management machine and developer machine is connected to the kubernetes on-prem cluster that has the network address 192.168.5.0/24 and they have ip addresses of 192.168.5.100 and 192.168.5.253 respectively.

Let us create the private key and CSR for the developer account on the management machine:

Reference: https://kubernetes.io/docs/concepts/cluster-administration/certificates/

openssl genrsa -out developer.key 2048

openssl req -new -key developer.key -out developer.csr -subj "/CN=developer/O=dev"

Let us create a new directory /home/vagrant/developer to handle all aspects regarding the user developer. We are using the account vagrant for handling all kubernetes related activities.

sudo mkdir /home/vagrant/developer

cd developer/

sudo mv /home/vagrant/developer.key .

sudo mv /home/vagrant/developer.csr .

Let us convert developer.csr to base64 for uploading in the kubernetes cluster.

sudo cat developer.csr | base64

Now, we have to generate a kubernetes csr object:

Reference: https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/

apiVersion: certificates.k8s.io/v1beta1
kind: CertificateSigningRequest
metadata:
  name: john
spec:
  groups:
  - system:authenticated
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1ZqQ0NBVDRDQVFBd0VURVBNQTBHQTFVRUF3d0dZVzVuWld4aE1JSUJJakFOQmdrcWhraUc5dzBCQVFFRgpBQU9DQVE4QU1JSUJDZ0tDQVFFQTByczhJTHRHdTYxakx2dHhWTTJSVlRWMDNHWlJTWWw0dWluVWo4RElaWjBOCnR2MUZtRVFSd3VoaUZsOFEzcWl0Qm0wMUFSMkNJVXBGd2ZzSjZ4MXF3ckJzVkhZbGlBNVhwRVpZM3ExcGswSDQKM3Z3aGJlK1o2MVNrVHF5SVBYUUwrTWM5T1Nsbm0xb0R2N0NtSkZNMUlMRVI3QTVGZnZKOEdFRjJ6dHBoaUlFMwpub1dtdHNZb3JuT2wzc2lHQ2ZGZzR4Zmd4eW8ybmlneFNVekl1bXNnVm9PM2ttT0x1RVF6cXpkakJ3TFJXbWlECklmMXBMWnoyalVnald4UkhCM1gyWnVVV1d1T09PZnpXM01LaE8ybHEvZi9DdS8wYk83c0x0MCt3U2ZMSU91TFcKcW90blZtRmxMMytqTy82WDNDKzBERHk5aUtwbXJjVDBnWGZLemE1dHJRSURBUUFCb0FBd0RRWUpLb1pJaHZjTgpBUUVMQlFBRGdnRUJBR05WdmVIOGR4ZzNvK21VeVRkbmFjVmQ1N24zSkExdnZEU1JWREkyQTZ1eXN3ZFp1L1BVCkkwZXpZWFV0RVNnSk1IRmQycVVNMjNuNVJsSXJ3R0xuUXFISUh5VStWWHhsdnZsRnpNOVpEWllSTmU3QlJvYXgKQVlEdUI5STZXT3FYbkFvczFqRmxNUG5NbFpqdU5kSGxpT1BjTU1oNndLaTZzZFhpVStHYTJ2RUVLY01jSVUyRgpvU2djUWdMYTk0aEpacGk3ZnNMdm1OQUxoT045UHdNMGM1dVJVejV4T0dGMUtCbWRSeEgvbUNOS2JKYjFRQm1HCkkwYitEUEdaTktXTU0xMzhIQXdoV0tkNjVoVHdYOWl4V3ZHMkh4TG1WQzg0L1BHT0tWQW9FNkpsYWFHdTlQVmkKdjlOSjVaZlZrcXdCd0hKbzZXdk9xVlA3SVFjZmg3d0drWm89Ci0tLS0tRU5EIENFUlRJRklDQVRFIFJFUVVFU1QtLS0tLQo=
  usages:
  - client auth


Let us edit the example found in the reference as given below:


apiVersion: certificates.k8s.io/v1beta1
kind: CertificateSigningRequest
metadata:
  name: developer-csr
spec:
  groups:
  - system:authenticated
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1p6Q0NBVThDQVFBd0lqRVNNQkFHQTFVRUF3d0paR1YyWld4dmNHVnlNUXd3Q2dZRFZRUUtEQU5rWlhZdwpnZ0VpTUEwR0NTcUdTSWIzRFFFQkFRVUFBNElCRHdBd2dnRUtBb0lCQVFEQ2VIelN0MUsxS0F1QXh6UFhwMzJKClladkNTckdQQ0w5N2N5VkZrUEJ5NWhXZFdqRXI4bFRXemhSbEdKSUNJajdzM3hKd3dGNnBwUVhCZmZOQTcvWDQKUGxTWW9FVUxJV2ZtbUZXMkxTZ1crRFZVVkI1NExHYlBWMEdwK1FjcVQ3UlhCQ3ZXTlBhSTJJY1hUWXV2YnpaOApHbFFvS0poeUd5NlR1MzV6R1hTU0pwQXFaQWZ1WUx4SmpDK3NRNUdseitjeWZZYTNRWU9WejBGdGI2aElKb0dWCmYrbzRxRmhiNnVwVjdyMkZVeHJjVGJxank3VFBYcmpKb2lGMFg3djU4Nm8vVEhHNmZBTTJyT1VRdHZNUjV3OEkKeTRoTCswYlIrRkZjN2ltdnVRYm1JYTg0UVh5ZmxYMGZjSkdCbnVZeGhlcjZyWTM2cGpid09tTVFVcy8ydktVegpBZ01CQUFHZ0FEQU5CZ2txaGtpRzl3MEJBUXNGQUFPQ0FRRUFiYk1JRXBUWDRLVlVKMk54RFI5b1FPaC9YN0s1CnNyWVdaZ1J3aXN3TmIyS2hkWllQYzJ3MElvcWlVb0dURUlmWmNnbDgvWm5Lbm5OT2E5T0pFdlRCMWFWSHZKUmcKZU5DUmFZMkNaOEFnSGNBUnFzeEdpR3JMWVhPT3JLeTJHNW1CbjdqbFljN0c2bFNpSCtMa1c0QnlrUURoWWZlOApwa0x4MDFLbW9jS0VWR08wOUJxQzdQT0hsbEV6Yi9vZ3ZRWlUyYk1MWGlIV2xnaEhEZ2xYbklX
  TWNzbnpWNGlzCnprM1Z1TjA4cm01WFQ1MmEvdkVoWDY2SUlNbUFTRmZNdTZ3N3loRWpWaEw3QThFNVh4NjRaRWRBL2dCTEQvYTcKd1B3bkRqRmFVQ1VVMDF4SVpYUnY2YXhCZGQ2TDYwaExUbVpkdnZpWmFmNm5sUzN2bmhnUXZPRTlBUT09Ci0tLS0tRU5EIENFUlRJRklDQVRFIFJFUVVFU1QtLS0tLQo=
  
  usages:
  - client auth


Let us create developer-csr-object.yaml using the content of the above csr object:

sudo vim developer-csr-object.yaml

kubectl create -f developer-csr-object.yaml

At this point we faced some issues with the alignment and number of lines for the base64 converted csr. 

error: error parsing developer-csr-object.yaml: error converting YAML to JSON: yaml: line 10: could not find expected ':'

After rearrangements, we got the below output:

certificatesigningrequest.certificates.k8s.io/developer-csr created

Check: kubectl get csr

Output:

NAME            AGE   SIGNERNAME                     REQUESTOR   CONDITION
developer-csr   54m   kubernetes.io/legacy-unknown   admin       Pending

kubectl certificate approve developer-csr

Output:

certificatesigningrequest.certificates.k8s.io/developer-csr approved

kubectl get csr

NAME            AGE   SIGNERNAME                     REQUESTOR   CONDITION
developer-csr   59m   kubernetes.io/legacy-unknown   admin       Approved,Issued

kubectl get csr developer-csr -o jsonpath='{.status.certificate}'

Let us put the output of the above command in the file developer-csr-encoded.

sudo vim developer-csr-encoded

Now let us scp the developer-csr-encoded and developer.key to the developer account fo the developer_machine:

scp developer-csr-encoded developer@10.9.108.252:/home/developer/

 scp developer.key  developer@10.9.108.252:/home/developer/

 Now, let us login to the developer_machine with developer account and create developer.crt using the below command:

 cat developer-csr-encoded | base64 -d > developer.crt

 ls -la

 Output: 
 
 drwx------. 16 developer developer 4096 Jul 20 01:47 .
drwxr-xr-x.  3 root      root        23 Jul 19 22:44 ..
-rw-------.  1 developer developer   95 Jul 19 23:52 .bash_history
-rw-r--r--.  1 developer developer   18 Apr 10  2018 .bash_logout
-rw-r--r--.  1 developer developer  193 Apr 10  2018 .bash_profile
-rw-r--r--.  1 developer developer  231 Apr 10  2018 .bashrc
drwxrwxr-x. 14 developer developer 4096 Jul 19 23:07 .cache
drwxrwxr-x. 14 developer developer  261 Jul 19 23:03 .config
drwx------.  3 developer developer   25 Jul 19 23:01 .dbus
drwxr-xr-x.  2 developer developer    6 Jul 19 23:02 Desktop
-rw-rw-r--.  1 developer developer 1070 Jul 20 01:45 developer.crt
-rw-r--r--.  1 developer developer 1429 Jul 20 01:44 developer-csr-encoded
-rw-rw-r--.  1 developer developer 1675 Jul 20 01:47 developer.key
drwxr-xr-x.  2 developer developer    6 Jul 19 23:02 Documents
drwxr-xr-x.  2 developer developer    6 Jul 19 23:02 Downloads
-rw-------.  1 developer developer   16 Jul 19 23:02 .esd_auth
-rw-------.  1 developer developer  310 Jul 19 23:02 .ICEauthority
drwx------.  3 developer developer   19 Jul 19 23:02 .local
drwxr-xr-x.  4 developer developer   39 Jul 19 22:24 .mozilla
drwxr-xr-x.  2 developer developer    6 Jul 19 23:02 Music
drwxr-xr-x.  2 developer developer    6 Jul 19 23:02 Pictures
drwxr-xr-x.  2 developer developer    6 Jul 19 23:02 Public
drwx------.  2 developer developer   29 Jul 19 22:59 .ssh
drwxr-xr-x.  2 developer developer    6 Jul 19 23:02 Templates
drwxr-xr-x.  2 developer developer    6 Jul 19 23:02 Videos

Let us download kubeclt on the developer_machine:

Reference: https://kubernetes.io/docs/tasks/tools/install-kubectl/

curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl

chmod +x ./kubectl

mv ./kubectl /usr/local/bin/kubectl

sudo mkdir .kube/

Now, let us edit the kubeconfig file from the management machine and scp to the developer_machine for developer user.

sudo cp .kube/config developer/


apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUNyRENDQVpRQ0NRRGw1aXRIV2cwdmRqQU5CZ2txaGtpRzl3MEJBUXNGQURBWU1SWXdGQVlEVlFRRERBMUwKVlVKRlVrNUZWRVZUTFVOQk1CNFhEVEl3TURjd01UQTJORE13TUZvWERUSXpNRE15T0RBMk5ETXdNRm93R0RFVwpNQlFHQTFVRUF3d05TMVZDUlZKT1JWUkZVeTFEUVRDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDCkFRb0NnZ0VCQVBOMkpueStzYVpuekRPYnhPWkh6SUx6cUIvSmlXNTNsVDZsVEFRbjJVd25YQzF3UzR5elFZaFgKNW80WHA5WE91OXdZR3Fsc1NwSzdLaVZTNXg5cEpzeUU0TXg0Zm1IODNFY1VVd0hYYjE4Y3h1SDZKd29obW5zVQoyeFd1eUdoUGJOTDhTVjhlQUIyQmNQZlpJMytSOVJCQ09RdEQxckduYWtXTHBibDBuTGdFa0ZwMmh4QVE0M3QyCksrRmUyRTVoWlZ0QkJQcy9lQ1hoQVhnUEVBRmgyTEhxYTNWMmRzdkZyMElQbjV5RmRyaEdXTitTaVJqZHJkYXEKWWJsOWdkcW05R29FY1ZwU1diM2tValZsQ0s3ZWtsTW8yTjlEeDF4SnN4UzdRTkZ1eEIvQW5Ic3VmeGg1RnVONwpDTlZKQ0J0NjZDUlJneVhCd3VFenIzOVQ0cW5tZGpVQ0F3RUFBVEFOQmdrcWhraUc5dzBCQVFzRkFBT0NBUUVBCjVJUlAvWnNGWVBDK0ZJN0dmL3A4TXZlUmFVY1h0eWpoZllFR2I2akdVL3JlM21jc2EwUnIwUHZSZlpUWWtHc1AKbzkxbmozVk1jUnRJZ21saEk3TFc2RGdveFl2MzZKRXRGaXllOXdlQVFNdEN2N3VNdjM2UThhSmlpY3Z1S0k2Ygo4Um1GUXNNdkYweW5pWHVJUks4S0REUFh5QkJ6MWk2YVdUV2RSRUR1elFKVjFQcnUyMUtic0JZQitrMUU2Nm8wCkI4dE5UZk42QzVQWWZuYjVRL2JEejRNaVZkL3d4WitOTnE5eTN4c3c1SytzS0JvUERmT1FWVkd6Zm9ScHJUWXkKbFZQL3JSY3AzcFh3V3k3YlRDOCs2ak5YUnBGcXlOVXlGOVhlMlJRQm9reUlhQkh4Z0hCKzROSXBTQXhhM0ZUUgp3M2hoMDJlbXJxVVFLT2VUZHBtZDlBPT0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
    server: https://192.168.5.15:6443
  name: hardway-cluster-a
contexts:
- context:
    cluster: hardway-cluster-a
    user: developer
  name: hardway-cluster-a
current-context: hardway-cluster-a
kind: Config
preferences: {}
users:
- name: developer
  user:
    client-certificate: /home/developer/developer.crt
    client-key: /home/developer/developer.key

Now let us copy the body of the config file and paste in the config file in /home/developer/.kube/ folder of the developer machine:

Then

kubectl get nodes

Output:

Error from server (Forbidden): nodes is forbidden: User "developer" cannot list resource "nodes" in API group "" at the cluster scope.

So far so good. Authentication is done but the authrization is preventing.

kubectl version --short

Client Version: v1.18.6
Server Version: v1.18.0

Let us create a namespace named developer from the mangement machine:

kubectl create namespace developer

Outputs:

namespace/developer created

kubectl get namespaces

Output:

NAME              STATUS   AGE
default           Active   18d
developer         Active   48s
kube-node-lease   Active   18d
kube-public       Active   18d
kube-system       Active   18d
metallb-system    Active   14d

Now let us create role and rolebinding for our user developer.

Reference: https://kubernetes.io/docs/reference/access-authn-authz/rbac/

apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: developer
  name: pod-reading-creating-deleting
rules:
- apiGroups: ["extensions", "apps"] 
  resources: ["pods"]
  verbs: ["get", "watch", "list", "create", "update", "patch", "delete"]



apiVersion: rbac.authorization.k8s.io/v1
# This role binding allows "jane" to read pods in the "default" namespace.
# You need to already have a Role named "pod-reader" in that namespace.
kind: RoleBinding
metadata:
  name: pod-reading-creating-deleting
  namespace: developer
subjects:
# You can specify more than one "subject"
- kind: User
  name: developer # "name" is case sensitive
  apiGroup: rbac.authorization.k8s.io
roleRef:
  # "roleRef" specifies the binding to a Role / ClusterRole
  kind: Role #this must be Role or ClusterRole
  name: pod-reading-creating-deleting # this must match the name of the Role or ClusterRole you wish to bind to
  apiGroup: rbac.authorization.k8s.io

Now let us create developer-role.yaml and developer-role-binding.yaml with the content of the above and create those resource from the mangement machine:

kubectl create -f developer-role.yaml

Output: role.rbac.authorization.k8s.io/pod-reading-creating-deleting created

kubectl create -f developer-role-binding.yaml

Output: rolebinding.rbac.authorization.k8s.io/pod-reading-creating-deleting created

Now, let us try from the developer_machine with developer account:

It did not work as the context for the user developer was not set.

Let us set the context on management machine:

kubectl config set-credentials developer --client-certificate=developer.crt --client-key=developer.key

kubectl config set-context developer-context --cluster hardway-cluster-a  --user=developer

Now, let us edit the config file for the developer with the context data:

apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUNyRENDQVpRQ0NRRGw1aXRIV2cwdmRqQU5CZ2txaGtpRzl3MEJBUXNGQURBWU1SWXdGQVlEVlFRRERBMUwKVlVKRlVrNUZWRVZUTFVOQk1CNFhEVEl3TURjd01UQTJORE13TUZvWERUSXpNRE15T0RBMk5ETXdNRm93R0RFVwpNQlFHQTFVRUF3d05TMVZDUlZKT1JWUkZVeTFEUVRDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDCkFRb0NnZ0VCQVBOMkpueStzYVpuekRPYnhPWkh6SUx6cUIvSmlXNTNsVDZsVEFRbjJVd25YQzF3UzR5elFZaFgKNW80WHA5WE91OXdZR3Fsc1NwSzdLaVZTNXg5cEpzeUU0TXg0Zm1IODNFY1VVd0hYYjE4Y3h1SDZKd29obW5zVQoyeFd1eUdoUGJOTDhTVjhlQUIyQmNQZlpJMytSOVJCQ09RdEQxckduYWtXTHBibDBuTGdFa0ZwMmh4QVE0M3QyCksrRmUyRTVoWlZ0QkJQcy9lQ1hoQVhnUEVBRmgyTEhxYTNWMmRzdkZyMElQbjV5RmRyaEdXTitTaVJqZHJkYXEKWWJsOWdkcW05R29FY1ZwU1diM2tValZsQ0s3ZWtsTW8yTjlEeDF4SnN4UzdRTkZ1eEIvQW5Ic3VmeGg1RnVONwpDTlZKQ0J0NjZDUlJneVhCd3VFenIzOVQ0cW5tZGpVQ0F3RUFBVEFOQmdrcWhraUc5dzBCQVFzRkFBT0NBUUVBCjVJUlAvWnNGWVBDK0ZJN0dmL3A4TXZlUmFVY1h0eWpoZllFR2I2akdVL3JlM21jc2EwUnIwUHZSZlpUWWtHc1AKbzkxbmozVk1jUnRJZ21saEk3TFc2RGdveFl2MzZKRXRGaXllOXdlQVFNdEN2N3VNdjM2UThhSmlpY3Z1S0k2Ygo4Um1GUXNNdkYweW5pWHVJUks4S0REUFh5QkJ6MWk2YVdUV2RSRUR1elFKVjFQcnUyMUtic0JZQitrMUU2Nm8wCkI4dE5UZk42QzVQWWZuYjVRL2JEejRNaVZkL3d4WitOTnE5eTN4c3c1SytzS0JvUERmT1FWVkd6Zm9ScHJUWXkKbFZQL3JSY3AzcFh3V3k3YlRDOCs2ak5YUnBGcXlOVXlGOVhlMlJRQm9reUlhQkh4Z0hCKzROSXBTQXhhM0ZUUgp3M2hoMDJlbXJxVVFLT2VUZHBtZDlBPT0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
    server: https://192.168.5.15:6443
  name: hardway-cluster-a
contexts:
- context:
    cluster: hardway-cluster-a
    user: developer
  name: developer-context
current-context: developer-context
kind: Config
preferences: {}
users:
- name: developer
  user:
    client-certificate: /home/developer/developer.crt
    client-key: /home/developer/developer.key

After even this it did not work and then I realized that the apiGroups in the role file needs to be changed to get corrected for pods as given below:

apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: developer
  name: pod-reading-creating-deleting
rules:
- apiGroups: [""] 
  resources: ["pods"]
  verbs: ["get", "watch", "list", "create", "update", "patch", "delete"]

Now after repplying, these worked.


Q33b ######################### NEW USER ACCESS IN KUBERNETES ###########################################

### You have to provide a user named developer using a machine developer_machine on your same network the access to the kubernetes cluster in the namespace named developer and with full access on that particular namespace. The developer would like to start just typing kubectl commands and no background works can be allocated to him. Assume that as the administrator you have access to the developer machine and developer account completely with password and keys.

### ANSWER

On management machine:

Command reference: https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/

openssl genrsa -out john.key 2048
openssl req -new -key john.key -out john.csr

Let us change the above command to fit our requirements:

openssl genrsa -out developer.key 2048
openssl req -new -key developer.key -out developer.csr -subj "/CN=developer"
[See openssl req -new -key developer.key --help]

cat developer.csr | base64

From https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/

apiVersion: certificates.k8s.io/v1beta1
kind: CertificateSigningRequest
metadata:
  name: john
spec:
  groups:
  - system:authenticated
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1ZqQ0NBVDRDQVFBd0VURVBNQTBHQTFVRUF3d0dZVzVuWld4aE1JSUJJakFOQmdrcWhraUc5dzBCQVFFRgpBQU9DQVE4QU1JSUJDZ0tDQVFFQTByczhJTHRHdTYxakx2dHhWTTJSVlRWMDNHWlJTWWw0dWluVWo4RElaWjBOCnR2MUZtRVFSd3VoaUZsOFEzcWl0Qm0wMUFSMkNJVXBGd2ZzSjZ4MXF3ckJzVkhZbGlBNVhwRVpZM3ExcGswSDQKM3Z3aGJlK1o2MVNrVHF5SVBYUUwrTWM5T1Nsbm0xb0R2N0NtSkZNMUlMRVI3QTVGZnZKOEdFRjJ6dHBoaUlFMwpub1dtdHNZb3JuT2wzc2lHQ2ZGZzR4Zmd4eW8ybmlneFNVekl1bXNnVm9PM2ttT0x1RVF6cXpkakJ3TFJXbWlECklmMXBMWnoyalVnald4UkhCM1gyWnVVV1d1T09PZnpXM01LaE8ybHEvZi9DdS8wYk83c0x0MCt3U2ZMSU91TFcKcW90blZtRmxMMytqTy82WDNDKzBERHk5aUtwbXJjVDBnWGZLemE1dHJRSURBUUFCb0FBd0RRWUpLb1pJaHZjTgpBUUVMQlFBRGdnRUJBR05WdmVIOGR4ZzNvK21VeVRkbmFjVmQ1N24zSkExdnZEU1JWREkyQTZ1eXN3ZFp1L1BVCkkwZXpZWFV0RVNnSk1IRmQycVVNMjNuNVJsSXJ3R0xuUXFISUh5VStWWHhsdnZsRnpNOVpEWllSTmU3QlJvYXgKQVlEdUI5STZXT3FYbkFvczFqRmxNUG5NbFpqdU5kSGxpT1BjTU1oNndLaTZzZFhpVStHYTJ2RUVLY01jSVUyRgpvU2djUWdMYTk0aEpacGk3ZnNMdm1OQUxoT045UHdNMGM1dVJVejV4T0dGMUtCbWRSeEgvbUNOS2JKYjFRQm1HCkkwYitEUEdaTktXTU0xMzhIQXdoV0tkNjVoVHdYOWl4V3ZHMkh4TG1WQzg0L1BHT0tWQW9FNkpsYWFHdTlQVmkKdjlOSjVaZlZrcXdCd0hKbzZXdk9xVlA3SVFjZmg3d0drWm89Ci0tLS0tRU5EIENFUlRJRklDQVRFIFJFUVVFU1QtLS0tLQo=
  usages:
  - client auth

Let us change with our csr output from  cat developer.csr | base64 as given below:

apiVersion: certificates.k8s.io/v1beta1
kind: CertificateSigningRequest
metadata:
  name: developer-csr
spec:
  groups:
  - system:authenticated
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1dUQ0NBVUVDQVFBd0ZERVNN
QkFHQTFVRUF3d0paR1YyWld4dmNHVnlNSUlCSWpBTkJna3Foa2lHOXcwQgpBUUVGQUFPQ0FROEFN
SUlCQ2dLQ0FRRUFxUjVWeVVhSnB3czJkc2k0YUVnb09ycC9sa0xZM3VQVjVqWG1HVmRBCmxwQ1JE
T2hvV05Ub0lWRmEyRGM2Rmp4RkJJd1hFenFvWU03OC8vbGpaQ3RVZnRUa0Y5ZWM1Tm5UU1Zia3Vl
dTEKR3FxRitYemV3dFdYcDY3UXFXUSsxOENMNGsvMW9QTlpiQTE5S2N2TEtZa1VlU0MrVlkwZjNh
SkN0RUp6MEdBTgo1d1Y2MGp2V1JTbXgwWlJQdklqYW1WU0gvQkFnc2pFSDhpS0tLUWN0SHpqbEJ1
enhrWHYzWVdTdEpjazljTzZoCllnVmZSeFFDWEx1TzdmMkVVTE1YaGJXTGljcFUxSUlOckRjVEFp
Z29NUmRxcGQ0QmUzL3VqRXBSTS85anV2TngKVy9aYlNoeE9CWFp2czRUSUJlTzRZS0VBbFJVVHdS
Vlc3Nk5aeDF1QWErNGhod0lEQVFBQm9BQXdEUVlKS29aSQpodmNOQVFFTEJRQURnZ0VCQUFONCtt
TGhtNzV6b1R5cFE4eGRTMTF0Q1EwR0l6MnQvaVFTZVh0RVh2UlJidVlGCmNjSFVkN21BWFVpbEV6
MzlWQ3M5OVNKMmZQQlMzVlJSNXR6YUo4bUhQdFRNaWU5QTZ0Y2tPTUVjY2ErbVpha1cKb2VBOWRK
a3BtNXY5NElSNW9CWnlzRHFYbVYxRGV1M09saFBCNTFIL1RMZlczYWlLZC9LZnRYK295enQxQnNa
ago2SDRsOXNySXpvYjRTcGpudTdIRENoRUJYaVc2NGtIVzV4S1QxUjF2Y0hSaXNhdG9McElVSEZG
c2FwaG9nUWZFCnVtV1BNSDBJNFM2YlB3ZEd5VnRhckwyMVRjVXZQU0cvV3NCU2JkdmxFMUt3NDFF
UlNZZ2pySlZxS3B4QlowQWoKMzV2VVY1UnNRdlQrK2NjeEtwVkFCUHZvTGsxNGVQcllwUmUvbFhN
PQotLS0tLUVORCBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0K
  usages:
  - client auth

kubectl create -f developer-csr-object.yaml

kubectl get csr

NAME            AGE   SIGNERNAME                     REQUESTOR   CONDITION
developer-csr   24s   kubernetes.io/legacy-unknown   admin       Pending

kubectl certificate approve developer-csr

kubectl get csr

NAME            AGE   SIGNERNAME                     REQUESTOR   CONDITION
developer-csr   41s   kubernetes.io/legacy-unknown   admin       Approved,Issued

To check the jsonpath: kubectl get csr developer-csr -o json

The jsonpath of the certificate was found to be .status.certificate

kubectl get csr developer-csr -o jsonpath='{.status.certificate}' | base64 -d > developer.crt

Now, let us create the kubeconfigfile for the user developer on our management machine so that we can trnasfer to the developer machine later.

mkdir fordeveloper

mv developer.crt developer.key ca.crt fordeveloper/

cd developer/

Reference: https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/

kubectl config --kubeconfig=config set-cluster hardway-cluster-a --server=https://192.168.5.15:6443 --certificate-authority=ca.crt

kubectl config --kubeconfig=config set-credentials developer --client-certificate=developer.crt --client-key=developer.key

kubectl config --kubeconfig=config set-context developer-context --cluster=hardway-cluster-a --namespace=developer --user=developer

On developer machine:

Ref: https://kubernetes.io/docs/tasks/tools/install-kubectl/

curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl

chmod +x ./kubectl

sudo mv ./kubectl /usr/local/bin/kubectl

kubectl version --client

mkdir -v /home/developer/.kube/

chown developer:developer /home/developer/.kube/

From management machine:

scp ca.crt developer.crt developer.key config developer@10.9.108.252:/home/developer/.kube/

On developer machine:

cd /home/developer/.kube/

ls -la

total 20
drwxr-xr-x.  2 developer developer   76 Jul 20 21:30 .
drwx------. 18 developer developer 4096 Jul 20 21:24 ..
-rw-r--r--.  1 developer developer  989 Jul 20 21:30 ca.crt
-rw-------.  1 developer developer  407 Jul 20 21:30 config
-rw-r--r--.  1 developer developer 1054 Jul 20 21:30 developer.crt
-rw-r--r--.  1 developer developer 1675 Jul 20 21:30 developer.key

On management machine let us create the RBAC role and role binding for the user developer as asked in the question:

Reference: https://kubernetes.io/docs/reference/access-authn-authz/rbac/

Let us create a file named developer-role.yaml using the below content:

apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: developer
  name: all-access-to-developer-namespace-role
rules:
- apiGroups: ["*"] 
  resources: ["*"]
  verbs: ["*"]

Let us create a file named  developer-role-binding.yaml using the below content:

apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: all-access-to-developer-namespace-role-binding
  namespace: developer
subjects:

- kind: User
  name: developer
  apiGroup: rbac.authorization.k8s.io
roleRef:
 
  kind: Role 
  name: all-access-to-developer-namespace-role
  apiGroup: rbac.authorization.k8s.io


kubectl create namespace developer

kubectl create -f developer-role.yaml

kubectl create -f developer-role-binding.yaml

Let us now login to the developer machine as user developer and change the current context in config file in .kube folder as developer-context.

kubectl get all

kubectl create deployment my-test-deployment --image=nginx --dry-run=client -oyaml > my-test-deployment.yaml

And let us change the replicaset to be 5 in the my-test-deployment.yaml file.

cat my-test-deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: my-test-deployment
  name: my-test-deployment
spec:
  replicas: 5
  selector:
    matchLabels:
      app: my-test-deployment
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: my-test-deployment
    spec:
      containers:
      - image: nginx
        name: nginx
        resources: {}
status: {}

kubectl create -f my-test-deployment.yaml

kubectl get all

NAME                                      READY   STATUS    RESTARTS   AGE
pod/my-test-deployment-5b9dff5f6b-hqtxv   1/1     Running   0          31s
pod/my-test-deployment-5b9dff5f6b-lzfc9   1/1     Running   0          31s
pod/my-test-deployment-5b9dff5f6b-mmmsg   1/1     Running   0          31s
pod/my-test-deployment-5b9dff5f6b-pm8h6   1/1     Running   0          31s
pod/my-test-deployment-5b9dff5f6b-r8x7t   1/1     Running   0          31s

NAME                                 READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/my-test-deployment   5/5     5            5           31s

NAME                                            DESIRED   CURRENT   READY   AGE
replicaset.apps/my-test-deployment-5b9dff5f6b   5         5         5       31s

kubectl delete deployment.apps/my-test-deployment

From now on the user developer can do whatever he needs to do in the namespace of developer.

