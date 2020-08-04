#### LAB A LAB A LAB A LAB A LAB A LAB A LAB A LAB A LAB A
#### LAB A LAB A LAB A LAB A LAB A LAB A LAB A LAB A LAB A
#### LAB A LAB A LAB A LAB A LAB A LAB A LAB A LAB A LAB A

# Provisioning a CA and Generating TLS Certificates

In this lab you will provision a [PKI Infrastructure](https://en.wikipedia.org/wiki/Public_key_infrastructure) using the popular openssl tool, then use it to bootstrap a Certificate Authority, and generate TLS certificates for the following components: etcd, kube-apiserver, kube-controller-manager, kube-scheduler, kubelet, and kube-proxy.

# Where to do these?

You can do these on any machine with `openssl` on it. But you should be able to copy the generated files to the provisioned VMs. Or just do these from one of the master nodes.

In our case we do it on the k8smanagement node, as we have set it up to be the administrative client.


## Certificate Authority
## We will use the same CA for all 6 clusters and also the same admin account for all the clusters.

In this section you will provision a Certificate Authority that can be used to generate additional TLS certificates.

Create a CA certificate, then generate a Certificate Signing Request and use it to create a private key:


```
# Create private key for CA
openssl genrsa -out ca.key 2048

# Comment line starting with RANDFILE in /etc/ssl/openssl.cnf definition to avoid permission issues
sudo sed -i '0,/RANDFILE/{s/RANDFILE/\#&/}' /etc/ssl/openssl.cnf

# Create CSR using the private key
openssl req -new -key ca.key -subj "/CN=KUBERNETES-CA" -out ca.csr

# Self sign the csr using its own private key
openssl x509 -req -in ca.csr -signkey ca.key -CAcreateserial  -out ca.crt -days 1000
```
Results:

```
ca.crt
ca.key
```

Reference : https://kubernetes.io/docs/concepts/cluster-administration/certificates/#openssl

The ca.crt is the Kubernetes Certificate Authority certificate and ca.key is the Kubernetes Certificate Authority private key.
You will use the ca.crt file in many places, so it will be copied to many places.
The ca.key is used by the CA for signing certificates. And it should be securely stored. In this case our master node(s) is our CA server as well, so we will store it on master node(s). There is not need to copy this file to elsewhere.

## Client and Server Certificates

In this section you will generate client and server certificates for each Kubernetes component and a client certificate for the Kubernetes `admin` user.

### The Admin Client Certificate

Generate the `admin` client certificate and private key:

```
# Generate private key for admin user
openssl genrsa -out admin.key 2048

# Generate CSR for admin user. Note the OU.
openssl req -new -key admin.key -subj "/CN=admin/O=system:masters" -out admin.csr

# Sign certificate for admin user using CA servers private key
openssl x509 -req -in admin.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out admin.crt -days 1000
```

Note that the admin user is part of the **system:masters** group. This is how we are able to perform any administrative operations on Kubernetes cluster using kubectl utility.

Results:

```
admin.key
admin.crt
```

The admin.crt and admin.key file gives you administrative access. We will configure these to be used with the kubectl tool to perform administrative functions on kubernetes.

### The Kubelet Client Certificates

We are going to skip certificate configuration for Worker Nodes for now. We will deal with them when we configure the workers.
For now let's just focus on the control plane components.

### The Controller Manager Client Certificate

Generate the `kube-controller-manager` client certificate and private key:

```
openssl genrsa -out clustera-kube-controller-manager.key 2048
openssl req -new -key clustera-kube-controller-manager.key -subj "/CN=system:kube-controller-manager" -out clustera-kube-controller-manager.csr
openssl x509 -req -in clustera-kube-controller-manager.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out clustera-kube-controller-manager.crt -days 1000
```

Results:

```
clustera-kube-controller-manager.key
clustera-kube-controller-manager.crt
```


### The Kube Proxy Client Certificate

Generate the `kube-proxy` client certificate and private key:


```
openssl genrsa -out clustera-kube-proxy.key 2048
openssl req -new -key clustera-kube-proxy.key -subj "/CN=system:kube-proxy" -out clustera-kube-proxy.csr
openssl x509 -req -in clustera-kube-proxy.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out clustera-kube-proxy.crt -days 1000
```

Results:

```
clustera-kube-proxy.key
clustera-kube-proxy.crt
```

### The Scheduler Client Certificate

Generate the `kube-scheduler` client certificate and private key:



```
openssl genrsa -out clustera-kube-scheduler.key 2048
openssl req -new -key clustera-kube-scheduler.key -subj "/CN=system:kube-scheduler" -out clustera-kube-scheduler.csr
openssl x509 -req -in clustera-kube-scheduler.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out clustera-kube-scheduler.crt -days 1000
```

Results:

```
clustera-kube-scheduler.key
clustera-kube-scheduler.crt
```

### The Kubernetes API Server Certificate

The kube-apiserver certificate requires all names that various components may reach it to be part of the alternate names. These include the different DNS names, and IP addresses such as the master servers IP address, the load balancers IP address, the kube-api service IP address etc.

The `openssl` command cannot take alternate names as command line parameter. So we must create a `conf` file for it:

```
cat > clustera-openssl.cnf <<EOF
[req]
req_extensions = v3_req
distinguished_name = req_distinguished_name
[req_distinguished_name]
[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @alt_names
[alt_names]
DNS.1 = kubernetes
DNS.2 = kubernetes.default
DNS.3 = kubernetes.default.svc
DNS.4 = kubernetes.default.svc.cluster.local
IP.1 = 10.96.0.1
IP.2 = 192.168.5.10
IP.3 = 192.168.5.11
IP.4 = 192.168.5.20
IP.5 = 127.0.0.1
EOF
```

Generates certs for kube-apiserver

```
openssl genrsa -out clustera-kube-apiserver.key 2048
openssl req -new -key clustera-kube-apiserver.key -subj "/CN=kube-apiserver" -out clustera-kube-apiserver.csr -config clustera-openssl.cnf
openssl x509 -req -in clustera-kube-apiserver.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out clustera-kube-apiserver.crt -extensions v3_req -extfile clustera-openssl.cnf -days 1000
```

Results:

```
clustera-kube-apiserver.crt
clustera-kube-apiserver.key
```

### The ETCD Server Certificate

Similarly ETCD server certificate must have addresses of all the servers part of the ETCD cluster

The `openssl` command cannot take alternate names as command line parameter. So we must create a `conf` file for it:

```
cat > clustera-openssl-etcd.cnf <<EOF
[req]
req_extensions = v3_req
distinguished_name = req_distinguished_name
[req_distinguished_name]
[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @alt_names
[alt_names]
IP.1 = 192.168.5.10
IP.2 = 192.168.5.11
IP.3 = 127.0.0.1
EOF
```

Generates certs for ETCD

```
openssl genrsa -out clustera-etcd-server.key 2048
openssl req -new -key clustera-etcd-server.key -subj "/CN=etcd-server" -out clustera-etcd-server.csr -config clustera-openssl-etcd.cnf
openssl x509 -req -in clustera-etcd-server.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out clustera-etcd-server.crt -extensions v3_req -extfile clustera-openssl-etcd.cnf -days 1000
```

Results:

```
clustera-etcd-server.key
clustera-etcd-server.crt
```

## The Service Account Key Pair

The Kubernetes Controller Manager leverages a key pair to generate and sign service account tokens as describe in the [managing service accounts](https://kubernetes.io/docs/admin/service-accounts-admin/) documentation.

Generate the `service-account` certificate and private key:

```
openssl genrsa -out clustera-service-account.key 2048
openssl req -new -key clustera-service-account.key -subj "/CN=service-accounts" -out clustera-service-account.csr
openssl x509 -req -in clustera-service-account.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out clustera-service-account.crt -days 1000
```

Results:

```
clustera-service-account.key
clustera-service-account.crt
```


## Distribute the Certificates

Copy the appropriate certificates and private keys to each controller instance:

```
for instance in clustera-master-1 clustera-master-2; do
  scp ca.crt ca.key clustera-kube-apiserver.key clustera-kube-apiserver.crt \
    clustera-service-account.key clustera-service-account.crt \
    clustera-etcd-server.key clustera-etcd-server.crt \
    ${instance}:~/
done
```

> The `kube-proxy`, `kube-controller-manager`, `kube-scheduler`, and `kubelet` client certificates will be used to generate client authentication configuration files in the next lab. These certificates will be embedded into the client authentication configuration files. We will then copy those configuration files to the other master nodes.

Next: [Generating Kubernetes Configuration Files for Authentication](05-kubernetes-configuration-files.md)


#### LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B
#### LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B
#### LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B


Generate the `kube-controller-manager` client certificate and private key:

```
openssl genrsa -out clusterb-kube-controller-manager.key 2048
openssl req -new -key clusterb-kube-controller-manager.key -subj "/CN=system:kube-controller-manager" -out clusterb-kube-controller-manager.csr
openssl x509 -req -in clusterb-kube-controller-manager.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out clusterb-kube-controller-manager.crt -days 1000
```

Results:

```
clusterb-kube-controller-manager.key
clusterb-kube-controller-manager.crt
```


### The Kube Proxy Client Certificate

Generate the `kube-proxy` client certificate and private key:


```
openssl genrsa -out clusterb-kube-proxy.key 2048
openssl req -new -key clusterb-kube-proxy.key -subj "/CN=system:kube-proxy" -out clusterb-kube-proxy.csr
openssl x509 -req -in clusterb-kube-proxy.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out clusterb-kube-proxy.crt -days 1000
```

Results:

```
clusterb-kube-proxy.key
clusterb-kube-proxy.crt
```

### The Scheduler Client Certificate

Generate the `kube-scheduler` client certificate and private key:



```
openssl genrsa -out clusterb-kube-scheduler.key 2048
openssl req -new -key clusterb-kube-scheduler.key -subj "/CN=system:kube-scheduler" -out clusterb-kube-scheduler.csr
openssl x509 -req -in clusterb-kube-scheduler.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out clusterb-kube-scheduler.crt -days 1000
```

Results:

```
clusterb-kube-scheduler.key
clusterb-kube-scheduler.crt
```

### The Kubernetes API Server Certificate

The kube-apiserver certificate requires all names that various components may reach it to be part of the alternate names. These include the different DNS names, and IP addresses such as the master servers IP address, the load balancers IP address, the kube-api service IP address etc.

The `openssl` command cannot take alternate names as command line parameter. So we must create a `conf` file for it:

```
cat > clusterb-openssl.cnf <<EOF
[req]
req_extensions = v3_req
distinguished_name = req_distinguished_name
[req_distinguished_name]
[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @alt_names
[alt_names]
DNS.1 = kubernetes
DNS.2 = kubernetes.default
DNS.3 = kubernetes.default.svc
DNS.4 = kubernetes.default.svc.cluster.local
IP.1 = 10.96.0.1
IP.2 = 192.168.5.30
IP.3 = 192.168.5.31
IP.4 = 192.168.5.40
IP.5 = 127.0.0.1
EOF
```

Generates certs for kube-apiserver

```
openssl genrsa -out clusterb-kube-apiserver.key 2048
openssl req -new -key clusterb-kube-apiserver.key -subj "/CN=kube-apiserver" -out clusterb-kube-apiserver.csr -config clusterb-openssl.cnf
openssl x509 -req -in clusterb-kube-apiserver.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out clusterb-kube-apiserver.crt -extensions v3_req -extfile clusterb-openssl.cnf -days 1000
```

Results:

```
clusterb-kube-apiserver.crt
clusterb-kube-apiserver.key
```

### The ETCD Server Certificate

Similarly ETCD server certificate must have addresses of all the servers part of the ETCD cluster

The `openssl` command cannot take alternate names as command line parameter. So we must create a `conf` file for it:

```
cat > clusterb-openssl-etcd.cnf <<EOF
[req]
req_extensions = v3_req
distinguished_name = req_distinguished_name
[req_distinguished_name]
[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @alt_names
[alt_names]
IP.1 = 192.168.5.30
IP.2 = 192.168.5.31
IP.3 = 127.0.0.1
EOF
```

Generates certs for ETCD

```
openssl genrsa -out clusterb-etcd-server.key 2048
openssl req -new -key clusterb-etcd-server.key -subj "/CN=etcd-server" -out clusterb-etcd-server.csr -config clusterb-openssl-etcd.cnf
openssl x509 -req -in clusterb-etcd-server.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out clusterb-etcd-server.crt -extensions v3_req -extfile clusterb-openssl-etcd.cnf -days 1000
```

Results:

```
clusterb-etcd-server.key
clusterb-etcd-server.crt
```

## The Service Account Key Pair

The Kubernetes Controller Manager leverages a key pair to generate and sign service account tokens as describe in the [managing service accounts](https://kubernetes.io/docs/admin/service-accounts-admin/) documentation.

Generate the `service-account` certificate and private key:

```
openssl genrsa -out clusterb-service-account.key 2048
openssl req -new -key clusterb-service-account.key -subj "/CN=service-accounts" -out clusterb-service-account.csr
openssl x509 -req -in clusterb-service-account.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out clusterb-service-account.crt -days 1000
```

Results:

```
clusterb-service-account.key
clusterb-service-account.crt
```


## Distribute the Certificates

Copy the appropriate certificates and private keys to each controller instance:

```
for instance in clusterb-master-1 clusterb-master-2; do
  scp ca.crt ca.key clusterb-kube-apiserver.key clusterb-kube-apiserver.crt \
    clusterb-service-account.key clusterb-service-account.crt \
    clusterb-etcd-server.key clusterb-etcd-server.crt \
    ${instance}:~/
done
```

> The `kube-proxy`, `kube-controller-manager`, `kube-scheduler`, and `kubelet` client certificates will be used to generate client authentication configuration files in the next lab. These certificates will be embedded into the client authentication configuration files. We will then copy those configuration files to the other master nodes.

Next: [Generating Kubernetes Configuration Files for Authentication](05-kubernetes-configuration-files.md)



