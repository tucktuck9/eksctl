# KMS Envelope Encryption for EKS clusters

EKS supports using [AWS KMS](https://aws.amazon.com/about-aws/whats-new/2020/03/amazon-eks-adds-envelope-encryption-for-secrets-with-aws-kms/) keys to provide envelope encryption of Kubernetes secrets stored in EKS. Implementing envelope encryption is considered a security best practice for applications that store sensitive data and is part of a [_defense in depth_ security strategy](https://www.us-cert.gov/bsi/articles/knowledge/principles/defense-in-depth).


## Creating a cluster with KMS encryption enabled

```yaml
# kms-cluster.yaml
# A cluster with KMS encryption enabled
---
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: kms-cluster
  region: us-west-2

managedNodeGroups:
- name: ng
# more config

secretsEncryption:
  # KMS key used for envelope encryption of Kubernetes secrets
  keyARN: arn:aws:kms:us-west-2:<account>:key/<key>

```

```shell
$ eksctl create cluster -f kms-cluster.yaml
```


## Enabling KMS encryption on an existing cluster

To enable KMS encryption on a cluster that doesn't already have it enabled, run

```shell
$ eksctl utils enable-secrets-encryption -f kms-cluster.yaml
```

or without a config file:

```shell
$ eksctl utils enable-secrets-encryption --cluster=kms-cluster --key-arn=arn:aws:kms:us-west-2:<account>:key/<key> --region=<region>
```

In addition to enabling KMS encryption on the EKS cluster, eksctl also re-encrypts all existing Kubernetes secrets using the new KMS key
by updating them with the annotation `eksctl.io/kms-encryption-timestamp`. This behaviour can be disabled by passing `--encrypt-existing-secrets=false`, as in:


```shell
$ eksctl utils enable-secrets-encryption --cluster=kms-cluster --key-arn=arn:aws:kms:us-west-2:<account>:key/<key> --encrypt-existing-secrets=false --region=<region>
```

If a cluster already has KMS encryption enabled, eksctl will proceed to re-encrypting all existing secrets.


???+ note
    Once KMS encryption is enabled, it cannot be disabled or updated to use a different KMS key.
