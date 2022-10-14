# Daemonsets

```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: exemplo-ds
spec:
  selector:
    matchLabels:
      app: exemplo-ds
  template:
    metadata:
      labels:
        app: exemplo-ds
    spec:
      containers:
      - image: nginx:alpine
        name: nginx
```
