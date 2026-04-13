# syntax=docker/dockerfile:1

############################
# 1) Build stage
############################
FROM node:24-alpine AS build

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .
ARG VITE_TASKS_API_URL
ARG VITE_STATS_API_URL
ENV VITE_TASKS_API_URL=$VITE_TASKS_API_URL
ENV VITE_STATS_API_URL=$VITE_STATS_API_URL
RUN npm run build


############################
# 2) Serve stage (Nginx)
############################
FROM nginx:alpine AS runner

# Copier le build Vite vers Nginx
COPY --from=build /app/dist /usr/share/nginx/html

# (Optionnel mais recommandé) : config SPA pour gérer le routing côté client
# Si vous n'utilisez pas de routing (React Router), vous pouvez omettre.
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
