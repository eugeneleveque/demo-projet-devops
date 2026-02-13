# syntax=docker/dockerfile:1

############################
# 1) Dépendances
############################
FROM node:24-alpine AS deps

WORKDIR /app

# Installer uniquement les dépendances (lockfile obligatoire)
COPY package*.json ./
RUN npm ci


############################
# 2) Build (TypeScript)
############################
FROM node:24-alpine AS build

WORKDIR /app

# Réutiliser node_modules depuis deps
COPY --from=deps /app/node_modules ./node_modules

# Copier le code source
COPY . .

# Compiler l'application
RUN npm run build


############################
# 3) Production
############################
FROM node:24-alpine AS runner

WORKDIR /app
ENV NODE_ENV=production

# Sécurité : utilisateur non-root
# Ensure we prepare the directory permissions before switching user
RUN chown node:node /app

USER node

# Installer uniquement les dépendances de prod
COPY --chown=node:node package*.json ./
RUN npm ci --omit=dev && npm cache clean --force

# Copier la sortie compilée
COPY --from=build /app/dist ./dist

# Exposer le port applicatif
EXPOSE 3001

# Lancer l'application
CMD ["node", "dist/server.js"]
