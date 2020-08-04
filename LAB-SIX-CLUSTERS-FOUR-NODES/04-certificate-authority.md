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
openssl genrsa -out laba-kube-controller-manager.key 2048
openssl req -new -key laba-kube-controller-manager.key -subj "/CN=system:kube-controller-manager" -out laba-kube-controller-manager.csr
openssl x509 -req -in laba-kube-controller-manager.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out laba-kube-controller-manager.crt -days 1000
```

Results:

```
laba-kube-controller-manager.key
laba-kube-controller-manager.crt
```


### The Kube Proxy Client Certificate

Generate the `kube-proxy` client certificate and private key:


```
openssl genrsa -out laba-kube-proxy.key 2048
openssl req -new -key laba-kube-proxy.key -subj "/CN=system:kube-proxy" -out laba-kube-proxy.csr
openssl x509 -req -in laba-kube-proxy.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out laba-kube-proxy.crt -days 1000
```

Results:

```
laba-kube-proxy.key
laba-kube-proxy.crt
```

### The Scheduler Client Certificate

Generate the `kube-scheduler` client certificate and private key:



```
openssl genrsa -out laba-kube-scheduler.key 2048
openssl req -new -key laba-kube-scheduler.key -subj "/CN=system:kube-scheduler" -out laba-kube-scheduler.csr
openssl x509 -req -in laba-kube-scheduler.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out laba-kube-scheduler.crt -days 1000
```

Results:

```
laba-kube-scheduler.key
laba-kube-scheduler.crt
```

### The Kubernetes API Server Certificate

The kube-apiserver certificate requires all names that various components may reach it to be part of the alternate names. These include the different DNS names, and IP addresses such as the master servers IP address, the load balancers IP address, the kube-api service IP address etc.

The `openssl` command cannot take alternate names as command line parameter. So we must create a `conf` file for it:

```
cat > laba-openssl.cnf <<EOF
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
IP.2 = 192.168.5.6
IP.3 = 192.168.5.7
IP.4 = 192.168.5.15
IP.5 = 127.0.0.1
EOF
```

Generates certs for kube-apiserver

```
openssl genrsa -out laba-kube-apiserver.key 2048
openssl req -new -key laba-kube-apiserver.key -subj "/CN=kube-apiserver" -out laba-kube-apiserver.csr -config laba-openssl.cnf
openssl x509 -req -in laba-kube-apiserver.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out laba-kube-apiserver.crt -extensions v3_req -extfile laba-openssl.cnf -days 1000
```

Results:

```
laba-kube-apiserver.crt
laba-kube-apiserver.key
```

### The ETCD Server Certificate

Similarly ETCD server certificate must have addresses of all the servers part of the ETCD cluster

The `openssl` command cannot take alternate names as command line parameter. So we must create a `conf` file for it:

```
cat > laba-openssl-etcd.cnf <<EOF
[req]
req_extensions = v3_req
distinguished_name = req_distinguished_name
[req_distinguished_name]
[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @alt_names
[alt_names]
IP.1 = 192.168.5.6
IP.2 = 192.168.5.7
IP.3 = 127.0.0.1
EOF
```

Generates certs for ETCD

```
openssl genrsa -out laba-etcd-server.key 2048
openssl req -new -key laba-etcd-server.key -subj "/CN=etcd-server" -out laba-etcd-server.csr -config laba-openssl-etcd.cnf
openssl x509 -req -in laba-etcd-server.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out laba-etcd-server.crt -extensions v3_req -extfile laba-openssl-etcd.cnf -days 1000
```

Results:

```
laba-etcd-server.key
laba-etcd-server.crt
```

## The Service Account Key Pair

The Kubernetes Controller Manager leverages a key pair to generate and sign service account tokens as describe in the [managing service accounts](https://kubernetes.io/docs/admin/service-accounts-admin/) documentation.

Generate the `service-account` certificate and private key:

```
openssl genrsa -out laba-service-account.key 2048
openssl req -new -key laba-service-account.key -subj "/CN=service-accounts" -out laba-service-account.csr
openssl x509 -req -in laba-service-account.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out laba-service-account.crt -days 1000
```

Results:

```
laba-service-account.key
laba-service-account.crt
```


## Distribute the Certificates

Copy the appropriate certificates and private keys to each controller instance:

```
for instance in laba-master-1 laba-master-2; do
  scp ca.crt ca.key laba-kube-apiserver.key laba-kube-apiserver.crt \
    laba-service-account.key laba-service-account.crt \
    laba-etcd-server.key laba-etcd-server.crt \
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
openssl genrsa -out labb-kube-controller-manager.key 2048
openssl req -new -key labb-kube-controller-manager.key -subj "/CN=system:kube-controller-manager" -out labb-kube-controller-manager.csr
openssl x509 -req -in labb-kube-controller-manager.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out labb-kube-controller-manager.crt -days 1000
```

Results:

```
labb-kube-controller-manager.key
labb-kube-controller-manager.crt
```


### The Kube Proxy Client Certificate

Generate the `kube-proxy` client certificate and private key:


```
openssl genrsa -out labb-kube-proxy.key 2048
openssl req -new -key labb-kube-proxy.key -subj "/CN=system:kube-proxy" -out labb-kube-proxy.csr
openssl x509 -req -in labb-kube-proxy.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out labb-kube-proxy.crt -days 1000
```

Results:

```
labb-kube-proxy.key
labb-kube-proxy.crt
```

### The Scheduler Client Certificate

Generate the `kube-scheduler` client certificate and private key:



```
openssl genrsa -out labb-kube-scheduler.key 2048
openssl req -new -key labb-kube-scheduler.key -subj "/CN=system:kube-scheduler" -out labb-kube-scheduler.csr
openssl x509 -req -in labb-kube-scheduler.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out labb-kube-scheduler.crt -days 1000
```

Results:

```
labb-kube-scheduler.key
labb-kube-scheduler.crt
```

### The Kubernetes API Server Certificate

The kube-apiserver certificate requires all names that various components may reach it to be part of the alternate names. These include the different DNS names, and IP addresses such as the master servers IP address, the load balancers IP address, the kube-api service IP address etc.

The `openssl` command cannot take alternate names as command line parameter. So we must create a `conf` file for it:

```
cat > labb-openssl.cnf <<EOF
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
IP.2 = 192.168.5.21
IP.3 = 192.168.5.22
IP.4 = 192.168.5.30
IP.5 = 127.0.0.1
EOF
```

Generates certs for kube-apiserver

```
openssl genrsa -out labb-kube-apiserver.key 2048
openssl req -new -key labb-kube-apiserver.key -subj "/CN=kube-apiserver" -out labb-kube-apiserver.csr -config labb-openssl.cnf
openssl x509 -req -in labb-kube-apiserver.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out labb-kube-apiserver.crt -extensions v3_req -extfile labb-openssl.cnf -days 1000
```

Results:

```
labb-kube-apiserver.crt
labb-kube-apiserver.key
```

### The ETCD Server Certificate

Similarly ETCD server certificate must have addresses of all the servers part of the ETCD cluster

The `openssl` command cannot take alternate names as command line parameter. So we must create a `conf` file for it:

```
cat > labb-openssl-etcd.cnf <<EOF
[req]
req_extensions = v3_req
distinguished_name = req_distinguished_name
[req_distinguished_name]
[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @alt_names
[alt_names]
IP.1 = 192.168.5.21
IP.2 = 192.168.5.22
IP.3 = 127.0.0.1
EOF
```

Generates certs for ETCD

```
openssl genrsa -out labb-etcd-server.key 2048
openssl req -new -key labb-etcd-server.key -subj "/CN=etcd-server" -out labb-etcd-server.csr -config labb-openssl-etcd.cnf
openssl x509 -req -in labb-etcd-server.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out labb-etcd-server.crt -extensions v3_req -extfile labb-openssl-etcd.cnf -days 1000
```

Results:

```
labb-etcd-server.key
labb-etcd-server.crt
```

## The Service Account Key Pair

The Kubernetes Controller Manager leverages a key pair to generate and sign service account tokens as describe in the [managing service accounts](https://kubernetes.io/docs/admin/service-accounts-admin/) documentation.

Generate the `service-account` certificate and private key:

```
openssl genrsa -out labb-service-account.key 2048
openssl req -new -key labb-service-account.key -subj "/CN=service-accounts" -out labb-service-account.csr
openssl x509 -req -in labb-service-account.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out labb-service-account.crt -days 1000
```

Results:

```
labb-service-account.key
labb-service-account.crt
```


## Distribute the Certificates

Copy the appropriate certificates and private keys to each controller instance:

```
for instance in labb-master-1 labb-master-2; do
  scp ca.crt ca.key labb-kube-apiserver.key labb-kube-apiserver.crt \
    labb-service-account.key labb-service-account.crt \
    labb-etcd-server.key labb-etcd-server.crt \
    ${instance}:~/
done
```

> The `kube-proxy`, `kube-controller-manager`, `kube-scheduler`, and `kubelet` client certificates will be used to generate client authentication configuration files in the next lab. These certificates will be embedded into the client authentication configuration files. We will then copy those configuration files to the other master nodes.

Next: [Generating Kubernetes Configuration Files for Authentication](05-kubernetes-configuration-files.md)


#### LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C
#### LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C labc
#### LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C


Generate the `kube-controller-manager` client certificate and private key:

```
openssl genrsa -out labc-kube-controller-manager.key 2048
openssl req -new -key labc-kube-controller-manager.key -subj "/CN=system:kube-controller-manager" -out labc-kube-controller-manager.csr
openssl x509 -req -in labc-kube-controller-manager.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out labc-kube-controller-manager.crt -days 1000
```

Results:

```
labc-kube-controller-manager.key
labc-kube-controller-manager.crt
```


### The Kube Proxy Client Certificate

Generate the `kube-proxy` client certificate and private key:


```
openssl genrsa -out labc-kube-proxy.key 2048
openssl req -new -key labc-kube-proxy.key -subj "/CN=system:kube-proxy" -out labc-kube-proxy.csr
openssl x509 -req -in labc-kube-proxy.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out labc-kube-proxy.crt -days 1000
```

Results:

```
labc-kube-proxy.key
labc-kube-proxy.crt
```

### The Scheduler Client Certificate

Generate the `kube-scheduler` client certificate and private key:



```
openssl genrsa -out labc-kube-scheduler.key 2048
openssl req -new -key labc-kube-scheduler.key -subj "/CN=system:kube-scheduler" -out labc-kube-scheduler.csr
openssl x509 -req -in labc-kube-scheduler.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out labc-kube-scheduler.crt -days 1000
```

Results:

```
labc-kube-scheduler.key
labc-kube-scheduler.crt
```

### The Kubernetes API Server Certificate

The kube-apiserver certificate requires all names that various components may reach it to be part of the alternate names. These include the different DNS names, and IP addresses such as the master servers IP address, the load balancers IP address, the kube-api service IP address etc.

The `openssl` command cannot take alternate names as command line parameter. So we must create a `conf` file for it:

```
cat > labc-openssl.cnf <<EOF
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
IP.2 = 192.168.5.36
IP.3 = 192.168.5.37
IP.4 = 192.168.5.45
IP.5 = 127.0.0.1
EOF
```

Generates certs for kube-apiserver

```
openssl genrsa -out labc-kube-apiserver.key 2048
openssl req -new -key labc-kube-apiserver.key -subj "/CN=kube-apiserver" -out labc-kube-apiserver.csr -config labc-openssl.cnf
openssl x509 -req -in labc-kube-apiserver.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out labc-kube-apiserver.crt -extensions v3_req -extfile labc-openssl.cnf -days 1000
```

Results:

```
labc-kube-apiserver.crt
labc-kube-apiserver.key
```

### The ETCD Server Certificate

Similarly ETCD server certificate must have addresses of all the servers part of the ETCD cluster

The `openssl` command cannot take alternate names as command line parameter. So we must create a `conf` file for it:

```
cat > labc-openssl-etcd.cnf <<EOF
[req]
req_extensions = v3_req
distinguished_name = req_distinguished_name
[req_distinguished_name]
[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @alt_names
[alt_names]
IP.1 = 192.168.5.36
IP.2 = 192.168.5.37
IP.3 = 127.0.0.1
EOF
```

Generates certs for ETCD

```
openssl genrsa -out labc-etcd-server.key 2048
openssl req -new -key labc-etcd-server.key -subj "/CN=etcd-server" -out labc-etcd-server.csr -config labc-openssl-etcd.cnf
openssl x509 -req -in labc-etcd-server.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out labc-etcd-server.crt -extensions v3_req -extfile labc-openssl-etcd.cnf -days 1000
```

Results:

```
labc-etcd-server.key
labc-etcd-server.crt
```

## The Service Account Key Pair

The Kubernetes Controller Manager leverages a key pair to generate and sign service account tokens as describe in the [managing service accounts](https://kubernetes.io/docs/admin/service-accounts-admin/) documentation.

Generate the `service-account` certificate and private key:

```
openssl genrsa -out labc-service-account.key 2048
openssl req -new -key labc-service-account.key -subj "/CN=service-accounts" -out labc-service-account.csr
openssl x509 -req -in labc-service-account.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out labc-service-account.crt -days 1000
```

Results:

```
labc-service-account.key
labc-service-account.crt
```


## Distribute the Certificates

Copy the appropriate certificates and private keys to each controller instance:

```
for instance in labc-master-1 labc-master-2; do
  scp ca.crt ca.key labc-kube-apiserver.key labc-kube-apiserver.crt \
    labc-service-account.key labc-service-account.crt \
    labc-etcd-server.key labc-etcd-server.crt \
    ${instance}:~/
done
```

> The `kube-proxy`, `kube-controller-manager`, `kube-scheduler`, and `kubelet` client certificates will be used to generate client authentication configuration files in the next lab. These certificates will be embedded into the client authentication configuration files. We will then copy those configuration files to the other master nodes.

Next: [Generating Kubernetes Configuration Files for Authentication](05-kubernetes-configuration-files.md)

#### LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D
#### LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D labd
#### LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D


Generate the `kube-controller-manager` client certificate and private key:

```
openssl genrsa -out labd-kube-controller-manager.key 2048
openssl req -new -key labd-kube-controller-manager.key -subj "/CN=system:kube-controller-manager" -out labd-kube-controller-manager.csr
openssl x509 -req -in labd-kube-controller-manager.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out labd-kube-controller-manager.crt -days 1000
```

Results:

```
labd-kube-controller-manager.key
labd-kube-controller-manager.crt
```


### The Kube Proxy Client Certificate

Generate the `kube-proxy` client certificate and private key:


```
openssl genrsa -out labd-kube-proxy.key 2048
openssl req -new -key labd-kube-proxy.key -subj "/CN=system:kube-proxy" -out labd-kube-proxy.csr
openssl x509 -req -in labd-kube-proxy.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out labd-kube-proxy.crt -days 1000
```

Results:

```
labd-kube-proxy.key
labd-kube-proxy.crt
```

### The Scheduler Client Certificate

Generate the `kube-scheduler` client certificate and private key:



```
openssl genrsa -out labd-kube-scheduler.key 2048
openssl req -new -key labd-kube-scheduler.key -subj "/CN=system:kube-scheduler" -out labd-kube-scheduler.csr
openssl x509 -req -in labd-kube-scheduler.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out labd-kube-scheduler.crt -days 1000
```

Results:

```
labd-kube-scheduler.key
labd-kube-scheduler.crt
```

### The Kubernetes API Server Certificate

The kube-apiserver certificate requires all names that various components may reach it to be part of the alternate names. These include the different DNS names, and IP addresses such as the master servers IP address, the load balancers IP address, the kube-api service IP address etc.

The `openssl` command cannot take alternate names as command line parameter. So we must create a `conf` file for it:

```
cat > labd-openssl.cnf <<EOF
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
IP.2 = 192.168.5.51
IP.3 = 192.168.5.52
IP.4 = 192.168.5.60
IP.5 = 127.0.0.1
EOF
```

Generates certs for kube-apiserver

```
openssl genrsa -out labd-kube-apiserver.key 2048
openssl req -new -key labd-kube-apiserver.key -subj "/CN=kube-apiserver" -out labd-kube-apiserver.csr -config labd-openssl.cnf
openssl x509 -req -in labd-kube-apiserver.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out labd-kube-apiserver.crt -extensions v3_req -extfile labd-openssl.cnf -days 1000
```

Results:

```
labd-kube-apiserver.crt
labd-kube-apiserver.key
```

### The ETCD Server Certificate

Similarly ETCD server certificate must have addresses of all the servers part of the ETCD cluster

The `openssl` command cannot take alternate names as command line parameter. So we must create a `conf` file for it:

```
cat > labd-openssl-etcd.cnf <<EOF
[req]
req_extensions = v3_req
distinguished_name = req_distinguished_name
[req_distinguished_name]
[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @alt_names
[alt_names]
IP.1 = 192.168.5.51
IP.2 = 192.168.5.52
IP.3 = 127.0.0.1
EOF
```

Generates certs for ETCD

```
openssl genrsa -out labd-etcd-server.key 2048
openssl req -new -key labd-etcd-server.key -subj "/CN=etcd-server" -out labd-etcd-server.csr -config labd-openssl-etcd.cnf
openssl x509 -req -in labd-etcd-server.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out labd-etcd-server.crt -extensions v3_req -extfile labd-openssl-etcd.cnf -days 1000
```

Results:

```
labd-etcd-server.key
labd-etcd-server.crt
```

## The Service Account Key Pair

The Kubernetes Controller Manager leverages a key pair to generate and sign service account tokens as describe in the [managing service accounts](https://kubernetes.io/docs/admin/service-accounts-admin/) documentation.

Generate the `service-account` certificate and private key:

```
openssl genrsa -out labd-service-account.key 2048
openssl req -new -key labd-service-account.key -subj "/CN=service-accounts" -out labd-service-account.csr
openssl x509 -req -in labd-service-account.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out labd-service-account.crt -days 1000
```

Results:

```
labd-service-account.key
labd-service-account.crt
```


## Distribute the Certificates

Copy the appropriate certificates and private keys to each controller instance:

```
for instance in labd-master-1 labd-master-2; do
  scp ca.crt ca.key labd-kube-apiserver.key labd-kube-apiserver.crt \
    labd-service-account.key labd-service-account.crt \
    labd-etcd-server.key labd-etcd-server.crt \
    ${instance}:~/
done
```

> The `kube-proxy`, `kube-controller-manager`, `kube-scheduler`, and `kubelet` client certificates will be used to generate client authentication configuration files in the next lab. These certificates will be embedded into the client authentication configuration files. We will then copy those configuration files to the other master nodes.

Next: [Generating Kubernetes Configuration Files for Authentication](05-kubernetes-configuration-files.md)

#### LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E
#### LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E labe
#### LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E


Generate the `kube-controller-manager` client certificate and private key:

```
openssl genrsa -out labe-kube-controller-manager.key 2048
openssl req -new -key labe-kube-controller-manager.key -subj "/CN=system:kube-controller-manager" -out labe-kube-controller-manager.csr
openssl x509 -req -in labe-kube-controller-manager.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out labe-kube-controller-manager.crt -days 1000
```

Results:

```
labe-kube-controller-manager.key
labe-kube-controller-manager.crt
```


### The Kube Proxy Client Certificate

Generate the `kube-proxy` client certificate and private key:


```
openssl genrsa -out labe-kube-proxy.key 2048
openssl req -new -key labe-kube-proxy.key -subj "/CN=system:kube-proxy" -out labe-kube-proxy.csr
openssl x509 -req -in labe-kube-proxy.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out labe-kube-proxy.crt -days 1000
```

Results:

```
labe-kube-proxy.key
labe-kube-proxy.crt
```

### The Scheduler Client Certificate

Generate the `kube-scheduler` client certificate and private key:



```
openssl genrsa -out labe-kube-scheduler.key 2048
openssl req -new -key labe-kube-scheduler.key -subj "/CN=system:kube-scheduler" -out labe-kube-scheduler.csr
openssl x509 -req -in labe-kube-scheduler.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out labe-kube-scheduler.crt -days 1000
```

Results:

```
labe-kube-scheduler.key
labe-kube-scheduler.crt
```

### The Kubernetes API Server Certificate

The kube-apiserver certificate requires all names that various components may reach it to be part of the alternate names. These include the different DNS names, and IP addresses such as the master servers IP address, the load balancers IP address, the kube-api service IP address etc.

The `openssl` command cannot take alternate names as command line parameter. So we must create a `conf` file for it:

```
cat > labe-openssl.cnf <<EOF
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
IP.2 = 192.168.5.66
IP.3 = 192.168.5.67
IP.4 = 192.168.5.75
IP.5 = 127.0.0.1
EOF
```

Generates certs for kube-apiserver

```
openssl genrsa -out labe-kube-apiserver.key 2048
openssl req -new -key labe-kube-apiserver.key -subj "/CN=kube-apiserver" -out labe-kube-apiserver.csr -config labe-openssl.cnf
openssl x509 -req -in labe-kube-apiserver.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out labe-kube-apiserver.crt -extensions v3_req -extfile labe-openssl.cnf -days 1000
```

Results:

```
labe-kube-apiserver.crt
labe-kube-apiserver.key
```

### The ETCD Server Certificate

Similarly ETCD server certificate must have addresses of all the servers part of the ETCD cluster

The `openssl` command cannot take alternate names as command line parameter. So we must create a `conf` file for it:

```
cat > labe-openssl-etcd.cnf <<EOF
[req]
req_extensions = v3_req
distinguished_name = req_distinguished_name
[req_distinguished_name]
[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @alt_names
[alt_names]
IP.1 = 192.168.5.66
IP.2 = 192.168.5.67
IP.3 = 127.0.0.1
EOF
```

Generates certs for ETCD

```
openssl genrsa -out labe-etcd-server.key 2048
openssl req -new -key labe-etcd-server.key -subj "/CN=etcd-server" -out labe-etcd-server.csr -config labe-openssl-etcd.cnf
openssl x509 -req -in labe-etcd-server.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out labe-etcd-server.crt -extensions v3_req -extfile labe-openssl-etcd.cnf -days 1000
```

Results:

```
labe-etcd-server.key
labe-etcd-server.crt
```

## The Service Account Key Pair

The Kubernetes Controller Manager leverages a key pair to generate and sign service account tokens as describe in the [managing service accounts](https://kubernetes.io/docs/admin/service-accounts-admin/) documentation.

Generate the `service-account` certificate and private key:

```
openssl genrsa -out labe-service-account.key 2048
openssl req -new -key labe-service-account.key -subj "/CN=service-accounts" -out labe-service-account.csr
openssl x509 -req -in labe-service-account.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out labe-service-account.crt -days 1000
```

Results:

```
labe-service-account.key
labe-service-account.crt
```


## Distribute the Certificates

Copy the appropriate certificates and private keys to each controller instance:

```
for instance in labe-master-1 labe-master-2; do
  scp ca.crt ca.key labe-kube-apiserver.key labe-kube-apiserver.crt \
    labe-service-account.key labe-service-account.crt \
    labe-etcd-server.key labe-etcd-server.crt \
    ${instance}:~/
done
```

> The `kube-proxy`, `kube-controller-manager`, `kube-scheduler`, and `kubelet` client certificates will be used to generate client authentication configuration files in the next lab. These certificates will be embedded into the client authentication configuration files. We will then copy those configuration files to the other master nodes.

Next: [Generating Kubernetes Configuration Files for Authentication](05-kubernetes-configuration-files.md)


#### LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F
#### LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F labf
#### LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F


Generate the `kube-controller-manager` client certificate and private key:

```
openssl genrsa -out labf-kube-controller-manager.key 2048
openssl req -new -key labf-kube-controller-manager.key -subj "/CN=system:kube-controller-manager" -out labf-kube-controller-manager.csr
openssl x509 -req -in labf-kube-controller-manager.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out labf-kube-controller-manager.crt -days 1000
```

Results:

```
labf-kube-controller-manager.key
labf-kube-controller-manager.crt
```


### The Kube Proxy Client Certificate

Generate the `kube-proxy` client certificate and private key:


```
openssl genrsa -out labf-kube-proxy.key 2048
openssl req -new -key labf-kube-proxy.key -subj "/CN=system:kube-proxy" -out labf-kube-proxy.csr
openssl x509 -req -in labf-kube-proxy.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out labf-kube-proxy.crt -days 1000
```

Results:

```
labf-kube-proxy.key
labf-kube-proxy.crt
```

### The Scheduler Client Certificate

Generate the `kube-scheduler` client certificate and private key:



```
openssl genrsa -out labf-kube-scheduler.key 2048
openssl req -new -key labf-kube-scheduler.key -subj "/CN=system:kube-scheduler" -out labf-kube-scheduler.csr
openssl x509 -req -in labf-kube-scheduler.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out labf-kube-scheduler.crt -days 1000
```

Results:

```
labf-kube-scheduler.key
labf-kube-scheduler.crt
```

### The Kubernetes API Server Certificate

The kube-apiserver certificate requires all names that various components may reach it to be part of the alternate names. These include the different DNS names, and IP addresses such as the master servers IP address, the load balancers IP address, the kube-api service IP address etc.

The `openssl` command cannot take alternate names as command line parameter. So we must create a `conf` file for it:

```
cat > labf-openssl.cnf <<EOF
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
IP.2 = 192.168.5.81
IP.3 = 192.168.5.82
IP.4 = 192.168.5.90
IP.5 = 127.0.0.1
EOF
```

Generates certs for kube-apiserver

```
openssl genrsa -out labf-kube-apiserver.key 2048
openssl req -new -key labf-kube-apiserver.key -subj "/CN=kube-apiserver" -out labf-kube-apiserver.csr -config labf-openssl.cnf
openssl x509 -req -in labf-kube-apiserver.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out labf-kube-apiserver.crt -extensions v3_req -extfile labf-openssl.cnf -days 1000
```

Results:

```
labf-kube-apiserver.crt
labf-kube-apiserver.key
```

### The ETCD Server Certificate

Similarly ETCD server certificate must have addresses of all the servers part of the ETCD cluster

The `openssl` command cannot take alternate names as command line parameter. So we must create a `conf` file for it:

```
cat > labf-openssl-etcd.cnf <<EOF
[req]
req_extensions = v3_req
distinguished_name = req_distinguished_name
[req_distinguished_name]
[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @alt_names
[alt_names]
IP.1 = 192.168.5.81
IP.2 = 192.168.5.82
IP.3 = 127.0.0.1
EOF
```

Generates certs for ETCD

```
openssl genrsa -out labf-etcd-server.key 2048
openssl req -new -key labf-etcd-server.key -subj "/CN=etcd-server" -out labf-etcd-server.csr -config labf-openssl-etcd.cnf
openssl x509 -req -in labf-etcd-server.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out labf-etcd-server.crt -extensions v3_req -extfile labf-openssl-etcd.cnf -days 1000
```

Results:

```
labf-etcd-server.key
labf-etcd-server.crt
```

## The Service Account Key Pair

The Kubernetes Controller Manager leverages a key pair to generate and sign service account tokens as describe in the [managing service accounts](https://kubernetes.io/docs/admin/service-accounts-admin/) documentation.

Generate the `service-account` certificate and private key:

```
openssl genrsa -out labf-service-account.key 2048
openssl req -new -key labf-service-account.key -subj "/CN=service-accounts" -out labf-service-account.csr
openssl x509 -req -in labf-service-account.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out labf-service-account.crt -days 1000
```

Results:

```
labf-service-account.key
labf-service-account.crt
```


## Distribute the Certificates

Copy the appropriate certificates and private keys to each controller instance:

```
for instance in labf-master-1 labf-master-2; do
  scp ca.crt ca.key labf-kube-apiserver.key labf-kube-apiserver.crt \
    labf-service-account.key labf-service-account.crt \
    labf-etcd-server.key labf-etcd-server.crt \
    ${instance}:~/
done
```

> The `kube-proxy`, `kube-controller-manager`, `kube-scheduler`, and `kubelet` client certificates will be used to generate client authentication configuration files in the next lab. These certificates will be embedded into the client authentication configuration files. We will then copy those configuration files to the other master nodes.

Next: [Generating Kubernetes Configuration Files for Authentication](05-kubernetes-configuration-files.md)

