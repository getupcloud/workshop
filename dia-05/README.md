# DIA 05 - Resources e Probes

## Introdução

### Usando Probes

#### 01 - Liveness com comando

```
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-exec
spec:
  containers:
  - name: liveness
    image: k8s.gcr.io/busybox
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -f /tmp/healthy; sleep 600
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5
```

```
watch kubectl get ev,pods
```

```
kubectl apply -f kubectl apply -f exemplo-01.yaml
```

#### 02 - Readiness com comando

```
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: readiness
  name: readiness-exec
spec:
  containers:
  - name: readiness
    image: k8s.gcr.io/busybox
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -f /tmp/healthy; sleep 600
    readinessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5
```

```
watch kubectl get ev,pods
```

```
kubectl apply -f exemplo-02.yaml
```

#### 03 - Liveness com requisição HTTP

O Liveness probe envia requisições HTTP relugares. Se o retorno for maior ou igual a 200 e menor do que 400, o pod é considerado saudável, caso contrário ele é reiniciado pelo kubelet.

```
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-http
spec:
  containers:
  - name: liveness
    image: k8s.gcr.io/liveness
    args:
    - /server
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
        httpHeaders:
        - name: Custom-Header
          value: Awesome
      initialDelaySeconds: 3
      periodSeconds: 3
```

```
watch kubectl get ev,pods
```

```
kubectl apply -f exemplo-03.yaml
```

#### 04 - Liveness e Readiness com requisição TCP

```
apiVersion: v1
kind: Pod
metadata:
  name: goproxy
  labels:
    app: goproxy
spec:
  containers:
  - name: goproxy
    image: k8s.gcr.io/goproxy:0.1
    ports:
    - containerPort: 8080
    readinessProbe:
      tcpSocket:
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 10
    livenessProbe:
      tcpSocket:
        port: 8080
      initialDelaySeconds: 15
      periodSeconds: 20
```

```
watch kubectl get ev,pods
```

```
kubectl apply -f exemplo-04.yaml
```

### Usando Resources

Instale o metrics-server como pré-requisito para os exemplos a seguir:

```
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

Para concluir a instalação, edite o deployment adicionando um campo `--kubelet-insecure-tls` da lista de `args` do container.

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


