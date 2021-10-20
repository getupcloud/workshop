# Instalação de ferramentas

## Preparando o ambiente
Criar o diretório para instalação dos binários
```shell
mkdir -p ~/bin
```

Adicionar o diretório de instaladores no $PATH do usuário.
```shell
export PATH=$PATH:$HOME/bin
```

Fixar no .bashrc:
```shell
echo 'export PATH=$PATH:$HOME/bin' >> ~/.bashrc
```
Fixar
```shell
echo 'export PATH=$PATH:$HOME/bin' >> ~/.zshrc
```

## Instalando ferramentas
### kubectl
##### Cliente da API do Kubernetes
```shell
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
mv ./kubectl ~/bin/kubectl
chmod +x ~/bin/kubectl
```

### helm v3
##### Gerenciador de pacotes para Kubernetes
```shell
curl -LO https://get.helm.sh/helm-v3.2.1-linux-amd64.tar.gz
tar -zxvf helm-v3.2.1-linux-amd64.tar.gz
mv ./linux-amd64/helm ~/bin/helm
```

### kubectx
##### Mudar contexto do kubeconfig (alterar apontamento entre clusters)
```shell
curl -LO https://github.com/ahmetb/kubectx/releases/download/v0.9.4/kubectx_v0.9.4_linux_x86_64.tar.gz
tar -zxvf kubectx_v0.9.4_linux_x86_64.tar.gz
mv ./kubectx ~/bin/kubectx
chmod +x ~/bin/kubectx
```

### kubens
##### Navegar entre namespaces do cluster
```shell
curl -LO https://github.com/ahmetb/kubectx/releases/download/v0.9.4/kubens_v0.9.4_linux_x86_64.tar.gz
tar -zxvf kubens_v0.9.4_linux_x86_64.tar.gz
mv ./kubens ~/bin/kubens
chmod +x ~/bin/kubens
```
### kconf
##### Gerenciar o arquivo de configuração do kubectl
```shell
curl -LO https://github.com/particledecay/kconf/releases/download/v1.10.1/kconf-linux-x86_64-1.10.1.tar.gz
tar -zxvf kconf-linux-x86_64-1.10.1.tar.gz
mv ./kconf ~/bin/kconf
chmod +x ~/bin/kconf
```
### kind
##### Criar clusters locais para desenvolvimento
```shell
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.11.1/kind-linux-amd64
mv ./kind ~/bin/kind
chmod +x ~/bin/kind
```

### k9s
##### Interface de usuário baseada em terminal para interagir com o Kubernetes
```shell
curl -LO https://github.com/derailed/k9s/releases/download/v0.24.15/k9s_Linux_arm64.tar.gz
tar -zxvf k9s_Linux_arm64.tar.gz
mv ./k9s ~/bin/k9s
chmod +x ~/bin/k9s
```

### LENS
##### Interface de usuário gráfica para interagir com o Kubernetes
Acessar https://k8slens.dev/ e fazer o download para a versão do seu sistema operacional
