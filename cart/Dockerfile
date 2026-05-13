FROM node:20
# this creates /app and move there
WORKDIR /app
COPY package.json .
COPY *.js .
RUN npm install
ENV REDIS_HOST="redis" \
    CATALOGUE_HOST="catalogue" \
    CATALOGUE_PORT="8080"
CMD ["node", "server.js"]