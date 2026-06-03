FROM node:18-alpine

WORKDIR /app

# 1. On copie le package.json de la racine ET celui du sous-dossier backend
COPY package*.json ./
COPY backend/package*.json ./backend/

# 2. On installe les dépendances (grâce aux deux fichiers copiés, tout est complet)
RUN npm install

# 3. On copie le reste du code du dossier backend uniquement
COPY backend ./backend

# 4. On expose le port de l'API Node.js
EXPOSE 5000

# 5. Commande de démarrage (on utilise "start" plutôt que "server" car nodemon n'est pas fait pour la prod)
CMD ["npm", "start"]