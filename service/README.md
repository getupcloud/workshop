# Services

## Introdução

Service é uma abstração a nível de rede para expor os pods de forma perene, possibilitando o acesso através de um nome e criando um balanceador para distribuir a carga igualmente entre os pods. A definicão de quais pods serão balanceados pelo service se dá através de um selector, que varre os pods que possuem uma determinada label.

![service](service-example.png)

### Usando Services

#### 01 - Expondo um pod

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app.kubernetes.io/name: proxy
spec:
  containers:
  - name: nginx
    image: nginx:stable
    ports:
      - containerPort: 80
        name: http-web-svc

---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app.kubernetes.io/name: proxy
  ports:
  - name: name-of-service-port
    protocol: TCP
    port: 80
    targetPort: http-web-svc
```

#### 02 - Expondo um deployment - ClusterIP

![clusterip](clusterip.png)

```
kubectl create deploy nginx --image=nginx:alpine
```

```
kubectl expose deploy/nginx --port=80 --target-port=80 --type=ClusterIP
```

#### 03 - NodePort

Crie um deployment com 2 réplicas:

```
kubectl create deploy nginx --image=nginx:alpine --replicas=2
```

Crie um service do tipo NodePort para esse deploy:

```
kubectl expose deploy/nginx --port=80 --target-port=80 --type=NodePort
```

Identifique o endereço IP de cada nó:

```
kubectl get nodes -o wide
```

Verifique se a conexão com porta alta do service é fechada com sucesso:

```
nc -vz IP PORTA

ou

curl IP:PORTA
```

#### 04 - Port forward a service

```
kubectl create deploy nginx --image=nginx:alpine
```

```
kubectl expose deploy/nginx --port=80 --target-port=80 --type=ClusterIP
```

```
kubectl port-forward --address 0.0.0.0 svc/nginx 8080:80
```

#### 05 - LoadBalancer

Instale o metallb:

```
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.14.9/config/manifests/metallb-native.yaml
```

Aplique o seguinte manifesto no cluster:

```
---
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: first-pool
  namespace: metallb-system
spec:
  addresses:
  - 172.30.0.100-172.30.0.200
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: example
  namespace: metallb-system
spec:
  ipAddressPools:
  - first-pool
```

Crie um deployment com nginx com um service do tipo LoadBalancer:

```
kubectl create deploy nginx --image=nginx:alpine
```

```
kubectl expose deploy/nginx --port=80 --target-port=80 --type=LoadBalancer
```

#### 06 - Headless - Deployment

Crie os seguintes recursos no cluster:

```
apiVersion: v1
kind: Service
metadata:
  name: regular-svc
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: headless-svc
spec:
  clusterIP: None
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

Rode um pod para avaliar a resolução de DNS em cima de cada service criado:

```
$ kubectl run -it --rm testedns --image=nicolaka/netshoot
> nslookup regular-svc.default.svc.cluster.local
> nslookup headless-svc.default.svc.cluster.local
```

#### 07 - Headless - StatefulSet

Crie os seguintes recursos no cluster:

```
---
apiVersion: v1
kind: Service
metadata:
  name: headless-svc-sts
spec:
  clusterIP: None
  selector:
    app: web-sts
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-sts
  labels:
    app: server-sts
spec:
  serviceName: headless-svc-sts
  replicas: 3
  selector:
    matchLabels:
      app: web-sts
  template:
    metadata:
      labels:
        app: web-sts
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

Rode um pod para avaliar a resolução de DNS em cima de cada service e pod criados:

```
$ kubectl run -it --rm testedns --image=nicolaka/netshoot
> nslookup headless-svc-sts.default.svc.cluster.local
> nslookup app-sts-0.headless-svc-sts.default.svc.cluster.local
```

