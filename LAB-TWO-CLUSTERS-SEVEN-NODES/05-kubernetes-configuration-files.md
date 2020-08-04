#### LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A
#### LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A clustera
#### LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A


# Generating Kubernetes Configuration Files for Authentication
# We will generate these in seperate folder for each kubernetes cluster. We will do these on the k8smanagement node

In this lab you will generate [Kubernetes configuration files](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/), also known as kubeconfigs, which enable Kubernetes clients to locate and authenticate to the Kubernetes API Servers.

## Client Authentication Configs

In this section you will generate kubeconfig files for the `controller manager`, `clustera-kube-proxy`, `scheduler` clients and the `admin` user.

### Kubernetes Public IP Address hardway-cluster-a  clustera-kube-proxy clustera-clustera-kube-controller-manager

Each kubeconfig requires a Kubernetes API Server to connect to. To support high availability the IP address assigned to the load balancer will be used. In our case it is `192.168.5.20, 192.168.5.40 for clusters hardway-cluster-a, hardway-cluster-b respectively `

```
LOADBALANCER_ADDRESS=192.168.5.20
```

### The clustera-kube-proxy Kubernetes Configuration File

Generate a kubeconfig file for the `clustera-kube-proxy` service:

```
{
  kubectl config set-cluster hardway-cluster-a \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://${LOADBALANCER_ADDRESS}:6443 \
    --kubeconfig=clustera-kube-proxy.kubeconfig

  kubectl config set-credentials system:clustera-kube-proxy \
    --client-certificate=clustera-kube-proxy.crt \
    --client-key=clustera-kube-proxy.key \
    --embed-certs=true \
    --kubeconfig=clustera-kube-proxy.kubeconfig

  kubectl config set-context default \
    --cluster=hardway-cluster-a \
    --user=system:clustera-kube-proxy \
    --kubeconfig=clustera-kube-proxy.kubeconfig

  kubectl config use-context default --kubeconfig=clustera-kube-proxy.kubeconfig
}
```

Results:

```
clustera-kube-proxy.kubeconfig


Reference docs for clustera-kube-proxy [here](https://kubernetes.io/docs/reference/command-line-tools-reference/clustera-kube-proxy/)
```

### The clustera-kube-controller-manager Kubernetes Configuration File

Generate a kubeconfig file for the `clustera-kube-controller-manager` service:

```
{
  kubectl config set-cluster hardway-cluster-a \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=clustera-kube-controller-manager.kubeconfig

  kubectl config set-credentials system:clustera-kube-controller-manager \
    --client-certificate=clustera-kube-controller-manager.crt \
    --client-key=clustera-kube-controller-manager.key \
    --embed-certs=true \
    --kubeconfig=clustera-kube-controller-manager.kubeconfig

  kubectl config set-context default \
    --cluster=hardway-cluster-a \
    --user=system:clustera-kube-controller-manager \
    --kubeconfig=clustera-kube-controller-manager.kubeconfig

  kubectl config use-context default --kubeconfig=clustera-kube-controller-manager.kubeconfig
}
```

Results:

```
clustera-kube-controller-manager.kubeconfig
```

Reference docs for clustera-kube-controller-manager [here](https://kubernetes.io/docs/reference/command-line-tools-reference/clustera-kube-controller-manager/)

### The clustera-kube-scheduler Kubernetes Configuration File clustera-clustera-kube-scheduler

Generate a kubeconfig file for the `clustera-kube-scheduler` service:

```
{
  kubectl config set-cluster hardway-cluster-a \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=clustera-kube-scheduler.kubeconfig

  kubectl config set-credentials system:clustera-kube-scheduler \
    --client-certificate=clustera-kube-scheduler.crt \
    --client-key=clustera-kube-scheduler.key \
    --embed-certs=true \
    --kubeconfig=clustera-kube-scheduler.kubeconfig

  kubectl config set-context default \
    --cluster=hardway-cluster-a \
    --user=system:clustera-kube-scheduler \
    --kubeconfig=clustera-kube-scheduler.kubeconfig

  kubectl config use-context default --kubeconfig=clustera-kube-scheduler.kubeconfig
}
```

Results:

```
clustera-kube-scheduler.kubeconfig
```

Reference docs for clustera-kube-scheduler [here](https://kubernetes.io/docs/reference/command-line-tools-reference/clustera-kube-scheduler/)

### The admin Kubernetes Configuration File

Generate a kubeconfig file for the `admin` user:

```
{
  kubectl config set-cluster hardway-cluster-a \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=admin.kubeconfig

  kubectl config set-credentials admin \
    --client-certificate=admin.crt \
    --client-key=admin.key \
    --embed-certs=true \
    --kubeconfig=admin.kubeconfig

  kubectl config set-context default \
    --cluster=hardway-cluster-a \
    --user=admin \
    --kubeconfig=admin.kubeconfig

  kubectl config use-context default --kubeconfig=admin.kubeconfig
}
```

Results:

```
admin.kubeconfig
```

Reference docs for kubeconfig [here](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/)

##

## Distribute the Kubernetes Configuration Files

Copy the appropriate `clustera-kube-proxy` kubeconfig files to each worker instance:

```
for instance in clustera-worker-1 clustera-worker-2 clustera-worker-3 clustera-worker-4 clustera-worker-5; do
  scp clustera-kube-proxy.kubeconfig ${instance}:~/
done
```

Copy the appropriate `admin.kubeconfig`, `clustera-kube-controller-manager` and `clustera-kube-scheduler` kubeconfig files to each controller instance:

```
for instance in clustera-master-1 clustera-master-2; do
  scp admin.kubeconfig clustera-kube-controller-manager.kubeconfig clustera-kube-scheduler.kubeconfig ${instance}:~/
done
```

Next: [Generating the Data Encryption Config and Key](06-data-encryption-keys.md)

#### LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B
#### LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B clusterb
#### LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B

LOADBALANCER_ADDRESS=192.168.5.40
```

### The clusterb-kube-proxy Kubernetes Configuration File hardway-cluster-b clusterb 

Generate a kubeconfig file for the `clusterb-kube-proxy` service:

```
{
  kubectl config set-cluster hardway-cluster-b \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://${LOADBALANCER_ADDRESS}:6443 \
    --kubeconfig=clusterb-kube-proxy.kubeconfig

  kubectl config set-credentials system:clusterb-kube-proxy \
    --client-certificate=clusterb-kube-proxy.crt \
    --client-key=clusterb-kube-proxy.key \
    --embed-certs=true \
    --kubeconfig=clusterb-kube-proxy.kubeconfig

  kubectl config set-context default \
    --cluster=hardway-cluster-b \
    --user=system:clusterb-kube-proxy \
    --kubeconfig=clusterb-kube-proxy.kubeconfig

  kubectl config use-context default --kubeconfig=clusterb-kube-proxy.kubeconfig
}
```

Results:

```
clusterb-kube-proxy.kubeconfig


Reference docs for clusterb-kube-proxy [here](https://kubernetes.io/docs/reference/command-line-tools-reference/clusterb-kube-proxy/)
```

### The clusterb-kube-controller-manager Kubernetes Configuration File

Generate a kubeconfig file for the `clusterb-kube-controller-manager` service:

```
{
  kubectl config set-cluster hardway-cluster-b \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=clusterb-kube-controller-manager.kubeconfig

  kubectl config set-credentials system:clusterb-kube-controller-manager \
    --client-certificate=clusterb-kube-controller-manager.crt \
    --client-key=clusterb-kube-controller-manager.key \
    --embed-certs=true \
    --kubeconfig=clusterb-kube-controller-manager.kubeconfig

  kubectl config set-context default \
    --cluster=hardway-cluster-b \
    --user=system:clusterb-kube-controller-manager \
    --kubeconfig=clusterb-kube-controller-manager.kubeconfig

  kubectl config use-context default --kubeconfig=clusterb-kube-controller-manager.kubeconfig
}
```

Results:

```
clusterb-kube-controller-manager.kubeconfig
```

Reference docs for clusterb-kube-controller-manager [here](https://kubernetes.io/docs/reference/command-line-tools-reference/clusterb-kube-controller-manager/)

### The clusterb-kube-scheduler Kubernetes Configuration File clusterb-clusterb-kube-scheduler

Generate a kubeconfig file for the `clusterb-kube-scheduler` service:

```
{
  kubectl config set-cluster hardway-cluster-b \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=clusterb-kube-scheduler.kubeconfig

  kubectl config set-credentials system:clusterb-kube-scheduler \
    --client-certificate=clusterb-kube-scheduler.crt \
    --client-key=clusterb-kube-scheduler.key \
    --embed-certs=true \
    --kubeconfig=clusterb-kube-scheduler.kubeconfig

  kubectl config set-context default \
    --cluster=hardway-cluster-b \
    --user=system:clusterb-kube-scheduler \
    --kubeconfig=clusterb-kube-scheduler.kubeconfig

  kubectl config use-context default --kubeconfig=clusterb-kube-scheduler.kubeconfig
}
```

Results:

```
clusterb-kube-scheduler.kubeconfig
```

Reference docs for clusterb-kube-scheduler [here](https://kubernetes.io/docs/reference/command-line-tools-reference/clusterb-kube-scheduler/)

### The admin Kubernetes Configuration File

Generate a kubeconfig file for the `admin` user:

```
{
  kubectl config set-cluster hardway-cluster-b \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=admin.kubeconfig

  kubectl config set-credentials admin \
    --client-certificate=admin.crt \
    --client-key=admin.key \
    --embed-certs=true \
    --kubeconfig=admin.kubeconfig

  kubectl config set-context default \
    --cluster=hardway-cluster-b \
    --user=admin \
    --kubeconfig=admin.kubeconfig

  kubectl config use-context default --kubeconfig=admin.kubeconfig
}
```

Results:

```
admin.kubeconfig
```

Reference docs for kubeconfig [here](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/)

##

## Distribute the Kubernetes Configuration Files

Copy the appropriate `clusterb-kube-proxy` kubeconfig files to each worker instance:

```
for instance in clusterb-worker-1 clusterb-worker-2 clusterb-worker-3 clusterb-worker-4 clusterb-worker-5; do
  scp clusterb-kube-proxy.kubeconfig ${instance}:~/
done
```

Copy the appropriate `admin.kubeconfig`, `clusterb-kube-controller-manager` and `clusterb-kube-scheduler` kubeconfig files to each controller instance:

```
for instance in clusterb-master-1 clusterb-master-2; do
  scp admin.kubeconfig clusterb-kube-controller-manager.kubeconfig clusterb-kube-scheduler.kubeconfig ${instance}:~/
done
```
