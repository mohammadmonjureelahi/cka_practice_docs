#### LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A
#### LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A clustera
#### LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A

# Bootstrapping the Kubernetes Worker Nodes

In this lab you will bootstrap 2 Kubernetes worker nodes. We already have [Docker](https://www.docker.com) installed on these nodes.

We will now install the kubernetes components
- [kubelet](https://kubernetes.io/docs/admin/kubelet)
- [kube-proxy](https://kubernetes.io/docs/concepts/cluster-administration/proxies).

## Prerequisites

The Certificates and Configuration are created on `k8smanagement` node and then copied over to workers using `scp`. 
Once this is done, the commands are to be run on first worker instance: `clustera-worker-1`, `clustera-worker-2`, `clustera-worker-3`, `clustera-worker-4` and `clustera-worker-5`. Login to first worker instance using SSH Terminal.

### Provisioning  Kubelet Client Certificates

Kubernetes uses a [special-purpose authorization mode](https://kubernetes.io/docs/admin/authorization/node/) called Node Authorizer, that specifically authorizes API requests made by [Kubelets](https://kubernetes.io/docs/concepts/overview/components/#kubelet). In order to be authorized by the Node Authorizer, Kubelets must use a credential that identifies them as being in the `system:nodes` group, with a username of `system:node:<nodeName>`. In this section you will create a certificate for each Kubernetes worker node that meets the Node Authorizer requirements.

Generate a certificate and private key for one worker node:

On management:

```
cat > clustera-openssl-clustera-worker-1.cnf <<EOF
[req]
req_extensions = v3_req
distinguished_name = req_distinguished_name
[req_distinguished_name]
[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @alt_names
[alt_names]
DNS.1 = clustera-worker-1
IP.1 = 192.168.5.15
EOF

openssl genrsa -out clustera-worker-1.key 2048
openssl req -new -key clustera-worker-1.key -subj "/CN=system:node:clustera-worker-1/O=system:nodes" -out clustera-worker-1.csr -config clustera-openssl-clustera-worker-1.cnf
openssl x509 -req -in clustera-worker-1.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out clustera-worker-1.crt -extensions v3_req -extfile clustera-openssl-clustera-worker-1.cnf -days 1000
```

Results:

```
clustera-worker-1.key
clustera-worker-1.crt
```
```
cat > clustera-openssl-clustera-worker-2.cnf <<EOF
[req]
req_extensions = v3_req
distinguished_name = req_distinguished_name
[req_distinguished_name]
[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @alt_names
[alt_names]
DNS.1 = clustera-worker-2
IP.1 = 192.168.5.16
EOF

openssl genrsa -out clustera-worker-2.key 2048
openssl req -new -key clustera-worker-2.key -subj "/CN=system:node:clustera-worker-2/O=system:nodes" -out clustera-worker-2.csr -config clustera-openssl-clustera-worker-2.cnf
openssl x509 -req -in clustera-worker-2.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out clustera-worker-2.crt -extensions v3_req -extfile clustera-openssl-clustera-worker-2.cnf -days 1000
```

Results:

```
clustera-worker-2.key
clustera-worker-2.crt
```

cat > clustera-openssl-clustera-worker-3.cnf <<EOF
[req]
req_extensions = v3_req
distinguished_name = req_distinguished_name
[req_distinguished_name]
[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @alt_names
[alt_names]
DNS.1 = clustera-worker-3
IP.1 = 192.168.5.17
EOF

openssl genrsa -out clustera-worker-3.key 2048
openssl req -new -key clustera-worker-3.key -subj "/CN=system:node:clustera-worker-3/O=system:nodes" -out clustera-worker-3.csr -config clustera-openssl-clustera-worker-3.cnf
openssl x509 -req -in clustera-worker-3.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out clustera-worker-3.crt -extensions v3_req -extfile clustera-openssl-clustera-worker-3.cnf -days 1000
```

Results:

```
clustera-worker-3.key
clustera-worker-3.crt
```
cat > clustera-openssl-clustera-worker-4.cnf <<EOF
[req]
req_extensions = v3_req
distinguished_name = req_distinguished_name
[req_distinguished_name]
[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @alt_names
[alt_names]
DNS.1 = clustera-worker-4
IP.1 = 192.168.5.18
EOF

openssl genrsa -out clustera-worker-4.key 2048
openssl req -new -key clustera-worker-4.key -subj "/CN=system:node:clustera-worker-4/O=system:nodes" -out clustera-worker-4.csr -config clustera-openssl-clustera-worker-4.cnf
openssl x509 -req -in clustera-worker-4.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out clustera-worker-4.crt -extensions v3_req -extfile clustera-openssl-clustera-worker-4.cnf -days 1000
```

Results:

```
clustera-worker-4.key
clustera-worker-4.crt
```

cat > clustera-openssl-clustera-worker-5.cnf <<EOF
[req]
req_extensions = v3_req
distinguished_name = req_distinguished_name
[req_distinguished_name]
[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @alt_names
[alt_names]
DNS.1 = clustera-worker-5
IP.1 = 192.168.5.19
EOF

openssl genrsa -out clustera-worker-5.key 2048
openssl req -new -key clustera-worker-5.key -subj "/CN=system:node:clustera-worker-5/O=system:nodes" -out clustera-worker-5.csr -config clustera-openssl-clustera-worker-5.cnf
openssl x509 -req -in clustera-worker-5.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out clustera-worker-5.crt -extensions v3_req -extfile clustera-openssl-clustera-worker-5.cnf -days 1000
```

Results:

```
clustera-worker-5.key
clustera-worker-5.crt
```


### The kubelet Kubernetes Configuration File

When generating kubeconfig files for Kubelets the client certificate matching the Kubelet's node name must be used. This will ensure Kubelets are properly authorized by the Kubernetes [Node Authorizer](https://kubernetes.io/docs/admin/authorization/node/).

Get the kub-api server load-balancer IP.
```
LOADBALANCER_ADDRESS=192.168.5.20
```

Generate a kubeconfig file for the first worker node.

On management:
```
{
  kubectl config set-cluster hardway-cluster-a \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://${LOADBALANCER_ADDRESS}:6443 \
    --kubeconfig=clustera-worker-1.kubeconfig

  kubectl config set-credentials system:node:clustera-worker-1 \
    --client-certificate=clustera-worker-1.crt \
    --client-key=clustera-worker-1.key \
    --embed-certs=true \
    --kubeconfig=clustera-worker-1.kubeconfig

  kubectl config set-context default \
    --cluster=hardway-cluster-a \
    --user=system:node:clustera-worker-1 \
    --kubeconfig=clustera-worker-1.kubeconfig

  kubectl config use-context default --kubeconfig=clustera-worker-1.kubeconfig
}
```

Results:

```
clustera-worker-1.kubeconfig
```

{
  kubectl config set-cluster hardway-cluster-a \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://${LOADBALANCER_ADDRESS}:6443 \
    --kubeconfig=clustera-worker-2.kubeconfig

  kubectl config set-credentials system:node:clustera-worker-2 \
    --client-certificate=clustera-worker-2.crt \
    --client-key=clustera-worker-2.key \
    --embed-certs=true \
    --kubeconfig=clustera-worker-2.kubeconfig

  kubectl config set-context default \
    --cluster=hardway-cluster-a \
    --user=system:node:clustera-worker-2 \
    --kubeconfig=clustera-worker-2.kubeconfig

  kubectl config use-context default --kubeconfig=clustera-worker-2.kubeconfig
}
```

Results:

```
clustera-worker-2.kubeconfig
```

{
  kubectl config set-cluster hardway-cluster-a \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://${LOADBALANCER_ADDRESS}:6443 \
    --kubeconfig=clustera-worker-3.kubeconfig

  kubectl config set-credentials system:node:clustera-worker-3 \
    --client-certificate=clustera-worker-3.crt \
    --client-key=clustera-worker-3.key \
    --embed-certs=true \
    --kubeconfig=clustera-worker-3.kubeconfig

  kubectl config set-context default \
    --cluster=hardway-cluster-a \
    --user=system:node:clustera-worker-3 \
    --kubeconfig=clustera-worker-3.kubeconfig

  kubectl config use-context default --kubeconfig=clustera-worker-3.kubeconfig
}
```

Results:

```
clustera-worker-3.kubeconfig
```

{
  kubectl config set-cluster hardway-cluster-a \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://${LOADBALANCER_ADDRESS}:6443 \
    --kubeconfig=clustera-worker-4.kubeconfig

  kubectl config set-credentials system:node:clustera-worker-4 \
    --client-certificate=clustera-worker-4.crt \
    --client-key=clustera-worker-4.key \
    --embed-certs=true \
    --kubeconfig=clustera-worker-4.kubeconfig

  kubectl config set-context default \
    --cluster=hardway-cluster-a \
    --user=system:node:clustera-worker-4 \
    --kubeconfig=clustera-worker-4.kubeconfig

  kubectl config use-context default --kubeconfig=clustera-worker-4.kubeconfig
}
```

Results:

```
clustera-worker-4.kubeconfig
```


{
  kubectl config set-cluster hardway-cluster-a \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://${LOADBALANCER_ADDRESS}:6443 \
    --kubeconfig=clustera-worker-5.kubeconfig

  kubectl config set-credentials system:node:clustera-worker-5 \
    --client-certificate=clustera-worker-5.crt \
    --client-key=clustera-worker-5.key \
    --embed-certs=true \
    --kubeconfig=clustera-worker-5.kubeconfig

  kubectl config set-context default \
    --cluster=hardway-cluster-a \
    --user=system:node:clustera-worker-5 \
    --kubeconfig=clustera-worker-5.kubeconfig

  kubectl config use-context default --kubeconfig=clustera-worker-5.kubeconfig
}
```

Results:

```
clustera-worker-5.kubeconfig
```

### Copy certificates, private keys and kubeconfig files to the worker node:
On management:
```
scp ca.crt clustera-worker-1.crt clustera-worker-1.key clustera-worker-1.kubeconfig clustera-worker-1:~/
scp ca.crt clustera-worker-2.crt clustera-worker-2.key clustera-worker-2.kubeconfig clustera-worker-2:~/
scp ca.crt clustera-worker-3.crt clustera-worker-3.key clustera-worker-3.kubeconfig clustera-worker-3:~/
scp ca.crt clustera-worker-4.crt clustera-worker-4.key clustera-worker-4.kubeconfig clustera-worker-4:~/
scp ca.crt clustera-worker-5.crt clustera-worker-5.key clustera-worker-5.kubeconfig clustera-worker-5:~/
```

### Download and Install Worker Binaries

Going forward all activities are to be done on the `clustera-worker-1` node.

On clustera-worker-1:
```
clustera-worker-1$ wget -q --show-progress --https-only --timestamping \
  https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kubectl \
  https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kube-proxy \
  https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kubelet
```

Reference: https://kubernetes.io/docs/setup/release/#node-binaries

Create the installation directories:

```
clustera-worker-1$ sudo mkdir -p \
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
clustera-worker-1$ cat <<EOF | sudo tee /var/lib/kubelet/kubelet-config.yaml
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
clustera-worker-1$ cat <<EOF | sudo tee /etc/systemd/system/kubelet.service
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
clustera-worker-1$ sudo mv clustera-kube-proxy.kubeconfig /var/lib/kube-proxy/kubeconfig
```

Create the `kube-proxy-config.yaml` configuration file:

```
clustera-worker-1$ cat <<EOF | sudo tee /var/lib/kube-proxy/kube-proxy-config.yaml
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
clustera-worker-1$ cat <<EOF | sudo tee /etc/systemd/system/kube-proxy.service
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

> Remember to run the above commands on worker node: `clustera-worker-1`


On clustera-worker-2:  
```
clustera-worker-2$ wget -q --show-progress --https-only --timestamping \
  https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kubectl \
  https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kube-proxy \
  https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kubelet
```

Reference: https://kubernetes.io/docs/setup/release/#node-binaries

Create the installation directories:

```
clustera-worker-2$ sudo mkdir -p \
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
clustera-worker-2$ cat <<EOF | sudo tee /var/lib/kubelet/kubelet-config.yaml
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
clustera-worker-2$ cat <<EOF | sudo tee /etc/systemd/system/kubelet.service
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
clustera-worker-2$ sudo mv clustera-kube-proxy.kubeconfig /var/lib/kube-proxy/kubeconfig
```

Create the `kube-proxy-config.yaml` configuration file:

```
clustera-worker-2$ cat <<EOF | sudo tee /var/lib/kube-proxy/kube-proxy-config.yaml
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
clustera-worker-2$ cat <<EOF | sudo tee /etc/systemd/system/kube-proxy.service
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


On clustera-worker-3:  
```
clustera-worker-3$ wget -q --show-progress --https-only --timestamping \
  https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kubectl \
  https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kube-proxy \
  https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kubelet
```

Reference: https://kubernetes.io/docs/setup/release/#node-binaries

Create the installation directories:

```
clustera-worker-3$ sudo mkdir -p \
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
clustera-worker-3$ cat <<EOF | sudo tee /var/lib/kubelet/kubelet-config.yaml
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
clustera-worker-3$ cat <<EOF | sudo tee /etc/systemd/system/kubelet.service
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
clustera-worker-3$ sudo mv clustera-kube-proxy.kubeconfig /var/lib/kube-proxy/kubeconfig
```

Create the `kube-proxy-config.yaml` configuration file:

```
clustera-worker-3$ cat <<EOF | sudo tee /var/lib/kube-proxy/kube-proxy-config.yaml
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
clustera-worker-3$ cat <<EOF | sudo tee /etc/systemd/system/kube-proxy.service
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


On clustera-worker-4:  
```
clustera-worker-4$ wget -q --show-progress --https-only --timestamping \
  https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kubectl \
  https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kube-proxy \
  https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kubelet
```

Reference: https://kubernetes.io/docs/setup/release/#node-binaries

Create the installation directories:

```
clustera-worker-4$ sudo mkdir -p \
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
clustera-worker-4$ cat <<EOF | sudo tee /var/lib/kubelet/kubelet-config.yaml
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
clustera-worker-4$ cat <<EOF | sudo tee /etc/systemd/system/kubelet.service
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
clustera-worker-4$ sudo mv clustera-kube-proxy.kubeconfig /var/lib/kube-proxy/kubeconfig
```

Create the `kube-proxy-config.yaml` configuration file:

```
clustera-worker-4$ cat <<EOF | sudo tee /var/lib/kube-proxy/kube-proxy-config.yaml
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
clustera-worker-4$ cat <<EOF | sudo tee /etc/systemd/system/kube-proxy.service
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


On clustera-worker-5:  
```
clustera-worker-5$ wget -q --show-progress --https-only --timestamping \
  https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kubectl \
  https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kube-proxy \
  https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kubelet
```

Reference: https://kubernetes.io/docs/setup/release/#node-binaries

Create the installation directories:

```
clustera-worker-5$ sudo mkdir -p \
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
clustera-worker-5$ cat <<EOF | sudo tee /var/lib/kubelet/kubelet-config.yaml
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
clustera-worker-5$ cat <<EOF | sudo tee /etc/systemd/system/kubelet.service
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
clustera-worker-5$ sudo mv clustera-kube-proxy.kubeconfig /var/lib/kube-proxy/kubeconfig
```

Create the `kube-proxy-config.yaml` configuration file:

```
clustera-worker-5$ cat <<EOF | sudo tee /var/lib/kube-proxy/kube-proxy-config.yaml
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
clustera-worker-5$ cat <<EOF | sudo tee /etc/systemd/system/kube-proxy.service
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

> Remember to run the above commands on worker node: `clustera-worker-2`

## Verification
On clustera-master-1:

List the registered Kubernetes nodes from the master node:

```
clustera-master-1$ kubectl get nodes --kubeconfig admin.kubeconfig
```

> output

```
NAME       STATUS     ROLES    AGE   VERSION
clustera-worker-1   NotReady   <none>   93s   v1.18.0
clustera-worker-2   NotReady   <none>   93s   v1.18.0
clustera-worker-3   NotReady   <none>   93s   v1.18.0
clustera-worker-4   NotReady   <none>   93s   v1.18.0
clustera-worker-5   NotReady   <none>   93s   v1.18.0
```

> Note: It is OK for the worker node to be in a NotReady state.
  That is because we haven't configured Networking yet.

Optional: At this point you may run the certificate verification script to make sure all certificates are configured correctly. Follow the instructions [here](verify-certificates.md)

Next: [TLS Bootstrapping Kubernetes Workers](10-tls-bootstrapping-kubernetes-workers.md)




#### LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B
#### LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B clusterb
#### LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B

# Bootstrapping the Kubernetes Worker Nodes

In this lab you will bootstrap 2 Kubernetes worker nodes. We already have [Docker](https://www.docker.com) installed on these nodes.

We will now install the kubernetes components
- [kubelet](https://kubernetes.io/docs/admin/kubelet)
- [kube-proxy](https://kubernetes.io/docs/concepts/cluster-bdministration/proxies).

## Prerequisites

The Certificates and Configuration are created on `k8smanagement` node and then copied over to workers using `scp`. 
Once this is done, the commands are to be run on first worker instance: `clusterb-worker-1`, `clusterb-worker-2`, `clusterb-worker-3`, `clusterb-worker-4`, `clusterb-worker-5`. Login to first worker instance using SSH Terminal.

### Provisioning  Kubelet Client Certificates

Kubernetes uses a [special-purpose authorization mode](https://kubernetes.io/docs/admin/authorization/node/) called Node Authorizer, that specifically authorizes API requests made by [Kubelets](https://kubernetes.io/docs/concepts/overview/components/#kubelet). In order to be authorized by the Node Authorizer, Kubelets must use a credential that identifies them as being in the `system:nodes` group, with a username of `system:node:<nodeName>`. In this section you will create a certificate for each Kubernetes worker node that meets the Node Authorizer requirements.

Generate a certificate and private key for one worker node:

On management:

```
cat > clusterb-openssl-clusterb-worker-1.cnf <<EOF
[req]
req_extensions = v3_req
distinguished_name = req_distinguished_name
[req_distinguished_name]
[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @alt_names
[alt_names]
DNS.1 = clusterb-worker-1
IP.1 = 192.168.5.35
EOF

openssl genrsa -out clusterb-worker-1.key 2048
openssl req -new -key clusterb-worker-1.key -subj "/CN=system:node:clusterb-worker-1/O=system:nodes" -out clusterb-worker-1.csr -config clusterb-openssl-clusterb-worker-1.cnf
openssl x509 -req -in clusterb-worker-1.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out clusterb-worker-1.crt -extensions v3_req -extfile clusterb-openssl-clusterb-worker-1.cnf -days 1000
```

Results:

```
clusterb-worker-1.key
clusterb-worker-1.crt
```
```
cat > clusterb-openssl-clusterb-worker-2.cnf <<EOF
[req]
req_extensions = v3_req
distinguished_name = req_distinguished_name
[req_distinguished_name]
[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @alt_names
[alt_names]
DNS.1 = clusterb-worker-2
IP.1 = 192.168.5.36
EOF

openssl genrsa -out clusterb-worker-2.key 2048
openssl req -new -key clusterb-worker-2.key -subj "/CN=system:node:clusterb-worker-2/O=system:nodes" -out clusterb-worker-2.csr -config clusterb-openssl-clusterb-worker-2.cnf
openssl x509 -req -in clusterb-worker-2.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out clusterb-worker-2.crt -extensions v3_req -extfile clusterb-openssl-clusterb-worker-2.cnf -days 1000
```

Results:

```
clusterb-worker-2.key
clusterb-worker-2.crt
```

cat > clusterb-openssl-clusterb-worker-3.cnf <<EOF
[req]
req_extensions = v3_req
distinguished_name = req_distinguished_name
[req_distinguished_name]
[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @alt_names
[alt_names]
DNS.1 = clusterb-worker-3
IP.1 = 192.168.5.37
EOF

openssl genrsa -out clusterb-worker-3.key 2048
openssl req -new -key clusterb-worker-3.key -subj "/CN=system:node:clusterb-worker-3/O=system:nodes" -out clusterb-worker-3.csr -config clusterb-openssl-clusterb-worker-3.cnf
openssl x509 -req -in clusterb-worker-3.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out clusterb-worker-3.crt -extensions v3_req -extfile clusterb-openssl-clusterb-worker-3.cnf -days 1000
```

Results:

```
clusterb-worker-3.key
clusterb-worker-3.crt
```

cat > clusterb-openssl-clusterb-worker-4.cnf <<EOF
[req]
req_extensions = v3_req
distinguished_name = req_distinguished_name
[req_distinguished_name]
[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @alt_names
[alt_names]
DNS.1 = clusterb-worker-4
IP.1 = 192.168.5.38
EOF

openssl genrsa -out clusterb-worker-4.key 2048
openssl req -new -key clusterb-worker-4.key -subj "/CN=system:node:clusterb-worker-4/O=system:nodes" -out clusterb-worker-4.csr -config clusterb-openssl-clusterb-worker-4.cnf
openssl x509 -req -in clusterb-worker-4.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out clusterb-worker-4.crt -extensions v3_req -extfile clusterb-openssl-clusterb-worker-4.cnf -days 1000
```

Results:

```
clusterb-worker-4.key
clusterb-worker-4.crt
```
cat > clusterb-openssl-clusterb-worker-5.cnf <<EOF
[req]
req_extensions = v3_req
distinguished_name = req_distinguished_name
[req_distinguished_name]
[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @alt_names
[alt_names]
DNS.1 = clusterb-worker-5
IP.1 = 192.168.5.39
EOF

openssl genrsa -out clusterb-worker-5.key 2048
openssl req -new -key clusterb-worker-5.key -subj "/CN=system:node:clusterb-worker-5/O=system:nodes" -out clusterb-worker-5.csr -config clusterb-openssl-clusterb-worker-5.cnf
openssl x509 -req -in clusterb-worker-5.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out clusterb-worker-5.crt -extensions v3_req -extfile clusterb-openssl-clusterb-worker-5.cnf -days 1000
```

Results:

```
clusterb-worker-5.key
clusterb-worker-5.crt
```

### The kubelet Kubernetes Configuration File

When generating kubeconfig files for Kubelets the client certificate matching the Kubelet's node name must be used. This will ensure Kubelets are properly authorized by the Kubernetes [Node Authorizer](https://kubernetes.io/docs/admin/authorization/node/).

Get the kub-api server load-balancer IP.
```
LOADBALANCER_ADDRESS=192.168.5.40
```

Generate a kubeconfig file for the first worker node. 

On management:
```
{
  kubectl config set-cluster hardway-cluster-b \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://${LOADBALANCER_ADDRESS}:6443 \
    --kubeconfig=clusterb-worker-1.kubeconfig

  kubectl config set-credentials system:node:clusterb-worker-1 \
    --client-certificate=clusterb-worker-1.crt \
    --client-key=clusterb-worker-1.key \
    --embed-certs=true \
    --kubeconfig=clusterb-worker-1.kubeconfig

  kubectl config set-context default \
    --cluster=hardway-cluster-b \
    --user=system:node:clusterb-worker-1 \
    --kubeconfig=clusterb-worker-1.kubeconfig

  kubectl config use-context default --kubeconfig=clusterb-worker-1.kubeconfig
}
```

Results:

```
clusterb-worker-1.kubeconfig
```

{
  kubectl config set-cluster hardway-cluster-b \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://${LOADBALANCER_ADDRESS}:6443 \
    --kubeconfig=clusterb-worker-2.kubeconfig

  kubectl config set-credentials system:node:clusterb-worker-2 \
    --client-certificate=clusterb-worker-2.crt \
    --client-key=clusterb-worker-2.key \
    --embed-certs=true \
    --kubeconfig=clusterb-worker-2.kubeconfig

  kubectl config set-context default \
    --cluster=hardway-cluster-b \
    --user=system:node:clusterb-worker-2 \
    --kubeconfig=clusterb-worker-2.kubeconfig

  kubectl config use-context default --kubeconfig=clusterb-worker-2.kubeconfig
}
```

Results:

```
clusterb-worker-2.kubeconfig
```


{
  kubectl config set-cluster hardway-cluster-b \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://${LOADBALANCER_ADDRESS}:6443 \
    --kubeconfig=clusterb-worker-3.kubeconfig

  kubectl config set-credentials system:node:clusterb-worker-3 \
    --client-certificate=clusterb-worker-3.crt \
    --client-key=clusterb-worker-3.key \
    --embed-certs=true \
    --kubeconfig=clusterb-worker-3.kubeconfig

  kubectl config set-context default \
    --cluster=hardway-cluster-b \
    --user=system:node:clusterb-worker-3 \
    --kubeconfig=clusterb-worker-3.kubeconfig

  kubectl config use-context default --kubeconfig=clusterb-worker-3.kubeconfig
}
```

Results:

```
clusterb-worker-3.kubeconfig
```


{
  kubectl config set-cluster hardway-cluster-b \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://${LOADBALANCER_ADDRESS}:6443 \
    --kubeconfig=clusterb-worker-4.kubeconfig

  kubectl config set-credentials system:node:clusterb-worker-4 \
    --client-certificate=clusterb-worker-4.crt \
    --client-key=clusterb-worker-4.key \
    --embed-certs=true \
    --kubeconfig=clusterb-worker-4.kubeconfig

  kubectl config set-context default \
    --cluster=hardway-cluster-b \
    --user=system:node:clusterb-worker-4 \
    --kubeconfig=clusterb-worker-4.kubeconfig

  kubectl config use-context default --kubeconfig=clusterb-worker-4.kubeconfig
}
```

Results:

```
clusterb-worker-4.kubeconfig
```


{
  kubectl config set-cluster hardway-cluster-b \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://${LOADBALANCER_ADDRESS}:6443 \
    --kubeconfig=clusterb-worker-5.kubeconfig

  kubectl config set-credentials system:node:clusterb-worker-5 \
    --client-certificate=clusterb-worker-5.crt \
    --client-key=clusterb-worker-5.key \
    --embed-certs=true \
    --kubeconfig=clusterb-worker-5.kubeconfig

  kubectl config set-context default \
    --cluster=hardway-cluster-b \
    --user=system:node:clusterb-worker-5 \
    --kubeconfig=clusterb-worker-5.kubeconfig

  kubectl config use-context default --kubeconfig=clusterb-worker-5.kubeconfig
}
```

Results:

```
clusterb-worker-5.kubeconfig
```

### Copy certificates, private keys and kubeconfig files to the worker node:
On management:
```
scp ca.crt clusterb-worker-1.crt clusterb-worker-1.key clusterb-worker-1.kubeconfig clusterb-worker-1:~/
scp ca.crt clusterb-worker-2.crt clusterb-worker-2.key clusterb-worker-2.kubeconfig clusterb-worker-2:~/
scp ca.crt clusterb-worker-3.crt clusterb-worker-3.key clusterb-worker-3.kubeconfig clusterb-worker-3:~/
scp ca.crt clusterb-worker-4.crt clusterb-worker-4.key clusterb-worker-4.kubeconfig clusterb-worker-4:~/
scp ca.crt clusterb-worker-5.crt clusterb-worker-5.key clusterb-worker-5.kubeconfig clusterb-worker-5:~/
```

### Download and Install Worker Binaries

Going forward all activities are to be done on the `clusterb-worker-1` node.

On clusterb-worker-1:
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
sudo mv clusterb-kube-proxy.kubeconfig /var/lib/kube-proxy/kubeconfig
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

> Remember to run the above commands on worker node: `clusterb-worker-1`


On clusterb-worker-2:  
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
sudo mv clusterb-kube-proxy.kubeconfig /var/lib/kube-proxy/kubeconfig
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

On clusterb-worker-3:  
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
sudo mv clusterb-kube-proxy.kubeconfig /var/lib/kube-proxy/kubeconfig
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

On clusterb-worker-4:  
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
sudo mv clusterb-kube-proxy.kubeconfig /var/lib/kube-proxy/kubeconfig
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


On clusterb-worker-5:  
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
sudo mv clusterb-kube-proxy.kubeconfig /var/lib/kube-proxy/kubeconfig
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

> Remember to run the above commands on worker node: `clusterb-worker-2`

## Verification
On clusterb-master-1:

List the registered Kubernetes nodes from the master node:

```
clusterb-master-1$ kubectl get nodes --kubeconfig admin.kubeconfig
```

> output

```
NAME       STATUS     ROLES    AGE   VERSION
clusterb-worker-1   NotReady   <none>   93s   v1.18.0
clusterb-worker-2   NotReady   <none>   93s   v1.18.0
clusterb-worker-3   NotReady   <none>   93s   v1.18.0
clusterb-worker-4   NotReady   <none>   93s   v1.18.0
clusterb-worker-5   NotReady   <none>   93s   v1.18.0
```

> Note: It is OK for the worker node to be in a NotReady state.
  That is because we haven't configured Networking yet.

Optional: At this point you may run the certificate verification script to make sure all certificates are configured correctly. Follow the instructions [here](verify-certificates.md)





