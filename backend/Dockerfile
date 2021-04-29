FROM node:alpine
USER node
RUN mkdir -p /home/node/app
WORKDIR /home/node/app
COPY --chown=node:node ./package.json ./
RUN npm install
COPY --chown=node:node ./ ./
EXPOSE 80
ENV MONGODB_USERNAME=default-username
ENV MONGODB_PASSWORD=default-password
ENV MONGODB_URL=default-url

# CMD [ "node", "app.js" ]
# with nodemon 
CMD ["npm", "start"]
