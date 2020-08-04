#### LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A
#### LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A laba
#### LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A

# Bootstrapping the Kubernetes Worker Nodes

In this lab you will bootstrap 2 Kubernetes worker nodes. We already have [Docker](https://www.docker.com) installed on these nodes.

We will now install the kubernetes components
- [kubelet](https://kubernetes.io/docs/admin/kubelet)
- [kube-proxy](https://kubernetes.io/docs/concepts/cluster-administration/proxies).

## Prerequisites

The Certificates and Configuration are created on `k8smanagement` node and then copied over to workers using `scp`. 
Once this is done, the commands are to be run on first worker instance: `laba-worker-1` and `laba-worker-2`. Login to first worker instance using SSH Terminal.

### Provisioning  Kubelet Client Certificates

Kubernetes uses a [special-purpose authorization mode](https://kubernetes.io/docs/admin/authorization/node/) called Node Authorizer, that specifically authorizes API requests made by [Kubelets](https://kubernetes.io/docs/concepts/overview/components/#kubelet). In order to be authorized by the Node Authorizer, Kubelets must use a credential that identifies them as being in the `system:nodes` group, with a username of `system:node:<nodeName>`. In this section you will create a certificate for each Kubernetes worker node that meets the Node Authorizer requirements.

Generate a certificate and private key for one worker node:

On management:

```
cat > laba-openssl-laba-worker-1.cnf <<EOF
[req]
req_extensions = v3_req
distinguished_name = req_distinguished_name
[req_distinguished_name]
[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @alt_names
[alt_names]
DNS.1 = laba-worker-1
IP.1 = 192.168.5.11
EOF

openssl genrsa -out laba-worker-1.key 2048
openssl req -new -key laba-worker-1.key -subj "/CN=system:node:laba-worker-1/O=system:nodes" -out laba-worker-1.csr -config laba-openssl-laba-worker-1.cnf
openssl x509 -req -in laba-worker-1.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out laba-worker-1.crt -extensions v3_req -extfile laba-openssl-laba-worker-1.cnf -days 1000
```

Results:

```
laba-worker-1.key
laba-worker-1.crt
```
```
cat > laba-openssl-laba-worker-2.cnf <<EOF
[req]
req_extensions = v3_req
distinguished_name = req_distinguished_name
[req_distinguished_name]
[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @alt_names
[alt_names]
DNS.1 = laba-worker-2
IP.1 = 192.168.5.12
EOF

openssl genrsa -out laba-worker-2.key 2048
openssl req -new -key laba-worker-2.key -subj "/CN=system:node:laba-worker-2/O=system:nodes" -out laba-worker-2.csr -config laba-openssl-laba-worker-2.cnf
openssl x509 -req -in laba-worker-2.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out laba-worker-2.crt -extensions v3_req -extfile laba-openssl-laba-worker-2.cnf -days 1000
```

Results:

```
laba-worker-2.key
laba-worker-2.crt
```

### The kubelet Kubernetes Configuration File

When generating kubeconfig files for Kubelets the client certificate matching the Kubelet's node name must be used. This will ensure Kubelets are properly authorized by the Kubernetes [Node Authorizer](https://kubernetes.io/docs/admin/authorization/node/).

Get the kub-api server load-balancer IP.
```
LOADBALANCER_ADDRESS=192.168.5.15
```

Generate a kubeconfig file for the first worker node.

On management:
```
{
  kubectl config set-cluster hardway-cluster-a \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://${LOADBALANCER_ADDRESS}:6443 \
    --kubeconfig=laba-worker-1.kubeconfig

  kubectl config set-credentials system:node:laba-worker-1 \
    --client-certificate=laba-worker-1.crt \
    --client-key=laba-worker-1.key \
    --embed-certs=true \
    --kubeconfig=laba-worker-1.kubeconfig

  kubectl config set-context default \
    --cluster=hardway-cluster-a \
    --user=system:node:laba-worker-1 \
    --kubeconfig=laba-worker-1.kubeconfig

  kubectl config use-context default --kubeconfig=laba-worker-1.kubeconfig
}
```

Results:

```
laba-worker-1.kubeconfig
```

{
  kubectl config set-cluster hardway-cluster-a \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://${LOADBALANCER_ADDRESS}:6443 \
    --kubeconfig=laba-worker-2.kubeconfig

  kubectl config set-credentials system:node:laba-worker-2 \
    --client-certificate=laba-worker-2.crt \
    --client-key=laba-worker-2.key \
    --embed-certs=true \
    --kubeconfig=laba-worker-2.kubeconfig

  kubectl config set-context default \
    --cluster=hardway-cluster-a \
    --user=system:node:laba-worker-2 \
    --kubeconfig=laba-worker-2.kubeconfig

  kubectl config use-context default --kubeconfig=laba-worker-2.kubeconfig
}
```

Results:

```
laba-worker-2.kubeconfig
```

### Copy certificates, private keys and kubeconfig files to the worker node:
On management:
```
scp ca.crt laba-worker-1.crt laba-worker-1.key laba-worker-1.kubeconfig laba-worker-1:~/
scp ca.crt laba-worker-2.crt laba-worker-2.key laba-worker-2.kubeconfig laba-worker-2:~/
```

### Download and Install Worker Binaries

Going forward all activities are to be done on the `laba-worker-1` node.

On laba-worker-1:
```
laba-worker-1$ wget -q --show-progress --https-only --timestamping \
  https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kubectl \
  https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kube-proxy \
  https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kubelet
```

Reference: https://kubernetes.io/docs/setup/release/#node-binaries

Create the installation directories:

```
laba-worker-1$ sudo mkdir -p \
  /etc/cni/net.d \
  /opt/cni/bin \
  /var/lib/kubelet \
  /var/lib/kube-proxy \
  /var/lib/kubernetes \
  /var/run/kubernetes
```

Install the worker binaries:

```
{
  chmod +x kubectl kube-proxy kubelet
  sudo mv kubectl kube-proxy kubelet /usr/local/bin/
}
```

### Configure the Kubelet

```
{
  sudo mv ${HOSTNAME}.key ${HOSTNAME}.crt /var/lib/kubelet/
  sudo mv ${HOSTNAME}.kubeconfig /var/lib/kubelet/kubeconfig
  sudo mv ca.crt /var/lib/kubernetes/
}
```

Create the `kubelet-config.yaml` configuration file:

```
laba-worker-1$ cat <<EOF | sudo tee /var/lib/kubelet/kubelet-config.yaml
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
    enabled: false
  webhook:
    enabled: true
  x509:
    clientCAFile: "/var/lib/kubernetes/ca.crt"
authorization:
  mode: Webhook
clusterDomain: "cluster.local"
clusterDNS:
  - "10.96.0.10"
resolvConf: "/run/systemd/resolve/resolv.conf"
runtimeRequestTimeout: "15m"
EOF
```

> The `resolvConf` configuration is used to avoid loops when using CoreDNS for service discovery on systems running `systemd-resolved`.

Create the `kubelet.service` systemd unit file:

```
laba-worker-1$ cat <<EOF | sudo tee /etc/systemd/system/kubelet.service
[Unit]
Description=Kubernetes Kubelet
Documentation=https://github.com/kubernetes/kubernetes
After=docker.service
Requires=docker.service

[Service]
ExecStart=/usr/local/bin/kubelet \\
  --config=/var/lib/kubelet/kubelet-config.yaml \\
  --image-pull-progress-deadline=2m \\
  --kubeconfig=/var/lib/kubelet/kubeconfig \\
  --tls-cert-file=/var/lib/kubelet/${HOSTNAME}.crt \\
  --tls-private-key-file=/var/lib/kubelet/${HOSTNAME}.key \\
  --network-plugin=cni \\
  --register-node=true \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Configure the Kubernetes Proxy

```
laba-worker-1$ sudo mv laba-kube-proxy.kubeconfig /var/lib/kube-proxy/kubeconfig
```

Create the `kube-proxy-config.yaml` configuration file:

```
laba-worker-1$ cat <<EOF | sudo tee /var/lib/kube-proxy/kube-proxy-config.yaml
kind: KubeProxyConfiguration
apiVersion: kubeproxy.config.k8s.io/v1alpha1
clientConnection:
  kubeconfig: "/var/lib/kube-proxy/kubeconfig"
mode: "iptables"
clusterCIDR: "192.168.5.0/24"
EOF
```

Create the `kube-proxy.service` systemd unit file:

```
laba-worker-1$ cat <<EOF | sudo tee /etc/systemd/system/kube-proxy.service
[Unit]
Description=Kubernetes Kube Proxy
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-proxy \\
  --config=/var/lib/kube-proxy/kube-proxy-config.yaml
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Start the Worker Services

```
{
  sudo systemctl daemon-reload
  sudo systemctl enable kubelet kube-proxy
  sudo systemctl start kubelet kube-proxy
}
```

> Remember to run the above commands on worker node: `laba-worker-1`


On laba-worker-2:  
```
laba-worker-2$ wget -q --show-progress --https-only --timestamping \
  https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kubectl \
  https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kube-proxy \
  https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kubelet
```

Reference: https://kubernetes.io/docs/setup/release/#node-binaries

Create the installation directories:

```
laba-worker-2$ sudo mkdir -p \
  /etc/cni/net.d \
  /opt/cni/bin \
  /var/lib/kubelet \
  /var/lib/kube-proxy \
  /var/lib/kubernetes \
  /var/run/kubernetes
```

Install the worker binaries:

```
{
  chmod +x kubectl kube-proxy kubelet
  sudo mv kubectl kube-proxy kubelet /usr/local/bin/
}
```

### Configure the Kubelet

```
{
  sudo mv ${HOSTNAME}.key ${HOSTNAME}.crt /var/lib/kubelet/
  sudo mv ${HOSTNAME}.kubeconfig /var/lib/kubelet/kubeconfig
  sudo mv ca.crt /var/lib/kubernetes/
}
```

Create the `kubelet-config.yaml` configuration file:

```
laba-worker-2$ cat <<EOF | sudo tee /var/lib/kubelet/kubelet-config.yaml
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
    enabled: false
  webhook:
    enabled: true
  x509:
    clientCAFile: "/var/lib/kubernetes/ca.crt"
authorization:
  mode: Webhook
clusterDomain: "cluster.local"
clusterDNS:
  - "10.96.0.10"
resolvConf: "/run/systemd/resolve/resolv.conf"
runtimeRequestTimeout: "15m"
EOF
```

> The `resolvConf` configuration is used to avoid loops when using CoreDNS for service discovery on systems running `systemd-resolved`.

Create the `kubelet.service` systemd unit file:

```
laba-worker-2$ cat <<EOF | sudo tee /etc/systemd/system/kubelet.service
[Unit]
Description=Kubernetes Kubelet
Documentation=https://github.com/kubernetes/kubernetes
After=docker.service
Requires=docker.service

[Service]
ExecStart=/usr/local/bin/kubelet \\
  --config=/var/lib/kubelet/kubelet-config.yaml \\
  --image-pull-progress-deadline=2m \\
  --kubeconfig=/var/lib/kubelet/kubeconfig \\
  --tls-cert-file=/var/lib/kubelet/${HOSTNAME}.crt \\
  --tls-private-key-file=/var/lib/kubelet/${HOSTNAME}.key \\
  --network-plugin=cni \\
  --register-node=true \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Configure the Kubernetes Proxy

```
laba-worker-2$ sudo mv laba-kube-proxy.kubeconfig /var/lib/kube-proxy/kubeconfig
```

Create the `kube-proxy-config.yaml` configuration file:

```
laba-worker-2$ cat <<EOF | sudo tee /var/lib/kube-proxy/kube-proxy-config.yaml
kind: KubeProxyConfiguration
apiVersion: kubeproxy.config.k8s.io/v1alpha1
clientConnection:
  kubeconfig: "/var/lib/kube-proxy/kubeconfig"
mode: "iptables"
clusterCIDR: "192.168.5.0/24"
EOF
```

Create the `kube-proxy.service` systemd unit file:

```
laba-worker-2$ cat <<EOF | sudo tee /etc/systemd/system/kube-proxy.service
[Unit]
Description=Kubernetes Kube Proxy
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-proxy \\
  --config=/var/lib/kube-proxy/kube-proxy-config.yaml
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Start the Worker Services

```
{
  sudo systemctl daemon-reload
  sudo systemctl enable kubelet kube-proxy
  sudo systemctl start kubelet kube-proxy
}
```

> Remember to run the above commands on worker node: `laba-worker-2`

## Verification
On laba-master-1:

List the registered Kubernetes nodes from the master node:

```
laba-master-1$ kubectl get nodes --kubeconfig admin.kubeconfig
```

> output

```
NAME       STATUS     ROLES    AGE   VERSION
laba-worker-1   NotReady   <none>   93s   v1.13.0
laba-worker-2   NotReady   <none>   93s   v1.13.0
```

> Note: It is OK for the worker node to be in a NotReady state.
  That is because we haven't configured Networking yet.

Optional: At this point you may run the certificate verification script to make sure all certificates are configured correctly. Follow the instructions [here](verify-certificates.md)

Next: [TLS Bootstrapping Kubernetes Workers](10-tls-bootstrapping-kubernetes-workers.md)




#### LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B
#### LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B labb
#### LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B

# Bootstrapping the Kubernetes Worker Nodes

In this lab you will bootstrap 2 Kubernetes worker nodes. We already have [Docker](https://www.docker.com) installed on these nodes.

We will now install the kubernetes components
- [kubelet](https://kubernetes.io/docs/admin/kubelet)
- [kube-proxy](https://kubernetes.io/docs/concepts/cluster-bdministration/proxies).

## Prerequisites

The Certificates and Configuration are created on `k8smanagement` node and then copied over to workers using `scp`. 
Once this is done, the commands are to be run on first worker instance: `labb-worker-1` and `labb-worker-2`. Login to first worker instance using SSH Terminal.

### Provisioning  Kubelet Client Certificates

Kubernetes uses a [special-purpose authorization mode](https://kubernetes.io/docs/admin/authorization/node/) called Node Authorizer, that specifically authorizes API requests made by [Kubelets](https://kubernetes.io/docs/concepts/overview/components/#kubelet). In order to be authorized by the Node Authorizer, Kubelets must use a credential that identifies them as being in the `system:nodes` group, with a username of `system:node:<nodeName>`. In this section you will create a certificate for each Kubernetes worker node that meets the Node Authorizer requirements.

Generate a certificate and private key for one worker node:

On management:

```
cat > labb-openssl-labb-worker-1.cnf <<EOF
[req]
req_extensions = v3_req
distinguished_name = req_distinguished_name
[req_distinguished_name]
[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @alt_names
[alt_names]
DNS.1 = labb-worker-1
IP.1 = 192.168.5.26
EOF

openssl genrsa -out labb-worker-1.key 2048
openssl req -new -key labb-worker-1.key -subj "/CN=system:node:labb-worker-1/O=system:nodes" -out labb-worker-1.csr -config labb-openssl-labb-worker-1.cnf
openssl x509 -req -in labb-worker-1.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out labb-worker-1.crt -extensions v3_req -extfile labb-openssl-labb-worker-1.cnf -days 1000
```

Results:

```
labb-worker-1.key
labb-worker-1.crt
```
```
cat > labb-openssl-labb-worker-2.cnf <<EOF
[req]
req_extensions = v3_req
distinguished_name = req_distinguished_name
[req_distinguished_name]
[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @alt_names
[alt_names]
DNS.1 = labb-worker-2
IP.1 = 192.168.5.27
EOF

openssl genrsa -out labb-worker-2.key 2048
openssl req -new -key labb-worker-2.key -subj "/CN=system:node:labb-worker-2/O=system:nodes" -out labb-worker-2.csr -config labb-openssl-labb-worker-2.cnf
openssl x509 -req -in labb-worker-2.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out labb-worker-2.crt -extensions v3_req -extfile labb-openssl-labb-worker-2.cnf -days 1000
```

Results:

```
labb-worker-2.key
labb-worker-2.crt
```

### The kubelet Kubernetes Configuration File

When generating kubeconfig files for Kubelets the client certificate matching the Kubelet's node name must be used. This will ensure Kubelets are properly authorized by the Kubernetes [Node Authorizer](https://kubernetes.io/docs/admin/authorization/node/).

Get the kub-api server load-balancer IP.
```
LOADBALANCER_ADDRESS=192.168.5.30
```

Generate a kubeconfig file for the first worker node. 

On management:
```
{
  kubectl config set-cluster hardway-cluster-b \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://${LOADBALANCER_ADDRESS}:6443 \
    --kubeconfig=labb-worker-1.kubeconfig

  kubectl config set-credentials system:node:labb-worker-1 \
    --client-certificate=labb-worker-1.crt \
    --client-key=labb-worker-1.key \
    --embed-certs=true \
    --kubeconfig=labb-worker-1.kubeconfig

  kubectl config set-context default \
    --cluster=hardway-cluster-b \
    --user=system:node:labb-worker-1 \
    --kubeconfig=labb-worker-1.kubeconfig

  kubectl config use-context default --kubeconfig=labb-worker-1.kubeconfig
}
```

Results:

```
labb-worker-1.kubeconfig
```

{
  kubectl config set-cluster hardway-cluster-b \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://${LOADBALANCER_ADDRESS}:6443 \
    --kubeconfig=labb-worker-2.kubeconfig

  kubectl config set-credentials system:node:labb-worker-2 \
    --client-certificate=labb-worker-2.crt \
    --client-key=labb-worker-2.key \
    --embed-certs=true \
    --kubeconfig=labb-worker-2.kubeconfig

  kubectl config set-context default \
    --cluster=hardway-cluster-b \
    --user=system:node:labb-worker-2 \
    --kubeconfig=labb-worker-2.kubeconfig

  kubectl config use-context default --kubeconfig=labb-worker-2.kubeconfig
}
```

Results:

```
labb-worker-2.kubeconfig
```

### Copy certificates, private keys and kubeconfig files to the worker node:
On management:
```
scp ca.crt labb-worker-1.crt labb-worker-1.key labb-worker-1.kubeconfig labb-worker-1:~/
scp ca.crt labb-worker-2.crt labb-worker-2.key labb-worker-2.kubeconfig labb-worker-2:~/
```

### Download and Install Worker Binaries

Going forward all activities are to be done on the `labb-worker-1` node.

On labb-worker-1:
```
wget -q --show-progress --https-only --timestamping \
  https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kubectl \
  https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kube-proxy \
  https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kubelet
```

Reference: https://kubernetes.io/docs/setup/release/#node-binaries

Create the installation directories:

```
sudo mkdir -p \
  /etc/cni/net.d \
  /opt/cni/bin \
  /var/lib/kubelet \
  /var/lib/kube-proxy \
  /var/lib/kubernetes \
  /var/run/kubernetes
```

Install the worker binaries:

```
{
  chmod +x kubectl kube-proxy kubelet
  sudo mv kubectl kube-proxy kubelet /usr/local/bin/
}
```

### Configure the Kubelet

```
{
  sudo mv ${HOSTNAME}.key ${HOSTNAME}.crt /var/lib/kubelet/
  sudo mv ${HOSTNAME}.kubeconfig /var/lib/kubelet/kubeconfig
  sudo mv ca.crt /var/lib/kubernetes/
}
```

Create the `kubelet-config.yaml` configuration file:

```
cat <<EOF | sudo tee /var/lib/kubelet/kubelet-config.yaml
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
    enabled: false
  webhook:
    enabled: true
  x509:
    clientCAFile: "/var/lib/kubernetes/ca.crt"
authorization:
  mode: Webhook
clusterDomain: "cluster.local"
clusterDNS:
  - "10.96.0.10"
resolvConf: "/run/systemd/resolve/resolv.conf"
runtimeRequestTimeout: "15m"
EOF
```

> The `resolvConf` configuration is used to avoid loops when using CoreDNS for service discovery on systems running `systemd-resolved`.

Create the `kubelet.service` systemd unit file:

```
cat <<EOF | sudo tee /etc/systemd/system/kubelet.service
[Unit]
Description=Kubernetes Kubelet
Documentation=https://github.com/kubernetes/kubernetes
After=docker.service
Requires=docker.service

[Service]
ExecStart=/usr/local/bin/kubelet \\
  --config=/var/lib/kubelet/kubelet-config.yaml \\
  --image-pull-progress-deadline=2m \\
  --kubeconfig=/var/lib/kubelet/kubeconfig \\
  --tls-cert-file=/var/lib/kubelet/${HOSTNAME}.crt \\
  --tls-private-key-file=/var/lib/kubelet/${HOSTNAME}.key \\
  --network-plugin=cni \\
  --register-node=true \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Configure the Kubernetes Proxy

```
sudo mv labb-kube-proxy.kubeconfig /var/lib/kube-proxy/kubeconfig
```

Create the `kube-proxy-config.yaml` configuration file:

```
cat <<EOF | sudo tee /var/lib/kube-proxy/kube-proxy-config.yaml
kind: KubeProxyConfiguration
apiVersion: kubeproxy.config.k8s.io/v1alpha1
clientConnection:
  kubeconfig: "/var/lib/kube-proxy/kubeconfig"
mode: "iptables"
clusterCIDR: "192.168.5.0/24"
EOF
```

Create the `kube-proxy.service` systemd unit file:

```
cat <<EOF | sudo tee /etc/systemd/system/kube-proxy.service
[Unit]
Description=Kubernetes Kube Proxy
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-proxy \\
  --config=/var/lib/kube-proxy/kube-proxy-config.yaml
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Start the Worker Services

```
{
  sudo systemctl daemon-reload
  sudo systemctl enable kubelet kube-proxy
  sudo systemctl start kubelet kube-proxy
}
```

> Remember to run the above commands on worker node: `labb-worker-1`


On labb-worker-2:  
```
wget -q --show-progress --https-only --timestamping \
  https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kubectl \
  https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kube-proxy \
  https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kubelet
```

Reference: https://kubernetes.io/docs/setup/release/#node-binaries

Create the installation directories:

```
sudo mkdir -p \
  /etc/cni/net.d \
  /opt/cni/bin \
  /var/lib/kubelet \
  /var/lib/kube-proxy \
  /var/lib/kubernetes \
  /var/run/kubernetes
```

Install the worker binaries:

```
{
  chmod +x kubectl kube-proxy kubelet
  sudo mv kubectl kube-proxy kubelet /usr/local/bin/
}
```

### Configure the Kubelet

```
{
  sudo mv ${HOSTNAME}.key ${HOSTNAME}.crt /var/lib/kubelet/
  sudo mv ${HOSTNAME}.kubeconfig /var/lib/kubelet/kubeconfig
  sudo mv ca.crt /var/lib/kubernetes/
}
```

Create the `kubelet-config.yaml` configuration file:

```
cat <<EOF | sudo tee /var/lib/kubelet/kubelet-config.yaml
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
    enabled: false
  webhook:
    enabled: true
  x509:
    clientCAFile: "/var/lib/kubernetes/ca.crt"
authorization:
  mode: Webhook
clusterDomain: "cluster.local"
clusterDNS:
  - "10.96.0.10"
resolvConf: "/run/systemd/resolve/resolv.conf"
runtimeRequestTimeout: "15m"
EOF
```

> The `resolvConf` configuration is used to avoid loops when using CoreDNS for service discovery on systems running `systemd-resolved`.

Create the `kubelet.service` systemd unit file:

```
cat <<EOF | sudo tee /etc/systemd/system/kubelet.service
[Unit]
Description=Kubernetes Kubelet
Documentation=https://github.com/kubernetes/kubernetes
After=docker.service
Requires=docker.service

[Service]
ExecStart=/usr/local/bin/kubelet \\
  --config=/var/lib/kubelet/kubelet-config.yaml \\
  --image-pull-progress-deadline=2m \\
  --kubeconfig=/var/lib/kubelet/kubeconfig \\
  --tls-cert-file=/var/lib/kubelet/${HOSTNAME}.crt \\
  --tls-private-key-file=/var/lib/kubelet/${HOSTNAME}.key \\
  --network-plugin=cni \\
  --register-node=true \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Configure the Kubernetes Proxy

```
sudo mv labb-kube-proxy.kubeconfig /var/lib/kube-proxy/kubeconfig
```

Create the `kube-proxy-config.yaml` configuration file:

```
cat <<EOF | sudo tee /var/lib/kube-proxy/kube-proxy-config.yaml
kind: KubeProxyConfiguration
apiVersion: kubeproxy.config.k8s.io/v1alpha1
clientConnection:
  kubeconfig: "/var/lib/kube-proxy/kubeconfig"
mode: "iptables"
clusterCIDR: "192.168.5.0/24"
EOF
```

Create the `kube-proxy.service` systemd unit file:

```
cat <<EOF | sudo tee /etc/systemd/system/kube-proxy.service
[Unit]
Description=Kubernetes Kube Proxy
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-proxy \\
  --config=/var/lib/kube-proxy/kube-proxy-config.yaml
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Start the Worker Services

```
{
  sudo systemctl daemon-reload
  sudo systemctl enable kubelet kube-proxy
  sudo systemctl start kubelet kube-proxy
}
```

> Remember to run the above commands on worker node: `labb-worker-2`

## Verification
On labb-master-1:

List the registered Kubernetes nodes from the master node:

```
labb-master-1$ kubectl get nodes --kubeconfig admin.kubeconfig
```

> output

```
NAME       STATUS     ROLES    AGE   VERSION
labb-worker-1   NotReady   <none>   93s   v1.13.0
labb-worker-2   NotReady   <none>   93s   v1.13.0
```

> Note: It is OK for the worker node to be in a NotReady state.
  That is because we haven't configured Networking yet.

Optional: At this point you may run the certificate verification script to make sure all certificates are configured correctly. Follow the instructions [here](verify-certificates.md)





#### LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C
#### LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C labc
#### LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C

# Bootstrapping the Kubernetes Worker Nodes

In this lab you will bootstrap 2 Kubernetes worker nodes. We already have [Docker](https://www.docker.com) installed on these nodes.

We will now install the kubernetes components
- [kubelet](https://kubernetes.io/docs/admin/kubelet)
- [kube-proxy](https://kubernetes.io/docs/concepts/cluster-bdministration/proxies).

## Prerequisites

The Certificates and Configuration are created on `k8smanagement` node and then copied over to workers using `scp`. 
Once this is done, the commands are to be run on first worker instance: `labc-worker-1` and `labc-worker-2`. Login to first worker instance using SSH Terminal.

### Provisioning  Kubelet Client Certificates

Kubernetes uses a [special-purpose authorization mode](https://kubernetes.io/docs/admin/authorization/node/) called Node Authorizer, that specifically authorizes API requests made by [Kubelets](https://kubernetes.io/docs/concepts/overview/components/#kubelet). In order to be authorized by the Node Authorizer, Kubelets must use a credential that identifies them as being in the `system:nodes` group, with a username of `system:node:<nodeName>`. In this section you will create a certificate for each Kubernetes worker node that meets the Node Authorizer requirements.

Generate a certificate and private key for one worker node:

On management:

```
cat > labc-openssl-labc-worker-1.cnf <<EOF
[req]
req_extensions = v3_req
distinguished_name = req_distinguished_name
[req_distinguished_name]
[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @alt_names
[alt_names]
DNS.1 = labc-worker-1
IP.1 = 192.168.5.41
EOF

openssl genrsa -out labc-worker-1.key 2048
openssl req -new -key labc-worker-1.key -subj "/CN=system:node:labc-worker-1/O=system:nodes" -out labc-worker-1.csr -config labc-openssl-labc-worker-1.cnf
openssl x509 -req -in labc-worker-1.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out labc-worker-1.crt -extensions v3_req -extfile labc-openssl-labc-worker-1.cnf -days 1000
```

Results:

```
labc-worker-1.key
labc-worker-1.crt
```
```
cat > labc-openssl-labc-worker-2.cnf <<EOF
[req]
req_extensions = v3_req
distinguished_name = req_distinguished_name
[req_distinguished_name]
[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @alt_names
[alt_names]
DNS.1 = labc-worker-2
IP.1 = 192.168.5.42
EOF

openssl genrsa -out labc-worker-2.key 2048
openssl req -new -key labc-worker-2.key -subj "/CN=system:node:labc-worker-2/O=system:nodes" -out labc-worker-2.csr -config labc-openssl-labc-worker-2.cnf
openssl x509 -req -in labc-worker-2.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out labc-worker-2.crt -extensions v3_req -extfile labc-openssl-labc-worker-2.cnf -days 1000
```

Results:

```
labc-worker-2.key
labc-worker-2.crt
```

### The kubelet Kubernetes Configuration File

When generating kubeconfig files for Kubelets the client certificate matching the Kubelet's node name must be used. This will ensure Kubelets are properly authorized by the Kubernetes [Node Authorizer](https://kubernetes.io/docs/admin/authorization/node/).

Get the kub-api server load-balancer IP.
```
LOADBALANCER_ADDRESS=192.168.5.45
```

Generate a kubeconfig file for the first worker node. 

On management:
```
{
  kubectl config set-cluster hardway-cluster-c \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://${LOADBALANCER_ADDRESS}:6443 \
    --kubeconfig=labc-worker-1.kubeconfig

  kubectl config set-credentials system:node:labc-worker-1 \
    --client-certificate=labc-worker-1.crt \
    --client-key=labc-worker-1.key \
    --embed-certs=true \
    --kubeconfig=labc-worker-1.kubeconfig

  kubectl config set-context default \
    --cluster=hardway-cluster-c \
    --user=system:node:labc-worker-1 \
    --kubeconfig=labc-worker-1.kubeconfig

  kubectl config use-context default --kubeconfig=labc-worker-1.kubeconfig
}
```

Results:

```
labc-worker-1.kubeconfig
```

{
  kubectl config set-cluster hardway-cluster-c \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://${LOADBALANCER_ADDRESS}:6443 \
    --kubeconfig=labc-worker-2.kubeconfig

  kubectl config set-credentials system:node:labc-worker-2 \
    --client-certificate=labc-worker-2.crt \
    --client-key=labc-worker-2.key \
    --embed-certs=true \
    --kubeconfig=labc-worker-2.kubeconfig

  kubectl config set-context default \
    --cluster=hardway-cluster-c \
    --user=system:node:labc-worker-2 \
    --kubeconfig=labc-worker-2.kubeconfig

  kubectl config use-context default --kubeconfig=labc-worker-2.kubeconfig
}
```

Results:

```
labc-worker-2.kubeconfig
```

### Copy certificates, private keys and kubeconfig files to the worker node:
On management:
```
scp ca.crt labc-worker-1.crt labc-worker-1.key labc-worker-1.kubeconfig labc-worker-1:~/
scp ca.crt labc-worker-2.crt labc-worker-2.key labc-worker-2.kubeconfig labc-worker-2:~/
```



### Download and Install Worker Binaries

Going forward all activities are to be done on the `labc-worker-1` node.

On labc-worker-1:
```
labc-worker-1$ wget -q --show-progress --https-only --timestamping \
  https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kubectl \
  https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kube-proxy \
  https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kubelet
```

Reference: https://kubernetes.io/docs/setup/release/#node-binaries

Create the installation directories:

```
labc-worker-1$ sudo mkdir -p \
  /etc/cni/net.d \
  /opt/cni/bin \
  /var/lib/kubelet \
  /var/lib/kube-proxy \
  /var/lib/kubernetes \
  /var/run/kubernetes
```

Install the worker binaries:

```
{
  chmod +x kubectl kube-proxy kubelet
  sudo mv kubectl kube-proxy kubelet /usr/local/bin/
}
```

### Configure the Kubelet

```
{
  sudo mv ${HOSTNAME}.key ${HOSTNAME}.crt /var/lib/kubelet/
  sudo mv ${HOSTNAME}.kubeconfig /var/lib/kubelet/kubeconfig
  sudo mv ca.crt /var/lib/kubernetes/
}
```

Create the `kubelet-config.yaml` configuration file:

```
labc-worker-1$ cat <<EOF | sudo tee /var/lib/kubelet/kubelet-config.yaml
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
    enabled: false
  webhook:
    enabled: true
  x509:
    clientCAFile: "/var/lib/kubernetes/ca.crt"
authorization:
  mode: Webhook
clusterDomain: "cluster.local"
clusterDNS:
  - "10.96.0.10"
resolvConf: "/run/systemd/resolve/resolv.conf"
runtimeRequestTimeout: "15m"
EOF
```

> The `resolvConf` configuration is used to avoid loops when using CoreDNS for service discovery on systems running `systemd-resolved`.

Create the `kubelet.service` systemd unit file:

```
labc-worker-1$ cat <<EOF | sudo tee /etc/systemd/system/kubelet.service
[Unit]
Description=Kubernetes Kubelet
Documentation=https://github.com/kubernetes/kubernetes
After=docker.service
Requires=docker.service

[Service]
ExecStart=/usr/local/bin/kubelet \\
  --config=/var/lib/kubelet/kubelet-config.yaml \\
  --image-pull-progress-deadline=2m \\
  --kubeconfig=/var/lib/kubelet/kubeconfig \\
  --tls-cert-file=/var/lib/kubelet/${HOSTNAME}.crt \\
  --tls-private-key-file=/var/lib/kubelet/${HOSTNAME}.key \\
  --network-plugin=cni \\
  --register-node=true \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Configure the Kubernetes Proxy

```
labc-worker-1$ sudo mv labc-kube-proxy.kubeconfig /var/lib/kube-proxy/kubeconfig
```

Create the `kube-proxy-config.yaml` configuration file:

```
labc-worker-1$ cat <<EOF | sudo tee /var/lib/kube-proxy/kube-proxy-config.yaml
kind: KubeProxyConfiguration
apiVersion: kubeproxy.config.k8s.io/v1alpha1
clientConnection:
  kubeconfig: "/var/lib/kube-proxy/kubeconfig"
mode: "iptables"
clusterCIDR: "192.168.5.0/24"
EOF
```

Create the `kube-proxy.service` systemd unit file:

```
labc-worker-1$ cat <<EOF | sudo tee /etc/systemd/system/kube-proxy.service
[Unit]
Description=Kubernetes Kube Proxy
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-proxy \\
  --config=/var/lib/kube-proxy/kube-proxy-config.yaml
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Start the Worker Services

```
{
  sudo systemctl daemon-reload
  sudo systemctl enable kubelet kube-proxy
  sudo systemctl start kubelet kube-proxy
}
```

> Remember to run the above commands on worker node: `labc-worker-1`


On labc-worker-2:  
```
labc-worker-2$ wget -q --show-progress --https-only --timestamping \
  https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kubectl \
  https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kube-proxy \
  https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kubelet
```

Reference: https://kubernetes.io/docs/setup/release/#node-binaries

Create the installation directories:

```
labc-worker-2$ sudo mkdir -p \
  /etc/cni/net.d \
  /opt/cni/bin \
  /var/lib/kubelet \
  /var/lib/kube-proxy \
  /var/lib/kubernetes \
  /var/run/kubernetes
```

Install the worker binaries:

```
{
  chmod +x kubectl kube-proxy kubelet
  sudo mv kubectl kube-proxy kubelet /usr/local/bin/
}
```

### Configure the Kubelet

```
{
  sudo mv ${HOSTNAME}.key ${HOSTNAME}.crt /var/lib/kubelet/
  sudo mv ${HOSTNAME}.kubeconfig /var/lib/kubelet/kubeconfig
  sudo mv ca.crt /var/lib/kubernetes/
}
```

Create the `kubelet-config.yaml` configuration file:

```
labc-worker-2$ cat <<EOF | sudo tee /var/lib/kubelet/kubelet-config.yaml
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
    enabled: false
  webhook:
    enabled: true
  x509:
    clientCAFile: "/var/lib/kubernetes/ca.crt"
authorization:
  mode: Webhook
clusterDomain: "cluster.local"
clusterDNS:
  - "10.96.0.10"
resolvConf: "/run/systemd/resolve/resolv.conf"
runtimeRequestTimeout: "15m"
EOF
```

> The `resolvConf` configuration is used to avoid loops when using CoreDNS for service discovery on systems running `systemd-resolved`.

Create the `kubelet.service` systemd unit file:

```
labc-worker-2$ cat <<EOF | sudo tee /etc/systemd/system/kubelet.service
[Unit]
Description=Kubernetes Kubelet
Documentation=https://github.com/kubernetes/kubernetes
After=docker.service
Requires=docker.service

[Service]
ExecStart=/usr/local/bin/kubelet \\
  --config=/var/lib/kubelet/kubelet-config.yaml \\
  --image-pull-progress-deadline=2m \\
  --kubeconfig=/var/lib/kubelet/kubeconfig \\
  --tls-cert-file=/var/lib/kubelet/${HOSTNAME}.crt \\
  --tls-private-key-file=/var/lib/kubelet/${HOSTNAME}.key \\
  --network-plugin=cni \\
  --register-node=true \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Configure the Kubernetes Proxy

```
labc-worker-2$ sudo mv labc-kube-proxy.kubeconfig /var/lib/kube-proxy/kubeconfig
```

Create the `kube-proxy-config.yaml` configuration file:

```
labc-worker-2$ cat <<EOF | sudo tee /var/lib/kube-proxy/kube-proxy-config.yaml
kind: KubeProxyConfiguration
apiVersion: kubeproxy.config.k8s.io/v1alpha1
clientConnection:
  kubeconfig: "/var/lib/kube-proxy/kubeconfig"
mode: "iptables"
clusterCIDR: "192.168.5.0/24"
EOF
```

Create the `kube-proxy.service` systemd unit file:

```
labc-worker-2$ cat <<EOF | sudo tee /etc/systemd/system/kube-proxy.service
[Unit]
Description=Kubernetes Kube Proxy
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-proxy \\
  --config=/var/lib/kube-proxy/kube-proxy-config.yaml
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Start the Worker Services

```
{
  sudo systemctl daemon-reload
  sudo systemctl enable kubelet kube-proxy
  sudo systemctl start kubelet kube-proxy
}
```

> Remember to run the above commands on worker node: `labc-worker-2`

## Verification
On labc-master-1:

List the registered Kubernetes nodes from the master node:

```
labc-master-1$ kubectl get nodes --kubeconfig admin.kubeconfig
```

> output

```
NAME       STATUS     ROLES    AGE   VERSION
labc-worker-1   NotReady   <none>   93s   v1.13.0
labc-worker-2   NotReady   <none>   93s   v1.13.0
```

> Note: It is OK for the worker node to be in a NotReady state.
  That is because we haven't configured Networking yet.

Optional: At this point you may run the certificate verification script to make sure all certificates are configured correctly. Follow the instructions [here](verify-certificates.md)

Next: [TLS Bootstrapping Kubernetes Workers](10-tls-bootstrapping-kubernetes-workers.md)


#### LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D
#### LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D labd
#### LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D

# Bootstrapping the Kubernetes Worker Nodes

In this lab you will bootstrap 2 Kubernetes worker nodes. We already have [Docker](https://www.docker.com) installed on these nodes.

We will now install the kubernetes components
- [kubelet](https://kubernetes.io/docs/admin/kubelet)
- [kube-proxy](https://kubernetes.io/docs/concepts/cluster-bdministration/proxies).

## Prerequisites

The Certificates and Configuration are created on `k8smanagement` node and then copied over to workers using `scp`. 
Once this is done, the commands are to be run on first worker instance: `labd-worker-1` and `labd-worker-2`. Login to first worker instance using SSH Terminal.

### Provisioning  Kubelet Client Certificates

Kubernetes uses a [special-purpose authorization mode](https://kubernetes.io/docs/admin/authorization/node/) called Node Authorizer, that specifically authorizes API requests made by [Kubelets](https://kubernetes.io/docs/concepts/overview/components/#kubelet). In order to be authorized by the Node Authorizer, Kubelets must use a credential that identifies them as being in the `system:nodes` group, with a username of `system:node:<nodeName>`. In this section you will create a certificate for each Kubernetes worker node that meets the Node Authorizer requirements.

Generate a certificate and private key for one worker node:

On management:

```
cat > labd-openssl-labd-worker-1.cnf <<EOF
[req]
req_extensions = v3_req
distinguished_name = req_distinguished_name
[req_distinguished_name]
[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @alt_names
[alt_names]
DNS.1 = labd-worker-1
IP.1 = 192.168.5.56
EOF

openssl genrsa -out labd-worker-1.key 2048
openssl req -new -key labd-worker-1.key -subj "/CN=system:node:labd-worker-1/O=system:nodes" -out labd-worker-1.csr -config labd-openssl-labd-worker-1.cnf
openssl x509 -req -in labd-worker-1.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out labd-worker-1.crt -extensions v3_req -extfile labd-openssl-labd-worker-1.cnf -days 1000
```

Results:

```
labd-worker-1.key
labd-worker-1.crt
```
```
cat > labd-openssl-labd-worker-2.cnf <<EOF
[req]
req_extensions = v3_req
distinguished_name = req_distinguished_name
[req_distinguished_name]
[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @alt_names
[alt_names]
DNS.1 = labd-worker-2
IP.1 = 192.168.5.57
EOF

openssl genrsa -out labd-worker-2.key 2048
openssl req -new -key labd-worker-2.key -subj "/CN=system:node:labd-worker-2/O=system:nodes" -out labd-worker-2.csr -config labd-openssl-labd-worker-2.cnf
openssl x509 -req -in labd-worker-2.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out labd-worker-2.crt -extensions v3_req -extfile labd-openssl-labd-worker-2.cnf -days 1000
```

Results:

```
labd-worker-2.key
labd-worker-2.crt
```

### The kubelet Kubernetes Configuration File

When generating kubeconfig files for Kubelets the client certificate matching the Kubelet's node name must be used. This will ensure Kubelets are properly authorized by the Kubernetes [Node Authorizer](https://kubernetes.io/docs/admin/authorization/node/).

Get the kub-api server load-balancer IP.
```
LOADBALANCER_ADDRESS=192.168.5.60
```

Generate a kubeconfig file for the first worker node. 

On management:
```
{
  kubectl config set-cluster hardway-cluster-d \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://${LOADBALANCER_ADDRESS}:6443 \
    --kubeconfig=labd-worker-1.kubeconfig

  kubectl config set-credentials system:node:labd-worker-1 \
    --client-certificate=labd-worker-1.crt \
    --client-key=labd-worker-1.key \
    --embed-certs=true \
    --kubeconfig=labd-worker-1.kubeconfig

  kubectl config set-context default \
    --cluster=hardway-cluster-d \
    --user=system:node:labd-worker-1 \
    --kubeconfig=labd-worker-1.kubeconfig

  kubectl config use-context default --kubeconfig=labd-worker-1.kubeconfig
}
```

Results:

```
labd-worker-1.kubeconfig
```

{
  kubectl config set-cluster hardway-cluster-d \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://${LOADBALANCER_ADDRESS}:6443 \
    --kubeconfig=labd-worker-2.kubeconfig

  kubectl config set-credentials system:node:labd-worker-2 \
    --client-certificate=labd-worker-2.crt \
    --client-key=labd-worker-2.key \
    --embed-certs=true \
    --kubeconfig=labd-worker-2.kubeconfig

  kubectl config set-context default \
    --cluster=hardway-cluster-d \
    --user=system:node:labd-worker-2 \
    --kubeconfig=labd-worker-2.kubeconfig

  kubectl config use-context default --kubeconfig=labd-worker-2.kubeconfig
}
```

Results:

```
labd-worker-2.kubeconfig
```

### Copy certificates, private keys and kubeconfig files to the worker node:
On management:
```
scp ca.crt labd-worker-1.crt labd-worker-1.key labd-worker-1.kubeconfig labd-worker-1:~/
scp ca.crt labd-worker-2.crt labd-worker-2.key labd-worker-2.kubeconfig labd-worker-2:~/
```

### Download and Install Worker Binaries

Going forward all activities are to be done on the `labd-worker-1` node.

On labd-worker-1:
```
labd-worker-1$ wget -q --show-progress --https-only --timestamping \
  https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kubectl \
  https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kube-proxy \
  https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kubelet
```

Reference: https://kubernetes.io/docs/setup/release/#node-binaries

Create the installation directories:

```
labd-worker-1$ sudo mkdir -p \
  /etc/cni/net.d \
  /opt/cni/bin \
  /var/lib/kubelet \
  /var/lib/kube-proxy \
  /var/lib/kubernetes \
  /var/run/kubernetes
```

Install the worker binaries:

```
{
  chmod +x kubectl kube-proxy kubelet
  sudo mv kubectl kube-proxy kubelet /usr/local/bin/
}
```

### Configure the Kubelet

```
{
  sudo mv ${HOSTNAME}.key ${HOSTNAME}.crt /var/lib/kubelet/
  sudo mv ${HOSTNAME}.kubeconfig /var/lib/kubelet/kubeconfig
  sudo mv ca.crt /var/lib/kubernetes/
}
```

Create the `kubelet-config.yaml` configuration file:

```
cat <<EOF | sudo tee /var/lib/kubelet/kubelet-config.yaml
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
    enabled: false
  webhook:
    enabled: true
  x509:
    clientCAFile: "/var/lib/kubernetes/ca.crt"
authorization:
  mode: Webhook
clusterDomain: "cluster.local"
clusterDNS:
  - "10.96.0.10"
resolvConf: "/run/systemd/resolve/resolv.conf"
runtimeRequestTimeout: "15m"
EOF
```

> The `resolvConf` configuration is used to avoid loops when using CoreDNS for service discovery on systems running `systemd-resolved`.

Create the `kubelet.service` systemd unit file:

```
cat <<EOF | sudo tee /etc/systemd/system/kubelet.service
[Unit]
Description=Kubernetes Kubelet
Documentation=https://github.com/kubernetes/kubernetes
After=docker.service
Requires=docker.service

[Service]
ExecStart=/usr/local/bin/kubelet \\
  --config=/var/lib/kubelet/kubelet-config.yaml \\
  --image-pull-progress-deadline=2m \\
  --kubeconfig=/var/lib/kubelet/kubeconfig \\
  --tls-cert-file=/var/lib/kubelet/${HOSTNAME}.crt \\
  --tls-private-key-file=/var/lib/kubelet/${HOSTNAME}.key \\
  --network-plugin=cni \\
  --register-node=true \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Configure the Kubernetes Proxy

```
labd-worker-1$ sudo mv labd-kube-proxy.kubeconfig /var/lib/kube-proxy/kubeconfig
```

Create the `kube-proxy-config.yaml` configuration file:

```
labd-worker-1$ cat <<EOF | sudo tee /var/lib/kube-proxy/kube-proxy-config.yaml
kind: KubeProxyConfiguration
apiVersion: kubeproxy.config.k8s.io/v1alpha1
clientConnection:
  kubeconfig: "/var/lib/kube-proxy/kubeconfig"
mode: "iptables"
clusterCIDR: "192.168.5.0/24"
EOF
```

Create the `kube-proxy.service` systemd unit file:

```
labd-worker-1$ cat <<EOF | sudo tee /etc/systemd/system/kube-proxy.service
[Unit]
Description=Kubernetes Kube Proxy
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-proxy \\
  --config=/var/lib/kube-proxy/kube-proxy-config.yaml
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Start the Worker Services

```
{
  sudo systemctl daemon-reload
  sudo systemctl enable kubelet kube-proxy
  sudo systemctl start kubelet kube-proxy
}
```

> Remember to run the above commands on worker node: `labd-worker-1`


On labd-worker-2:  
```
labd-worker-2$ wget -q --show-progress --https-only --timestamping \
  https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kubectl \
  https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kube-proxy \
  https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kubelet
```

Reference: https://kubernetes.io/docs/setup/release/#node-binaries

Create the installation directories:

```
labd-worker-2$ sudo mkdir -p \
  /etc/cni/net.d \
  /opt/cni/bin \
  /var/lib/kubelet \
  /var/lib/kube-proxy \
  /var/lib/kubernetes \
  /var/run/kubernetes
```

Install the worker binaries:

```
{
  chmod +x kubectl kube-proxy kubelet
  sudo mv kubectl kube-proxy kubelet /usr/local/bin/
}
```

### Configure the Kubelet

```
{
  sudo mv ${HOSTNAME}.key ${HOSTNAME}.crt /var/lib/kubelet/
  sudo mv ${HOSTNAME}.kubeconfig /var/lib/kubelet/kubeconfig
  sudo mv ca.crt /var/lib/kubernetes/
}
```

Create the `kubelet-config.yaml` configuration file:

```
labd-worker-2$ cat <<EOF | sudo tee /var/lib/kubelet/kubelet-config.yaml
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
    enabled: false
  webhook:
    enabled: true
  x509:
    clientCAFile: "/var/lib/kubernetes/ca.crt"
authorization:
  mode: Webhook
clusterDomain: "cluster.local"
clusterDNS:
  - "10.96.0.10"
resolvConf: "/run/systemd/resolve/resolv.conf"
runtimeRequestTimeout: "15m"
EOF
```

> The `resolvConf` configuration is used to avoid loops when using CoreDNS for service discovery on systems running `systemd-resolved`.

Create the `kubelet.service` systemd unit file:

```
labd-worker-2$ cat <<EOF | sudo tee /etc/systemd/system/kubelet.service
[Unit]
Description=Kubernetes Kubelet
Documentation=https://github.com/kubernetes/kubernetes
After=docker.service
Requires=docker.service

[Service]
ExecStart=/usr/local/bin/kubelet \\
  --config=/var/lib/kubelet/kubelet-config.yaml \\
  --image-pull-progress-deadline=2m \\
  --kubeconfig=/var/lib/kubelet/kubeconfig \\
  --tls-cert-file=/var/lib/kubelet/${HOSTNAME}.crt \\
  --tls-private-key-file=/var/lib/kubelet/${HOSTNAME}.key \\
  --network-plugin=cni \\
  --register-node=true \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Configure the Kubernetes Proxy

```
labd-worker-2$ sudo mv labd-kube-proxy.kubeconfig /var/lib/kube-proxy/kubeconfig
```

Create the `kube-proxy-config.yaml` configuration file:

```
labd-worker-2$ cat <<EOF | sudo tee /var/lib/kube-proxy/kube-proxy-config.yaml
kind: KubeProxyConfiguration
apiVersion: kubeproxy.config.k8s.io/v1alpha1
clientConnection:
  kubeconfig: "/var/lib/kube-proxy/kubeconfig"
mode: "iptables"
clusterCIDR: "192.168.5.0/24"
EOF
```

Create the `kube-proxy.service` systemd unit file:

```
labd-worker-2$ cat <<EOF | sudo tee /etc/systemd/system/kube-proxy.service
[Unit]
Description=Kubernetes Kube Proxy
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-proxy \\
  --config=/var/lib/kube-proxy/kube-proxy-config.yaml
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Start the Worker Services

```
{
  sudo systemctl daemon-reload
  sudo systemctl enable kubelet kube-proxy
  sudo systemctl start kubelet kube-proxy
}
```

> Remember to run the above commands on worker node: `labd-worker-2`

## Verification
On labd-master-1:

List the registered Kubernetes nodes from the master node:

```
labd-master-1$ kubectl get nodes --kubeconfig admin.kubeconfig
```

> output

```
NAME       STATUS     ROLES    AGE   VERSION
labd-worker-1   NotReady   <none>   93s   v1.13.0
labd-worker-2   NotReady   <none>   93s   v1.13.0
```

> Note: It is OK for the worker node to be in a NotReady state.
  That is because we haven't configured Networking yet.

Optional: At this point you may run the certificate verification script to make sure all certificates are configured correctly. Follow the instructions [here](verify-certificates.md)

Next: [TLS Bootstrapping Kubernetes Workers](10-tls-bootstrapping-kubernetes-workers.md)



#### LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E
#### LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E labe
#### LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E

# Bootstrapping the Kubernetes Worker Nodes

In this lab you will bootstrap 2 Kubernetes worker nodes. We already have [Docker](https://www.docker.com) installed on these nodes.

We will now install the kubernetes components
- [kubelet](https://kubernetes.io/docs/admin/kubelet)
- [kube-proxy](https://kubernetes.io/docs/concepts/cluster-bdministration/proxies).

## Prerequisites

The Certificates and Configuration are created on `k8smanagement` node and then copied over to workers using `scp`. 
Once this is done, the commands are to be run on first worker instance: `labe-worker-1` and `labe-worker-2`. Login to first worker instance using SSH Terminal.

### Provisioning  Kubelet Client Certificates

Kubernetes uses a [special-purpose authorization mode](https://kubernetes.io/docs/admin/authorization/node/) called Node Authorizer, that specifically authorizes API requests made by [Kubelets](https://kubernetes.io/docs/concepts/overview/components/#kubelet). In order to be authorized by the Node Authorizer, Kubelets must use a credential that identifies them as being in the `system:nodes` group, with a username of `system:node:<nodeName>`. In this section you will create a certificate for each Kubernetes worker node that meets the Node Authorizer requirements.

Generate a certificate and private key for one worker node:

On management:

```
master-1$ cat > labe-openssl-labe-worker-1.cnf <<EOF
[req]
req_extensions = v3_req
distinguished_name = req_distinguished_name
[req_distinguished_name]
[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @alt_names
[alt_names]
DNS.1 = labe-worker-1
IP.1 = 192.168.5.71
EOF

openssl genrsa -out labe-worker-1.key 2048
openssl req -new -key labe-worker-1.key -subj "/CN=system:node:labe-worker-1/O=system:nodes" -out labe-worker-1.csr -config labe-openssl-labe-worker-1.cnf
openssl x509 -req -in labe-worker-1.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out labe-worker-1.crt -extensions v3_req -extfile labe-openssl-labe-worker-1.cnf -days 1000
```

Results:

```
labe-worker-1.key
labe-worker-1.crt
```
```
master-1$ cat > labe-openssl-labe-worker-2.cnf <<EOF
[req]
req_extensions = v3_req
distinguished_name = req_distinguished_name
[req_distinguished_name]
[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @alt_names
[alt_names]
DNS.1 = labe-worker-2
IP.1 = 192.168.5.72
EOF

openssl genrsa -out labe-worker-2.key 2048
openssl req -new -key labe-worker-2.key -subj "/CN=system:node:labe-worker-2/O=system:nodes" -out labe-worker-2.csr -config labe-openssl-labe-worker-2.cnf
openssl x509 -req -in labe-worker-2.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out labe-worker-2.crt -extensions v3_req -extfile labe-openssl-labe-worker-2.cnf -days 1000
```

Results:

```
labe-worker-2.key
labe-worker-2.crt
```

### The kubelet Kubernetes Configuration File

When generating kubeconfig files for Kubelets the client certificate matching the Kubelet's node name must be used. This will ensure Kubelets are properly authorized by the Kubernetes [Node Authorizer](https://kubernetes.io/docs/admin/authorization/node/).

Get the kub-api server load-balancer IP.
```
LOADBALANCER_ADDRESS=192.168.5.75
```

Generate a kubeconfig file for the first worker node. 

On management:
```
{
  kubectl config set-cluster hardway-cluster-e \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://${LOADBALANCER_ADDRESS}:6443 \
    --kubeconfig=labe-worker-1.kubeconfig

  kubectl config set-credentials system:node:labe-worker-1 \
    --client-certificate=labe-worker-1.crt \
    --client-key=labe-worker-1.key \
    --embed-certs=true \
    --kubeconfig=labe-worker-1.kubeconfig

  kubectl config set-context default \
    --cluster=hardway-cluster-e \
    --user=system:node:labe-worker-1 \
    --kubeconfig=labe-worker-1.kubeconfig

  kubectl config use-context default --kubeconfig=labe-worker-1.kubeconfig
}
```

Results:

```
labe-worker-1.kubeconfig
```

{
  kubectl config set-cluster hardway-cluster-e \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://${LOADBALANCER_ADDRESS}:6443 \
    --kubeconfig=labe-worker-2.kubeconfig

  kubectl config set-credentials system:node:labe-worker-2 \
    --client-certificate=labe-worker-2.crt \
    --client-key=labe-worker-2.key \
    --embed-certs=true \
    --kubeconfig=labe-worker-2.kubeconfig

  kubectl config set-context default \
    --cluster=hardway-cluster-e \
    --user=system:node:labe-worker-2 \
    --kubeconfig=labe-worker-2.kubeconfig

  kubectl config use-context default --kubeconfig=labe-worker-2.kubeconfig
}
```

Results:

```
labe-worker-2.kubeconfig
```

### Copy certificates, private keys and kubeconfig files to the worker node:
On management:
```
scp ca.crt labe-worker-1.crt labe-worker-1.key labe-worker-1.kubeconfig labe-worker-1:~/
scp ca.crt labe-worker-2.crt labe-worker-2.key labe-worker-2.kubeconfig labe-worker-2:~/
```

### Download and Install Worker Binaries

Going forward all activities are to be done on the `labe-worker-1` node.

On labe-worker-1:
```
labe-worker-1$ wget -q --show-progress --https-only --timestamping \
  https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kubectl \
  https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kube-proxy \
  https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kubelet
```

Reference: https://kubernetes.io/docs/setup/release/#node-binaries

Create the installation directories:

```
labe-worker-1$ sudo mkdir -p \
  /etc/cni/net.d \
  /opt/cni/bin \
  /var/lib/kubelet \
  /var/lib/kube-proxy \
  /var/lib/kubernetes \
  /var/run/kubernetes
```

Install the worker binaries:

```
{
  chmod +x kubectl kube-proxy kubelet
  sudo mv kubectl kube-proxy kubelet /usr/local/bin/
}
```

### Configure the Kubelet

```
{
  sudo mv ${HOSTNAME}.key ${HOSTNAME}.crt /var/lib/kubelet/
  sudo mv ${HOSTNAME}.kubeconfig /var/lib/kubelet/kubeconfig
  sudo mv ca.crt /var/lib/kubernetes/
}
```

Create the `kubelet-config.yaml` configuration file:

```
labe-worker-1$ cat <<EOF | sudo tee /var/lib/kubelet/kubelet-config.yaml
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
    enabled: false
  webhook:
    enabled: true
  x509:
    clientCAFile: "/var/lib/kubernetes/ca.crt"
authorization:
  mode: Webhook
clusterDomain: "cluster.local"
clusterDNS:
  - "10.96.0.10"
resolvConf: "/run/systemd/resolve/resolv.conf"
runtimeRequestTimeout: "15m"
EOF
```

> The `resolvConf` configuration is used to avoid loops when using CoreDNS for service discovery on systems running `systemd-resolved`.

Create the `kubelet.service` systemd unit file:

```
labe-worker-1$ cat <<EOF | sudo tee /etc/systemd/system/kubelet.service
[Unit]
Description=Kubernetes Kubelet
Documentation=https://github.com/kubernetes/kubernetes
After=docker.service
Requires=docker.service

[Service]
ExecStart=/usr/local/bin/kubelet \\
  --config=/var/lib/kubelet/kubelet-config.yaml \\
  --image-pull-progress-deadline=2m \\
  --kubeconfig=/var/lib/kubelet/kubeconfig \\
  --tls-cert-file=/var/lib/kubelet/${HOSTNAME}.crt \\
  --tls-private-key-file=/var/lib/kubelet/${HOSTNAME}.key \\
  --network-plugin=cni \\
  --register-node=true \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Configure the Kubernetes Proxy

```
labe-worker-1$ sudo mv labe-kube-proxy.kubeconfig /var/lib/kube-proxy/kubeconfig
```

Create the `kube-proxy-config.yaml` configuration file:

```
labe-worker-1$ cat <<EOF | sudo tee /var/lib/kube-proxy/kube-proxy-config.yaml
kind: KubeProxyConfiguration
apiVersion: kubeproxy.config.k8s.io/v1alpha1
clientConnection:
  kubeconfig: "/var/lib/kube-proxy/kubeconfig"
mode: "iptables"
clusterCIDR: "192.168.5.0/24"
EOF
```

Create the `kube-proxy.service` systemd unit file:

```
labe-worker-1$ cat <<EOF | sudo tee /etc/systemd/system/kube-proxy.service
[Unit]
Description=Kubernetes Kube Proxy
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-proxy \\
  --config=/var/lib/kube-proxy/kube-proxy-config.yaml
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Start the Worker Services

```
{
  sudo systemctl daemon-reload
  sudo systemctl enable kubelet kube-proxy
  sudo systemctl start kubelet kube-proxy
}
```

> Remember to run the above commands on worker node: `labe-worker-1`


On labe-worker-2:  
```
labe-worker-2$ wget -q --show-progress --https-only --timestamping \
  https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kubectl \
  https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kube-proxy \
  https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kubelet
```

Reference: https://kubernetes.io/docs/setup/release/#node-binaries

Create the installation directories:

```
labe-worker-2$ sudo mkdir -p \
  /etc/cni/net.d \
  /opt/cni/bin \
  /var/lib/kubelet \
  /var/lib/kube-proxy \
  /var/lib/kubernetes \
  /var/run/kubernetes
```

Install the worker binaries:

```
{
  chmod +x kubectl kube-proxy kubelet
  sudo mv kubectl kube-proxy kubelet /usr/local/bin/
}
```

### Configure the Kubelet

```
{
  sudo mv ${HOSTNAME}.key ${HOSTNAME}.crt /var/lib/kubelet/
  sudo mv ${HOSTNAME}.kubeconfig /var/lib/kubelet/kubeconfig
  sudo mv ca.crt /var/lib/kubernetes/
}
```

Create the `kubelet-config.yaml` configuration file:

```
labe-worker-2$ cat <<EOF | sudo tee /var/lib/kubelet/kubelet-config.yaml
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
    enabled: false
  webhook:
    enabled: true
  x509:
    clientCAFile: "/var/lib/kubernetes/ca.crt"
authorization:
  mode: Webhook
clusterDomain: "cluster.local"
clusterDNS:
  - "10.96.0.10"
resolvConf: "/run/systemd/resolve/resolv.conf"
runtimeRequestTimeout: "15m"
EOF
```

> The `resolvConf` configuration is used to avoid loops when using CoreDNS for service discovery on systems running `systemd-resolved`.

Create the `kubelet.service` systemd unit file:

```
labe-worker-2$ cat <<EOF | sudo tee /etc/systemd/system/kubelet.service
[Unit]
Description=Kubernetes Kubelet
Documentation=https://github.com/kubernetes/kubernetes
After=docker.service
Requires=docker.service

[Service]
ExecStart=/usr/local/bin/kubelet \\
  --config=/var/lib/kubelet/kubelet-config.yaml \\
  --image-pull-progress-deadline=2m \\
  --kubeconfig=/var/lib/kubelet/kubeconfig \\
  --tls-cert-file=/var/lib/kubelet/${HOSTNAME}.crt \\
  --tls-private-key-file=/var/lib/kubelet/${HOSTNAME}.key \\
  --network-plugin=cni \\
  --register-node=true \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Configure the Kubernetes Proxy

```
labe-worker-2$ sudo mv labe-kube-proxy.kubeconfig /var/lib/kube-proxy/kubeconfig
```

Create the `kube-proxy-config.yaml` configuration file:

```
labe-worker-2$ cat <<EOF | sudo tee /var/lib/kube-proxy/kube-proxy-config.yaml
kind: KubeProxyConfiguration
apiVersion: kubeproxy.config.k8s.io/v1alpha1
clientConnection:
  kubeconfig: "/var/lib/kube-proxy/kubeconfig"
mode: "iptables"
clusterCIDR: "192.168.5.0/24"
EOF
```

Create the `kube-proxy.service` systemd unit file:

```
labe-worker-2$ cat <<EOF | sudo tee /etc/systemd/system/kube-proxy.service
[Unit]
Description=Kubernetes Kube Proxy
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-proxy \\
  --config=/var/lib/kube-proxy/kube-proxy-config.yaml
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Start the Worker Services

```
{
  sudo systemctl daemon-reload
  sudo systemctl enable kubelet kube-proxy
  sudo systemctl start kubelet kube-proxy
}
```

> Remember to run the above commands on worker node: `labe-worker-2`

## Verification
On labe-master-1:

List the registered Kubernetes nodes from the master node:

```
labe-master-1$ kubectl get nodes --kubeconfig admin.kubeconfig
```

> output

```
NAME       STATUS     ROLES    AGE   VERSION
labe-worker-1   NotReady   <none>   93s   v1.13.0
labe-worker-2   NotReady   <none>   93s   v1.13.0
```

> Note: It is OK for the worker node to be in a NotReady state.
  That is because we haven't configured Networking yet.

Optional: At this point you may run the certificate verification script to make sure all certificates are configured correctly. Follow the instructions [here](verify-certificates.md)

Next: [TLS Bootstrapping Kubernetes Workers](10-tls-bootstrapping-kubernetes-workers.md)


#### LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F
#### LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F labf
#### LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F

# Bootstrapping the Kubernetes Worker Nodes

In this lab you will bootstrap 2 Kubernetes worker nodes. We already have [Docker](https://www.docker.com) installed on these nodes.

We will now install the kubernetes components
- [kubelet](https://kubernetes.io/docs/admin/kubelet)
- [kube-proxy](https://kubernetes.io/docs/concepts/cluster-bdministration/proxies).

## Prerequisites

The Certificates and Configuration are created on `k8smanagement` node and then copied over to workers using `scp`. 
Once this is done, the commands are to be run on first worker instance: `labf-worker-1` and `labf-worker-2`. Login to first worker instance using SSH Terminal.

### Provisioning  Kubelet Client Certificates

Kubernetes uses a [special-purpose authorization mode](https://kubernetes.io/docs/admin/authorization/node/) called Node Authorizer, that specifically authorizes API requests made by [Kubelets](https://kubernetes.io/docs/concepts/overview/components/#kubelet). In order to be authorized by the Node Authorizer, Kubelets must use a credential that identifies them as being in the `system:nodes` group, with a username of `system:node:<nodeName>`. In this section you will create a certificate for each Kubernetes worker node that meets the Node Authorizer requirements.

Generate a certificate and private key for one worker node:

On management:

```
master-1$ cat > labf-openssl-labf-worker-1.cnf <<EOF
[req]
req_extensions = v3_req
distinguished_name = req_distinguished_name
[req_distinguished_name]
[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @alt_names
[alt_names]
DNS.1 = labf-worker-1
IP.1 = 192.168.5.86
EOF

openssl genrsa -out labf-worker-1.key 2048
openssl req -new -key labf-worker-1.key -subj "/CN=system:node:labf-worker-1/O=system:nodes" -out labf-worker-1.csr -config labf-openssl-labf-worker-1.cnf
openssl x509 -req -in labf-worker-1.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out labf-worker-1.crt -extensions v3_req -extfile labf-openssl-labf-worker-1.cnf -days 1000
```

Results:

```
labf-worker-1.key
labf-worker-1.crt
```
```
master-1$ cat > labf-openssl-labf-worker-2.cnf <<EOF
[req]
req_extensions = v3_req
distinguished_name = req_distinguished_name
[req_distinguished_name]
[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @alt_names
[alt_names]
DNS.1 = labf-worker-2
IP.1 = 192.168.5.87
EOF

openssl genrsa -out labf-worker-2.key 2048
openssl req -new -key labf-worker-2.key -subj "/CN=system:node:labf-worker-2/O=system:nodes" -out labf-worker-2.csr -config labf-openssl-labf-worker-2.cnf
openssl x509 -req -in labf-worker-2.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out labf-worker-2.crt -extensions v3_req -extfile labf-openssl-labf-worker-2.cnf -days 1000
```

Results:

```
labf-worker-2.key
labf-worker-2.crt
```

### The kubelet Kubernetes Configuration File

When generating kubeconfig files for Kubelets the client certificate matching the Kubelet's node name must be used. This will ensure Kubelets are properly authorized by the Kubernetes [Node Authorizer](https://kubernetes.io/docs/admin/authorization/node/).

Get the kub-api server load-balancer IP.
```
LOADBALANCER_ADDRESS=192.168.5.90
```

Generate a kubeconfig file for the first worker node. 

On management:
```
{
  kubectl config set-cluster hardway-cluster-f \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://${LOADBALANCER_ADDRESS}:6443 \
    --kubeconfig=labf-worker-1.kubeconfig

  kubectl config set-credentials system:node:labf-worker-1 \
    --client-certificate=labf-worker-1.crt \
    --client-key=labf-worker-1.key \
    --embed-certs=true \
    --kubeconfig=labf-worker-1.kubeconfig

  kubectl config set-context default \
    --cluster=hardway-cluster-f \
    --user=system:node:labf-worker-1 \
    --kubeconfig=labf-worker-1.kubeconfig

  kubectl config use-context default --kubeconfig=labf-worker-1.kubeconfig
}
```

Results:

```
labf-worker-1.kubeconfig
```

{
  kubectl config set-cluster hardway-cluster-f \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://${LOADBALANCER_ADDRESS}:6443 \
    --kubeconfig=labf-worker-2.kubeconfig

  kubectl config set-credentials system:node:labf-worker-2 \
    --client-certificate=labf-worker-2.crt \
    --client-key=labf-worker-2.key \
    --embed-certs=true \
    --kubeconfig=labf-worker-2.kubeconfig

  kubectl config set-context default \
    --cluster=hardway-cluster-f \
    --user=system:node:labf-worker-2 \
    --kubeconfig=labf-worker-2.kubeconfig

  kubectl config use-context default --kubeconfig=labf-worker-2.kubeconfig
}
```

Results:

```
labf-worker-2.kubeconfig
```

### Copy certificates, private keys and kubeconfig files to the worker node:
On management:
```
scp ca.crt labf-worker-1.crt labf-worker-1.key labf-worker-1.kubeconfig labf-worker-1:~/
scp ca.crt labf-worker-2.crt labf-worker-2.key labf-worker-2.kubeconfig labf-worker-2:~/
```

### Download and Install Worker Binaries

Going forward all activities are to be done on the `labf-worker-1` node.

On labf-worker-1:
```
labf-worker-1$ wget -q --show-progress --https-only --timestamping \
  https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kubectl \
  https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kube-proxy \
  https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kubelet
```

Reference: https://kubernetes.io/docs/setup/release/#node-binaries

Create the installation directories:

```
labf-worker-1$ sudo mkdir -p \
  /etc/cni/net.d \
  /opt/cni/bin \
  /var/lib/kubelet \
  /var/lib/kube-proxy \
  /var/lib/kubernetes \
  /var/run/kubernetes
```

Install the worker binaries:

```
{
  chmod +x kubectl kube-proxy kubelet
  sudo mv kubectl kube-proxy kubelet /usr/local/bin/
}
```

### Configure the Kubelet

```
{
  sudo mv ${HOSTNAME}.key ${HOSTNAME}.crt /var/lib/kubelet/
  sudo mv ${HOSTNAME}.kubeconfig /var/lib/kubelet/kubeconfig
  sudo mv ca.crt /var/lib/kubernetes/
}
```

Create the `kubelet-config.yaml` configuration file:

```
labf-worker-1$ cat <<EOF | sudo tee /var/lib/kubelet/kubelet-config.yaml
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
    enabled: false
  webhook:
    enabled: true
  x509:
    clientCAFile: "/var/lib/kubernetes/ca.crt"
authorization:
  mode: Webhook
clusterDomain: "cluster.local"
clusterDNS:
  - "10.96.0.10"
resolvConf: "/run/systemd/resolve/resolv.conf"
runtimeRequestTimeout: "15m"
EOF
```

> The `resolvConf` configuration is used to avoid loops when using CoreDNS for service discovery on systems running `systemd-resolved`.

Create the `kubelet.service` systemd unit file:

```
labf-worker-1$ cat <<EOF | sudo tee /etc/systemd/system/kubelet.service
[Unit]
Description=Kubernetes Kubelet
Documentation=https://github.com/kubernetes/kubernetes
After=docker.service
Requires=docker.service

[Service]
ExecStart=/usr/local/bin/kubelet \\
  --config=/var/lib/kubelet/kubelet-config.yaml \\
  --image-pull-progress-deadline=2m \\
  --kubeconfig=/var/lib/kubelet/kubeconfig \\
  --tls-cert-file=/var/lib/kubelet/${HOSTNAME}.crt \\
  --tls-private-key-file=/var/lib/kubelet/${HOSTNAME}.key \\
  --network-plugin=cni \\
  --register-node=true \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Configure the Kubernetes Proxy

```
labf-worker-1$ sudo mv labf-kube-proxy.kubeconfig /var/lib/kube-proxy/kubeconfig
```

Create the `kube-proxy-config.yaml` configuration file:

```
labf-worker-1$ cat <<EOF | sudo tee /var/lib/kube-proxy/kube-proxy-config.yaml
kind: KubeProxyConfiguration
apiVersion: kubeproxy.config.k8s.io/v1alpha1
clientConnection:
  kubeconfig: "/var/lib/kube-proxy/kubeconfig"
mode: "iptables"
clusterCIDR: "192.168.5.0/24"
EOF
```

Create the `kube-proxy.service` systemd unit file:

```
labf-worker-1$ cat <<EOF | sudo tee /etc/systemd/system/kube-proxy.service
[Unit]
Description=Kubernetes Kube Proxy
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-proxy \\
  --config=/var/lib/kube-proxy/kube-proxy-config.yaml
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Start the Worker Services

```
{
  sudo systemctl daemon-reload
  sudo systemctl enable kubelet kube-proxy
  sudo systemctl start kubelet kube-proxy
}
```

> Remember to run the above commands on worker node: `labf-worker-1`


On labf-worker-2:  
```
labf-worker-2$ wget -q --show-progress --https-only --timestamping \
  https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kubectl \
  https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kube-proxy \
  https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kubelet
```

Reference: https://kubernetes.io/docs/setup/release/#node-binaries

Create the installation directories:

```
labf-worker-2$ sudo mkdir -p \
  /etc/cni/net.d \
  /opt/cni/bin \
  /var/lib/kubelet \
  /var/lib/kube-proxy \
  /var/lib/kubernetes \
  /var/run/kubernetes
```

Install the worker binaries:

```
{
  chmod +x kubectl kube-proxy kubelet
  sudo mv kubectl kube-proxy kubelet /usr/local/bin/
}
```

### Configure the Kubelet

```
{
  sudo mv ${HOSTNAME}.key ${HOSTNAME}.crt /var/lib/kubelet/
  sudo mv ${HOSTNAME}.kubeconfig /var/lib/kubelet/kubeconfig
  sudo mv ca.crt /var/lib/kubernetes/
}
```

Create the `kubelet-config.yaml` configuration file:

```
labf-worker-2$ cat <<EOF | sudo tee /var/lib/kubelet/kubelet-config.yaml
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
    enabled: false
  webhook:
    enabled: true
  x509:
    clientCAFile: "/var/lib/kubernetes/ca.crt"
authorization:
  mode: Webhook
clusterDomain: "cluster.local"
clusterDNS:
  - "10.96.0.10"
resolvConf: "/run/systemd/resolve/resolv.conf"
runtimeRequestTimeout: "15m"
EOF
```

> The `resolvConf` configuration is used to avoid loops when using CoreDNS for service discovery on systems running `systemd-resolved`.

Create the `kubelet.service` systemd unit file:

```
labf-worker-2$ cat <<EOF | sudo tee /etc/systemd/system/kubelet.service
[Unit]
Description=Kubernetes Kubelet
Documentation=https://github.com/kubernetes/kubernetes
After=docker.service
Requires=docker.service

[Service]
ExecStart=/usr/local/bin/kubelet \\
  --config=/var/lib/kubelet/kubelet-config.yaml \\
  --image-pull-progress-deadline=2m \\
  --kubeconfig=/var/lib/kubelet/kubeconfig \\
  --tls-cert-file=/var/lib/kubelet/${HOSTNAME}.crt \\
  --tls-private-key-file=/var/lib/kubelet/${HOSTNAME}.key \\
  --network-plugin=cni \\
  --register-node=true \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Configure the Kubernetes Proxy

```
labf-worker-2$ sudo mv labf-kube-proxy.kubeconfig /var/lib/kube-proxy/kubeconfig
```

Create the `kube-proxy-config.yaml` configuration file:

```
labf-worker-2$ cat <<EOF | sudo tee /var/lib/kube-proxy/kube-proxy-config.yaml
kind: KubeProxyConfiguration
apiVersion: kubeproxy.config.k8s.io/v1alpha1
clientConnection:
  kubeconfig: "/var/lib/kube-proxy/kubeconfig"
mode: "iptables"
clusterCIDR: "192.168.5.0/24"
EOF
```

Create the `kube-proxy.service` systemd unit file:

```
labf-worker-2$ cat <<EOF | sudo tee /etc/systemd/system/kube-proxy.service
[Unit]
Description=Kubernetes Kube Proxy
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-proxy \\
  --config=/var/lib/kube-proxy/kube-proxy-config.yaml
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Start the Worker Services

```
{
  sudo systemctl daemon-reload
  sudo systemctl enable kubelet kube-proxy
  sudo systemctl start kubelet kube-proxy
}
```

> Remember to run the above commands on worker node: `labf-worker-2`

## Verification
On labf-master-1:

List the registered Kubernetes nodes from the master node:

```
labf-master-1$ kubectl get nodes --kubeconfig admin.kubeconfig
```

> output

```
NAME       STATUS     ROLES    AGE   VERSION
labf-worker-1   NotReady   <none>   93s   v1.13.0
labf-worker-2   NotReady   <none>   93s   v1.13.0
```

> Note: It is OK for the worker node to be in a NotReady state.
  That is because we haven't configured Networking yet.

Optional: At this point you may run the certificate verification script to make sure all certificates are configured correctly. Follow the instructions [here](verify-certificates.md)

Next: [TLS Bootstrapping Kubernetes Workers](10-tls-bootstrapping-kubernetes-workers.md)
