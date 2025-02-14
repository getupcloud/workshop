#### 01 - Liveness com comando

```sh
kubectl create deployment liveness-exec --image=nginx -o yaml --dry-run > probes.yaml
```

Adicione ao deployment:

```yaml
command:
- /bin/sh
- -c
- |
  echo $(date) creating /tmp/healthy
  touch /tmp/healthy
  sleep 30
  echo $(date) removing /tmp/healthy
  rm -f /tmp/healthy
  sleep $((10 * 60))

livenessProbe:
  exec:
    command:
    - cat
    - /tmp/healthy
  initialDelaySeconds: 5
  periodSeconds: 2
  failureThreshold: 2
```

```sh
watch -d -n1 kubectl get ev,ep,pods
kubectl apply -f kubectl apply -f probes.yaml
```

#### 02 - Readiness com comando

Substitua `livenessProbe` por `readinessProbe`:

```sh
kubectl apply -f probe.yaml
```

#### 03 - Probe com requisição HTTP

O Liveness probe envia requisições HTTP relugares. Se o retorno for maior ou igual a 200 e menor do que 400, o pod é considerado saudável, caso contrário ele é reiniciado pelo kubelet.

Altera `readinessProbe` para:

```yaml
readinessProbe:
  httpGet:
    path: /
    port: 80
    httpHeaders:
    - name: Custom-Header
      value: Awesome
  initialDelaySeconds: 3
  periodSeconds: 3
  failureThreshold: 1
```

```sh
kubectl apply -f probe.yaml
```

#### 04 - Probe com requisição TCP

Altera `readinessProbe` para:

```yaml
readinessProbe:
  tcpSocket:
    port: 80
  initialDelaySeconds: 5
  periodSeconds: 5
```

```
kubectl apply -f probe.yaml
```

#### 05 - Startup Probe

Startup probe executa apenas na inicialização do container até que este retorne sucesso. Logo após, os outros probes entram em ação.
Serve para dar um tempo de inicialização para o processo, sem depender do `initialDelaySeconds`.

Substitua `readinessProbe` por `startupProbe` e modifique o `command`:

```yaml

command:
- /bin/sh
- -c
- |
  echo $(date) initializing process...
  sleep 15
  echo $(date) creating /tmp/healthy
  touch /tmp/healthy
  sleep $((10 * 60))

startupProbe:
  exec:
    command:
    - cat
    - /tmp/healthy
  initialDelaySeconds: 5
  periodSeconds: 2
  failureThreshold: 10
```

```sh
kubectl apply -f probe.yaml
```
