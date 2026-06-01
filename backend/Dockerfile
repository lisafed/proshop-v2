FROM node:18-alpine 
WORKDIR /app 
COPY package*.json ./ 
RUN npm install 
COPY backend ./backend 
COPY uploads ./uploads
COPY frontend ./frontend 
EXPOSE 5000 
CMD ["npm", "run", "server"]