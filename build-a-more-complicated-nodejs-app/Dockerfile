FROM node:20

USER node

WORKDIR /home/node/code

COPY --chown=node package*.json ./

RUN npm ci

COPY --chown=node . .

CMD [ "node", "index.js"]
