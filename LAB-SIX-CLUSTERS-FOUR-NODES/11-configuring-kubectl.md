#### LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A
#### LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A laba
#### LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A

# Configuring kubectl for Remote Access

In this lab you will generate a kubeconfig file for the `kubectl` command line utility based on the `admin` user credentials.

> Run the commands in this lab from the same directory used to generate the admin client certificates on the k8smanagement node.

## The Admin Kubernetes Configuration File

Each kubeconfig requires a Kubernetes API Server to connect to. To support high availability the IP address assigned to the external load balancer fronting the Kubernetes API Servers will be used.

Generate a kubeconfig file suitable for authenticating as the `admin` user:

```
{
  KUBERNETES_LB_ADDRESS=192.168.5.15

  kubectl config set-cluster hardway-cluster-a \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://${KUBERNETES_LB_ADDRESS}:6443

  kubectl config set-credentials admin \
    --client-certificate=admin.crt \
    --client-key=admin.key

  kubectl config set-context hardway-cluster-a \
    --cluster=hardway-cluster-a \
    --user=admin

  kubectl config use-context hardway-cluster-a
}

Reference doc for kubectl config [here](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/)
```

## Verification

Check the health of the remote Kubernetes cluster:

```
kubectl get componentstatuses
```

> output

```
NAME                 STATUS    MESSAGE             ERROR
controller-manager   Healthy   ok
scheduler            Healthy   ok
etcd-1               Healthy   {"health":"true"}
etcd-0               Healthy   {"health":"true"}
```

List the nodes in the remote Kubernetes cluster:

```
kubectl get nodes
```

> output

```
NAME       STATUS   ROLES    AGE    VERSION
laba-worker-1   NotReady    <none>   118s   v1.13.0
laba-worker-2   NotReady    <none>   118s   v1.13.0
```

Note: It is OK for the worker node to be in a `NotReady` state. Worker nodes will come into `Ready` state once networking is configured.

Next: [Deploy Pod Networking](12-configure-pod-networking.md)


#### LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B
#### LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B labb
#### LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B

Generate a kubeconfig file suitable for authenticating as the `admin` user: 

```
{
  KUBERNETES_LB_ADDRESS=192.168.5.30

  kubectl config set-cluster hardway-cluster-b \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://${KUBERNETES_LB_ADDRESS}:6443

  kubectl config set-credentials admin \
    --client-certificate=admin.crt \
    --client-key=admin.key

  kubectl config set-context hardway-cluster-b \
    --cluster=hardway-cluster-b \
    --user=admin

  kubectl config use-context hardway-cluster-b
}

Reference doc for kubectl config [here](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/)
```

## Verification

Check the health of the remote Kubernetes cluster:

```
kubectl get componentstatuses
```

> output

```
NAME                 STATUS    MESSAGE             ERROR
controller-manager   Healthy   ok
scheduler            Healthy   ok
etcd-1               Healthy   {"health":"true"}
etcd-0               Healthy   {"health":"true"}
```

List the nodes in the remote Kubernetes cluster:

```
kubectl get nodes
```

> output

```
NAME       STATUS   ROLES    AGE    VERSION
labb-worker-1   NotReady    <none>   118s   v1.13.0
labb-worker-2   NotReady    <none>   118s   v1.13.0
```

Note: It is OK for the worker node to be in a `NotReady` state. Worker nodes will come into `Ready` state once networking is configured.

Next: [Deploy Pod Networking](12-configure-pod-networking.md)


#### LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C
#### LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C labc
#### LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C

Generate a kubeconfig file suitable for authenticating as the `admin` user: 

```
{
  KUBERNETES_LB_ADDRESS=192.168.5.45

  kubectl config set-cluster hardway-cluster-c \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://${KUBERNETES_LB_ADDRESS}:6443

  kubectl config set-credentials admin \
    --client-certificate=admin.crt \
    --client-key=admin.key

  kubectl config set-context hardway-cluster-c \
    --cluster=hardway-cluster-c \
    --user=admin

  kubectl config use-context hardway-cluster-c
}

Reference doc for kubectl config [here](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/)
```

## Verification

Check the health of the remote Kubernetes cluster:

```
kubectl get componentstatuses
```

> output

```
NAME                 STATUS    MESSAGE             ERROR
controller-manager   Healthy   ok
scheduler            Healthy   ok
etcd-1               Healthy   {"health":"true"}
etcd-0               Healthy   {"health":"true"}
```

List the nodes in the remote Kubernetes cluster:

```
kubectl get nodes
```

> output

```
NAME       STATUS   ROLES    AGE    VERSION
labc-worker-1   NotReady    <none>   118s   v1.13.0
labc-worker-2   NotReady    <none>   118s   v1.13.0
```

Note: It is OK for the worker node to be in a `NotReady` state. Worker nodes will come into `Ready` state once networking is configured.

Next: [Deploy Pod Networking](12-configure-pod-networking.md)


### LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D
#### LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D labd
#### LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D

Generate a kubeconfig file suitable for authenticating as the `admin` user: 

```
{
  KUBERNETES_LB_ADDRESS=192.168.5.60

  kubectl config set-cluster hardway-cluster-d \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://${KUBERNETES_LB_ADDRESS}:6443

  kubectl config set-credentials admin \
    --client-certificate=admin.crt \
    --client-key=admin.key

  kubectl config set-context hardway-cluster-d \
    --cluster=hardway-cluster-d \
    --user=admin

  kubectl config use-context hardway-cluster-d
}

Reference doc for kubectl config [here](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/)
```

## Verification

Check the health of the remote Kubernetes cluster:

```
kubectl get componentstatuses
```

> output

```
NAME                 STATUS    MESSAGE             ERROR
controller-manager   Healthy   ok
scheduler            Healthy   ok
etcd-1               Healthy   {"health":"true"}
etcd-0               Healthy   {"health":"true"}
```

List the nodes in the remote Kubernetes cluster:

```
kubectl get nodes
```

> output

```
NAME       STATUS   ROLES    AGE    VERSION
labd-worker-1   NotReady    <none>   118s   v1.13.0
labd-worker-2   NotReady    <none>   118s   v1.13.0
```

Note: It is OK for the worker node to be in a `NotReady` state. Worker nodes will come into `Ready` state once networking is configured.

Next: [Deploy Pod Networking](12-configure-pod-networking.md)


#### LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E
#### LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E labe
#### LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E

Generate a kubeconfig file suitable for authenticating as the `admin` user: 

```
{
  KUBERNETES_LB_ADDRESS=192.168.5.75

  kubectl config set-cluster hardway-cluster-e \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://${KUBERNETES_LB_ADDRESS}:6443

  kubectl config set-credentials admin \
    --client-certificate=admin.crt \
    --client-key=admin.key

  kubectl config set-context hardway-cluster-e \
    --cluster=hardway-cluster-e \
    --user=admin

  kubectl config use-context hardway-cluster-e
}

Reference doc for kubectl config [here](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/)
```

## Verification

Check the health of the remote Kubernetes cluster:

```
kubectl get componentstatuses
```

> output

```
NAME                 STATUS    MESSAGE             ERROR
controller-manager   Healthy   ok
scheduler            Healthy   ok
etcd-1               Healthy   {"health":"true"}
etcd-0               Healthy   {"health":"true"}
```

List the nodes in the remote Kubernetes cluster:

```
kubectl get nodes
```

> output

```
NAME       STATUS   ROLES    AGE    VERSION
labe-worker-1   NotReady    <none>   118s   v1.13.0
labe-worker-2   NotReady    <none>   118s   v1.13.0
```

Note: It is OK for the worker node to be in a `NotReady` state. Worker nodes will come into `Ready` state once networking is configured.

Next: [Deploy Pod Networking](12-configure-pod-networking.md)


#### LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F
#### LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F labf
#### LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F

Generate a kubeconfig file suitable for authenticating as the `admin` user: 

```
{
  KUBERNETES_LB_ADDRESS=192.168.5.90

  kubectl config set-cluster hardway-cluster-f \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://${KUBERNETES_LB_ADDRESS}:6443

  kubectl config set-credentials admin \
    --client-certificate=admin.crt \
    --client-key=admin.key

  kubectl config set-context hardway-cluster-f \
    --cluster=hardway-cluster-f \
    --user=admin

  kubectl config use-context hardway-cluster-f
}

Reference doc for kubectl config [here](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/)
```

## Verification

Check the health of the remote Kubernetes cluster:

```
kubectl get componentstatuses
```

> output

```
NAME                 STATUS    MESSAGE             ERROR
controller-manager   Healthy   ok
scheduler            Healthy   ok
etcd-1               Healthy   {"health":"true"}
etcd-0               Healthy   {"health":"true"}
```

List the nodes in the remote Kubernetes cluster:

```
kubectl get nodes
```

> output

```
NAME       STATUS   ROLES    AGE    VERSION
labf-worker-1   NotReady    <none>   118s   v1.13.0
labf-worker-2   NotReady    <none>   118s   v1.13.0
```

Note: It is OK for the worker node to be in a `NotReady` state. Worker nodes will come into `Ready` state once networking is configured.

Next: [Deploy Pod Networking](12-configure-pod-networking.md)

