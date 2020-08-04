#### LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A
#### LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A laba
#### LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A

# Bootstrapping the etcd Cluster

Kubernetes components are stateless and store cluster state in [etcd](https://github.com/coreos/etcd). In this lab you will bootstrap a two node etcd cluster and configure it for high availability and secure remote access.

## Prerequisites

The commands in this lab must be run on each controller instance: `laba-master-1`, and `laba-master-2`. Login to each of these using an SSH terminal.

### Running commands in parallel with tmux

[tmux](https://github.com/tmux/tmux/wiki) can be used to run commands on multiple compute instances at the same time. See the [Running commands in parallel with tmux](01-prerequisites.md#running-commands-in-parallel-with-tmux) section in the Prerequisites lab.

## Bootstrapping an etcd Cluster Member

### Download and Install the etcd Binaries

Download the official etcd release binaries from the [coreos/etcd](https://github.com/coreos/etcd) GitHub project:

```
wget -q --show-progress --https-only --timestamping \
  "https://github.com/coreos/etcd/releases/download/v3.4.9/etcd-v3.4.9-linux-amd64.tar.gz"
```

Extract and install the `etcd` server and the `etcdctl` command line utility:

```
{
  tar -xvf etcd-v3.4.9-linux-amd64.tar.gz
  sudo mv etcd-v3.4.9-linux-amd64/etcd* /usr/local/bin/
}
```

### Configure the etcd Server

```
{
  sudo mkdir -p /etc/etcd /var/lib/etcd
  sudo cp ca.crt laba-etcd-server.key laba-etcd-server.crt /etc/etcd/
}
```

The instance internal IP address will be used to serve client requests and communicate with etcd cluster peers. Retrieve the internal IP address of the master(etcd) nodes:

```
INTERNAL_IP=$(ip addr show enp0s8 | grep "inet " | awk '{print $2}' | cut -d / -f 1)
```

Each etcd member must have a unique name within an etcd cluster. Set the etcd name to match the hostname of the current compute instance:

```
ETCD_NAME=$(hostname -s)
```

Create the `etcd.service` systemd unit file:

```
cat <<EOF | sudo tee /etc/systemd/system/etcd.service
[Unit]
Description=etcd
Documentation=https://github.com/coreos

[Service]
ExecStart=/usr/local/bin/etcd \\
  --name ${ETCD_NAME} \\
  --cert-file=/etc/etcd/laba-etcd-server.crt \\
  --key-file=/etc/etcd/laba-etcd-server.key \\
  --peer-cert-file=/etc/etcd/laba-etcd-server.crt \\
  --peer-key-file=/etc/etcd/laba-etcd-server.key \\
  --trusted-ca-file=/etc/etcd/ca.crt \\
  --peer-trusted-ca-file=/etc/etcd/ca.crt \\
  --peer-client-cert-auth \\
  --client-cert-auth \\
  --initial-advertise-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-client-urls https://${INTERNAL_IP}:2379,https://127.0.0.1:2379 \\
  --advertise-client-urls https://${INTERNAL_IP}:2379 \\
  --initial-cluster-token etcd-cluster-0 \\
  --initial-cluster laba-master-1=https://192.168.5.6:2380,laba-master-2=https://192.168.5.7:2380 \\
  --initial-cluster-state new \\
  --data-dir=/var/lib/etcd
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Start the etcd Server

```
{
  sudo systemctl daemon-reload
  sudo systemctl enable etcd
  sudo systemctl start etcd
}
```

> Remember to run the above commands on each controller node: `laba-master-1`, and `laba-master-2`.

## Verification

List the etcd cluster members:

```
sudo ETCDCTL_API=3 etcdctl member list \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.crt \
  --cert=/etc/etcd/laba-etcd-server.crt \
  --key=/etc/etcd/laba-etcd-server.key
```

> output

```
45bf9ccad8d8900a, started, laba-master-2, https://192.168.5.7:2380, https://192.168.5.7:2379
54a5796a6803f252, started, laba-master-1, https://192.168.5.6:2380, https://192.168.5.6:2379
```

Reference: https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#starting-etcd-clusters

Next: [Bootstrapping the Kubernetes Control Plane](08-bootstrapping-kubernetes-controllers.md)


#### LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B
#### LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B labb
#### LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B

## Bootstrapping the etcd Cluster

Kubernetes components are stateless and store cluster state in [etcd](https://github.com/coreos/etcd). In this lab you will bootstrap a two node etcd cluster and configure it for high availability and secure remote access.

## Prerequisites

The commands in this lab must be run on each controller instance: `labb-master-1`, and `labb-master-2`. Login to each of these using an SSH terminal.

### Running commands in parallel with tmux

[tmux](https://github.com/tmux/tmux/wiki) can be used to run commands on multiple compute instances at the same time. See the [Running commands in parallel with tmux](01-prerequisites.md#running-commands-in-parallel-with-tmux) section in the Prerequisites lab.

## Bootstrapping an etcd Cluster Member

### Download and Install the etcd Binaries

Download the official etcd release binaries from the [coreos/etcd](https://github.com/coreos/etcd) GitHub project:

```
wget -q --show-progress --https-only --timestamping \
  "https://github.com/coreos/etcd/releases/download/v3.4.9/etcd-v3.4.9-linux-amd64.tar.gz"
```

Extract and install the `etcd` server and the `etcdctl` command line utility:

```
{
  tar -xvf etcd-v3.4.9-linux-amd64.tar.gz
  sudo mv etcd-v3.4.9-linux-amd64/etcd* /usr/local/bin/
}
```

### Configure the etcd Server

```
{
  sudo mkdir -p /etc/etcd /var/lib/etcd
  sudo cp ca.crt labb-etcd-server.key labb-etcd-server.crt /etc/etcd/
}
```

The instance internal IP address will be used to serve client requests and communicate with etcd cluster peers. Retrieve the internal IP address of the master(etcd) nodes:

```
INTERNAL_IP=$(ip addr show enp0s8 | grep "inet " | awk '{print $2}' | cut -d / -f 1)
```

Each etcd member must have a unique name within an etcd cluster. Set the etcd name to match the hostname of the current compute instance:

```
ETCD_NAME=$(hostname -s)
```

Create the `etcd.service` systemd unit file:

```
cat <<EOF | sudo tee /etc/systemd/system/etcd.service
[Unit]
Description=etcd
Documentation=https://github.com/coreos

[Service]
ExecStart=/usr/local/bin/etcd \\
  --name ${ETCD_NAME} \\
  --cert-file=/etc/etcd/labb-etcd-server.crt \\
  --key-file=/etc/etcd/labb-etcd-server.key \\
  --peer-cert-file=/etc/etcd/labb-etcd-server.crt \\
  --peer-key-file=/etc/etcd/labb-etcd-server.key \\
  --trusted-ca-file=/etc/etcd/ca.crt \\
  --peer-trusted-ca-file=/etc/etcd/ca.crt \\
  --peer-client-cert-auth \\
  --client-cert-auth \\
  --initial-advertise-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-client-urls https://${INTERNAL_IP}:2379,https://127.0.0.1:2379 \\
  --advertise-client-urls https://${INTERNAL_IP}:2379 \\
  --initial-cluster-token etcd-cluster-0 \\
  --initial-cluster labb-master-1=https://192.168.5.21:2380,labb-master-2=https://192.168.5.22:2380 \\
  --initial-cluster-state new \\
  --data-dir=/var/lib/etcd
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Start the etcd Server

```
{
  sudo systemctl daemon-reload
  sudo systemctl enable etcd
  sudo systemctl start etcd
}
```

> Remember to run the above commands on each controller node: `labb-master-1`, and `labb-master-2`.

## Verification

List the etcd cluster members:

```
sudo ETCDCTL_API=3 etcdctl member list \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.crt \
  --cert=/etc/etcd/labb-etcd-server.crt \
  --key=/etc/etcd/labb-etcd-server.key
```

> output

```
45bf9ccad8d8900a, started, labb-master-2, https://192.168.5.22:2380, https://192.168.5.22:2379
54a5796a6803f252, started, labb-master-1, https://192.168.5.21:2380, https://192.168.5.21:2379
```

#### LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C
#### LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C labc
#### LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C

## Bootstrapping the etcd Cluster

Kubernetes components are stateless and store cluster state in [etcd](https://github.com/coreos/etcd). In this lab you will bootstrap a two node etcd cluster and configure it for high availability and secure remote access.

## Prerequisites

The commands in this lab must be run on each controller instance: `labc-master-1`, and `labc-master-2`. Login to each of these using an SSH terminal.

### Running commands in parallel with tmux

[tmux](https://github.com/tmux/tmux/wiki) can be used to run commands on multiple compute instances at the same time. See the [Running commands in parallel with tmux](01-prerequisites.md#running-commands-in-parallel-with-tmux) section in the Prerequisites lab.

## Bootstrapping an etcd Cluster Member

### Download and Install the etcd Binaries

Download the official etcd release binaries from the [coreos/etcd](https://github.com/coreos/etcd) GitHub project:

```
wget -q --show-progress --https-only --timestamping \
  "https://github.com/coreos/etcd/releases/download/v3.4.9/etcd-v3.4.9-linux-amd64.tar.gz"
```

Extract and install the `etcd` server and the `etcdctl` command line utility:

```
{
  tar -xvf etcd-v3.4.9-linux-amd64.tar.gz
  sudo mv etcd-v3.4.9-linux-amd64/etcd* /usr/local/bin/
}
```

### Configure the etcd Server

```
{
  sudo mkdir -p /etc/etcd /var/lib/etcd
  sudo cp ca.crt labc-etcd-server.key labc-etcd-server.crt /etc/etcd/
}
```

The instance internal IP address will be used to serve client requests and communicate with etcd cluster peers. Retrieve the internal IP address of the master(etcd) nodes:

```
INTERNAL_IP=$(ip addr show enp0s8 | grep "inet " | awk '{print $2}' | cut -d / -f 1)
```

Each etcd member must have a unique name within an etcd cluster. Set the etcd name to match the hostname of the current compute instance:

```
ETCD_NAME=$(hostname -s)
```

Create the `etcd.service` systemd unit file:

```
cat <<EOF | sudo tee /etc/systemd/system/etcd.service
[Unit]
Description=etcd
Documentation=https://github.com/coreos

[Service]
ExecStart=/usr/local/bin/etcd \\
  --name ${ETCD_NAME} \\
  --cert-file=/etc/etcd/labc-etcd-server.crt \\
  --key-file=/etc/etcd/labc-etcd-server.key \\
  --peer-cert-file=/etc/etcd/labc-etcd-server.crt \\
  --peer-key-file=/etc/etcd/labc-etcd-server.key \\
  --trusted-ca-file=/etc/etcd/ca.crt \\
  --peer-trusted-ca-file=/etc/etcd/ca.crt \\
  --peer-client-cert-auth \\
  --client-cert-auth \\
  --initial-advertise-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-client-urls https://${INTERNAL_IP}:2379,https://127.0.0.1:2379 \\
  --advertise-client-urls https://${INTERNAL_IP}:2379 \\
  --initial-cluster-token etcd-cluster-0 \\
  --initial-cluster labc-master-1=https://192.168.5.36:2380,labc-master-2=https://192.168.5.37:2380 \\
  --initial-cluster-state new \\
  --data-dir=/var/lib/etcd
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Start the etcd Server

```
{
  sudo systemctl daemon-reload
  sudo systemctl enable etcd
  sudo systemctl start etcd
}
```

> Remember to run the above commands on each controller node: `labc-master-1`, and `labc-master-2`.

## Verification

List the etcd cluster members:

```
sudo ETCDCTL_API=3 etcdctl member list \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.crt \
  --cert=/etc/etcd/labc-etcd-server.crt \
  --key=/etc/etcd/labc-etcd-server.key
```

> output

```
45bf9ccad8d8900a, started, labc-master-2, https://192.168.5.37:2380, https://192.168.5.37:2379
54a5796a6803f252, started, labc-master-1, https://192.168.5.36:2380, https://192.168.5.36:2379
```


#### LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D
#### LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D labd
#### LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D

## Bootstrapping the etcd Cluster

Kubernetes components are stateless and store cluster state in [etcd](https://github.com/coreos/etcd). In this lab you will bootstrap a two node etcd cluster and configure it for high availability and secure remote access.

## Prerequisites

The commands in this lab must be run on each controller instance: `labd-master-1`, and `labd-master-2`. Login to each of these using an SSH terminal.

### Running commands in parallel with tmux

[tmux](https://github.com/tmux/tmux/wiki) can be used to run commands on multiple compute instances at the same time. See the [Running commands in parallel with tmux](01-prerequisites.md#running-commands-in-parallel-with-tmux) section in the Prerequisites lab.

## Bootstrapping an etcd Cluster Member

### Download and Install the etcd Binaries

Download the official etcd release binaries from the [coreos/etcd](https://github.com/coreos/etcd) GitHub project:

```
wget -q --show-progress --https-only --timestamping \
  "https://github.com/coreos/etcd/releases/download/v3.4.9/etcd-v3.4.9-linux-amd64.tar.gz"
```

Extract and install the `etcd` server and the `etcdctl` command line utility:

```
{
  tar -xvf etcd-v3.4.9-linux-amd64.tar.gz
  sudo mv etcd-v3.4.9-linux-amd64/etcd* /usr/local/bin/
}
```

### Configure the etcd Server

```
{
  sudo mkdir -p /etc/etcd /var/lib/etcd
  sudo cp ca.crt labd-etcd-server.key labd-etcd-server.crt /etc/etcd/
}
```

The instance internal IP address will be used to serve client requests and communicate with etcd cluster peers. Retrieve the internal IP address of the master(etcd) nodes:

```
INTERNAL_IP=$(ip addr show enp0s8 | grep "inet " | awk '{print $2}' | cut -d / -f 1)
```

Each etcd member must have a unique name within an etcd cluster. Set the etcd name to match the hostname of the current compute instance:

```
ETCD_NAME=$(hostname -s)
```

Create the `etcd.service` systemd unit file:

```
cat <<EOF | sudo tee /etc/systemd/system/etcd.service
[Unit]
Description=etcd
Documentation=https://github.com/coreos

[Service]
ExecStart=/usr/local/bin/etcd \\
  --name ${ETCD_NAME} \\
  --cert-file=/etc/etcd/labd-etcd-server.crt \\
  --key-file=/etc/etcd/labd-etcd-server.key \\
  --peer-cert-file=/etc/etcd/labd-etcd-server.crt \\
  --peer-key-file=/etc/etcd/labd-etcd-server.key \\
  --trusted-ca-file=/etc/etcd/ca.crt \\
  --peer-trusted-ca-file=/etc/etcd/ca.crt \\
  --peer-client-cert-auth \\
  --client-cert-auth \\
  --initial-advertise-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-client-urls https://${INTERNAL_IP}:2379,https://127.0.0.1:2379 \\
  --advertise-client-urls https://${INTERNAL_IP}:2379 \\
  --initial-cluster-token etcd-cluster-0 \\
  --initial-cluster labd-master-1=https://192.168.5.51:2380,labd-master-2=https://192.168.5.52:2380 \\
  --initial-cluster-state new \\
  --data-dir=/var/lib/etcd
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Start the etcd Server

```
{
  sudo systemctl daemon-reload
  sudo systemctl enable etcd
  sudo systemctl start etcd
}
```

> Remember to run the above commands on each controller node: `labd-master-1`, and `labd-master-2`.

## Verification

List the etcd cluster members:

```
sudo ETCDCTL_API=3 etcdctl member list \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.crt \
  --cert=/etc/etcd/labd-etcd-server.crt \
  --key=/etc/etcd/labd-etcd-server.key
```

> output

```
45bf9ccad8d8900a, started, labd-master-2, https://192.168.5.52:2380, https://192.168.5.52:2379
54a5796a6803f252, started, labd-master-1, https://192.168.5.51:2380, https://192.168.5.51:2379
```

#### LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E
#### LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E labe
#### LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E

## Bootstrapping the etcd Cluster

Kubernetes components are stateless and store cluster state in [etcd](https://github.com/coreos/etcd). In this lab you will bootstrap a two node etcd cluster and configure it for high availability and secure remote access.

## Prerequisites

The commands in this lab must be run on each controller instance: `labe-master-1`, and `labe-master-2`. Login to each of these using an SSH terminal.

### Running commands in parallel with tmux

[tmux](https://github.com/tmux/tmux/wiki) can be used to run commands on multiple compute instances at the same time. See the [Running commands in parallel with tmux](01-prerequisites.md#running-commands-in-parallel-with-tmux) section in the Prerequisites lab.

## Bootstrapping an etcd Cluster Member

### Download and Install the etcd Binaries

Download the official etcd release binaries from the [coreos/etcd](https://github.com/coreos/etcd) GitHub project:

```
wget -q --show-progress --https-only --timestamping \
  "https://github.com/coreos/etcd/releases/download/v3.4.9/etcd-v3.4.9-linux-amd64.tar.gz"
```

Extract and install the `etcd` server and the `etcdctl` command line utility:

```
{
  tar -xvf etcd-v3.4.9-linux-amd64.tar.gz
  sudo mv etcd-v3.4.9-linux-amd64/etcd* /usr/local/bin/
}
```

### Configure the etcd Server

```
{
  sudo mkdir -p /etc/etcd /var/lib/etcd
  sudo cp ca.crt labe-etcd-server.key labe-etcd-server.crt /etc/etcd/
}
```

The instance internal IP address will be used to serve client requests and communicate with etcd cluster peers. Retrieve the internal IP address of the master(etcd) nodes:

```
INTERNAL_IP=$(ip addr show enp0s8 | grep "inet " | awk '{print $2}' | cut -d / -f 1)
```

Each etcd member must have a unique name within an etcd cluster. Set the etcd name to match the hostname of the current compute instance:

```
ETCD_NAME=$(hostname -s)
```

Create the `etcd.service` systemd unit file:

```
cat <<EOF | sudo tee /etc/systemd/system/etcd.service
[Unit]
Description=etcd
Documentation=https://github.com/coreos

[Service]
ExecStart=/usr/local/bin/etcd \\
  --name ${ETCD_NAME} \\
  --cert-file=/etc/etcd/labe-etcd-server.crt \\
  --key-file=/etc/etcd/labe-etcd-server.key \\
  --peer-cert-file=/etc/etcd/labe-etcd-server.crt \\
  --peer-key-file=/etc/etcd/labe-etcd-server.key \\
  --trusted-ca-file=/etc/etcd/ca.crt \\
  --peer-trusted-ca-file=/etc/etcd/ca.crt \\
  --peer-client-cert-auth \\
  --client-cert-auth \\
  --initial-advertise-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-client-urls https://${INTERNAL_IP}:2379,https://127.0.0.1:2379 \\
  --advertise-client-urls https://${INTERNAL_IP}:2379 \\
  --initial-cluster-token etcd-cluster-0 \\
  --initial-cluster labe-master-1=https://192.168.5.66:2380,labe-master-2=https://192.168.5.67:2380 \\
  --initial-cluster-state new \\
  --data-dir=/var/lib/etcd
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Start the etcd Server

```
{
  sudo systemctl daemon-reload
  sudo systemctl enable etcd
  sudo systemctl start etcd
}
```

> Remember to run the above commands on each controller node: `labe-master-1`, and `labe-master-2`.

## Verification

List the etcd cluster members:

```
sudo ETCDCTL_API=3 etcdctl member list \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.crt \
  --cert=/etc/etcd/labe-etcd-server.crt \
  --key=/etc/etcd/labe-etcd-server.key
```

> output

```
45bf9ccad8d8900a, started, labe-master-2, https://192.168.5.67:2380, https://192.168.5.67:2379
54a5796a6803f252, started, labe-master-1, https://192.168.5.66:2380, https://192.168.5.66:2379
```


#### LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F
#### LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F labf
#### LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F

## Bootstrapping the etcd Cluster

Kubernetes components are stateless and store cluster state in [etcd](https://github.com/coreos/etcd). In this lab you will bootstrap a two node etcd cluster and configure it for high availability and secure remote access.

## Prerequisites

The commands in this lab must be run on each controller instance: `labf-master-1`, and `labf-master-2`. Login to each of these using an SSH terminal.

### Running commands in parallel with tmux

[tmux](https://github.com/tmux/tmux/wiki) can be used to run commands on multiple compute instances at the same time. See the [Running commands in parallel with tmux](01-prerequisites.md#running-commands-in-parallel-with-tmux) section in the Prerequisites lab.

## Bootstrapping an etcd Cluster Member

### Download and Install the etcd Binaries

Download the official etcd release binaries from the [coreos/etcd](https://github.com/coreos/etcd) GitHub project:

```
wget -q --show-progress --https-only --timestamping \
  "https://github.com/coreos/etcd/releases/download/v3.4.9/etcd-v3.4.9-linux-amd64.tar.gz"
```

Extract and install the `etcd` server and the `etcdctl` command line utility:

```
{
  tar -xvf etcd-v3.4.9-linux-amd64.tar.gz
  sudo mv etcd-v3.4.9-linux-amd64/etcd* /usr/local/bin/
}
```

### Configure the etcd Server

```
{
  sudo mkdir -p /etc/etcd /var/lib/etcd
  sudo cp ca.crt labf-etcd-server.key labf-etcd-server.crt /etc/etcd/
}
```

The instance internal IP address will be used to serve client requests and communicate with etcd cluster peers. Retrieve the internal IP address of the master(etcd) nodes:

```
INTERNAL_IP=$(ip addr show enp0s8 | grep "inet " | awk '{print $2}' | cut -d / -f 1)
```

Each etcd member must have a unique name within an etcd cluster. Set the etcd name to match the hostname of the current compute instance:

```
ETCD_NAME=$(hostname -s)
```

Create the `etcd.service` systemd unit file:

```
cat <<EOF | sudo tee /etc/systemd/system/etcd.service
[Unit]
Description=etcd
Documentation=https://github.com/coreos

[Service]
ExecStart=/usr/local/bin/etcd \\
  --name ${ETCD_NAME} \\
  --cert-file=/etc/etcd/labf-etcd-server.crt \\
  --key-file=/etc/etcd/labf-etcd-server.key \\
  --peer-cert-file=/etc/etcd/labf-etcd-server.crt \\
  --peer-key-file=/etc/etcd/labf-etcd-server.key \\
  --trusted-ca-file=/etc/etcd/ca.crt \\
  --peer-trusted-ca-file=/etc/etcd/ca.crt \\
  --peer-client-cert-auth \\
  --client-cert-auth \\
  --initial-advertise-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-client-urls https://${INTERNAL_IP}:2379,https://127.0.0.1:2379 \\
  --advertise-client-urls https://${INTERNAL_IP}:2379 \\
  --initial-cluster-token etcd-cluster-0 \\
  --initial-cluster labf-master-1=https://192.168.5.81:2380,labf-master-2=https://192.168.5.82:2380 \\
  --initial-cluster-state new \\
  --data-dir=/var/lib/etcd
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Start the etcd Server

```
{
  sudo systemctl daemon-reload
  sudo systemctl enable etcd
  sudo systemctl start etcd
}
```

> Remember to run the above commands on each controller node: `labf-master-1`, and `labf-master-2`.

## Verification

List the etcd cluster members:

```
sudo ETCDCTL_API=3 etcdctl member list \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.crt \
  --cert=/etc/etcd/labf-etcd-server.crt \
  --key=/etc/etcd/labf-etcd-server.key
```

> output

```
45bf9ccad8d8900a, started, labf-master-2, https://192.168.5.82:2380, https://192.168.5.82:2379
54a5796a6803f252, started, labf-master-1, https://192.168.5.81:2380, https://192.168.5.81:2379
```
