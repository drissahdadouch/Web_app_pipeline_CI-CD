FROM node:20-buster AS builder

WORKDIR /app
COPY package*.json ./
RUN npm install  --force
COPY . .
RUN npm run build  # Commande pour construire l'application Next.js

FROM node:16 AS stage-1

WORKDIR /app
COPY --from=builder /app .
EXPOSE 3000
CMD ["npm", "start"]
