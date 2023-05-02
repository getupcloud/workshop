# Kubernetes Dashboard

Instale o repositório de Helm:

```
helm repo add kubernetes-dashboard https://kubernetes.github.io/dashboard/
helm repo update
```

Instale o chart do Kubernetes Dashboard:

```
helm install kubernetes-dashboard kubernetes-dashboard/kubernetes-dashboard \
-n kubernetes-dashboard --create-namespace \
--version 6.0.7 \
--set protocolHttp=true \
--set extraArgs[0]='--enable-skip-login' \
--set extraArgs[1]='--disable-settings-authorizer' \
--set extraArgs[2]='--enable-insecure-login' \
--set extraArgs[3]='--insecure-bind-address=0.0.0.0' \
--set metricsScraper.enabled=true \
--wait
```

Crie os seguintes objetos de permissionamento para visualização do cluster:

```
kubectl -n kubernetes-dashboard create sa admin-user
kubectl create clusterrolebinding admin-user --clusterrole cluster-admin --serviceaccount kubernetes-dashboard:admin-user
kubectl -n kubernetes-dashboard create token admin-user
```

Execute o direcionamento da porta do service para outra porta da máquina:

```
kubectl port-forward -n kubernetes-dashboard --address 0.0.0.0 svc/kubernetes-dashboard 8080:443
```

Por fim, acesse a opção de tráfego de portas do killercoda e selecione a porta 8080 do Host1 para ter acesso ao dashboard do seu cluster. Lembre-se de usar o token gerado nos comandos anteriores para se autenticar.


## Referências usadas:

- https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/
- https://github.com/killercoda/scenario-examples/blob/main/kubernetes-dashboard/assets/dashboard.yaml
- https://artifacthub.io/packages/helm/k8s-dashboard/kubernetes-dashboard
