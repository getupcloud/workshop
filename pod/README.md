# Pods

[Pod](https://kubernetes.io/docs/concepts/workloads/pods/) é um grupo de um ou mais containers, que compartilham armazenamento, rede e uma especificação de como devem rodar no ambiente.

Características de um pod:

- executa um ou mais containers, sendo escalonado de forma conjunta;
- compartilha recursos de rede (interfaces, IPs, rotas, etc) com outros containers do mesmo pod;
- compartilha, opcionalmente, storage com outros containers do mesmo pod (emptyDir, PVC, secrets/configmaps);
- possui um endereço IP único no cluster, acessível a partir de qualquer outro pod do cluster;

![pod](../img/pod.png)

## Usando pods

### 01 - Pod simples

Exemplo de manifesto de um pod contendo uma imagem de nginx:

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

  - name: web
    image: nginx
    ports:
    - containerPort: 80

  - name: sidecar
    image: curlimages/curl
    command:        ## executa command sem shell
    - /bin/sh
    - -c
    - "echo Hello from the sidecar container; sleep inf"
```

Salve o manifesto acima com o nome `sidecar.yaml` e execute o seguinte comando:

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
kubectl logs nginx-02 -c web
kubectl logs nginx-02 -c sidecar
```


### 02.1 - Variaveis de ambiente

Variaveis de ambiente podem ser definidas em `pod.spec.containers.env[].{name, value}`. Tanteo `name` quanto `value` devem ser strings, portanto defina `value: "123"` ao inves de `value: 123`.

```
apiVersion: v1
kind: Pod
metadata:
  name: envs
  labels:
    app: envs
spec:
  containers:

  - name: app
    image: nginx
    env:
    - name: MENSAGEM
      value: "Hello from the sidecar container"
    command:        ## executa command sem shell
    - /bin/sh
    - -c
    - "echo $MENSAGEM; sleep inf"
```

Variáveis de abviente podem ser expandidas previamente, antes da criação do container usando a notação `$(NAME)`:


```
apiVersion: v1
kind: Pod
metadata:
  name: envs
  labels:
    app: envs
spec:
  containers:

  - name: app
    image: nginx
    env:
    - name: MY_SHELL
      value: "/bin/bash"
    command:
    - $(MY_SHELL)
    - -c
    - "echo Hello world; sleep inf"
```
### 03 - Sinais

Ao deletar um pod (`kubectl delete pod/$name`), o kubelet do node envia o sinal `TERM`[1] para o processo terminar de forma graciosa.
Após `pods.spec.terminationGracePeriodSeconds` segundos, caso o container não termine, o sinal `KILL` é enviado para finalizar o processo forçadamente.

```
apiVersion: v1
kind: Pod
metadata:
  name: envs
  labels:
    app: envs
spec:
  terminationGracePeriodSeconds: 5
  containers:
  - name: app
    image: ubuntu
    command:
    - /bin/bash
    - -c
    - |+
      echo Entrando...
      trap 'echo "Sinal $(( $? - 128 )) recebido. Saindo..."' SIGTERM
      sleep inf
```

> 1. Veja https://docs.docker.com/reference/dockerfile/#stopsignal para utilizar outro sinal.

### 04 - Init containers

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
