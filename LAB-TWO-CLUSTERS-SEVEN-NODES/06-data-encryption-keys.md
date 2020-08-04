#### LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A
#### LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A clustera
#### LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A

# Generating the Data Encryption Config and Key
# We will perform these on the k8smanagement node

Kubernetes stores a variety of data including cluster state, application configurations, and secrets. Kubernetes supports the ability to [encrypt](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data) cluster data at rest.

In this lab you will generate an encryption key and an [encryption config](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/#understanding-the-encryption-at-rest-configuration) suitable for encrypting Kubernetes Secrets.

## The Encryption Key

Generate an encryption key:

```
ENCRYPTION_KEY=$(head -c 32 /dev/urandom | base64)
```

## The Encryption Config File  clustera-clustera-encryption-config.yaml

Create the `clustera-encryption-config.yaml` encryption config file:

```
cat > clustera-encryption-config.yaml <<EOF
kind: EncryptionConfig
apiVersion: v1
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: ${ENCRYPTION_KEY}
      - identity: {}
EOF
```

Copy the `clustera-encryption-config.yaml` encryption config file to each controller instance:

```
for instance in clustera-master-1 clustera-master-2; do
  scp clustera-encryption-config.yaml ${instance}:~/
done
```
Reference: https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/#encrypting-your-data

Next: [Bootstrapping the etcd Cluster](07-bootstrapping-etcd.md)

#### LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B
#### LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B clusterb
#### LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B

## The Encryption Key

Generate an encryption key:

```
ENCRYPTION_KEY=$(head -c 32 /dev/urandom | base64)
```

## The Encryption Config File  clusterb-encryption-config.yaml

Create the `clusterb-encryption-config.yaml` encryption config file:

```
cat > clusterb-encryption-config.yaml <<EOF
kind: EncryptionConfig
apiVersion: v1
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: ${ENCRYPTION_KEY}
      - identity: {}
EOF
```

Copy the `clusterb-encryption-config.yaml` encryption config file to each controller instance:

```
for instance in clusterb-master-1 clusterb-master-2; do
  scp clusterb-encryption-config.yaml ${instance}:~/
done
```
