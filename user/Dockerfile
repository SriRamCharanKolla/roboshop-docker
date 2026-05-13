FROM node:20
# this creates /app and move there
WORKDIR /app
COPY package.json .
COPY *.js .
ENV MONGO="true" \
    REDIS_URL="redis://redis:6379" \
    MONGO_URL="mongodb://mongodb:27017/users"
EXPOSE 8080
RUN npm install
CMD ["node", "server.js"]