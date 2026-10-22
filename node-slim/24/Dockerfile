FROM node:24.22-bookworm-slim

ENV PORT=80
WORKDIR /app
EXPOSE 80

RUN apt update && apt upgrade -y && apt install -y --no-install-recommends \
    curl \
 && rm -rf /var/lib/apt/lists/*

RUN npm install -g npm@latest && npm cache clean --force
