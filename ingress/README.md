# Ingress

## Introdução

Ingress permite acessar serviços internos HTTP(S) de fora do cluster através de mapeamentos de hostnames, URIs e PATHs.

### Usando Ingress

Utilizaremos o controlador metallb para permitir a criação de services do tipo LoadBalancer, que implementa um VIP dentro do kubernetes.


Primeiro é necessário garantir queo kube-proxy está devidamente configurado para rodar junto com metallb. Edite o config map abaixo e mude, se necessário, os campos `mode` e `ipvs.strictARP`.

```
kubectl edit configmap -n kube-system kube-proxy
```

Verifique os valores abaixo:

```
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
...
mode: "ipvs"             <---- MODE deve ser `ipvs`
ipvs:
  strictARP: true        <---- strictARP=true impede que a interface kube-ipvs0 respondar por requisições ARP, o que interfere no funcionamento do metallb
```

Por fim, instale o metallb:

```
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.14.9/config/manifests/metallb-native.yaml
```

Aplique o seguinte manifesto no cluster para atribuir automaticamente IPs do range `172.30.1.100 - 172.30.1.200` aos services do tipo LoadBalancer.

```
---
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: first-pool
  namespace: metallb-system
spec:
  addresses:
  - 172.30.1.100-172.30.1.200
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

Instale o ingress nginx:

```
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install ingress-nginx ingress-nginx/ingress-nginx -n ingress-nginx --create-namespace
```

Crie um depoyment para sua aplicação, expondo a porta 9899 do container na porta 80 do service:

```
kubectl create deploy podinfo --port=9898 --image=ghcr.io/stefanprodan/podinfo:latest
kubectl expose deploy/podinfo --port=80 --target-port=9898
```

Crie um objeto Ingress para expor o service para fora do cluster. Este será totalmente gerenciado pelo controlador `ingress-nginx`

```
kubectl create ingress podinfo --class=nginx --rule="podinfo.exemplo.com/*=podinfo:80"
```

Configure o "DNS" do host para responder a URL do Ingress com o IP do service:

```
kubectl get service ingress-nginx-controller -o wide          #<------------ Anote o EXTERNAL-IP
echo EXTERNAL-IP podinfo.exemplo.com >>  /etc/hosts
ping podinfo.exemplo.com

curl http://podinfo.exemplo.com
```
