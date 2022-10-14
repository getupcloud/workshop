# Namespaces

É um mecanismo criado para segregação de recursos dentro dos clusters. Vale ressaltar que há duas categorias de objetos no Kubernetes em relação ao namespace, uns possuem escopo restrito por ele, outros não. Isso quer dizer que existem unidades com abrangência global ao nível de todo o cluster.

Por padrão, o kubernetes recém instalado possui quatro namespaces. Para listá-los, execute o seguinte comando:

```
kubectl get namespaces
```

- `default`: namespace padrão para instalação de quaisquer objetos sem não precisam de um dedicado;
- `kube-system`: namespace para objetos do sistema do Kubernetes;
- `kube-public`: namespace criado com permissão de leitura para todos os usuários, mesmo que não estejam autenticados;
- `kube-node-lease`: namespace para guardar objetos do tipo `Lease`, com a finalidade de permitir o kubelet enviar `heartbeats` para o control-plane detectar falhas nos nós.


Para criação de um novo namespace:

```
kubectl create namespace NAME
```

Uma boa prática para registro da informação é documentar os objetos criados no cluster. Sob essa ótica, é possível criar o manifesto para documentar a ação aplicada em um repositório git. Segue abaixo um exemplo com um namespace de nome `getup`:

```
apiVersion: v1
kind: Namespace
metadata:
  name: getup
spec: {}
```
Outras razões para se usar namespaces:

- evitar conflitos de aplicações com o mesmo nome em diferentes times;
- compartilhar recursos de um mesmo cluster para ambientes distintos, como por exemplo *staging* e *production*;
- limitar acesso de um determinado time apenas à aplicação que eles usam;
- limitar recursos de cpu e memória por namespace;

Dica de como mudar o namespace corrente via kubectl:

```
kubectl config set-context --current --namespace=NAME
```
