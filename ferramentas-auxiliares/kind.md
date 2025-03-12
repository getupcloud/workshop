### Kind

Kind é uma implementaçào de Kubernetes dentro do Docker (Kubernetes-IN-Docker).

Baixe o binário para sua arquitetura em https://github.com/kubernetes-sigs/kind/releases/tag/v0.27.0
Crie um cluster Kind com o comando:

```
$ kind create cluster
Creating cluster "kind" ...
 ✓ Ensuring node image (kindest/node:v1.32.0) 🖼
 ✓ Preparing nodes 📦  
 ✓ Writing configuration 📜 
 ✓ Starting control-plane 🕹️ 
 ✓ Installing CNI 🔌 
 ✓ Installing StorageClass 💾 
Set kubectl context to "kind-kind"
You can now use your cluster with:

kubectl cluster-info --context kind-kind

Have a nice day! 👋

$ kubectl get node
NAME                 STATUS   ROLES           AGE     VERSION
kind-control-plane   Ready    control-plane   5m55s   v1.32.0
```

Instale o MetalLB para usar services do tipo LoadBalancer:


```sh
DOCKER_CIDR=$(docker network inspect kind --format '{{(index .IPAM.Config 0).Subnet}}')\
PREFIX=$(cut -f1-2 -d. <<<$DOCKER_CIDR)
NET=$(shuf -n1 -i 100-200)
START=$(shuf -n1 -i 100-200)
METALLB_RANGE=$PREFIX.$NET.$START-$PREFIX.$NET.$END

$ cat >metallb-values.yaml <<EOF
configInline:
  address-pools:
  - name: default
    protocol: layer2
    addresses:
    - $METALLB_RANGE
EOF

helm install metallb-controller metallb \
    --repo https://charts.bitnami.com/bitnami \
    --version 3.0.12 \
    --values ./metallb-values.yaml \
    --namespace metallb-system \
    --create-namespace

```

Instale o Nginx Ingress Controller para usar objetos Ingress:

```sh
helm install ingress-nginx ingress-nginx \
    --repo https://kubernetes.github.io/ingress-nginx \
    --version 4.10.1 \
    --namespace ingress-nginx \
    --create-namespace

kubectl get svc -n ingress-nginx ingress-nginx-ingress-nginx-controller -o jsonpath={.status.loadBalancer.ingress[0].ip}
```
