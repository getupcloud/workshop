# Ferramentas Auxiliares

Seção dedicada com as instruções de instalação de aplicações auxiliares aos exercícios do Workshop.

- [Kubernetes Dashboard](#dashboard)
- [Metrics Server](#metrics-server)

## Dashboard

Bloco de comandos para instalação da aplicação:
```
helm repo add kubernetes-dashboard https://kubernetes.github.io/dashboard
helm repo update
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
kubectl wait --for=condition=ready -n kubernetes-dashboard pod -l "app.kubernetes.io/component=kubernetes-dashboard"
kubectl -n kubernetes-dashboard create sa admin-user
kubectl create clusterrolebinding admin-user --clusterrole cluster-admin --serviceaccount kubernetes-dashboard:admin-user
kubectl port-forward -n kubernetes-dashboard --address 0.0.0.0 svc/kubernetes-dashboard 8080:443 &
kubectl -n kubernetes-dashboard create token admin-user
```

Por fim, acesse a opção de tráfego de portas do killercoda e selecione a porta 8080 do Host1 para ter acesso ao dashboard do seu cluster. Lembre-se de usar o token gerado nos comandos anteriores para se autenticar.


### Referências usadas:

- https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/
- https://github.com/killercoda/scenario-examples/blob/main/kubernetes-dashboard/assets/dashboard.yaml
- https://artifacthub.io/packages/helm/k8s-dashboard/kubernetes-dashboard

## Metrics Server

Bloco de comandos para instalação da aplicação:


```
helm repo add metrics-server https://kubernetes-sigs.github.io/metrics-server/
helm repo update
helm upgrade --install metrics-server metrics-server/metrics-server -n metrics-server --create-namespace \
--version 3.10.0 \
--set args={--kubelet-insecure-tls}
```

### Referências usadas:

- https://github.com/kubernetes-sigs/metrics-server
- https://artifacthub.io/packages/helm/metrics-server/metrics-server
