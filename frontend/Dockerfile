# Étape 1 : Compilation de l'application React
FROM node:18-alpine AS build
WORKDIR /app

# Copie des fichiers package du frontend
COPY frontend/package*.json ./
RUN npm install

# Copie de tout le code source du dossier frontend
COPY frontend/ .
RUN npm run build

# Étape 2 : Serveur Web de Production (Nginx)
FROM nginx:alpine

# Copie des fichiers statiques compilés de React vers Nginx
COPY --from=build /app/build /usr/share/nginx/html

# Copie du fichier de configuration Nginx (optionnel)
COPY frontend/nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]