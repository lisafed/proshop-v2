FROM node:18-alpine 
WORKDIR /app 
COPY package*.json ./ 
RUN npm install 
COPY backend ./backend 
COPY frontend ./frontend 
EXPOSE 5000 
CMD ["npm", "run", "server"]