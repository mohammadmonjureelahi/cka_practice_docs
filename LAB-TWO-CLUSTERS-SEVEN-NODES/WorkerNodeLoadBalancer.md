## Let us create an yaml file named metallb.yaml and run kubectl apply -f metallb.yam in each cluster


apiVersion: v1
kind: Namespace
metadata:
  labels:
    app: metallb
  name: metallb-system
---
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  labels:
    app: metallb
  name: speaker
  namespace: metallb-system
spec:
  allowPrivilegeEscalation: false
  allowedCapabilities:
  - NET_ADMIN
  - NET_RAW
  - SYS_ADMIN
  fsGroup:
    rule: RunAsAny
  hostNetwork: true
  hostPorts:
  - max: 7472
    min: 7472
  privileged: true
  runAsUser:
    rule: RunAsAny
  seLinux:
    rule: RunAsAny
  supplementalGroups:
    rule: RunAsAny
  volumes:
  - '*'
---
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    app: metallb
  name: controller
  namespace: metallb-system
---
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    app: metallb
  name: speaker
  namespace: metallb-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  labels:
    app: metallb
  name: metallb-system:controller
rules:
- apiGroups:
  - ''
  resources:
  - services
  verbs:
  - get
  - list
  - watch
  - update
- apiGroups:
  - ''
  resources:
  - services/status
  verbs:
  - update
- apiGroups:
  - ''
  resources:
  - events
  verbs:
  - create
  - patch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  labels:
    app: metallb
  name: metallb-system:speaker
rules:
- apiGroups:
  - ''
  resources:
  - services
  - endpoints
  - nodes
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - ''
  resources:
  - events
  verbs:
  - create
  - patch
- apiGroups:
  - extensions
  resourceNames:
  - speaker
  resources:
  - podsecuritypolicies
  verbs:
  - use
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  labels:
    app: metallb
  name: config-watcher
  namespace: metallb-system
rules:
- apiGroups:
  - ''
  resources:
  - configmaps
  verbs:
  - get
  - list
  - watch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  labels:
    app: metallb
  name: metallb-system:controller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: metallb-system:controller
subjects:
- kind: ServiceAccount
  name: controller
  namespace: metallb-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  labels:
    app: metallb
  name: metallb-system:speaker
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: metallb-system:speaker
subjects:
- kind: ServiceAccount
  name: speaker
  namespace: metallb-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  labels:
    app: metallb
  name: config-watcher
  namespace: metallb-system
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: config-watcher
subjects:
- kind: ServiceAccount
  name: controller
- kind: ServiceAccount
  name: speaker
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  labels:
    app: metallb
    component: speaker
  name: speaker
  namespace: metallb-system
spec:
  selector:
    matchLabels:
      app: metallb
      component: speaker
  template:
    metadata:
      annotations:
        prometheus.io/port: '7472'
        prometheus.io/scrape: 'true'
      labels:
        app: metallb
        component: speaker
    spec:
      containers:
      - args:
        - --port=7472
        - --config=config
        env:
        - name: METALLB_NODE_NAME
          valueFrom:
            fieldRef:
              fieldPath: spec.nodeName
        - name: METALLB_HOST
          valueFrom:
            fieldRef:
              fieldPath: status.hostIP
        image: metallb/speaker:v0.8.1
        imagePullPolicy: IfNotPresent
        name: speaker
        ports:
        - containerPort: 7472
          name: monitoring
        resources:
          limits:
            cpu: 100m
            memory: 100Mi
        securityContext:
          allowPrivilegeEscalation: false
          capabilities:
            add:
            - NET_ADMIN
            - NET_RAW
            - SYS_ADMIN
            drop:
            - ALL
          readOnlyRootFilesystem: true
      hostNetwork: true
      nodeSelector:
        beta.kubernetes.io/os: linux
      serviceAccountName: speaker
      terminationGracePeriodSeconds: 0
      tolerations:
      - effect: NoSchedule
        key: node-role.kubernetes.io/master
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: metallb
    component: controller
  name: controller
  namespace: metallb-system
spec:
  revisionHistoryLimit: 3
  selector:
    matchLabels:
      app: metallb
      component: controller
  template:
    metadata:
      annotations:
        prometheus.io/port: '7472'
        prometheus.io/scrape: 'true'
      labels:
        app: metallb
        component: controller
    spec:
      containers:
      - args:
        - --port=7472
        - --config=config
        image: metallb/controller:v0.8.1
        imagePullPolicy: IfNotPresent
        name: controller
        ports:
        - containerPort: 7472
          name: monitoring
        resources:
          limits:
            cpu: 100m
            memory: 100Mi
        securityContext:
          allowPrivilegeEscalation: false
          capabilities:
            drop:
            - all
          readOnlyRootFilesystem: true
      nodeSelector:
        beta.kubernetes.io/os: linux
      securityContext:
        runAsNonRoot: true
        runAsUser: 65534
      serviceAccountName: controller
      terminationGracePeriodSeconds: 0
  


###########################################################################

Then let us apply the below configuration files for various clusters:

hardway-cluster-a:


apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - 192.168.51.-192.168.5.100
   


hardway-cluster-b:


apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - 192.168.5.101-192.168.5.150


###########################################################################

Changing the Role for the worker nodes from none to worker.

kubectl label node clustera-worker-1 node-role.kubernetes.io/worker=worker --context=hardway-cluster-a
kubectl label node clustera-worker-2 node-role.kubernetes.io/worker=worker --context=hardway-cluster-a
kubectl label node clustera-worker-3 node-role.kubernetes.io/worker=worker --context=hardway-cluster-a
kubectl label node clustera-worker-4 node-role.kubernetes.io/worker=worker --context=hardway-cluster-a
kubectl label node clustera-worker-5 node-role.kubernetes.io/worker=worker --context=hardway-cluster-a



kubectl label node clusterb-worker-1 node-role.kubernetes.io/worker=worker --context=hardway-cluster-b
kubectl label node clusterb-worker-2 node-role.kubernetes.io/worker=worker --context=hardway-cluster-b
kubectl label node clusterb-worker-3 node-role.kubernetes.io/worker=worker --context=hardway-cluster-b
kubectl label node clusterb-worker-4 node-role.kubernetes.io/worker=worker --context=hardway-cluster-b
kubectl label node clusterb-worker-5 node-role.kubernetes.io/worker=worker --context=hardway-cluster-b

Outputs:

kubectl get nodes --context=hardway-cluster-a

clustera-worker-1   Ready    worker   10h   v1.18.0
clustera-worker-2   Ready    worker   10h   v1.18.0
clustera-worker-3   Ready    worker   10h   v1.18.0
clustera-worker-4   Ready    worker   10h   v1.18.0
clustera-worker-5   Ready    worker   10h   v1.18.0


kubectl get nodes --context=hardway-cluster-b

clusterb-worker-1   Ready    worker   10h   v1.18.0
clusterb-worker-2   Ready    worker   10h   v1.18.0
clusterb-worker-3   Ready    worker   10h   v1.18.0
clusterb-worker-4   Ready    worker   10h   v1.18.0
clusterb-worker-5   Ready    worker   10h   v1.18.0





