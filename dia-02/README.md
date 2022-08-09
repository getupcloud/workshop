# DIA 02 - Primeiros comandos com o kubectl

## Introdução

Esse dia é usado para a apresentação do kubectl e do pod, o menor objeto dentro de um cluster Kubernetes. Serão detalhados os principais campos do seu manifeto e o seu comportamento dentro dos nós.

## Kubectl

O [kubectl](https://kubernetes.io/docs/tasks/tools/) é um binário criado na linguagem [GO](https://go.dev/) para interagir com a API do cluster.

No decorrer do treinamento serão evidenciados os principais subcomandos do kubectl e seus parâmetros. Abaixo segue a referência para todos os subcomandos e alguns exemplos:

```
https://kubectl.docs.kubernetes.io/references/kubectl/
```

## Namespace

É um mecanismo criado para segregação de recursos dentro dos clusters. Vale ressaltar que há duas categorias de objetos no Kubernetes em relação ao namespace, uns possuem escopo restrito por ele, outros não. Isso quer dizer que existem unidades com abrangência global ao nível de todo o cluster.

Por padrão, o kubernetes recém instalado possui quatro namespaces. Para listá-los, execute o seguinte comando:

```
kubectl get namespaces
```

- `default`: namespace padrão para instalação de quaisquer objetos sem não precisam de um dedicado;
- `kube-system`: namespace para objetos do sistema do Kubernetes;
- `kube-public`: namespace criado com permissão de leitura para todos os usuários, mesmo que não estejam autenticados;
- `kube-node-lease`: namespace para guardar objetos do tipo `Lease`, com a finalidade de permitir o kubelet enviar `heartbeats` para o control-plane detectar falhas nos nós.


Para criação de um novo namespace:

```
kubectl create namespace NAME
```

Uma boa prática para registro da informação é documentar os objetos criados no cluster. Sob essa ótica, é possível criar o manifesto para documentar a ação aplicada em um repositório git. Segue abaixo um exemplo com um namespace de nome `getup`:

```
apiVersion: v1
kind: Namespace
metadata:
  name: getup
spec: {}
```
Outras razões para se usar namespaces:

- evitar conflitos de aplicações com o mesmo nome em diferentes times;
- compartilhar recursos de um mesmo cluster para ambientes distintos, como por exemplo *staging* e *production*;
- limitar acesso de um determinado time apenas à aplicação que eles usam;
- limitar recursos de cpu e memória por namespace;

Dica de como mudar o namespace corrente via kubectl:

```
kubectl config set-context --current --namespace=NAME
```

## Pod
[Pod](https://kubernetes.io/docs/concepts/workloads/pods/) é um grupo de um ou mais containers, que compartilham armazenamento, rede e uma especificação de como devem rodar no ambiente.

Características de um pod:

- cada pod possui um endereço ip único, acessível de qualquer outro dentro do cluster;
- dentro de um mesmo pod, os containers podem se comunicar através de localhost:porta


### Usando pods

#### 01 - Pod simples

Exemplo de manifesto de um pod contento uma imagem de nginx:

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
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

#### 02 - Pod com dois containers

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
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
kubectl exec nginx -c sidecar
netstat -tupan
curl localhost
```

Depois veja os logs separadamente de cada container:

```
kubectl logs nginx -c nginx
kubectl logs nginx -c sidecar
```

#### 03 - Init containers

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

kubectl apply -f services.yaml
