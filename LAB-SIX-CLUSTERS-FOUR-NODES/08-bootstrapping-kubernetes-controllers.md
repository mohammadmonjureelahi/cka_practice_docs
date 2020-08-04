#### LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A
#### LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A laba
#### LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A

# Bootstrapping the Kubernetes Control Plane

In this lab you will bootstrap the Kubernetes control plane across 2 compute instances and configure it for high availability. You will also create an external load balancer that exposes the Kubernetes API Servers to remote clients. The following components will be installed on each node: Kubernetes API Server, Scheduler, and Controller Manager.

## Prerequisites

The commands in this lab must be run on each controller instance: `laba-master-1`, and `laba-master-2`. Login to each controller instance using SSH Terminal. Example:

### Running commands in parallel with tmux

[tmux](https://github.com/tmux/tmux/wiki) can be used to run commands on multiple compute instances at the same time. See the [Running commands in parallel with tmux](01-prerequisites.md#running-commands-in-parallel-with-tmux) section in the Prerequisites lab.

## Provision the Kubernetes Control Plane

Create the Kubernetes configuration directory:

```
sudo mkdir -p /etc/kubernetes/config
```

### Download and Install the Kubernetes Controller Binaries

Download the official Kubernetes release binaries:

```
wget -q --show-progress --https-only --timestamping \
  "https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kube-apiserver" \
  "https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kube-controller-manager" \
  "https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kube-scheduler" \
  "https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kubectl"
```

Reference: https://kubernetes.io/docs/setup/release/#server-binaries

Install the Kubernetes binaries:

```
{
  chmod +x kube-apiserver kube-controller-manager kube-scheduler kubectl
  sudo mv kube-apiserver kube-controller-manager kube-scheduler kubectl /usr/local/bin/
}
```

### Configure the Kubernetes API Server

```
{
  sudo mkdir -p /var/lib/kubernetes/

  sudo cp ca.crt ca.key laba-kube-apiserver.crt laba-kube-apiserver.key \
    laba-service-account.key laba-service-account.crt \
    laba-etcd-server.key laba-etcd-server.crt \
    laba-encryption-config.yaml /var/lib/kubernetes/
}
```

The instance internal IP address will be used to advertise the API Server to members of the cluster. Retrieve the internal IP address for the current compute instance:

```
INTERNAL_IP=$(ip addr show enp0s8 | grep "inet " | awk '{print $2}' | cut -d / -f 1)
```

Verify it is set

```
echo $INTERNAL_IP
```

Create the `kube-apiserver.service` systemd unit file:

```
cat <<EOF | sudo tee /etc/systemd/system/kube-apiserver.service
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-apiserver \\
  --advertise-address=${INTERNAL_IP} \\
  --allow-privileged=true \\
  --apiserver-count=3 \\
  --audit-log-maxage=30 \\
  --audit-log-maxbackup=3 \\
  --audit-log-maxsize=100 \\
  --audit-log-path=/var/log/audit.log \\
  --authorization-mode=Node,RBAC \\
  --bind-address=0.0.0.0 \\
  --client-ca-file=/var/lib/kubernetes/ca.crt \\
  --enable-admission-plugins=NodeRestriction,ServiceAccount \\
  --enable-swagger-ui=true \\
  --enable-bootstrap-token-auth=true \\
  --etcd-cafile=/var/lib/kubernetes/ca.crt \\
  --etcd-certfile=/var/lib/kubernetes/laba-etcd-server.crt \\
  --etcd-keyfile=/var/lib/kubernetes/laba-etcd-server.key \\
  --etcd-servers=https://192.168.5.6:2379,https://192.168.5.7:2379 \\
  --event-ttl=1h \\
  --encryption-provider-config=/var/lib/kubernetes/laba-encryption-config.yaml \\
  --kubelet-certificate-authority=/var/lib/kubernetes/ca.crt \\
  --kubelet-client-certificate=/var/lib/kubernetes/laba-kube-apiserver.crt \\
  --kubelet-client-key=/var/lib/kubernetes/laba-kube-apiserver.key \\
  --kubelet-https=true \\
  --runtime-config=api/all=true \\
  --service-account-key-file=/var/lib/kubernetes/laba-service-account.crt \\
  --service-cluster-ip-range=10.96.0.0/24 \\
  --service-node-port-range=30000-32767 \\
  --tls-cert-file=/var/lib/kubernetes/laba-kube-apiserver.crt \\
  --tls-private-key-file=/var/lib/kubernetes/laba-kube-apiserver.key \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Configure the Kubernetes Controller Manager

Copy the `kube-controller-manager` kubeconfig into place:

```
sudo cp laba-kube-controller-manager.kubeconfig /var/lib/kubernetes/
```

Create the `kube-controller-manager.service` systemd unit file:

```
cat <<EOF | sudo tee /etc/systemd/system/kube-controller-manager.service
[Unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-controller-manager \\
  --address=0.0.0.0 \\
  --cluster-cidr=192.168.5.0/24 \\
  --cluster-name=kubernetes \\
  --cluster-signing-cert-file=/var/lib/kubernetes/ca.crt \\
  --cluster-signing-key-file=/var/lib/kubernetes/ca.key \\
  --kubeconfig=/var/lib/kubernetes/laba-kube-controller-manager.kubeconfig \\
  --leader-elect=true \\
  --root-ca-file=/var/lib/kubernetes/ca.crt \\
  --service-account-private-key-file=/var/lib/kubernetes/laba-service-account.key \\
  --service-cluster-ip-range=10.96.0.0/24 \\
  --use-service-account-credentials=true \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Configure the Kubernetes Scheduler

Copy the `kube-scheduler` kubeconfig into place:

```
sudo cp laba-kube-scheduler.kubeconfig /var/lib/kubernetes/
```

Create the `kube-scheduler.service` systemd unit file:

```
cat <<EOF | sudo tee /etc/systemd/system/kube-scheduler.service
[Unit]
Description=Kubernetes Scheduler
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-scheduler \\
  --kubeconfig=/var/lib/kubernetes/laba-kube-scheduler.kubeconfig \\
  --address=127.0.0.1 \\
  --leader-elect=true \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Start the Controller Services

```
{
  sudo systemctl daemon-reload
  sudo systemctl enable kube-apiserver kube-controller-manager kube-scheduler
  sudo systemctl start kube-apiserver kube-controller-manager kube-scheduler
}
```

> Allow up to 10 seconds for the Kubernetes API Server to fully initialize.


### Verification

```
kubectl get componentstatuses --kubeconfig admin.kubeconfig
```

```
NAME                 STATUS    MESSAGE              ERROR
controller-manager   Healthy   ok
scheduler            Healthy   ok
etcd-0               Healthy   {"health": "true"}
etcd-1               Healthy   {"health": "true"}
```

> Remember to run the above commands on each controller node: `laba-master-1`, and `laba-master-2`.

## The Kubernetes Frontend Load Balancer

In this section you will provision an external load balancer to front the Kubernetes API Servers. The `kubernetes-the-hard-way` static IP address will be attached to the resulting load balancer.


### Provision a Network Load Balancer

Login to `laba-loadbalancer` instance using SSH Terminal.

```
#Install HAProxy
loadbalancer# sudo apt-get update && sudo apt-get install -y haproxy

```

```
loadbalancer# cat <<EOF | sudo tee /etc/haproxy/haproxy.cfg 
frontend kubernetes
    bind 192.168.5.15:6443
    option tcplog
    mode tcp
    default_backend kubernetes-master-nodes

backend kubernetes-master-nodes
    mode tcp
    balance roundrobin
    option tcp-check
    server laba-master-1 192.168.5.6:6443 check fall 3 rise 2
    server laba-master-2 192.168.5.7:6443 check fall 3 rise 2
EOF
```

```
loadbalancer# sudo service haproxy restart
```

### Verification

Make a HTTP request for the Kubernetes version info:

```
curl  https://192.168.5.15:6443/version -k
```

> output

```
{
  "major": "1",
  "minor": "13",
  "gitVersion": "v1.13.0",
  "gitCommit": "ddf47ac13c1a9483ea035a79cd7c10005ff21a6d",
  "gitTreeState": "clean",
  "buildDate": "2018-12-03T20:56:12Z",
  "goVersion": "go1.11.2",
  "compiler": "gc",
  "platform": "linux/amd64"
}
```

Next: [Bootstrapping the Kubernetes Worker Nodes](09-bootstrapping-kubernetes-workers.md)

#### LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B
#### LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B labb
#### LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B

# Bootstrapping the Kubernetes Control Plane

In this lab you will bootstrap the Kubernetes control plane across 2 compute instances and configure it for high availability. You will also create an external load balancer that exposes the Kubernetes API Servers to remote clients. The following components will be installed on each node: Kubernetes API Server, Scheduler, and Controller Manager.

## Prerequisites

The commands in this lab must be run on each controller instance: `labb-master-1`, and `labb-master-2`. Login to each controller instance using SSH Terminal. Example:

### Running commands in parallel with tmux

[tmux](https://github.com/tmux/tmux/wiki) can be used to run commands on multiple compute instances at the same time. See the [Running commands in parallel with tmux](01-prerequisites.md#running-commands-in-parallel-with-tmux) section in the Prerequisites lab.

## Provision the Kubernetes Control Plane

Create the Kubernetes configuration directory:

```
sudo mkdir -p /etc/kubernetes/config
```

### Download and Install the Kubernetes Controller Binaries

Download the official Kubernetes release binaries:

```
wget -q --show-progress --https-only --timestamping \
  "https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kube-apiserver" \
  "https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kube-controller-manager" \
  "https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kube-scheduler" \
  "https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kubectl"
```

Reference: https://kubernetes.io/docs/setup/release/#server-binaries

Install the Kubernetes binaries:

```
{
  chmod +x kube-apiserver kube-controller-manager kube-scheduler kubectl
  sudo mv kube-apiserver kube-controller-manager kube-scheduler kubectl /usr/local/bin/
}
```

### Configure the Kubernetes API Server

```
{
  sudo mkdir -p /var/lib/kubernetes/

  sudo cp ca.crt ca.key labb-kube-apiserver.crt labb-kube-apiserver.key \
    labb-service-account.key labb-service-account.crt \
    labb-etcd-server.key labb-etcd-server.crt \
    labb-encryption-config.yaml /var/lib/kubernetes/
}
```

The instance internal IP address will be used to advertise the API Server to members of the cluster. Retrieve the internal IP address for the current compute instance:

```
INTERNAL_IP=$(ip addr show enp0s8 | grep "inet " | awk '{print $2}' | cut -d / -f 1)
```

Verify it is set

```
echo $INTERNAL_IP
```

Create the `kube-apiserver.service` systemd unit file:

```
cat <<EOF | sudo tee /etc/systemd/system/kube-apiserver.service
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-apiserver \\
  --advertise-address=${INTERNAL_IP} \\
  --allow-privileged=true \\
  --apiserver-count=3 \\
  --audit-log-maxage=30 \\
  --audit-log-maxbackup=3 \\
  --audit-log-maxsize=100 \\
  --audit-log-path=/var/log/audit.log \\
  --authorization-mode=Node,RBAC \\
  --bind-address=0.0.0.0 \\
  --client-ca-file=/var/lib/kubernetes/ca.crt \\
  --enable-admission-plugins=NodeRestriction,ServiceAccount \\
  --enable-swagger-ui=true \\
  --enable-bootstrap-token-auth=true \\
  --etcd-cafile=/var/lib/kubernetes/ca.crt \\
  --etcd-certfile=/var/lib/kubernetes/labb-etcd-server.crt \\
  --etcd-keyfile=/var/lib/kubernetes/labb-etcd-server.key \\
  --etcd-servers=https://192.168.5.21:2379,https://192.168.5.22:2379 \\
  --event-ttl=1h \\
  --encryption-provider-config=/var/lib/kubernetes/labb-encryption-config.yaml \\
  --kubelet-certificate-authority=/var/lib/kubernetes/ca.crt \\
  --kubelet-client-certificate=/var/lib/kubernetes/labb-kube-apiserver.crt \\
  --kubelet-client-key=/var/lib/kubernetes/labb-kube-apiserver.key \\
  --kubelet-https=true \\
  --runtime-config=api/all=true \\
  --service-account-key-file=/var/lib/kubernetes/labb-service-account.crt \\
  --service-cluster-ip-range=10.96.0.0/24 \\
  --service-node-port-range=30000-32767 \\
  --tls-cert-file=/var/lib/kubernetes/labb-kube-apiserver.crt \\
  --tls-private-key-file=/var/lib/kubernetes/labb-kube-apiserver.key \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Configure the Kubernetes Controller Manager

Copy the `kube-controller-manager` kubeconfig into place:

```
sudo cp labb-kube-controller-manager.kubeconfig /var/lib/kubernetes/
```

Create the `kube-controller-manager.service` systemd unit file:

```
cat <<EOF | sudo tee /etc/systemd/system/kube-controller-manager.service
[Unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-controller-manager \\
  --address=0.0.0.0 \\
  --cluster-cidr=192.168.5.0/24 \\
  --cluster-name=kubernetes \\
  --cluster-signing-cert-file=/var/lib/kubernetes/ca.crt \\
  --cluster-signing-key-file=/var/lib/kubernetes/ca.key \\
  --kubeconfig=/var/lib/kubernetes/labb-kube-controller-manager.kubeconfig \\
  --leader-elect=true \\
  --root-ca-file=/var/lib/kubernetes/ca.crt \\
  --service-account-private-key-file=/var/lib/kubernetes/labb-service-account.key \\
  --service-cluster-ip-range=10.96.0.0/24 \\
  --use-service-account-credentials=true \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Configure the Kubernetes Scheduler

Copy the `kube-scheduler` kubeconfig into place:

```
sudo cp labb-kube-scheduler.kubeconfig /var/lib/kubernetes/
```

Create the `kube-scheduler.service` systemd unit file:

```
cat <<EOF | sudo tee /etc/systemd/system/kube-scheduler.service
[Unit]
Description=Kubernetes Scheduler
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-scheduler \\
  --kubeconfig=/var/lib/kubernetes/labb-kube-scheduler.kubeconfig \\
  --address=127.0.0.1 \\
  --leader-elect=true \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Start the Controller Services

```
{
  sudo systemctl daemon-reload
  sudo systemctl enable kube-apiserver kube-controller-manager kube-scheduler
  sudo systemctl start kube-apiserver kube-controller-manager kube-scheduler
}
```

> Allow up to 10 seconds for the Kubernetes API Server to fully initialize.


### Verification

```
kubectl get componentstatuses --kubeconfig admin.kubeconfig
```

```
NAME                 STATUS    MESSAGE              ERROR
controller-manager   Healthy   ok
scheduler            Healthy   ok
etcd-0               Healthy   {"health": "true"}
etcd-1               Healthy   {"health": "true"}
```

> Remember to run the above commands on each controller node: `labb-master-1`, and `labb-master-2`.

## The Kubernetes Frontend Load Balancer

In this section you will provision an external load balancer to front the Kubernetes API Servers. The `hardway-cluster-b` static IP address will be attached to the resulting load balancer.


### Provision a Network Load Balancer

Login to `labb-loadbalancer` instance using SSH Terminal.

```
#Install HAProxy
loadbalancer# sudo apt-get update && sudo apt-get install -y haproxy

```

```
cat <<EOF | sudo tee /etc/haproxy/haproxy.cfg 
frontend kubernetes
    bind 192.168.5.30:6443
    option tcplog
    mode tcp
    default_backend kubernetes-master-nodes

backend kubernetes-master-nodes
    mode tcp
    balance roundrobin
    option tcp-check
    server labb-master-1 192.168.5.21:6443 check fall 3 rise 2
    server labb-master-2 192.168.5.22:6443 check fall 3 rise 2
EOF
```

```
loadbalancer# sudo service haproxy restart
```

### Verification

Make a HTTP request for the Kubernetes version info:

```
curl  https://192.168.5.30:6443/version -k
```

> output

```
{
  "major": "1",
  "minor": "13",
  "gitVersion": "v1.13.0",
  "gitCommit": "ddf47ac13c1a9483ea035a79cd7c10005ff21a6d",
  "gitTreeState": "clean",
  "buildDate": "2018-12-03T20:56:12Z",
  "goVersion": "go1.11.2",
  "compiler": "gc",
  "platform": "linux/amd64"
}
```

#### LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C
#### LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C labc
#### LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C

# Bootstrapping the Kubernetes Control Plane

In this lab you will bootstrap the Kubernetes control plane across 2 compute instances and configure it for high availability. You will also create an external load balancer that exposes the Kubernetes API Servers to remote clients. The following components will be installed on each node: Kubernetes API Server, Scheduler, and Controller Manager.

## Prerequisites

The commands in this lab must be run on each controller instance: `labc-master-1`, and `labc-master-2`. Login to each controller instance using SSH Terminal. Example:

### Running commands in parallel with tmux

[tmux](https://github.com/tmux/tmux/wiki) can be used to run commands on multiple compute instances at the same time. See the [Running commands in parallel with tmux](01-prerequisites.md#running-commands-in-parallel-with-tmux) section in the Prerequisites lab.

## Provision the Kubernetes Control Plane

Create the Kubernetes configuration directory:

```
sudo mkdir -p /etc/kubernetes/config
```

### Download and Install the Kubernetes Controller Binaries

Download the official Kubernetes release binaries:

```
wget -q --show-progress --https-only --timestamping \
  "https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kube-apiserver" \
  "https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kube-controller-manager" \
  "https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kube-scheduler" \
  "https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kubectl"
```

Reference: https://kubernetes.io/docs/setup/release/#server-binaries

Install the Kubernetes binaries:

```
{
  chmod +x kube-apiserver kube-controller-manager kube-scheduler kubectl
  sudo mv kube-apiserver kube-controller-manager kube-scheduler kubectl /usr/local/bin/
}
```

### Configure the Kubernetes API Server

```
{
  sudo mkdir -p /var/lib/kubernetes/

  sudo cp ca.crt ca.key labc-kube-apiserver.crt labc-kube-apiserver.key \
    labc-service-account.key labc-service-account.crt \
    labc-etcd-server.key labc-etcd-server.crt \
    labc-encryption-config.yaml /var/lib/kubernetes/
}
```

The instance internal IP address will be used to advertise the API Server to members of the cluster. Retrieve the internal IP address for the current compute instance:

```
INTERNAL_IP=$(ip addr show enp0s8 | grep "inet " | awk '{print $2}' | cut -d / -f 1)
```

Verify it is set

```
echo $INTERNAL_IP
```

Create the `kube-apiserver.service` systemd unit file:

```
cat <<EOF | sudo tee /etc/systemd/system/kube-apiserver.service
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-apiserver \\
  --advertise-address=${INTERNAL_IP} \\
  --allow-privileged=true \\
  --apiserver-count=3 \\
  --audit-log-maxage=30 \\
  --audit-log-maxbackup=3 \\
  --audit-log-maxsize=100 \\
  --audit-log-path=/var/log/audit.log \\
  --authorization-mode=Node,RBAC \\
  --bind-address=0.0.0.0 \\
  --client-ca-file=/var/lib/kubernetes/ca.crt \\
  --enable-admission-plugins=NodeRestriction,ServiceAccount \\
  --enable-swagger-ui=true \\
  --enable-bootstrap-token-auth=true \\
  --etcd-cafile=/var/lib/kubernetes/ca.crt \\
  --etcd-certfile=/var/lib/kubernetes/labc-etcd-server.crt \\
  --etcd-keyfile=/var/lib/kubernetes/labc-etcd-server.key \\
  --etcd-servers=https://192.168.5.36:2379,https://192.168.5.37:2379 \\
  --event-ttl=1h \\
  --encryption-provider-config=/var/lib/kubernetes/labc-encryption-config.yaml \\
  --kubelet-certificate-authority=/var/lib/kubernetes/ca.crt \\
  --kubelet-client-certificate=/var/lib/kubernetes/labc-kube-apiserver.crt \\
  --kubelet-client-key=/var/lib/kubernetes/labc-kube-apiserver.key \\
  --kubelet-https=true \\
  --runtime-config=api/all=true \\
  --service-account-key-file=/var/lib/kubernetes/labc-service-account.crt \\
  --service-cluster-ip-range=10.96.0.0/24 \\
  --service-node-port-range=30000-32767 \\
  --tls-cert-file=/var/lib/kubernetes/labc-kube-apiserver.crt \\
  --tls-private-key-file=/var/lib/kubernetes/labc-kube-apiserver.key \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Configure the Kubernetes Controller Manager

Copy the `kube-controller-manager` kubeconfig into place:

```
sudo cp labc-kube-controller-manager.kubeconfig /var/lib/kubernetes/
```

Create the `kube-controller-manager.service` systemd unit file:

```
cat <<EOF | sudo tee /etc/systemd/system/kube-controller-manager.service
[Unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-controller-manager \\
  --address=0.0.0.0 \\
  --cluster-cidr=192.168.5.0/24 \\
  --cluster-name=kubernetes \\
  --cluster-signing-cert-file=/var/lib/kubernetes/ca.crt \\
  --cluster-signing-key-file=/var/lib/kubernetes/ca.key \\
  --kubeconfig=/var/lib/kubernetes/labc-kube-controller-manager.kubeconfig \\
  --leader-elect=true \\
  --root-ca-file=/var/lib/kubernetes/ca.crt \\
  --service-account-private-key-file=/var/lib/kubernetes/labc-service-account.key \\
  --service-cluster-ip-range=10.96.0.0/24 \\
  --use-service-account-credentials=true \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Configure the Kubernetes Scheduler

Copy the `kube-scheduler` kubeconfig into place:

```
sudo cp labc-kube-scheduler.kubeconfig /var/lib/kubernetes/
```

Create the `kube-scheduler.service` systemd unit file:

```
cat <<EOF | sudo tee /etc/systemd/system/kube-scheduler.service
[Unit]
Description=Kubernetes Scheduler
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-scheduler \\
  --kubeconfig=/var/lib/kubernetes/labc-kube-scheduler.kubeconfig \\
  --address=127.0.0.1 \\
  --leader-elect=true \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Start the Controller Services

```
{
  sudo systemctl daemon-reload
  sudo systemctl enable kube-apiserver kube-controller-manager kube-scheduler
  sudo systemctl start kube-apiserver kube-controller-manager kube-scheduler
}
```

> Allow up to 10 seconds for the Kubernetes API Server to fully initialize.


### Verification

```
kubectl get componentstatuses --kubeconfig admin.kubeconfig
```

```
NAME                 STATUS    MESSAGE              ERROR
controller-manager   Healthy   ok
scheduler            Healthy   ok
etcd-0               Healthy   {"health": "true"}
etcd-1               Healthy   {"health": "true"}
```

> Remember to run the above commands on each controller node: `labc-master-1`, and `labc-master-2`.

## The Kubernetes Frontend Load Balancer

In this section you will provision an external load balancer to front the Kubernetes API Servers. The `kubernetes-the-hard-way` static IP address will be attached to the resulting load balancer.


### Provision a Network Load Balancer

Login to `labc-loadbalancer` instance using SSH Terminal.

```
#Install HAProxy
loadbalancer# sudo apt-get update && sudo apt-get install -y haproxy

```

```
loadbalancer# cat <<EOF | sudo tee /etc/haproxy/haproxy.cfg 
frontend kubernetes
    bind 192.168.5.45:6443
    option tcplog
    mode tcp
    default_backend kubernetes-master-nodes

backend kubernetes-master-nodes
    mode tcp
    balance roundrobin
    option tcp-check
    server labc-master-1 192.168.5.36:6443 check fall 3 rise 2
    server labc-master-2 192.168.5.37:6443 check fall 3 rise 2
EOF
```

```
loadbalancer# sudo service haproxy restart
```

### Verification

Make a HTTP request for the Kubernetes version info:

```
curl  https://192.168.5.45:6443/version -k
```

> output

```
{
  "major": "1",
  "minor": "13",
  "gitVersion": "v1.13.0",
  "gitCommit": "ddf47ac13c1a9483ea035a79cd7c10005ff21a6d",
  "gitTreeState": "clean",
  "buildDate": "2018-12-03T20:56:12Z",
  "goVersion": "go1.11.2",
  "compiler": "gc",
  "platform": "linux/amd64"
}
```


#### LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D
#### LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D labd
#### LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D

# Bootstrapping the Kubernetes Control Plane

In this lab you will bootstrap the Kubernetes control plane across 2 compute instances and configure it for high availability. You will also create an external load balancer that exposes the Kubernetes API Servers to remote clients. The following components will be installed on each node: Kubernetes API Server, Scheduler, and Controller Manager.

## Prerequisites

The commands in this lab must be run on each controller instance: `labd-master-1`, and `labd-master-2`. Login to each controller instance using SSH Terminal. Example:

### Running commands in parallel with tmux

[tmux](https://github.com/tmux/tmux/wiki) can be used to run commands on multiple compute instances at the same time. See the [Running commands in parallel with tmux](01-prerequisites.md#running-commands-in-parallel-with-tmux) section in the Prerequisites lab.

## Provision the Kubernetes Control Plane

Create the Kubernetes configuration directory:

```
sudo mkdir -p /etc/kubernetes/config
```

### Download and Install the Kubernetes Controller Binaries

Download the official Kubernetes release binaries:

```
wget -q --show-progress --https-only --timestamping \
  "https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kube-apiserver" \
  "https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kube-controller-manager" \
  "https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kube-scheduler" \
  "https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kubectl"
```

Reference: https://kubernetes.io/docs/setup/release/#server-binaries

Install the Kubernetes binaries:

```
{
  chmod +x kube-apiserver kube-controller-manager kube-scheduler kubectl
  sudo mv kube-apiserver kube-controller-manager kube-scheduler kubectl /usr/local/bin/
}
```

### Configure the Kubernetes API Server

```
{
  sudo mkdir -p /var/lib/kubernetes/

  sudo cp ca.crt ca.key labd-kube-apiserver.crt labd-kube-apiserver.key \
    labd-service-account.key labd-service-account.crt \
    labd-etcd-server.key labd-etcd-server.crt \
    labd-encryption-config.yaml /var/lib/kubernetes/
}
```

The instance internal IP address will be used to advertise the API Server to members of the cluster. Retrieve the internal IP address for the current compute instance:

```
INTERNAL_IP=$(ip addr show enp0s8 | grep "inet " | awk '{print $2}' | cut -d / -f 1)
```

Verify it is set

```
echo $INTERNAL_IP
```

Create the `kube-apiserver.service` systemd unit file:

```
cat <<EOF | sudo tee /etc/systemd/system/kube-apiserver.service
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-apiserver \\
  --advertise-address=${INTERNAL_IP} \\
  --allow-privileged=true \\
  --apiserver-count=3 \\
  --audit-log-maxage=30 \\
  --audit-log-maxbackup=3 \\
  --audit-log-maxsize=100 \\
  --audit-log-path=/var/log/audit.log \\
  --authorization-mode=Node,RBAC \\
  --bind-address=0.0.0.0 \\
  --client-ca-file=/var/lib/kubernetes/ca.crt \\
  --enable-admission-plugins=NodeRestriction,ServiceAccount \\
  --enable-swagger-ui=true \\
  --enable-bootstrap-token-auth=true \\
  --etcd-cafile=/var/lib/kubernetes/ca.crt \\
  --etcd-certfile=/var/lib/kubernetes/labd-etcd-server.crt \\
  --etcd-keyfile=/var/lib/kubernetes/labd-etcd-server.key \\
  --etcd-servers=https://192.168.5.51:2379,https://192.168.5.52:2379 \\
  --event-ttl=1h \\
  --encryption-provider-config=/var/lib/kubernetes/labd-encryption-config.yaml \\
  --kubelet-certificate-authority=/var/lib/kubernetes/ca.crt \\
  --kubelet-client-certificate=/var/lib/kubernetes/labd-kube-apiserver.crt \\
  --kubelet-client-key=/var/lib/kubernetes/labd-kube-apiserver.key \\
  --kubelet-https=true \\
  --runtime-config=api/all=true \\
  --service-account-key-file=/var/lib/kubernetes/labd-service-account.crt \\
  --service-cluster-ip-range=10.96.0.0/24 \\
  --service-node-port-range=30000-32767 \\
  --tls-cert-file=/var/lib/kubernetes/labd-kube-apiserver.crt \\
  --tls-private-key-file=/var/lib/kubernetes/labd-kube-apiserver.key \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Configure the Kubernetes Controller Manager

Copy the `kube-controller-manager` kubeconfig into place:

```
sudo cp labd-kube-controller-manager.kubeconfig /var/lib/kubernetes/
```

Create the `kube-controller-manager.service` systemd unit file:

```
cat <<EOF | sudo tee /etc/systemd/system/kube-controller-manager.service
[Unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-controller-manager \\
  --address=0.0.0.0 \\
  --cluster-cidr=192.168.5.0/24 \\
  --cluster-name=kubernetes \\
  --cluster-signing-cert-file=/var/lib/kubernetes/ca.crt \\
  --cluster-signing-key-file=/var/lib/kubernetes/ca.key \\
  --kubeconfig=/var/lib/kubernetes/labd-kube-controller-manager.kubeconfig \\
  --leader-elect=true \\
  --root-ca-file=/var/lib/kubernetes/ca.crt \\
  --service-account-private-key-file=/var/lib/kubernetes/labd-service-account.key \\
  --service-cluster-ip-range=10.96.0.0/24 \\
  --use-service-account-credentials=true \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Configure the Kubernetes Scheduler

Copy the `kube-scheduler` kubeconfig into place:

```
sudo cp labd-kube-scheduler.kubeconfig /var/lib/kubernetes/
```

Create the `kube-scheduler.service` systemd unit file:

```
cat <<EOF | sudo tee /etc/systemd/system/kube-scheduler.service
[Unit]
Description=Kubernetes Scheduler
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-scheduler \\
  --kubeconfig=/var/lib/kubernetes/labd-kube-scheduler.kubeconfig \\
  --address=127.0.0.1 \\
  --leader-elect=true \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Start the Controller Services

```
{
  sudo systemctl daemon-reload
  sudo systemctl enable kube-apiserver kube-controller-manager kube-scheduler
  sudo systemctl start kube-apiserver kube-controller-manager kube-scheduler
}
```

> Allow up to 10 seconds for the Kubernetes API Server to fully initialize.


### Verification

```
kubectl get componentstatuses --kubeconfig admin.kubeconfig
```

```
NAME                 STATUS    MESSAGE              ERROR
controller-manager   Healthy   ok
scheduler            Healthy   ok
etcd-0               Healthy   {"health": "true"}
etcd-1               Healthy   {"health": "true"}
```

> Remember to run the above commands on each controller node: `labd-master-1`, and `labd-master-2`.

## The Kubernetes Frontend Load Balancer

In this section you will provision an external load balancer to front the Kubernetes API Servers. The `kubernetes-the-hard-way` static IP address will be attached to the resulting load balancer.


### Provision a Network Load Balancer

Login to `labd-loadbalancer` instance using SSH Terminal.

```
#Install HAProxy
loadbalancer# sudo apt-get update && sudo apt-get install -y haproxy

```

```
loadbalancer# cat <<EOF | sudo tee /etc/haproxy/haproxy.cfg 
frontend kubernetes
    bind 192.168.5.60:6443
    option tcplog
    mode tcp
    default_backend kubernetes-master-nodes

backend kubernetes-master-nodes
    mode tcp
    balance roundrobin
    option tcp-check
    server labd-master-1 192.168.5.51:6443 check fall 3 rise 2
    server labd-master-2 192.168.5.52:6443 check fall 3 rise 2
EOF
```

```
loadbalancer# sudo service haproxy restart
```

### Verification

Make a HTTP request for the Kubernetes version info:

```
curl  https://192.168.5.60:6443/version -k
```

> output

```
{
  "major": "1",
  "minor": "13",
  "gitVersion": "v1.13.0",
  "gitCommit": "ddf47ac13c1a9483ea035a79cd7c10005ff21a6d",
  "gitTreeState": "clean",
  "buildDate": "2018-12-03T20:56:12Z",
  "goVersion": "go1.11.2",
  "compiler": "gc",
  "platform": "linux/amd64"
}
```

#### LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E
#### LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E labe
#### LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E

# Bootstrapping the Kubernetes Control Plane

In this lab you will bootstrap the Kubernetes control plane across 2 compute instances and configure it for high availability. You will also create an external load balancer that exposes the Kubernetes API Servers to remote clients. The following components will be installed on each node: Kubernetes API Server, Scheduler, and Controller Manager.

## Prerequisites

The commands in this lab must be run on each controller instance: `labe-master-1`, and `labe-master-2`. Login to each controller instance using SSH Terminal. Example:

### Running commands in parallel with tmux

[tmux](https://github.com/tmux/tmux/wiki) can be used to run commands on multiple compute instances at the same time. See the [Running commands in parallel with tmux](01-prerequisites.md#running-commands-in-parallel-with-tmux) section in the Prerequisites lab.

## Provision the Kubernetes Control Plane

Create the Kubernetes configuration directory:

```
sudo mkdir -p /etc/kubernetes/config
```

### Download and Install the Kubernetes Controller Binaries

Download the official Kubernetes release binaries:

```
wget -q --show-progress --https-only --timestamping \
  "https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kube-apiserver" \
  "https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kube-controller-manager" \
  "https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kube-scheduler" \
  "https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kubectl"
```

Reference: https://kubernetes.io/docs/setup/release/#server-binaries

Install the Kubernetes binaries:

```
{
  chmod +x kube-apiserver kube-controller-manager kube-scheduler kubectl
  sudo mv kube-apiserver kube-controller-manager kube-scheduler kubectl /usr/local/bin/
}
```

### Configure the Kubernetes API Server

```
{
  sudo mkdir -p /var/lib/kubernetes/

  sudo cp ca.crt ca.key labe-kube-apiserver.crt labe-kube-apiserver.key \
    labe-service-account.key labe-service-account.crt \
    labe-etcd-server.key labe-etcd-server.crt \
    labe-encryption-config.yaml /var/lib/kubernetes/
}
```

The instance internal IP address will be used to advertise the API Server to members of the cluster. Retrieve the internal IP address for the current compute instance:

```
INTERNAL_IP=$(ip addr show enp0s8 | grep "inet " | awk '{print $2}' | cut -d / -f 1)
```

Verify it is set

```
echo $INTERNAL_IP
```

Create the `kube-apiserver.service` systemd unit file:

```
cat <<EOF | sudo tee /etc/systemd/system/kube-apiserver.service
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-apiserver \\
  --advertise-address=${INTERNAL_IP} \\
  --allow-privileged=true \\
  --apiserver-count=3 \\
  --audit-log-maxage=30 \\
  --audit-log-maxbackup=3 \\
  --audit-log-maxsize=100 \\
  --audit-log-path=/var/log/audit.log \\
  --authorization-mode=Node,RBAC \\
  --bind-address=0.0.0.0 \\
  --client-ca-file=/var/lib/kubernetes/ca.crt \\
  --enable-admission-plugins=NodeRestriction,ServiceAccount \\
  --enable-swagger-ui=true \\
  --enable-bootstrap-token-auth=true \\
  --etcd-cafile=/var/lib/kubernetes/ca.crt \\
  --etcd-certfile=/var/lib/kubernetes/labe-etcd-server.crt \\
  --etcd-keyfile=/var/lib/kubernetes/labe-etcd-server.key \\
  --etcd-servers=https://192.168.5.66:2379,https://192.168.5.67:2379 \\
  --event-ttl=1h \\
  --encryption-provider-config=/var/lib/kubernetes/labe-encryption-config.yaml \\
  --kubelet-certificate-authority=/var/lib/kubernetes/ca.crt \\
  --kubelet-client-certificate=/var/lib/kubernetes/labe-kube-apiserver.crt \\
  --kubelet-client-key=/var/lib/kubernetes/labe-kube-apiserver.key \\
  --kubelet-https=true \\
  --runtime-config=api/all=true \\
  --service-account-key-file=/var/lib/kubernetes/labe-service-account.crt \\
  --service-cluster-ip-range=10.96.0.0/24 \\
  --service-node-port-range=30000-32767 \\
  --tls-cert-file=/var/lib/kubernetes/labe-kube-apiserver.crt \\
  --tls-private-key-file=/var/lib/kubernetes/labe-kube-apiserver.key \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Configure the Kubernetes Controller Manager

Copy the `kube-controller-manager` kubeconfig into place:

```
sudo cp labe-kube-controller-manager.kubeconfig /var/lib/kubernetes/
```

Create the `kube-controller-manager.service` systemd unit file:

```
cat <<EOF | sudo tee /etc/systemd/system/kube-controller-manager.service
[Unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-controller-manager \\
  --address=0.0.0.0 \\
  --cluster-cidr=192.168.5.0/24 \\
  --cluster-name=kubernetes \\
  --cluster-signing-cert-file=/var/lib/kubernetes/ca.crt \\
  --cluster-signing-key-file=/var/lib/kubernetes/ca.key \\
  --kubeconfig=/var/lib/kubernetes/labe-kube-controller-manager.kubeconfig \\
  --leader-elect=true \\
  --root-ca-file=/var/lib/kubernetes/ca.crt \\
  --service-account-private-key-file=/var/lib/kubernetes/labe-service-account.key \\
  --service-cluster-ip-range=10.96.0.0/24 \\
  --use-service-account-credentials=true \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Configure the Kubernetes Scheduler

Copy the `kube-scheduler` kubeconfig into place:

```
sudo cp labe-kube-scheduler.kubeconfig /var/lib/kubernetes/
```

Create the `kube-scheduler.service` systemd unit file:

```
cat <<EOF | sudo tee /etc/systemd/system/kube-scheduler.service
[Unit]
Description=Kubernetes Scheduler
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-scheduler \\
  --kubeconfig=/var/lib/kubernetes/labe-kube-scheduler.kubeconfig \\
  --address=127.0.0.1 \\
  --leader-elect=true \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Start the Controller Services

```
{
  sudo systemctl daemon-reload
  sudo systemctl enable kube-apiserver kube-controller-manager kube-scheduler
  sudo systemctl start kube-apiserver kube-controller-manager kube-scheduler
}
```

> Allow up to 10 seconds for the Kubernetes API Server to fully initialize.


### Verification

```
kubectl get componentstatuses --kubeconfig admin.kubeconfig
```

```
NAME                 STATUS    MESSAGE              ERROR
controller-manager   Healthy   ok
scheduler            Healthy   ok
etcd-0               Healthy   {"health": "true"}
etcd-1               Healthy   {"health": "true"}
```

> Remember to run the above commands on each controller node: `labe-master-1`, and `labe-master-2`.

## The Kubernetes Frontend Load Balancer

In this section you will provision an external load balancer to front the Kubernetes API Servers. The `kubernetes-the-hard-way` static IP address will be attached to the resulting load balancer.


### Provision a Network Load Balancer

Login to `labe-loadbalancer` instance using SSH Terminal.

```
#Install HAProxy
loadbalancer# sudo apt-get update && sudo apt-get install -y haproxy

```

```
loadbalancer# cat <<EOF | sudo tee /etc/haproxy/haproxy.cfg 
frontend kubernetes
    bind 192.168.5.75:6443
    option tcplog
    mode tcp
    default_backend kubernetes-master-nodes

backend kubernetes-master-nodes
    mode tcp
    balance roundrobin
    option tcp-check
    server labe-master-1 192.168.5.66:6443 check fall 3 rise 2
    server labe-master-2 192.168.5.67:6443 check fall 3 rise 2
EOF
```

```
loadbalancer# sudo service haproxy restart
```

### Verification

Make a HTTP request for the Kubernetes version info:

```
curl  https://192.168.5.75:6443/version -k
```

> output

```
{
  "major": "1",
  "minor": "13",
  "gitVersion": "v1.13.0",
  "gitCommit": "ddf47ac13c1a9483ea035a79cd7c10005ff21a6d",
  "gitTreeState": "clean",
  "buildDate": "2018-12-03T20:56:12Z",
  "goVersion": "go1.11.2",
  "compiler": "gc",
  "platform": "linux/amd64"
}
```


#### LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F
#### LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F labf
#### LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F

# Bootstrapping the Kubernetes Control Plane

In this lab you will bootstrap the Kubernetes control plane across 2 compute instances and configure it for high availability. You will also create an external load balancer that exposes the Kubernetes API Servers to remote clients. The following components will be installed on each node: Kubernetes API Server, Scheduler, and Controller Manager.

## Prerequisites

The commands in this lab must be run on each controller instance: `labf-master-1`, and `labf-master-2`. Login to each controller instance using SSH Terminal. Example:

### Running commands in parallel with tmux

[tmux](https://github.com/tmux/tmux/wiki) can be used to run commands on multiple compute instances at the same time. See the [Running commands in parallel with tmux](01-prerequisites.md#running-commands-in-parallel-with-tmux) section in the Prerequisites lab.

## Provision the Kubernetes Control Plane

Create the Kubernetes configuration directory:

```
sudo mkdir -p /etc/kubernetes/config
```

### Download and Install the Kubernetes Controller Binaries

Download the official Kubernetes release binaries:

```
wget -q --show-progress --https-only --timestamping \
  "https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kube-apiserver" \
  "https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kube-controller-manager" \
  "https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kube-scheduler" \
  "https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kubectl"
```

Reference: https://kubernetes.io/docs/setup/release/#server-binaries

Install the Kubernetes binaries:

```
{
  chmod +x kube-apiserver kube-controller-manager kube-scheduler kubectl
  sudo mv kube-apiserver kube-controller-manager kube-scheduler kubectl /usr/local/bin/
}
```

### Configure the Kubernetes API Server

```
{
  sudo mkdir -p /var/lib/kubernetes/

  sudo cp ca.crt ca.key labf-kube-apiserver.crt labf-kube-apiserver.key \
    labf-service-account.key labf-service-account.crt \
    labf-etcd-server.key labf-etcd-server.crt \
    labf-encryption-config.yaml /var/lib/kubernetes/
}
```

The instance internal IP address will be used to advertise the API Server to members of the cluster. Retrieve the internal IP address for the current compute instance:

```
INTERNAL_IP=$(ip addr show enp0s8 | grep "inet " | awk '{print $2}' | cut -d / -f 1)
```

Verify it is set

```
echo $INTERNAL_IP
```

Create the `kube-apiserver.service` systemd unit file:

```
cat <<EOF | sudo tee /etc/systemd/system/kube-apiserver.service
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-apiserver \\
  --advertise-address=${INTERNAL_IP} \\
  --allow-privileged=true \\
  --apiserver-count=3 \\
  --audit-log-maxage=30 \\
  --audit-log-maxbackup=3 \\
  --audit-log-maxsize=100 \\
  --audit-log-path=/var/log/audit.log \\
  --authorization-mode=Node,RBAC \\
  --bind-address=0.0.0.0 \\
  --client-ca-file=/var/lib/kubernetes/ca.crt \\
  --enable-admission-plugins=NodeRestriction,ServiceAccount \\
  --enable-swagger-ui=true \\
  --enable-bootstrap-token-auth=true \\
  --etcd-cafile=/var/lib/kubernetes/ca.crt \\
  --etcd-certfile=/var/lib/kubernetes/labf-etcd-server.crt \\
  --etcd-keyfile=/var/lib/kubernetes/labf-etcd-server.key \\
  --etcd-servers=https://192.168.5.81:2379,https://192.168.5.82:2379 \\
  --event-ttl=1h \\
  --encryption-provider-config=/var/lib/kubernetes/labf-encryption-config.yaml \\
  --kubelet-certificate-authority=/var/lib/kubernetes/ca.crt \\
  --kubelet-client-certificate=/var/lib/kubernetes/labf-kube-apiserver.crt \\
  --kubelet-client-key=/var/lib/kubernetes/labf-kube-apiserver.key \\
  --kubelet-https=true \\
  --runtime-config=api/all=true \\
  --service-account-key-file=/var/lib/kubernetes/labf-service-account.crt \\
  --service-cluster-ip-range=10.96.0.0/24 \\
  --service-node-port-range=30000-32767 \\
  --tls-cert-file=/var/lib/kubernetes/labf-kube-apiserver.crt \\
  --tls-private-key-file=/var/lib/kubernetes/labf-kube-apiserver.key \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Configure the Kubernetes Controller Manager

Copy the `kube-controller-manager` kubeconfig into place:

```
sudo cp labf-kube-controller-manager.kubeconfig /var/lib/kubernetes/
```

Create the `kube-controller-manager.service` systemd unit file:

```
cat <<EOF | sudo tee /etc/systemd/system/kube-controller-manager.service
[Unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-controller-manager \\
  --address=0.0.0.0 \\
  --cluster-cidr=192.168.5.0/24 \\
  --cluster-name=kubernetes \\
  --cluster-signing-cert-file=/var/lib/kubernetes/ca.crt \\
  --cluster-signing-key-file=/var/lib/kubernetes/ca.key \\
  --kubeconfig=/var/lib/kubernetes/labf-kube-controller-manager.kubeconfig \\
  --leader-elect=true \\
  --root-ca-file=/var/lib/kubernetes/ca.crt \\
  --service-account-private-key-file=/var/lib/kubernetes/labf-service-account.key \\
  --service-cluster-ip-range=10.96.0.0/24 \\
  --use-service-account-credentials=true \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Configure the Kubernetes Scheduler

Copy the `kube-scheduler` kubeconfig into place:

```
sudo cp labf-kube-scheduler.kubeconfig /var/lib/kubernetes/
```

Create the `kube-scheduler.service` systemd unit file:

```
cat <<EOF | sudo tee /etc/systemd/system/kube-scheduler.service
[Unit]
Description=Kubernetes Scheduler
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-scheduler \\
  --kubeconfig=/var/lib/kubernetes/labf-kube-scheduler.kubeconfig \\
  --address=127.0.0.1 \\
  --leader-elect=true \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Start the Controller Services

```
{
  sudo systemctl daemon-reload
  sudo systemctl enable kube-apiserver kube-controller-manager kube-scheduler
  sudo systemctl start kube-apiserver kube-controller-manager kube-scheduler
}
```

> Allow up to 10 seconds for the Kubernetes API Server to fully initialize.


### Verification

```
kubectl get componentstatuses --kubeconfig admin.kubeconfig
```

```
NAME                 STATUS    MESSAGE              ERROR
controller-manager   Healthy   ok
scheduler            Healthy   ok
etcd-0               Healthy   {"health": "true"}
etcd-1               Healthy   {"health": "true"}
```

> Remember to run the above commands on each controller node: `labf-master-1`, and `labf-master-2`.

## The Kubernetes Frontend Load Balancer

In this section you will provision an external load balancer to front the Kubernetes API Servers. The `kubernetes-the-hard-way` static IP address will be attached to the resulting load balancer.


### Provision a Network Load Balancer

Login to `labf-loadbalancer` instance using SSH Terminal.

```
#Install HAProxy
loadbalancer# sudo apt-get update && sudo apt-get install -y haproxy

```

```
loadbalancer# cat <<EOF | sudo tee /etc/haproxy/haproxy.cfg 
frontend kubernetes
    bind 192.168.5.90:6443
    option tcplog
    mode tcp
    default_backend kubernetes-master-nodes

backend kubernetes-master-nodes
    mode tcp
    balance roundrobin
    option tcp-check
    server labf-master-1 192.168.5.81:6443 check fall 3 rise 2
    server labf-master-2 192.168.5.82:6443 check fall 3 rise 2
EOF
```

```
loadbalancer# sudo service haproxy restart
```

### Verification

Make a HTTP request for the Kubernetes version info:

```
curl  https://192.168.5.90:6443/version -k
```

> output

```
{
  "major": "1",
  "minor": "13",
  "gitVersion": "v1.13.0",
  "gitCommit": "ddf47ac13c1a9483ea035a79cd7c10005ff21a6d",
  "gitTreeState": "clean",
  "buildDate": "2018-12-03T20:56:12Z",
  "goVersion": "go1.11.2",
  "compiler": "gc",
  "platform": "linux/amd64"
}
```

