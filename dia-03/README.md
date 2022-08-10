# DIA 03 - Deployments, ReplicaSet e HPA

## Introdução

Após o contato com os pods, precisamos subir um nível na abstração e apresentar novos objetos que permitem um controle e gestão mais refinado das aplicações. O próximo nível após os pods é o objeto chamado *ReplicaSet*, cuja função é manter um número estabelecido de pods idênticos no cluster. O mecanismo de identificação de quais pods serão geridos pelo ReplicaSet baseia-se no uso de labels aplicados aos pods. Acima desse nível, temos o Deployment, que gerencia os ReplicaSet e entrega um recuros de controle de atualizações. Veja a figura abaixo para visualizar a hierarquia de um deployment:

![deploymet](deployment.png)

### Usando Deployments

#### 01 - Deployment básico

Segue um exemplo básico de deployment:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

Para aplicar no cluster, execute:

```
kubectl apply -f https://k8s.io/examples/controllers/nginx-deployment.yaml
```

Verifique todos os nomes objetos que surgiram no cluster:

```
kubectl get deployment
kubectl get replicaset
kubectl get pods
kubectl describe deployment nginx-deployment
```

Inspecione os objetos `metadata.ownerReferences` do pod e do ReplicaSet criados.

Altere a imagem presente nesse deployment para `nginx:alpine` com o seguinte comando:

```
kubectl edit deploy nginx-deployment

ou

kubectl set image deployment/nginx-deployment nginx=nginx:alpine
```

Após a alteração, veja que novos pods rodam no cluster. Um outro ponto a se notar é que um novo ReplicaSet é criado para contemplar esse novo cenário.

Escale o deployment para cinco réplicas:

```
kubectl scale deployment/nginx-deployment --replicas=5
```

#### 02 - HorizontalPodAutoscaler (HPA)

Para utilização do HPA precisaremos instalar o [metrics-server](https://github.com/kubernetes-sigs/metrics-server) no cluster:

```
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/high-availability.yaml
```

Para concluir a instalação, edite o deployment adicionando um campo `--kubelet-insecure-tls` da lista de `args` do container.

Instale o deployment de exemplo abaixo e inspecione os objetos no cluster:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hpa-demo-deployment
spec:
  selector:
    matchLabels:
      run: hpa-demo-deployment
  replicas: 1
  template:
    metadata:
      labels:
        run: hpa-demo-deployment
    spec:
      containers:
      - name: hpa-demo-deployment
        image: k8s.gcr.io/hpa-example
        ports:
        - containerPort: 80
        resources:
          limits:
            cpu: 500m
          requests:
            cpu: 200m
```

Crie o service no cluster:

```
apiVersion: v1
kind: Service
metadata:
  name: hpa-demo-deployment
  labels:
    run: hpa-demo-deployment
spec:
  ports:
  - port: 80
  selector:
    run: hpa-demo-deployment
```

Crie o HPA no cluster:

```
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: hpa-demo-deployment
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: hpa-demo-deployment
  minReplicas: 1
  maxReplicas: 10
  targetCPUUtilizationPercentage: 50
```

Suba um container usando a image busybox para adicionar carga de CPU no pod:

```
kubectl run -i --tty load-generator --rm --image=busybox --restart=Never
```

Dentro do container rode o seguinte comando:

```
while sleep 0.01; do wget -q -O- http://hpa-demo-deployment; done
```

Monitore o HPA e o quantitativo de pods enquanto o comando acima é executado:

```
watch kubectl get pods,hpa
```
