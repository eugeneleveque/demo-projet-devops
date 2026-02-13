# syntax=docker/dockerfile:1

############################
# 1) Build stage
############################
FROM node:24-alpine AS build

WORKDIR /app

# Installer dépendances (inclut devDependencies pour compiler TypeScript)
COPY package*.json ./
RUN npm ci

# Copier le code et builder
COPY . .
RUN npm run build


############################
# 2) Production stage
############################
FROM node:24-alpine AS runner

WORKDIR /app
ENV NODE_ENV=production

# Installer uniquement les dépendances de prod
COPY package*.json ./
RUN npm ci --omit=dev && npm cache clean --force

# Copier la sortie compilée
COPY --from=build /app/dist ./dist

EXPOSE 3002

# TASKS_SERVICE_URL + PORT doivent être passés au runtime
CMD ["node", "dist/server.js"]
