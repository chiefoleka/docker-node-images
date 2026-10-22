FROM node:24.10.0-bookworm-slim

ENV PORT=80
WORKDIR /app
EXPOSE 80

RUN apt update && apt upgrade -y && apt install -y --no-install-recommends \
    build-essential \
    curl \
 && rm -rf /var/lib/apt/lists/*
