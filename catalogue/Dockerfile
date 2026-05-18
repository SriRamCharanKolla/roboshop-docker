# FROM node:20
# # this creates /app and move there
# WORKDIR /app
# COPY package.json .
# COPY *.js .
# RUN npm install
# ENV MONGO="true" \
#     MONGO_URL="mongodb://mongodb:27017/catalogue"
# CMD ["node", "server.js"]


FROM node:20.20.2-alpine3.23 AS builder
WORKDIR /app
COPY package.json .
COPY *.js .
RUN npm install

FROM node:20.20.2-alpine3.23
WORKDIR /app
EXPOSE 8080
COPY --from=builder /app /app
ENV MONGO="true" \
    MONGO_URL="mongodb://mongodb:27017/catalogue"
RUN addgroup -S roboshop && adduser -S roboshop -G roboshop # create new roboshop and created roboshop system user and assign him to roboshop group, -S means system user
RUN chown -R roboshop:roboshop /app # /app directory ownership chnaged to roboshop system user
USER roboshop # Switch to roboshop system user to avoid hacker attacks on application.
CMD ["server.js"]
ENTRYPOINT [ "node" ]
# both CMD & ENTRYPOINT are combined and execute like "node server.js"
