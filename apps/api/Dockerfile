# syntax=docker/dockerfile:1.6

########  Etapa 1: deps (Debian + toolchain)  ########
FROM node:20-bullseye-slim AS deps
WORKDIR /workspace

# Toolchain para paquetes nativos (swc/sharp/etc.)
RUN apt-get update && apt-get install -y --no-install-recommends \
    python3 make g++ git ca-certificates \
 && rm -rf /var/lib/apt/lists/*

# Archivos mínimos para resolver deps y cachear
COPY package*.json ./
COPY nx.json tsconfig.base.json ./

# <<< clave: ignorar scripts para evitar el Bus error >>>
ENV npm_config_ignore_scripts=true
RUN npm ci --legacy-peer-deps --no-audit --no-fund

########  Etapa 2: build ########
FROM deps AS build
WORKDIR /workspace
COPY . .
ENV NX_DAEMON=false
# (opcional) comprobar nx
RUN npx nx --version
# compila la app: genera dist/apps/api
RUN npx nx build api

########  Etapa 3: runtime ########
FROM node:20-bullseye-slim AS runtime
ENV NODE_ENV=production
WORKDIR /app

COPY package*.json ./
# también sin scripts (no necesitamos binarios nativos en runtime)
ENV npm_config_ignore_scripts=true
RUN npm ci --omit=dev --legacy-peer-deps --no-audit --no-fund && npm cache clean --force

# Copiamos artefactos compilados
COPY --from=build /workspace/dist/apps/api ./dist/apps/api

# Usuario no root (opcional)
RUN useradd -m nodeuser
USER nodeuser

ENV PORT=3333
EXPOSE 3333
CMD ["node", "dist/apps/api/main.js"]
