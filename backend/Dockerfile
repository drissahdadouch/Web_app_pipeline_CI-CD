FROM node:20-buster AS builder

WORKDIR /app

COPY package*.json ./

RUN npm install --force

COPY . .

FROM node:16

WORKDIR /app

COPY --from=builder /app .
EXPOSE 5000
RUN npm install --only=production --force

CMD ["npm", "start"]
