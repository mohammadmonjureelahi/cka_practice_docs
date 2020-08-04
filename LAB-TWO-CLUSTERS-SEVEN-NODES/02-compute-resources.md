########## kubernetes-the-hard-way-cluster-lab-a  ###############################

# Provisioning Compute Resources


CD into vagrant directory

`cd kubernetes-the-hard-way-cluster-lab-a\vagrant`

Run Vagrant up

`vagrant up`


This does the below:

- Deploys 5 VMs - 2 Master, 2 Worker and 1 Loadbalancer with the name 'laba-* '
    > This is the default settings. This can be changed at the top of the Vagrant file

- Set's IP addresses in the range 192.168.5.*

    | VM Host Name       |  VM Name          | Purpose       | IP           | Forwarded Port   |
    | -------------------| ------------------|:-------------:| ------------:| ----------------:|
    | laba-master-1      | k8s-laba-master-1 | Master        | 192.168.5.6  |     2706         |
    | laba-master-2      | k8s-laba-master-2 | Master        | 192.168.5.7  |     2707         |
    | laba-worker-1      | k8s-laba-worker-1 | Worker        | 192.168.5.11 |     2711         |
    | laba-worker-2      | k8s-laba-worker-2 | Worker        | 192.168.5.12 |     2712         |
    | laba-loadbalancer  | k8s-laba-lb       | LoadBalancer  | 192.168.5.15 |     2715         |

    > These are the default settings. These can be changed in the Vagrant file

- Add's a DNS entry to each of the nodes to access internet
    > DNS: 8.8.8.8

- Install's Docker on Worker nodes
- Runs the below command on all nodes to allow for network forwarding in IP Tables.
  This is required for kubernetes networking to function correctly.
    > sysctl net.bridge.bridge-nf-call-iptables=1




########## kubernetes-the-hard-way-cluster-lab-b  ###############################

# Provisioning Compute Resources


CD into vagrant directory

`cd kubernetes-the-hard-way-cluster-lab-b\vagrant`

Run Vagrant up

`vagrant up`


This does the below:

- Deploys 5 VMs - 2 Master, 2 Worker and 1 Loadbalancer with the name 'labb-* '
    > This is the default settings. This can be changed at the top of the Vagrant file

- Set's IP addresses in the range 192.168.5.*

    | VM Host Name       |  VM Name          | Purpose       | IP           | Forwarded Port   |
    | -------------------| ------------------|:-------------:| ------------:| ----------------:|
    | labb-master-1      | k8s-labb-master-1 | Master        | 192.168.5.21 |     2721         |
    | labb-master-2      | k8s-labb-master-2 | Master        | 192.168.5.22 |     2722         |
    | labb-worker-1      | k8s-labb-worker-1 | Worker        | 192.168.5.26 |     2726         |
    | labb-worker-2      | k8s-labb-worker-2 | Worker        | 192.168.5.27 |     2727         |
    | labb-loadbalancer  | k8s-labb-lb       | LoadBalancer  | 192.168.5.30 |     2730         |

    > These are the default settings. These can be changed in the Vagrant file

- Add's a DNS entry to each of the nodes to access internet
    > DNS: 8.8.8.8

- Install's Docker on Worker nodes
- Runs the below command on all nodes to allow for network forwarding in IP Tables.
  This is required for kubernetes networking to function correctly.
    > sysctl net.bridge.bridge-nf-call-iptables=1



########## kubernetes-the-hard-way-cluster-lab-c  ###############################

# Provisioning Compute Resources


CD into vagrant directory

`cd kubernetes-the-hard-way-cluster-lab-c\vagrant`

Run Vagrant up

`vagrant up`


This does the below:

- Deploys 5 VMs - 2 Master, 2 Worker and 1 Loadbalancer with the name 'labc-* '
    > This is the default settings. This can be changed at the top of the Vagrant file

- Set's IP addresses in the range 192.168.5.*

    | VM Host Name       |  VM Name          | Purpose       | IP           | Forwarded Port   |
    | -------------------| ------------------|:-------------:| ------------:| ----------------:|
    | labc-master-1      | k8s-labc-master-1 | Master        | 192.168.5.36 |     2736         |
    | labc-master-2      | k8s-labc-master-2 | Master        | 192.168.5.37 |     2737         |
    | labc-worker-1      | k8s-labc-worker-1 | Worker        | 192.168.5.41 |     2741         |
    | labc-worker-2      | k8s-labc-worker-2 | Worker        | 192.168.5.42 |     2742         |
    | labc-loadbalancer  | k8s-labc-lb       | LoadBalancer  | 192.168.5.45 |     2745         |

    > These are the default settings. These can be changed in the Vagrant file

- Add's a DNS entry to each of the nodes to access internet
    > DNS: 8.8.8.8

- Install's Docker on Worker nodes
- Runs the below command on all nodes to allow for network forwarding in IP Tables.
  This is required for kubernetes networking to function correctly.
    > sysctl net.bridge.bridge-nf-call-iptables=1


########## kubernetes-the-hard-way-cluster-lab-d  ###############################

# Provisioning Compute Resources


CD into vagrant directory

`cd kubernetes-the-hard-way-cluster-lab-d\vagrant`

Run Vagrant up

`vagrant up`


This does the below:

- Deploys 5 VMs - 2 Master, 2 Worker and 1 Loadbalancer with the name 'labd-* '
    > This is the default settings. This can be changed at the top of the Vagrant file

- Set's IP addresses in the range 192.168.5.*

    | VM Host Name       |  VM Name          | Purpose       | IP           | Forwarded Port   |
    | -------------------| ------------------|:-------------:| ------------:| ----------------:|
    | labd-master-1      | k8s-labd-master-1 | Master        | 192.168.5.51 |     2751         |
    | labd-master-2      | k8s-labd-master-2 | Master        | 192.168.5.52 |     2752         |
    | labd-worker-1      | k8s-labd-worker-1 | Worker        | 192.168.5.56 |     2756         |
    | labd-worker-2      | k8s-labd-worker-2 | Worker        | 192.168.5.57 |     2757         |
    | labd-loadbalancer  | k8s-labd-lb       | LoadBalancer  | 192.168.5.60 |     2760         |

    > These are the default settings. These can be changed in the Vagrant file

- Add's a DNS entry to each of the nodes to access internet
    > DNS: 8.8.8.8

- Install's Docker on Worker nodes
- Runs the below command on all nodes to allow for network forwarding in IP Tables.
  This is required for kubernetes networking to function correctly.
    > sysctl net.bridge.bridge-nf-call-iptables=1


########## kubernetes-the-hard-way-cluster-lab-e  ###############################

# Provisioning Compute Resources


CD into vagrant directory

`cd kubernetes-the-hard-way-cluster-lab-e\vagrant`

Run Vagrant up

`vagrant up`


This does the below:

- Deploys 5 VMs - 2 Master, 2 Worker and 1 Loadbalancer with the name 'labe-* '
    > This is the default settings. This can be changed at the top of the Vagrant file

- Set's IP addresses in the range 192.168.5.*

    | VM Host Name       |  VM Name          | Purpose       | IP           | Forwarded Port   |
    | -------------------| ------------------|:-------------:| ------------:| ----------------:|
    | labe-master-1      | k8s-labe-master-1 | Master        | 192.168.5.66 |     2766         |
    | labe-master-2      | k8s-labe-master-2 | Master        | 192.168.5.67 |     2767         |
    | labe-worker-1      | k8s-labe-worker-1 | Worker        | 192.168.5.71 |     2771         |
    | labe-worker-2      | k8s-labe-worker-2 | Worker        | 192.168.5.72 |     2772         |
    | labe-loadbalancer  | k8s-labe-lb       | LoadBalancer  | 192.168.5.75 |     2775         |

    > These are the default settings. These can be changed in the Vagrant file

- Add's a DNS entry to each of the nodes to access internet
    > DNS: 8.8.8.8

- Install's Docker on Worker nodes
- Runs the below command on all nodes to allow for network forwarding in IP Tables.
  This is required for kubernetes networking to function correctly.
    > sysctl net.bridge.bridge-nf-call-iptables=1


########## kubernetes-the-hard-way-cluster-lab-e  ###############################

# Provisioning Compute Resources


CD into vagrant directory

`cd kubernetes-the-hard-way-cluster-lab-e\vagrant`

Run Vagrant up

`vagrant up`


This does the below:

- Deploys 5 VMs - 2 Master, 2 Worker and 1 Loadbalancer with the name 'labf-* '
    > This is the default settings. This can be changed at the top of the Vagrant file

- Set's IP addresses in the range 192.168.5.*

    | VM Host Name       |  VM Name          | Purpose       | IP           | Forwarded Port   |
    | -------------------| ------------------|:-------------:| ------------:| ----------------:|
    | labf-master-1      | k8s-labf-master-1 | Master        | 192.168.5.81 |     2781         |
    | labf-master-2      | k8s-labf-master-2 | Master        | 192.168.5.82 |     2782         |
    | labf-worker-1      | k8s-labf-worker-1 | Worker        | 192.168.5.86 |     2786         |
    | labf-worker-2      | k8s-labf-worker-2 | Worker        | 192.168.5.87 |     2787         |
    | labf-loadbalancer  | k8s-labf-lb       | LoadBalancer  | 192.168.5.90 |     2790         |

    > These are the default settings. These can be changed in the Vagrant file

- Add's a DNS entry to each of the nodes to access internet
    > DNS: 8.8.8.8

- Install's Docker on Worker nodes
- Runs the below command on all nodes to allow for network forwarding in IP Tables.
  This is required for kubernetes networking to function correctly.
    > sysctl net.bridge.bridge-nf-call-iptables=1


########## kubernetes hardway management machine for all 6 clusters  ###############################

# Provisioning Compute Resources


    | VM Host Name       |  VM Name          | Purpose       | IP           -| 
    | -------------------| ------------------|:-------------:| -------------:| 
    | k8smanagement      | k8smanagement     | Management    | 192.168.5.100 |   



