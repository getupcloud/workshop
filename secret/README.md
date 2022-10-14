# Secrets

## Introdução

Secrets são objetos para armazenamento de informações sensiveis, como senhas ou tokens.

### Usando Secrets


#### 01 - Opaque

```
kubectl create secret generic test-secret --from-literal=username='my-app' --from-literal=password='39528$vdg7Jb'
```

```
apiVersion: v1
kind: Pod
metadata:
  name: envfrom-secret
spec:
  containers:
  - name: envars-test-container
    image: nginx
    envFrom:
    - secretRef:
        name: test-secret
```

```
apiVersion: v1
kind: Pod
metadata:
  name: env-secretkeyref
spec:
  containers:
  - name: envars-test-container
    image: nginx
    env:
    - name: username
      valueFrom:
        secretKeyRef:
          name: test-secret
          key: username
    - name: password
      valueFrom:
        secretKeyRef:
          name: test-secret
          key: password
```

#### 02 - Dockerconfigjson

```
kubectl create secret docker-registry regcred --docker-server=<your-registry-server> --docker-username=<your-name> --docker-password=<your-pword> --docker-email=<your-email>
```

Preencha com alguns dados fictícios e aplique no cluster. Inspecione o objeto criado para visualizar o formato do Yaml gerado.

Há duas formas de usar esse tipo de secret, diretamente no pod/deployment ou na serivceaccount.

Exemplo de uso no pod:

```
apiVersion: v1
kind: Pod
metadata:
  name: private-reg
spec:
  containers:
  - name: private-reg-container
    image: <your-private-image>
  imagePullSecrets:
  - name: regcred
```

Exemplo de uso na serviceaccount:

```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: default
imagePullSecrets:
- name: regcred
```

#### 03 - TLS secrets

O Kubernetes provê um tipo de secret própria para armazenar certificados TLS de aplicações (`kubernetes.io/tls`).

Crie um arquivo (`confs.cnf`) passando as configurações do certificado:

```
[req]
distinguished_name = req_distinguished_name
x509_extensions = v3_req
prompt = no
[req_distinguished_name]
C = BR
ST = RJ
L = Rio de Janeiro
O = Sua Empresa
OU = Departamento de TI
CN = *.workshop.getup.local
[v3_req]
keyUsage = critical, digitalSignature, keyAgreement
extendedKeyUsage = serverAuth
```

Rode o openssl para a criação do certificado (`tls.crt`) e da chave privada (`tls.key`):

```
openssl req -x509 -nodes -days 365 -newkey rsa:4096 -keyout tls.key -out tls.crt -config confs.cnf -sha256
```

Instale o ingress-nginx no cluster:

```
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
```

```
helm repo update
```

```
helm upgrade --install ingress-nginx ingress-nginx/ingress-nginx -n ingress-nginx --create-namespace \
--set controller.extraArgs.default-ssl-certificate=ingress-nginx/default-certificate
```

Uma vez o ingress instalado, basta criar a secret padrão no mesmo namespace do controlador:

```
kubectl create secret tls default-certificate -n ingress-nginx\
  --cert=tls.crt \
  --key=tls.key
```

Para o teste final, crie um deployment de nginx para expô-lo através de um ingress:

```
kubectl create deploy nginx --port=80 --image=nginx:alpine
```

```
kubectl expose deploy/nginx --port=80 --target-port=80
```

```
kubectl create ingress nginx --class=nginx --rule="nginx.workshop.getup.local/=nginx:80,tls"
```

```
curl -kv https://nginx.workshop.getup.local
```

```
apiVersion: v1
kind: Secret
metadata:
  name: secret-tls
type: kubernetes.io/tls
data:
  # the data is abbreviated in this example
  tls.crt: |
        MIIC2DCCAcCgAwIBAgIBATANBgkqh ...
  tls.key: |
        MIIEpgIBAAKCAQEA7yn3bRHQ5FHMQ ...
```

Como criar a secret do tipo `kubernetes.io/tls`:

```
kubectl create secret tls my-tls-secret \
  --cert=path/to/cert/file \
  --key=path/to/key/file
```
