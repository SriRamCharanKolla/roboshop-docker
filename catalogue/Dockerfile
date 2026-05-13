FROM node:20
# this creates /app and move there
WORKDIR /app
COPY package.json .
COPY *.js .
RUN npm install
ENV MONGO="true" \
    MONGO_URL="mongodb://mongodb:27017/catalogue"
CMD ["node", "server.js"]
