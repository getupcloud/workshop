
# Operators

```
curl -sL https://github.com/operator-framework/operator-lifecycle-manager/releases/download/v0.21.2/install.sh | bash -s v0.21.2
```

```
kubectl create -f https://operatorhub.io/install/cloud-native-postgresql.yaml
```

```
apiVersion: postgresql.k8s.enterprisedb.io/v1
kind: Cluster
metadata:
  name: cluster-sample
spec:
  instances: 3
  logLevel: info
  primaryUpdateStrategy: unsupervised
  storage:
    size: 1Gi
```
