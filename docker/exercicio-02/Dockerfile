FROM node:16

WORKDIR /usr/src/app

COPY package*.json ./

RUN npm install

COPY . .

EXPOSE 8080

RUN useradd -ms /bin/bash  nodejs

USER nodejs

CMD [ "node", "server.js" ]
