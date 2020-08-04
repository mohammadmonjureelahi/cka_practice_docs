#### LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A
#### LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A LAB-A laba
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

## The Encryption Config File  laba-laba-encryption-config.yaml

Create the `laba-encryption-config.yaml` encryption config file:

```
cat > laba-encryption-config.yaml <<EOF
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

Copy the `laba-encryption-config.yaml` encryption config file to each controller instance:

```
for instance in laba-master-1 laba-master-2; do
  scp laba-encryption-config.yaml ${instance}:~/
done
```
Reference: https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/#encrypting-your-data

Next: [Bootstrapping the etcd Cluster](07-bootstrapping-etcd.md)

#### LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B
#### LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B labb
#### LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B LAB-B

## The Encryption Key

Generate an encryption key:

```
ENCRYPTION_KEY=$(head -c 32 /dev/urandom | base64)
```

## The Encryption Config File  labb-encryption-config.yaml

Create the `labb-encryption-config.yaml` encryption config file:

```
cat > labb-encryption-config.yaml <<EOF
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

Copy the `labb-encryption-config.yaml` encryption config file to each controller instance:

```
for instance in labb-master-1 labb-master-2; do
  scp labb-encryption-config.yaml ${instance}:~/
done
```

#### LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C
#### LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C labc
#### LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C LAB-C

## The Encryption Key

Generate an encryption key:

```
ENCRYPTION_KEY=$(head -c 32 /dev/urandom | base64)
```

## The Encryption Config File  labc-encryption-config.yaml

Create the `labc-encryption-config.yaml` encryption config file:

```
cat > labc-encryption-config.yaml <<EOF
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

Copy the `labc-encryption-config.yaml` encryption config file to each controller instance:

```
for instance in labc-master-1 labc-master-2; do
  scp labc-encryption-config.yaml ${instance}:~/
done
```

#### LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D
#### LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D labd
#### LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D LAB-D

## The Encryption Key

Generate an encryption key:

```
ENCRYPTION_KEY=$(head -c 32 /dev/urandom | base64)
```

## The Encryption Config File  labd-encryption-config.yaml

Create the `labd-encryption-config.yaml` encryption config file:

```
cat > labd-encryption-config.yaml <<EOF
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

Copy the `labd-encryption-config.yaml` encryption config file to each controller instance:

```
for instance in labd-master-1 labd-master-2; do
  scp labd-encryption-config.yaml ${instance}:~/
done
```

#### LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E
#### LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E labe
#### LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E LAB-E

## The Encryption Key

Generate an encryption key:

```
ENCRYPTION_KEY=$(head -c 32 /dev/urandom | base64)
```

## The Encryption Config File  labe-encryption-config.yaml

Create the `labe-encryption-config.yaml` encryption config file:

```
cat > labe-encryption-config.yaml <<EOF
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

Copy the `labe-encryption-config.yaml` encryption config file to each controller instance:

```
for instance in labe-master-1 labe-master-2; do
  scp labe-encryption-config.yaml ${instance}:~/
done
```

#### LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F
#### LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F labf
#### LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F LAB-F

## The Encryption Key

Generate an encryption key:

```
ENCRYPTION_KEY=$(head -c 32 /dev/urandom | base64)
```

## The Encryption Config File  labf-encryption-config.yaml

Create the `labf-encryption-config.yaml` encryption config file:

```
cat > labf-encryption-config.yaml <<EOF
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

Copy the `labf-encryption-config.yaml` encryption config file to each controller instance:

```
for instance in labf-master-1 labf-master-2; do
  scp labf-encryption-config.yaml ${instance}:~/
done
```


