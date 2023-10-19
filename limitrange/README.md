# Limit Ranges

Limit Range é um objeto da api do Kubernetes que permite criar uma política a nível de namespace para restringir a alocação de recursos. Ele provê as seguintes restrições:

- Aplicar o uso mínimo e máximo de recursos computacionais por pod ou contêiner em um namespace.
- Impor a solicitação de armazenamento mínimo e máximo por PersistentVolumeClaim em um namespace.
- Impor a proporção entre solicitação e limite para um recurso em um namespace.
- Definir a solicitação/limite padrão para recursos computacionais em um namespace e utilizá-los automaticamente nos contêineres em tempo de execução.

## Exemplo 01

Aplique o seguinte LimitRange:

```
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-resource-constraint
spec:
  limits:
  - default: # this section defines default limits
      cpu: 500m
    defaultRequest: # this section defines default requests
      cpu: 500m
    max: # max and min define the limit range
      cpu: "1"
    min:
      cpu: 100m
    type: Container
```

Inspecione o objeto criado:

```
kubectl describe limitranges
kubectl get limitranges cpu-resource-constraint -o yaml
```

Crie um Pod com o seguinte manifesto:

```
apiVersion: v1
kind: Pod
metadata:
  name: example-conflict-with-limitrange-cpu
spec:
  containers:
  - name: demo
    image: registry.k8s.io/pause:2.0
    resources:
      requests:
        cpu: 700m
```

Porque o pod não é criado? Na sequência, aplique o seguinte manifesto:

```
apiVersion: v1
kind: Pod
metadata:
  name: example-no-conflict-with-limitrange-cpu
spec:
  containers:
  - name: demo
    image: registry.k8s.io/pause:2.0
    resources:
      requests:
        cpu: 700m
      limits:
        cpu: 700m
```
