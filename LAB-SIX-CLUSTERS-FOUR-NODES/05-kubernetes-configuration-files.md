#### LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A
#### LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A laba
#### LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A


# Generating Kubernetes Configuration Files for Authentication
# We will generate these in seperate folder for each kubernetes cluster. We will do these on the k8smanagement node

In this lab you will generate [Kubernetes configuration files](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/), also known as kubeconfigs, which enable Kubernetes clients to locate and authenticate to the Kubernetes API Servers.

## Client Authentication Configs

In this section you will generate kubeconfig files for the `controller manager`, `laba-kube-proxy`, `scheduler` clients and the `admin` user.

### Kubernetes Public IP Address hardway-cluster-a  laba-kube-proxy laba-laba-kube-controller-manager

Each kubeconfig requires a Kubernetes API Server to connect to. To support high availability the IP address assigned to the load balancer will be used. In our case it is `192.168.5.15, 192.168.5.30, 192.168.5.45, 192.168.5.60, 192.168.5.75, 192.168.5.90 for clusters hardway-cluster-a, hardway-cluster-b, hardway-cluster-c, hardway-cluster-d, hardway-cluster-e, hardway-cluster-f respectively `

```
LOADBALANCER_ADDRESS=192.168.5.15
```

### The laba-kube-proxy Kubernetes Configuration File

Generate a kubeconfig file for the `laba-kube-proxy` service:

```
{
  kubectl config set-cluster hardway-cluster-a \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://${LOADBALANCER_ADDRESS}:6443 \
    --kubeconfig=laba-kube-proxy.kubeconfig

  kubectl config set-credentials system:laba-kube-proxy \
    --client-certificate=laba-kube-proxy.crt \
    --client-key=laba-kube-proxy.key \
    --embed-certs=true \
    --kubeconfig=laba-kube-proxy.kubeconfig

  kubectl config set-context default \
    --cluster=hardway-cluster-a \
    --user=system:laba-kube-proxy \
    --kubeconfig=laba-kube-proxy.kubeconfig

  kubectl config use-context default --kubeconfig=laba-kube-proxy.kubeconfig
}
```

Results:

```
laba-kube-proxy.kubeconfig


Reference docs for laba-kube-proxy [here](https://kubernetes.io/docs/reference/command-line-tools-reference/laba-kube-proxy/)
```

### The laba-kube-controller-manager Kubernetes Configuration File

Generate a kubeconfig file for the `laba-kube-controller-manager` service:

```
{
  kubectl config set-cluster hardway-cluster-a \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=laba-kube-controller-manager.kubeconfig

  kubectl config set-credentials system:laba-kube-controller-manager \
    --client-certificate=laba-kube-controller-manager.crt \
    --client-key=laba-kube-controller-manager.key \
    --embed-certs=true \
    --kubeconfig=laba-kube-controller-manager.kubeconfig

  kubectl config set-context default \
    --cluster=hardway-cluster-a \
    --user=system:laba-kube-controller-manager \
    --kubeconfig=laba-kube-controller-manager.kubeconfig

  kubectl config use-context default --kubeconfig=laba-kube-controller-manager.kubeconfig
}
```

Results:

```
laba-kube-controller-manager.kubeconfig
```

Reference docs for laba-kube-controller-manager [here](https://kubernetes.io/docs/reference/command-line-tools-reference/laba-kube-controller-manager/)

### The laba-kube-scheduler Kubernetes Configuration File laba-laba-kube-scheduler

Generate a kubeconfig file for the `laba-kube-scheduler` service:

```
{
  kubectl config set-cluster hardway-cluster-a \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=laba-kube-scheduler.kubeconfig

  kubectl config set-credentials system:laba-kube-scheduler \
    --client-certificate=laba-kube-scheduler.crt \
    --client-key=laba-kube-scheduler.key \
    --embed-certs=true \
    --kubeconfig=laba-kube-scheduler.kubeconfig

  kubectl config set-context default \
    --cluster=hardway-cluster-a \
    --user=system:laba-kube-scheduler \
    --kubeconfig=laba-kube-scheduler.kubeconfig

  kubectl config use-context default --kubeconfig=laba-kube-scheduler.kubeconfig
}
```

Results:

```
laba-kube-scheduler.kubeconfig
```

Reference docs for laba-kube-scheduler [here](https://kubernetes.io/docs/reference/command-line-tools-reference/laba-kube-scheduler/)

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

Copy the appropriate `laba-kube-proxy` kubeconfig files to each worker instance:

```
for instance in laba-worker-1 laba-worker-2; do
  scp laba-kube-proxy.kubeconfig ${instance}:~/
done
```

Copy the appropriate `admin.kubeconfig`, `laba-kube-controller-manager` and `laba-kube-scheduler` kubeconfig files to each controller instance:

```
for instance in laba-master-1 laba-master-2; do
  scp admin.kubeconfig laba-kube-controller-manager.kubeconfig laba-kube-scheduler.kubeconfig ${instance}:~/
done
```

Next: [Generating the Data Encryption Config and Key](06-data-encryption-keys.md)

#### LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B
#### LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B labb
#### LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B

LOADBALANCER_ADDRESS=192.168.5.30
```

### The labb-kube-proxy Kubernetes Configuration File hardway-cluster-b labb 

Generate a kubeconfig file for the `labb-kube-proxy` service:

```
{
  kubectl config set-cluster hardway-cluster-b \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://${LOADBALANCER_ADDRESS}:6443 \
    --kubeconfig=labb-kube-proxy.kubeconfig

  kubectl config set-credentials system:labb-kube-proxy \
    --client-certificate=labb-kube-proxy.crt \
    --client-key=labb-kube-proxy.key \
    --embed-certs=true \
    --kubeconfig=labb-kube-proxy.kubeconfig

  kubectl config set-context default \
    --cluster=hardway-cluster-b \
    --user=system:labb-kube-proxy \
    --kubeconfig=labb-kube-proxy.kubeconfig

  kubectl config use-context default --kubeconfig=labb-kube-proxy.kubeconfig
}
```

Results:

```
labb-kube-proxy.kubeconfig


Reference docs for labb-kube-proxy [here](https://kubernetes.io/docs/reference/command-line-tools-reference/labb-kube-proxy/)
```

### The labb-kube-controller-manager Kubernetes Configuration File

Generate a kubeconfig file for the `labb-kube-controller-manager` service:

```
{
  kubectl config set-cluster hardway-cluster-b \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=labb-kube-controller-manager.kubeconfig

  kubectl config set-credentials system:labb-kube-controller-manager \
    --client-certificate=labb-kube-controller-manager.crt \
    --client-key=labb-kube-controller-manager.key \
    --embed-certs=true \
    --kubeconfig=labb-kube-controller-manager.kubeconfig

  kubectl config set-context default \
    --cluster=hardway-cluster-b \
    --user=system:labb-kube-controller-manager \
    --kubeconfig=labb-kube-controller-manager.kubeconfig

  kubectl config use-context default --kubeconfig=labb-kube-controller-manager.kubeconfig
}
```

Results:

```
labb-kube-controller-manager.kubeconfig
```

Reference docs for labb-kube-controller-manager [here](https://kubernetes.io/docs/reference/command-line-tools-reference/labb-kube-controller-manager/)

### The labb-kube-scheduler Kubernetes Configuration File labb-labb-kube-scheduler

Generate a kubeconfig file for the `labb-kube-scheduler` service:

```
{
  kubectl config set-cluster hardway-cluster-b \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=labb-kube-scheduler.kubeconfig

  kubectl config set-credentials system:labb-kube-scheduler \
    --client-certificate=labb-kube-scheduler.crt \
    --client-key=labb-kube-scheduler.key \
    --embed-certs=true \
    --kubeconfig=labb-kube-scheduler.kubeconfig

  kubectl config set-context default \
    --cluster=hardway-cluster-b \
    --user=system:labb-kube-scheduler \
    --kubeconfig=labb-kube-scheduler.kubeconfig

  kubectl config use-context default --kubeconfig=labb-kube-scheduler.kubeconfig
}
```

Results:

```
labb-kube-scheduler.kubeconfig
```

Reference docs for labb-kube-scheduler [here](https://kubernetes.io/docs/reference/command-line-tools-reference/labb-kube-scheduler/)

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

Copy the appropriate `labb-kube-proxy` kubeconfig files to each worker instance:

```
for instance in labb-worker-1 labb-worker-2; do
  scp labb-kube-proxy.kubeconfig ${instance}:~/
done
```

Copy the appropriate `admin.kubeconfig`, `labb-kube-controller-manager` and `labb-kube-scheduler` kubeconfig files to each controller instance:

```
for instance in labb-master-1 labb-master-2; do
  scp admin.kubeconfig labb-kube-controller-manager.kubeconfig labb-kube-scheduler.kubeconfig ${instance}:~/
done
```

#### LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C
#### LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C labc
#### LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C

LOADBALANCER_ADDRESS=192.168.5.45
```

### The labc-kube-proxy Kubernetes Configuration File hardway-cluster-c labc

Generate a kubeconfig file for the `labc-kube-proxy` service:

```
{
  kubectl config set-cluster hardway-cluster-c \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://${LOADBALANCER_ADDRESS}:6443 \
    --kubeconfig=labc-kube-proxy.kubeconfig

  kubectl config set-credentials system:labc-kube-proxy \
    --client-certificate=labc-kube-proxy.crt \
    --client-key=labc-kube-proxy.key \
    --embed-certs=true \
    --kubeconfig=labc-kube-proxy.kubeconfig

  kubectl config set-context default \
    --cluster=hardway-cluster-c \
    --user=system:labc-kube-proxy \
    --kubeconfig=labc-kube-proxy.kubeconfig

  kubectl config use-context default --kubeconfig=labc-kube-proxy.kubeconfig
}
```

Results:

```
labc-kube-proxy.kubeconfig


Reference docs for labc-kube-proxy [here](https://kubernetes.io/docs/reference/command-line-tools-reference/labc-kube-proxy/)
```

### The labc-kube-controller-manager Kubernetes Configuration File

Generate a kubeconfig file for the `labc-kube-controller-manager` service:

```
{
  kubectl config set-cluster hardway-cluster-c \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=labc-kube-controller-manager.kubeconfig

  kubectl config set-credentials system:labc-kube-controller-manager \
    --client-certificate=labc-kube-controller-manager.crt \
    --client-key=labc-kube-controller-manager.key \
    --embed-certs=true \
    --kubeconfig=labc-kube-controller-manager.kubeconfig

  kubectl config set-context default \
    --cluster=hardway-cluster-c \
    --user=system:labc-kube-controller-manager \
    --kubeconfig=labc-kube-controller-manager.kubeconfig

  kubectl config use-context default --kubeconfig=labc-kube-controller-manager.kubeconfig
}
```

Results:

```
labc-kube-controller-manager.kubeconfig
```

Reference docs for labc-kube-controller-manager [here](https://kubernetes.io/docs/reference/command-line-tools-reference/labc-kube-controller-manager/)

### The labc-kube-scheduler Kubernetes Configuration File labc-labc-kube-scheduler

Generate a kubeconfig file for the `labc-kube-scheduler` service:

```
{
  kubectl config set-cluster hardway-cluster-c \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=labc-kube-scheduler.kubeconfig

  kubectl config set-credentials system:labc-kube-scheduler \
    --client-certificate=labc-kube-scheduler.crt \
    --client-key=labc-kube-scheduler.key \
    --embed-certs=true \
    --kubeconfig=labc-kube-scheduler.kubeconfig

  kubectl config set-context default \
    --cluster=hardway-cluster-c \
    --user=system:labc-kube-scheduler \
    --kubeconfig=labc-kube-scheduler.kubeconfig

  kubectl config use-context default --kubeconfig=labc-kube-scheduler.kubeconfig
}
```

Results:

```
labc-kube-scheduler.kubeconfig
```

Reference docs for labc-kube-scheduler [here](https://kubernetes.io/docs/reference/command-line-tools-reference/labc-kube-scheduler/)

### The admin Kubernetes Configuration File

Generate a kubeconfig file for the `admin` user:

```
{
  kubectl config set-cluster hardway-cluster-c \
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
    --cluster=hardway-cluster-c \
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

Copy the appropriate `labc-kube-proxy` kubeconfig files to each worker instance:

```
for instance in labc-worker-1 labc-worker-2; do
  scp labc-kube-proxy.kubeconfig ${instance}:~/
done
```

Copy the appropriate `admin.kubeconfig`, `labc-kube-controller-manager` and `labc-kube-scheduler` kubeconfig files to each controller instance:

```
for instance in labc-master-1 labc-master-2; do
  scp admin.kubeconfig labc-kube-controller-manager.kubeconfig labc-kube-scheduler.kubeconfig ${instance}:~/
done
```

#### LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D
#### LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D labd
#### LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D

LOADBALANCER_ADDRESS=192.168.5.60
```

### The labd-kube-proxy Kubernetes Configuration File hardway-cluster-d labd

Generate a kubeconfig file for the `labd-kube-proxy` service:

```
{
  kubectl config set-cluster hardway-cluster-d \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://${LOADBALANCER_ADDRESS}:6443 \
    --kubeconfig=labd-kube-proxy.kubeconfig

  kubectl config set-credentials system:labd-kube-proxy \
    --client-certificate=labd-kube-proxy.crt \
    --client-key=labd-kube-proxy.key \
    --embed-certs=true \
    --kubeconfig=labd-kube-proxy.kubeconfig

  kubectl config set-context default \
    --cluster=hardway-cluster-d \
    --user=system:labd-kube-proxy \
    --kubeconfig=labd-kube-proxy.kubeconfig

  kubectl config use-context default --kubeconfig=labd-kube-proxy.kubeconfig
}
```

Results:

```
labd-kube-proxy.kubeconfig


Reference docs for labd-kube-proxy [here](https://kubernetes.io/docs/reference/command-line-tools-reference/labd-kube-proxy/)
```

### The labd-kube-controller-manager Kubernetes Configuration File

Generate a kubeconfig file for the `labd-kube-controller-manager` service:

```
{
  kubectl config set-cluster hardway-cluster-d \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=labd-kube-controller-manager.kubeconfig

  kubectl config set-credentials system:labd-kube-controller-manager \
    --client-certificate=labd-kube-controller-manager.crt \
    --client-key=labd-kube-controller-manager.key \
    --embed-certs=true \
    --kubeconfig=labd-kube-controller-manager.kubeconfig

  kubectl config set-context default \
    --cluster=hardway-cluster-d \
    --user=system:labd-kube-controller-manager \
    --kubeconfig=labd-kube-controller-manager.kubeconfig

  kubectl config use-context default --kubeconfig=labd-kube-controller-manager.kubeconfig
}
```

Results:

```
labd-kube-controller-manager.kubeconfig
```

Reference docs for labd-kube-controller-manager [here](https://kubernetes.io/docs/reference/command-line-tools-reference/labd-kube-controller-manager/)

### The labd-kube-scheduler Kubernetes Configuration File labd-labd-kube-scheduler

Generate a kubeconfig file for the `labd-kube-scheduler` service:

```
{
  kubectl config set-cluster hardway-cluster-d \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=labd-kube-scheduler.kubeconfig

  kubectl config set-credentials system:labd-kube-scheduler \
    --client-certificate=labd-kube-scheduler.crt \
    --client-key=labd-kube-scheduler.key \
    --embed-certs=true \
    --kubeconfig=labd-kube-scheduler.kubeconfig

  kubectl config set-context default \
    --cluster=hardway-cluster-d \
    --user=system:labd-kube-scheduler \
    --kubeconfig=labd-kube-scheduler.kubeconfig

  kubectl config use-context default --kubeconfig=labd-kube-scheduler.kubeconfig
}
```

Results:

```
labd-kube-scheduler.kubeconfig
```

Reference docs for labd-kube-scheduler [here](https://kubernetes.io/docs/reference/command-line-tools-reference/labd-kube-scheduler/)

### The admin Kubernetes Configuration File

Generate a kubeconfig file for the `admin` user:

```
{
  kubectl config set-cluster hardway-cluster-d \
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
    --cluster=hardway-cluster-d \
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

Copy the appropriate `labd-kube-proxy` kubeconfig files to each worker instance:

```
for instance in labd-worker-1 labd-worker-2; do
  scp labd-kube-proxy.kubeconfig ${instance}:~/
done
```

Copy the appropriate `admin.kubeconfig`, `labd-kube-controller-manager` and `labd-kube-scheduler` kubeconfig files to each controller instance:

```
for instance in labd-master-1 labd-master-2; do
  scp admin.kubeconfig labd-kube-controller-manager.kubeconfig labd-kube-scheduler.kubeconfig ${instance}:~/
done
```

#### LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E
#### LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E labe
#### LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E

LOADBALANCER_ADDRESS=192.168.5.75
```

### The labe-kube-proxy Kubernetes Configuration File hardway-cluster-e labe

Generate a kubeconfig file for the `labe-kube-proxy` service:

```
{
  kubectl config set-cluster hardway-cluster-e \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://${LOADBALANCER_ADDRESS}:6443 \
    --kubeconfig=labe-kube-proxy.kubeconfig

  kubectl config set-credentials system:labe-kube-proxy \
    --client-certificate=labe-kube-proxy.crt \
    --client-key=labe-kube-proxy.key \
    --embed-certs=true \
    --kubeconfig=labe-kube-proxy.kubeconfig

  kubectl config set-context default \
    --cluster=hardway-cluster-e \
    --user=system:labe-kube-proxy \
    --kubeconfig=labe-kube-proxy.kubeconfig

  kubectl config use-context default --kubeconfig=labe-kube-proxy.kubeconfig
}
```

Results:

```
labe-kube-proxy.kubeconfig


Reference docs for labe-kube-proxy [here](https://kubernetes.io/docs/reference/command-line-tools-reference/labe-kube-proxy/)
```

### The labe-kube-controller-manager Kubernetes Configuration File

Generate a kubeconfig file for the `labe-kube-controller-manager` service:

```
{
  kubectl config set-cluster hardway-cluster-e \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=labe-kube-controller-manager.kubeconfig

  kubectl config set-credentials system:labe-kube-controller-manager \
    --client-certificate=labe-kube-controller-manager.crt \
    --client-key=labe-kube-controller-manager.key \
    --embed-certs=true \
    --kubeconfig=labe-kube-controller-manager.kubeconfig

  kubectl config set-context default \
    --cluster=hardway-cluster-e \
    --user=system:labe-kube-controller-manager \
    --kubeconfig=labe-kube-controller-manager.kubeconfig

  kubectl config use-context default --kubeconfig=labe-kube-controller-manager.kubeconfig
}
```

Results:

```
labe-kube-controller-manager.kubeconfig
```

Reference docs for labe-kube-controller-manager [here](https://kubernetes.io/docs/reference/command-line-tools-reference/labe-kube-controller-manager/)

### The labe-kube-scheduler Kubernetes Configuration File labe-labe-kube-scheduler

Generate a kubeconfig file for the `labe-kube-scheduler` service:

```
{
  kubectl config set-cluster hardway-cluster-e \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=labe-kube-scheduler.kubeconfig

  kubectl config set-credentials system:labe-kube-scheduler \
    --client-certificate=labe-kube-scheduler.crt \
    --client-key=labe-kube-scheduler.key \
    --embed-certs=true \
    --kubeconfig=labe-kube-scheduler.kubeconfig

  kubectl config set-context default \
    --cluster=hardway-cluster-e \
    --user=system:labe-kube-scheduler \
    --kubeconfig=labe-kube-scheduler.kubeconfig

  kubectl config use-context default --kubeconfig=labe-kube-scheduler.kubeconfig
}
```

Results:

```
labe-kube-scheduler.kubeconfig
```

Reference docs for labe-kube-scheduler [here](https://kubernetes.io/docs/reference/command-line-tools-reference/labe-kube-scheduler/)

### The admin Kubernetes Configuration File

Generate a kubeconfig file for the `admin` user:

```
{
  kubectl config set-cluster hardway-cluster-e \
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
    --cluster=hardway-cluster-e \
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

Copy the appropriate `labe-kube-proxy` kubeconfig files to each worker instance:

```
for instance in labe-worker-1 labe-worker-2; do
  scp labe-kube-proxy.kubeconfig ${instance}:~/
done
```

Copy the appropriate `admin.kubeconfig`, `labe-kube-controller-manager` and `labe-kube-scheduler` kubeconfig files to each controller instance:

```
for instance in labe-master-1 labe-master-2; do
  scp admin.kubeconfig labe-kube-controller-manager.kubeconfig labe-kube-scheduler.kubeconfig ${instance}:~/
done
```

#### LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F
#### LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F labf
#### LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F

LOADBALANCER_ADDRESS=192.168.5.90
```

### The labf-kube-proxy Kubernetes Configuration File hardway-cluster-f labf

Generate a kubeconfig file for the `labf-kube-proxy` service:

```
{
  kubectl config set-cluster hardway-cluster-f \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://${LOADBALANCER_ADDRESS}:6443 \
    --kubeconfig=labf-kube-proxy.kubeconfig

  kubectl config set-credentials system:labf-kube-proxy \
    --client-certificate=labf-kube-proxy.crt \
    --client-key=labf-kube-proxy.key \
    --embed-certs=true \
    --kubeconfig=labf-kube-proxy.kubeconfig

  kubectl config set-context default \
    --cluster=hardway-cluster-f \
    --user=system:labf-kube-proxy \
    --kubeconfig=labf-kube-proxy.kubeconfig

  kubectl config use-context default --kubeconfig=labf-kube-proxy.kubeconfig
}
```

Results:

```
labf-kube-proxy.kubeconfig


Reference docs for labf-kube-proxy [here](https://kubernetes.io/docs/reference/command-line-tools-reference/labf-kube-proxy/)
```

### The labf-kube-controller-manager Kubernetes Configuration File

Generate a kubeconfig file for the `labf-kube-controller-manager` service:

```
{
  kubectl config set-cluster hardway-cluster-f \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=labf-kube-controller-manager.kubeconfig

  kubectl config set-credentials system:labf-kube-controller-manager \
    --client-certificate=labf-kube-controller-manager.crt \
    --client-key=labf-kube-controller-manager.key \
    --embed-certs=true \
    --kubeconfig=labf-kube-controller-manager.kubeconfig

  kubectl config set-context default \
    --cluster=hardway-cluster-f \
    --user=system:labf-kube-controller-manager \
    --kubeconfig=labf-kube-controller-manager.kubeconfig

  kubectl config use-context default --kubeconfig=labf-kube-controller-manager.kubeconfig
}
```

Results:

```
labf-kube-controller-manager.kubeconfig
```

Reference docs for labf-kube-controller-manager [here](https://kubernetes.io/docs/reference/command-line-tools-reference/labf-kube-controller-manager/)

### The labf-kube-scheduler Kubernetes Configuration File labf-labf-kube-scheduler

Generate a kubeconfig file for the `labf-kube-scheduler` service:

```
{
  kubectl config set-cluster hardway-cluster-f \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=labf-kube-scheduler.kubeconfig

  kubectl config set-credentials system:labf-kube-scheduler \
    --client-certificate=labf-kube-scheduler.crt \
    --client-key=labf-kube-scheduler.key \
    --embed-certs=true \
    --kubeconfig=labf-kube-scheduler.kubeconfig

  kubectl config set-context default \
    --cluster=hardway-cluster-f \
    --user=system:labf-kube-scheduler \
    --kubeconfig=labf-kube-scheduler.kubeconfig

  kubectl config use-context default --kubeconfig=labf-kube-scheduler.kubeconfig
}
```

Results:

```
labf-kube-scheduler.kubeconfig
```

Reference docs for labf-kube-scheduler [here](https://kubernetes.io/docs/reference/command-line-tools-reference/labf-kube-scheduler/)

### The admin Kubernetes Configuration File

Generate a kubeconfig file for the `admin` user:

```
{
  kubectl config set-cluster hardway-cluster-f \
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
    --cluster=hardway-cluster-f \
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

Copy the appropriate `labf-kube-proxy` kubeconfig files to each worker instance:

```
for instance in labf-worker-1 labf-worker-2; do
  scp labf-kube-proxy.kubeconfig ${instance}:~/
done
```

Copy the appropriate `admin.kubeconfig`, `labf-kube-controller-manager` and `labf-kube-scheduler` kubeconfig files to each controller instance:

```
for instance in labf-master-1 labf-master-2; do
  scp admin.kubeconfig labf-kube-controller-manager.kubeconfig labf-kube-scheduler.kubeconfig ${instance}:~/
done
```

