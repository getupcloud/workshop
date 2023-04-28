# HorizontalPodAutoscaler (HPA)

Para utilização do HPA precisaremos instalar o [metrics-server](https://github.com/kubernetes-sigs/metrics-server) no cluster:

```
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

Para concluir a instalação, edite o deployment adicionando um campo `--kubelet-insecure-tls` da lista de `args` do container.

```
kubectl edit -n kube-system deployment metrics-server
```

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
