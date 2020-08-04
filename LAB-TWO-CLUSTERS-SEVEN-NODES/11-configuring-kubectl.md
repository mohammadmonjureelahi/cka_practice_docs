#### LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A
#### LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A clustera
#### LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A

# Configuring kubectl for Remote Access

In this lab you will generate a kubeconfig file for the `kubectl` command line utility based on the `admin` user credentials.

> Run the commands in this lab from the same directory used to generate the admin client certificates on the k8smanagement node.

## The Admin Kubernetes Configuration File

Each kubeconfig requires a Kubernetes API Server to connect to. To support high availability the IP address assigned to the external load balancer fronting the Kubernetes API Servers will be used.

Generate a kubeconfig file suitable for authenticating as the `admin` user:

```
{
  KUBERNETES_LB_ADDRESS=192.168.5.20

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
clustera-worker-1   NotReady    <none>   118s   v1.18.0
clustera-worker-2   NotReady    <none>   118s   v1.18.0
clustera-worker-3   NotReady    <none>   118s   v1.18.0
clustera-worker-4   NotReady    <none>   118s   v1.18.0
clustera-worker-5   NotReady    <none>   118s   v1.18.0
```

Note: It is OK for the worker node to be in a `NotReady` state. Worker nodes will come into `Ready` state once networking is configured.

Next: [Deploy Pod Networking](12-configure-pod-networking.md)


#### LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B
#### LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B clusterb
#### LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B

Generate a kubeconfig file suitable for authenticating as the `admin` user: 

```
{
  KUBERNETES_LB_ADDRESS=192.168.5.40

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
clusterb-worker-1   NotReady    <none>   118s   v1.18.0
clusterb-worker-2   NotReady    <none>   118s   v1.18.0
clusterb-worker-3   NotReady    <none>   118s   v1.18.0
clusterb-worker-4   NotReady    <none>   118s   v1.18.0
clusterb-worker-5   NotReady    <none>   118s   v1.18.0
```

Note: It is OK for the worker node to be in a `NotReady` state. Worker nodes will come into `Ready` state once networking is configured.

Next: [Deploy Pod Networking](12-configure-pod-networking.md)

