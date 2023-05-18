# Ingress

## Introdução

![ingress](ingress.svg)

### Usando Ingress


Instale o metallb:

```
kubectl edit configmap -n kube-system kube-proxy
```
O campo `strictARP` deve ser alterado para `true`.
```
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
mode: "ipvs"
ipvs:
  strictARP: true
```

```
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.4/config/manifests/metallb-native.yaml
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

```
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
```

```
helm repo update
```

```
helm install ingress-nginx ingress-nginx/ingress-nginx -n ingress-nginx --create-namespace
```

```
kubectl create deploy nginx --port=80 --image=nginx:alpine
```

```
kubectl expose deploy/nginx --port=80 --target-port=80
```

```
kubectl create ingress nginx --class=nginx --rule="nginx.marcelo.local/=nginx:80"
```

```
curl http://nginx.marcelo.local
```
