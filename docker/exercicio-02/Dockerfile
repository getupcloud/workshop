FROM node:16

WORKDIR /usr/src/app

COPY package*.json ./

RUN npm install

COPY . .

EXPOSE 8080

RUN useradd -ms /bin/bash  nodejs 

USER nodejs #especificação de permissão de uso como não root

CMD [ "node", "server.js" ]
