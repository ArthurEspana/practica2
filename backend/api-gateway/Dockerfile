FROM node:20-alpine

WORKDIR /app

COPY package*.json ./
RUN npm install -g npm@latest && npm install --omit=dev

COPY . .

EXPOSE 3000
CMD ["node", "src/server.js"]
