# Resource Quotas

Resource Quota é um objeto nativo do Kubernetes para estabelecer restrições a nível de namespace. É possível estabelecer tetos para números de Pods, request e limit de CPU e memória, armazenameto através de volumes e outros objetos.

## Exemplo 01

- Aplique o seguinte resourecequota no namespace default:

```
apiVersion: v1
kind: ResourceQuota
metadata:
  name: pods-default
spec:
  hard:
    cpu: "500m"
    memory: 500Mi
    pods: "5"
```

Crie um deployment conforme o manifesto abaixo:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 1
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
        image: nginx:alpine
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "100Mi"
            cpu: "100m"
          limits:
            memory: "100Mi"
            cpu: "100m"
```

Escale esse deployment gradativamente e acompanhe o resultado do provisionamento dos Pods no cluster.

## Exemplo 02

- Aplique o seguinte objeto no namespace default:

```
apiVersion: v1
kind: ResourceQuota
metadata:
  name: object-counts
spec:
  hard:
    configmaps: "2"
    persistentvolumeclaims: "4"
    pods: "4"
    replicationcontrollers: "20"
    secrets: "2"
    services: "2"
    services.loadbalancers: "2"
```

- Crie objetos no cluster para atingir esses limites estabelecidos e verifique o comportamento.
