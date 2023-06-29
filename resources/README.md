### Usando Resources

#### 01 - Especificação de *requests* e *limits* de memória:


Crie o seguinte Pod no cluster:

```
apiVersion: v1
kind: Pod
metadata:
  name: memory-demo
  namespace: default
spec:
  containers:
  - name: memory-demo-ctr
    image: polinux/stress
    resources:
      requests:
        memory: "100Mi"
      limits:
        memory: "200Mi"
    command: ["stress"]
    args: ["--vm", "1", "--vm-bytes", "150M", "--vm-hang", "1"]
```

```
kubectl apply -f exemplo-01.yaml
```

#### 02 - Excendendo o limite de memória

Crie o seguinte Pod no cluster:

```
apiVersion: v1
kind: Pod
metadata:
  name: memory-demo-2
  namespace: default
spec:
  containers:
  - name: memory-demo-2-ctr
    image: polinux/stress
    resources:
      requests:
        memory: "50Mi"
      limits:
        memory: "100Mi"
    command: ["stress"]
    args: ["--vm", "1", "--vm-bytes", "250M", "--vm-hang", "1"]
```

```
kubectl apply -f exemplo-02.yaml
```

#### 03 - Nodes sem memória suficiente para entregar para o Pod

```
apiVersion: v1
kind: Pod
metadata:
  name: memory-demo-3
  namespace: default
spec:
  containers:
  - name: memory-demo-3-ctr
    image: polinux/stress
    resources:
      requests:
        memory: "1000Gi"
      limits:
        memory: "1000Gi"
    command: ["stress"]
    args: ["--vm", "1", "--vm-bytes", "150M", "--vm-hang", "1"]
```

```
kubectl apply -f exemplo-03.yaml
```
