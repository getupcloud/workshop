# ServiceAccount


- Crie um Pod no namespace default com a imagem do nginx

```
kubectl run teste --image=nginx
```

- Inspecione no manifesto qual é a serviceaccount usada por esse Pod

```
kubectl get pods teste -oyaml
```

- Acesse o shell do container nginx e inspecione a pasta `/var/run/secrets/kubernetes.io/serviceaccount`

```
kubectl exec -it teste -- bash
cd /var/run/secrets/kubernetes.io/serviceaccount
ls -la
```

- Faça requisições para a api do cluster

```
TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
echo $TOKEN
curl -k -H "Authorization: Bearer $TOKEN" https://kubernetes.default:443/api
curl -k -H "Authorization: Bearer $TOKEN" https://kubernetes.default:443/api/v1/namespaces/default/pods/teste
```

- Crie um novo namespace

```
kubectl create ns getup
```

- Verifique a serviceaccount desse novo namespace

```
kubectl get sa -n getup
kubectl get sa -n getup default -oyaml
```

- provisione um Pod de nginx novamente e avalie como ele usa o objeto serviceaccount

```
kubectl run teste --image=nginx -n getup
```
Repita os passos executados para o Pod do namespace default


- crie uma nova serviceaccount

```
kubectl create sa getup -n getup
```

- Dê permissão a serviceaccount `getup` para usar a clusterole `view`

```
kubectl create clusterrolebinding getup --clusterrole=view --serviceaccount=getup:getup
```

- Crie um novo Pod de nome getup-view usando a serviceaccount `getup` e repita os passos de requisições para a API do cluster.

```
kubectl run getup-view -n getup --image=nginx --dry-run=client -oyaml > getup-view.yaml
```
Edite esse o manifesto desse Pod e adicione a serviceaccount getup

```
kubectl exec -it getup-view -- bash
```

