# Pods

[Pod](https://kubernetes.io/docs/concepts/workloads/pods/) é um grupo de um ou mais containers, que compartilham armazenamento, rede e uma especificação de como devem rodar no ambiente.

Características de um pod:

- cada pod possui um endereço ip único, acessível de qualquer outro dentro do cluster;
- dentro de um mesmo pod, os containers podem se comunicar através de localhost:porta

![pod](../img/pod.png)



## Usando pods

### 01 - Pod simples

Exemplo de manifesto de um pod contento uma imagem de nginx:

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-01
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```

Para aplicar no cluster rode o seguinte comando:

```
kubectl apply -f https://k8s.io/examples/pods/simple-pod.yaml
```

### 02 - Pod com dois containers

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-02
  labels:
    app: nginx
spec:
  containers:
  - name: nginx-container
    image: nginx
    ports:
    - containerPort: 80
  - name: sidecar
    image: curlimages/curl
    command: ["/bin/sh"]
    args: ["-c", "echo Hello from the sidecar container; sleep 3000"]
```

Salve o manifesto acima sob o nome sidecar.yaml e execute o seguinte comando:

```
kubectl apply -f sidecar.yaml
```

Execute um *netstat* e *curl* dentro do container de nome sidecar:

```
kubectl exec -it nginx-02 -c sidecar -- sh
netstat -tupan
curl localhost
```

Depois veja os logs separadamente de cada container:

```
kubectl logs nginx-02 -c nginx-container
kubectl logs nginx-02 -c sidecar
```

### 03 - Init containers

Crie um manifesto de nome *myapp.yaml* com o conteúdo abaixo:

```
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app.kubernetes.io/name: MyApp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup myservice.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for myservice; sleep 2; done"]
  - name: init-mydb
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup mydb.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for mydb; sleep 2; done"]
```

Inicie o pod:

```
kubectl apply -f myapp.yaml
```

Verifique o status e o log do pod:

```
kubectl get pods
kubectl describe pod myapp-pod
kubectl logs myapp-pod -c init-myservice
kubectl logs myapp-pod -c init-mydb
```

Crie os services abaixo (services.yaml) e repita o procedimento de verificação:

```
---
apiVersion: v1
kind: Service
metadata:
  name: myservice
spec:
  ports:
  - protocol: TCP
    port: 80
    targetPort: 9376
---
apiVersion: v1
kind: Service
metadata:
  name: mydb
spec:
  ports:
  - protocol: TCP
    port: 80
    targetPort: 9377
```

```
kubectl apply -f services.yaml
```
