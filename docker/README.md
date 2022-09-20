# Docker

Este documento abordará os princípios básicos do Docker e a construção de imagens seguindo as [melhores práticas](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/). Uma imagem Docker consiste em uma estrutura de camadas apenas de leitura, criadas a partir de um arquivo de instrução `Dockerfile`.

## Exercício 01

### Tarefa:
- Baixar a imagem do banco de dados MySQL e inspecioná-la usando o software de nome [Dive](https://github.com/wagoodman/dive);

### Solução:
- Instale o binário do `Dive` com o seguinte comando:
```
wget https://github.com/wagoodman/dive/releases/download/v0.9.2/dive_0.9.2_linux_amd64.deb
```
```
sudo dpkg -i dive_0.9.2_linux_amd64.deb
```
- Execute o `Dive` passando a imagem desejada como argumento:
```
dive mysql:latest
```
- Navegue por todas as camadas da imagem para evidenciar as alterações geradas por cada instrução do Dockerfile;


## Exercício 02

### Tarefa:
- Crie uma imagem docker para a aplicação em Node.js de exemplo presente na pasta [exercicio-02](exercicio-02). Nessa pasta você encontrará os arquivos `package.json` e o `server.js`. Após a criação da imagem, rode um container e liste seus processos para avaliar seu comportamento. Encontre esse mesmo processo a partir do host e identifique com qual usuário ele está rodando.

### Solução:
- Crie o arquivo `.dockerignore` no mesmo diretório do código e adicione o seguinte conteúdo:

```
node_modules
npm-debug.log
```

Ele tem a função de impedir que os arquivos listados não sejam copiados para dentro da imagem durante o build, consequentemente reduzindo o seu tamanho. Essa é uma boa prática a ser seguida, adicione apenas o estritamente necessário para a sua aplicação.

- Crie o arquivo `Dockerfile` passando as instruções para construir sua imagem Docker. Para evidenciar alguns casos típicos de problemas na elaboração desse arquivo, o `Dockerfile-sample` foi adicionado para um comparativo.

Faça o build da imagem:

```
docker build . -t node-web-app
```

Rode o container:

```
docker run --name node-web-app -d node-web-app
```

Acesse o shell do container e liste os processos:

```
docker exec -it node-web-app bash
ps -aux
```
Repita os passos anteriores para o arquivo `Dockerfile-sample`:

```
docker build . -t node-web-app-sample -f Dockerfile-sample
docker run --name node-web-app-sample -d node-web-app-sample
docker exec -it node-web-app-sample bash
ps -aux
```





