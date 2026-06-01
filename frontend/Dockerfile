FROM node:18-alpine AS build 
WORKDIR /app 
COPY package*.json ./ 
RUN npm install 
COPY . . 
RUN npm run build
FROM nginx:alpine 
# Copie des fichiers React compilés 
COPY --from=build /app/build /usr/share/nginx/html 
# Configuration Nginx personnalisée 
COPY nginx.conf /etc/nginx/conf.d/default.conf 
EXPOSE 80 
CMD ["nginx", "-g", "daemon off;"]
